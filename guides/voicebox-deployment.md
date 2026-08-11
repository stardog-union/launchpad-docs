# Deploying the Voicebox Service

> [!IMPORTANT]
> **This documentation has moved.** The current version is published at
> [docs.stardog.com/voicebox/voicebox-service/deployment/](https://docs.stardog.com/voicebox/voicebox-service/deployment/) and is no longer
> updated here. This page is kept so existing links keep working.

> [!IMPORTANT]
> **Applies to:** Launchpad v4.0.0+ · Voicebox Service `v1.0.0+`
>
> Upgrading from an earlier Launchpad release requires a configuration change in every deployment. See the [4.0.0 release notes](../release-notes.md#400-release-2026-08-21) for the steps.

A practical guide for operating the Voicebox Service: how it stores the results it computes, which storage backend to choose, what to tune, and what to watch.

For a quickstart — pulling the image and running the service — see [Running the Voicebox Service](./voicebox-configuration.md#running-the-voicebox-service). For the `vbx-config.json` file and the supported LLM providers, see [Voicebox Configuration File](./voicebox-configuration.md#voicebox-configuration-file).

## Background

The Voicebox Service produces **frames**: the tabular results of the queries it runs against Stardog to answer a conversation turn. It persists them so a later turn can reuse a previous result without re-querying Stardog.

Conversation memory is separate from frames. Launchpad resends the conversation lineage on every turn, so the server holds no per-conversation memory between turns. The practical upshot is that **frames are an optimization, not a source of truth**. If a frame is missing — disk full, object expired, volume lost — the service recomputes it from Stardog. Nothing is corrupted; you pay recompute time.

This is the main way the Voicebox Service differs from the previous-generation service, which kept no frames and recomputed every result.

## Frame Stores

The backend is selected with `VOICEBOX_FRAME_STORE_BACKEND`. All three backends write frames as snappy-compressed Parquet, one object per frame, in the same layout: `{prefix}/{conversation_id}/{frame_id}.parquet`.

### Choosing a Backend

| | `local` (default) | `s3` | `azure` |
| :--- | :--- | :--- | :--- |
| Frames live on | the container's own disk | an S3 bucket | an Azure Blob container |
| Persistent volume | **required** | not needed | not needed |
| Replicas | **exactly one** | multiple | multiple |
| Expiring old frames | built-in sweeper, on a TTL | bucket lifecycle rule, which you configure | Lifecycle Management rule, which you configure |
| Credentials | none | standard AWS credential chain | credential ladder, see below |
| Suited to | a single-node deployment, or evaluation | a deployment already running in AWS | a deployment already running in Azure or AKS |

The deciding constraint is usually replica count. `local` pins the service to a single instance, because the store is that instance's own disk and a second replica would read an empty one. Either object backend removes both the volume requirement and the single-instance limit.

### Local Disk

The default. Frames are written under the frame store path, which defaults to `/var/lib/voicebox/frames`.

**Mount a writable volume at that path.** Without one, frames land in the container's writable layer and vanish on restart — the service still works, it just re-fetches from Stardog every time.

- **Docker (named volume):** Mount it at the frame store path. The image pre-creates that directory owned by the non-root container user, and a named volume inherits that ownership, so no extra steps are needed. Prefer a named volume over a bind mount.
- **Kubernetes:** Use a single-replica deployment with a `ReadWriteOnce` persistent volume mounted at the frame store path, a `Recreate` update strategy, and a security context that makes the mount writable by the non-root container user.

Two behaviors are specific to this backend:

- **Eviction is built in.** A background sweeper deletes frames older than the TTL and reaps orphaned `.tmp` files after an hour. No external cron or cleanup job is needed. The sweeper does **not** run on the object backends.
- **Disk-full is non-fatal.** When the volume fills, the user still gets their answer; the frame just isn't written and a `local_disk.disk_full` warning is logged. Recover by growing the volume or lowering the TTL.

| Environment Variable | Default | Notes |
| :--- | :--- | :--- |
| `VOICEBOX_FRAME_STORE_LOCAL_PATH` | `/var/lib/voicebox/frames` | Mount the volume here. |
| `VOICEBOX_FRAME_STORE_LOCAL_TTL_DAYS` | `7` | Lower on a small volume to reclaim disk faster; raise for longer-lived conversations. Must be `> 0` while the sweeper is enabled. |
| `VOICEBOX_FRAME_STORE_SWEEPER_ENABLED` | `true` | Set to `false` to disable the background eviction sweeper entirely. |
| `VOICEBOX_FRAME_STORE_SWEEP_INTERVAL_SECONDS` | half the TTL | How often a sweep runs. Leave unset unless you need a specific cadence. |
| `VOICEBOX_FRAME_STORE_LARGE_FRAME_WARN_MB` | `10` | Lower for earlier oversized-frame warnings; `0` disables them. |

> [!TIP]
> **Sizing.** Rough disk need is `avg frame size × frames per turn × turns per day × TTL days`. 20 GB is a comfortable start for a small team at the 7-day default. Use the `frame_store.capacity` log to trend real usage and resize from data.

### Amazon S3

Set `VOICEBOX_FRAME_STORE_BACKEND=s3`. Credentials come from the standard AWS credential chain, so on EKS an IAM role for the service account is usually all that is needed.

| Environment Variable | Default | Notes |
| :--- | :--- | :--- |
| `VOICEBOX_FRAME_STORE_S3_BUCKET` | not set | **Required** for this backend. The service fails to start without it. |
| `VOICEBOX_FRAME_STORE_S3_PREFIX` | `voicebox/frames` | Key prefix within the bucket. Bucket-relative. |

Configure expiry with an S3 lifecycle rule on the bucket. The built-in sweeper does not run on this backend, so without a lifecycle rule frames accumulate indefinitely.

### Azure Blob Storage

Set `VOICEBOX_FRAME_STORE_BACKEND=azure`.

| Environment Variable | Default | Notes |
| :--- | :--- | :--- |
| `VOICEBOX_FRAME_STORE_AZURE_ACCOUNT` | not set | Storage account name. Required unless a connection string is set. |
| `VOICEBOX_FRAME_STORE_AZURE_CONTAINER` | not set | **Required** for this backend. |
| `VOICEBOX_FRAME_STORE_AZURE_PREFIX` | `voicebox/frames` | Blob name prefix within the container. |

The storage account should be StorageV2, Standard, with hierarchical namespace **off** and blob soft delete **off**.

> [!IMPORTANT]
> Data-plane access is a separate role from account ownership. **Owner and Contributor grant no blob access at all** — the identity running the service needs **Storage Blob Data Contributor** on the account or container.

**Credentials** resolve as a ladder, first match wins:

1. `VOICEBOX_FRAME_STORE_AZURE_CONNECTION_STRING`
2. `VOICEBOX_FRAME_STORE_AZURE_ACCOUNT_KEY` — the shared account key, from the portal under **Security and networking → Access keys**
3. `VOICEBOX_FRAME_STORE_AZURE_SAS_TOKEN`
4. `DefaultAzureCredential` — used when none of the above are set. Covers service-principal environment variables, AKS Workload Identity, Managed Identity, and a developer `az login`.

Leaving all three credential variables unset — rung 4 — is the recommended path on AKS. The rung that resolved is named in the startup banner.

Configure expiry with an Azure Lifecycle Management rule.

> [!NOTE]
> Unlike S3, where prefixes are bucket-relative, an Azure lifecycle rule's `prefixMatch` must **start with the container name**.

### Startup Reachability Check

On the object backends, a missing frame and an unreachable bucket or container look identical from the outside: reads return nothing, the service reports healthy, and every turn silently falls back to re-querying Stardog. To make that visible, the service makes one cheap metadata call at startup and logs either `frame_store.startup_check_ok` or a classified `frame_store.startup_check_failed`.

The check is advisory and never blocks startup. **Watch for it after any storage configuration change** — a failure there is the difference between a working cache and an expensive no-op.

## Tuning the Agent

These apply to every backend. Most deployments only touch these; leave the rest at their defaults.

| Environment Variable | Default | When to change |
| :--- | :--- | :--- |
| `VOICEBOX_CODE_EXEC_TIMEOUT_SECONDS` | `30` | How long the service may spend analyzing query results while answering a question, in seconds. Raise if the `code_executed` log shows `timeout` status on large results; must be `>= 1`. |
| `VOICEBOX_QUERY_EXEC_TIMEOUT_SECONDS` | `60` | Timeout applied to each query the service runs against Stardog, in seconds. Raise for slow queries on large databases; set to `0` to apply no Voicebox timeout, so the Stardog endpoint's own configured query timeout governs. |
| `VOICEBOX_QUERY_MAX_RESULTS` | `100000` | Maximum rows a single query may return, applied as the query `LIMIT`. Lower it to reduce memory use and frame sizes on wide result sets; must be `>= 1`. |
| `VOICEBOX_RECURSION_LIMIT` | not set | Maximum number of steps the agent may take to answer one question. Unset uses the built-in default (35). Raise for complex, multi-step questions that report running out of steps; must be `>= 1`. |
| `VOICEBOX_FRAME_STORE_CACHE_SIZE` | `100` | Frames held in each instance's in-memory read cache. Raise to cut reads against the store on long conversations, at the cost of memory. |

## Health and Readiness Probes

- **Liveness:** `GET /system/health` (returns `204`).
- **Readiness:** `GET /system/storage-ready` (`200` when the frame path is writable, `503` otherwise).

Keep the disk-writability check on **readiness**, not liveness, so a transient full disk pulls the instance from traffic instead of restarting it into a crash loop.

> [!WARNING]
> On the `s3` and `azure` backends, `/system/storage-ready` always returns `200` with `"checked": false` in the body. That means *the check does not apply here*, *not* that the bucket or container was verified. Use the [startup reachability check](#startup-reachability-check) to confirm object storage is actually configured correctly.

## Log Events

Logs are emitted as structured JSON when `LOG_TYPE=JSON` (the default): each line is a JSON object keyed by an `event` field, so you can alert on the event name.

| Event | Level | Meaning / action |
| :--- | :--- | :--- |
| `voicebox_backends` | `INFO` | Startup banner: resolved backends, frame path or container, the Azure credential rung where applicable, and `frame_local_writable`. Use it to confirm your config took effect. |
| `frame_store.startup_check_ok` | `INFO` | Object backends only. The bucket or container was reachable at startup. |
| `frame_store.startup_check_failed` | `WARNING` | Object backends only. The bucket or container was not reachable, with a classified reason. Frames will not persist and every turn re-queries Stardog. **Alert.** |
| `local_disk.disk_full` | `WARNING` | Local backend. A write ran out of disk space. Users still get answers; frames aren't persisting. Carries `disk_free_bytes` / `disk_total_bytes`. **Alert.** Grow the volume or shorten the TTL. |
| `frame_store.capacity` | `INFO` | Local backend. Periodic volume telemetry: `disk_free_bytes`, `disk_total_bytes`, `file_count`. Alert proactively when free space drops below a threshold (for example 15 percent). |
| `local_disk.large_frame` | `WARNING` | Local backend. A single frame exceeded the warn threshold. Carries `size_bytes` / `threshold_bytes`. Informational; a spike can signal runaway result sizes. |
| `frame_store_sweep.cycle` | `INFO` | Local backend. A sweep pass finished: `scanned`, `deleted`, `skipped_recent`, `tmp_deleted`, `errors`, `elapsed_ms`. Watch `errors` and `elapsed_ms`. |
| `code_executed` | `INFO` / `ERROR` | A code-execution step finished: `status` (`success`, `timeout`, `op_cap`, or `error`), `duration_ms`, `input_frame_count`, `input_bytes`, `output_bytes`, `peak_rss_kb`. Repeated `timeout` suggests raising `VOICEBOX_CODE_EXEC_TIMEOUT_SECONDS`; trend `peak_rss_kb` to watch the service's memory headroom. |
