# Deploying the Voicebox Service

> [!IMPORTANT]
> **Applies to:** Launchpad v4.0.0+ · Voicebox Service `v1.0.0+`
>
> Upgrading from an earlier Launchpad release requires a configuration change in every deployment. See the [4.0.0 release notes](../release-notes.md#400-release-2026-08-21) for the steps.

A practical guide for operating the Voicebox Service: how it stores the results it computes, what that requires of your deployment, what to tune, and what to watch.

For a quickstart — pulling the image and running the service — see [Running the Voicebox Service](../voicebox.md#running-the-voicebox-service). For the `vbx-config.json` file and the supported LLM providers, see [Voicebox Configuration File](../voicebox.md#voicebox-configuration-file).

## Background

The Voicebox Service produces **frames**: the tabular results of the queries it runs against Stardog to answer a conversation turn. It persists them so a later turn can reuse a previous result without re-querying Stardog.

Conversation memory is separate from frames. Launchpad resends the conversation lineage on every turn, so the server holds no per-conversation memory between turns. The practical upshot is that **frames are an optimization, not a source of truth**. If a frame is missing — disk full, volume lost, expired — the service recomputes it from Stardog. Nothing is corrupted; you pay recompute time.

This is the main way the Voicebox Service differs from the previous-generation service, which kept no frames and recomputed every result.

## Frame Stores

<!-- TODO: confirm which frame store backends v1.0.0 ships. The table below documents local disk only.
     If object storage shipped, add a row and extend the tradeoff table. -->

### Local Disk

Frames are written as snappy-compressed Parquet files, one file per frame, laid out as `{frame_store_path}/{conversation_id}/{frame_id}.parquet` under the frame store path (default `/var/lib/voicebox/frames`).

Because the store is the container's own disk, a deployment using it **runs as a single instance** — a second replica would read an empty store.

## Deployment Requirements

- **A persistent volume is required.** Frames must outlive container restarts, so mount a volume at the frame store path. Without it, frames land in the container's writable layer and vanish on restart. The service still works; it just re-fetches from Stardog.
- **It runs as a single instance.** Its store is the pod's own disk, so a second replica would read an empty store.
- **Eviction is built in.** A background sweeper deletes frames older than a TTL (7 days by default) and reaps orphaned `.tmp` files after 1 hour. No external cron or cleanup job is needed.
- **Disk-full is non-fatal.** When the volume fills, the user still gets their answer; the frame just isn't written and a `local_disk.disk_full` warning is logged. Recover by growing the volume or lowering the TTL.
- **A readiness endpoint exists.** `GET /system/storage-ready` reports whether the frame path is actually writable, so a bad mount surfaces at startup rather than at first query.

## Configuration

> [!NOTE]
> This section covers **only** the storage and tuning settings. Everything else about running the Voicebox Service — the `vbx-config.json` configuration file, the supported LLM providers (Azure AI, AWS Bedrock, OpenAI, Anthropic, Databricks, Vertex AI, Fireworks), and the standard environment variables — is documented in [Voicebox Service Configuration](../voicebox.md#configuration) and the [Voicebox Configuration File](../voicebox.md#voicebox-configuration-file) reference.

### Required: Mount a Volume

Mount a writable volume at the frame store path (default `/var/lib/voicebox/frames`):

- **Docker (named volume):** Mount it at the frame store path. The image pre-creates that directory owned by the non-root container user, and a named volume inherits that ownership, so no extra steps are needed. Prefer a named volume over a bind mount.
- **Kubernetes:** Use a single-replica deployment with a `ReadWriteOnce` persistent volume mounted at the frame store path, a `Recreate` update strategy, and a security context that makes the mount writable by the non-root container user.

### Commonly Tuned Settings

Most deployments only touch these. Leave the rest at their defaults.

| Environment Variable | Default | When to change |
| :--- | :--- | :--- |
| `VOICEBOX_FRAME_STORE_LOCAL_TTL_DAYS` | `7` | Lower on a small volume to reclaim disk faster; raise for longer-lived conversations. Must be `> 0` while the sweeper is enabled. |
| `VOICEBOX_FRAME_STORE_SWEEPER_ENABLED` | `true` | Set to `false` to disable the background eviction sweeper entirely. |
| `VOICEBOX_FRAME_STORE_LARGE_FRAME_WARN_MB` | `10` | Lower for earlier oversized-frame warnings; `0` disables them. |
| `VOICEBOX_CODE_EXEC_TIMEOUT_SECONDS` | `30` | How long the service may spend analyzing query results while answering a question, in seconds. Raise if the `code_executed` log shows `timeout` status on large results; must be `>= 1`. |
| `VOICEBOX_QUERY_EXEC_TIMEOUT_SECONDS` | `60` | Timeout applied to each query the service runs against Stardog, in seconds. Raise for slow queries on large databases; set to `0` to apply no Voicebox timeout, so the Stardog endpoint's own configured query timeout governs. |
| `VOICEBOX_QUERY_MAX_RESULTS` | `100000` | Maximum rows a single query may return, applied as the query `LIMIT`. Lower it to reduce memory use and frame sizes on wide result sets; must be `>= 1`. |
| `VOICEBOX_RECURSION_LIMIT` | not set | Maximum number of steps the agent may take to answer one question. Unset uses the built-in default (35). Raise for complex, multi-step questions that report running out of steps; must be `>= 1`. |

> [!TIP]
> **Sizing.** Rough disk need is `avg frame size × frames per turn × turns per day × TTL days`. 20 GB is a comfortable start for a small team at the 7-day default. Use the `frame_store.capacity` log (below) to trend real usage and resize from data.

## Health and Readiness Probes

- **Liveness:** `GET /system/health` (returns `204`).
- **Readiness:** `GET /system/storage-ready` (`200` when the frame path is writable, `503` otherwise).

Keep the disk-writability check on **readiness**, not liveness, so a transient full disk pulls the instance from traffic instead of restarting it into a crash loop.

## Log Events

Logs are emitted as structured JSON when `LOG_TYPE=JSON` (the default): each line is a JSON object keyed by an `event` field, so you can alert on the event name.

| Event | Level | Meaning / action |
| :--- | :--- | :--- |
| `local_disk.disk_full` | `WARNING` | A write ran out of disk space. Users still get answers; frames aren't persisting. Carries `disk_free_bytes` / `disk_total_bytes`. **Alert.** Grow the volume or shorten the TTL. |
| `frame_store.capacity` | `INFO` | Periodic volume telemetry: `disk_free_bytes`, `disk_total_bytes`, `file_count`. Alert proactively when free space drops below a threshold (for example 15 percent). |
| `local_disk.large_frame` | `WARNING` | A single frame exceeded the warn threshold. Carries `size_bytes` / `threshold_bytes`. Informational; a spike can signal runaway result sizes. |
| `frame_store_sweep.cycle` | `INFO` | A sweep pass finished: `scanned`, `deleted`, `skipped_recent`, `tmp_deleted`, `errors`, `elapsed_ms`. Watch `errors` and `elapsed_ms`. |
| `code_executed` | `INFO` / `ERROR` | A code-execution step finished: `status` (`success`, `timeout`, `op_cap`, or `error`), `duration_ms`, `input_frame_count`, `input_bytes`, `output_bytes`, `peak_rss_kb`. Repeated `timeout` suggests raising `VOICEBOX_CODE_EXEC_TIMEOUT_SECONDS`; trend `peak_rss_kb` to watch the service's memory headroom. |
| `voicebox_backends` | `INFO` | Startup banner: resolved backends, frame path, and `frame_local_writable`. Use it to confirm your config and mount took effect. |
