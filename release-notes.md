# Launchpad Release Notes

Stardog Launchpad is distributed as a Docker image, which includes the Launchpad web app together with the Stardog Applications (Studio, Explorer, and Designer, Knowledge Catalog).

## Getting the Current Version of Launchpad

The latest release of Launchpad is available from Stardog's Docker registry:

1. Log in to the Docker registry:

```bash
docker login stardog-stardog-apps.jfrog.io
```

2. Pull the latest image:

```bash
docker pull stardog-stardog-apps.jfrog.io/launchpad:current
```

> [!IMPORTANT]
> The `current` tag always points to the latest release of Launchpad. You can also pull a specific version of Launchpad by using the version tag, for example, `v3.0.1`.
>
>```bash
>docker pull stardog-stardog-apps.jfrog.io/launchpad:v3.0.1 
>```


## Version Scheme

Launchpad v3 uses semantic versioning.

## Launchpad Releases

| Release | Image Tag | Designer Version | Explorer Version | Studio Version | Knowledge Catalog Version |
| ----- | ----------- | ------------- | -------------- | -------------- | ------------ |
| 3.2.0 | `v3.2.0` | [2.45.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2451-release) | [2.10.4](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2104-release) | [5.7.9](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v579-release) | [1.4.15](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1415-release) |
| 3.1.0 | `v3.1.0` | [2.43.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2432-release) | [2.10.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2102-release) | [5.7.7](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v577-release) | [1.4.13](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1413-release) |
| 3.0.1 | `v3.0.1` | [2.42.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2420-release) | [2.10.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2100-release) | [5.7.5](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v575-release) | [1.4.11](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1411-release) |
| 3.0.0 | `v3.0.0` | [2.41.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2410-release) | [2.9.3](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v293-release) | [5.7.5](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v575-release) | [1.4.10](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1410-release) |

## 3.2.0 Release (2025-05-06)

### New Features

- Adds support for using shared user authentication to log users into to Launchpad, bypassing the requirement for using an SSO login provider. This is **not** intended for production use. See [Shared User Authentication](./README.md#shared-user-authentication) for more information.
- Adds support for saving Designer projects to the Launchpad database instead of the user's browser's local storage. See [`DESIGNER_STORAGE_ENABLED`](./README.md#designer_storage_enabled) for more information.

## 3.1.0 Release (2025-04-03)

### New Features

- Adds support for using Okta as a login and connection provider
- Adds support for using PingOne as a login and connection provider
- Adds a new advanced configuration option, `GUNICORN_WORKERS`, to specify how many gunicorn workers should run interally for the Launchpad server. By default, this is set to `2 * number of CPU cores + 1`. The default value is recommended for most use cases, but can be overridden to increase or decrease the number of workers. This is useful for environments with limited resources like memory.

### Modifications

- Removes the default user `httpd` (with uid `1001`) from the Launchpad image. Any numeric user id can be used to run the Launchpad image, but the default is now `root` (with uid `0`). This change was made to increase flexibility in how the Launchpad image can be run, specifically in some Kubernetes environments where it may be advisable to run the container with a numeric user id > 100000. 

> [!NOTE]
> When running the Launchpad image with a numeric user id, the `--user` flag should be used to specify the user id. For example, to run the Launchpad image with a user and group id of `1001`, use the following command:
>
>```bash
>    docker run \
>      --user 1001:1001 \
>      --env-file /path/to/launchpad/.env.launchpad \
>      -p 8080:8080 \
>      -v /path/to/launchpad/data:/data \
>      stardog-stardog-apps.jfrog.io/launchpad:current
>```
>
> On Linux, you will want to ensure that the user id you specify has the appropriate permissions to access the files and directories used by Launchpad. macOS handles file permissions differently, so you may not need to change the ownership of the files and directories. However, if you are running Launchpad on Linux and using a numeric user id, you may need to change the ownership of the files and directories used by Launchpad to match the user id you specified. For example, if you are using a user id of `1001`, you can use the following command to change the ownership of the files and directories:
>
>```bash
>sudo chown -R 1001:1001 /path/to/launchpad/data
>```
>
> For Kubernetes, the `runAsUser` and `runAsGroup` fields in the pod security context can be used to specify the user id.


### Security

- Modifies the Launchpad image to support being run with the `--read-only` flag in Docker. This enforces the container’s root filesystem being mounted as read only. Similarly, in Kubernetes, the `readOnlyRootFilesystem` option can be set to `true` in the pod security context.
- Change from using `python:3.11-bookworm` to `python:3.11-slim-bookworm` as the base image for Launchpad. This change reduces the size of the image and improves security by removing unnecessary packages and files.
- Update internal packages and dependencies to eliminate all fixable CVEs (CVEs that do not have an upstream fix or patch).

## 3.0.1 Release (2025-02-21)

### Bug Fixes

- [CLOUD-3018]: Fixed an issue where the Azure Login provider was requesting unnecessary `Directory.Read.All` scope permissions.

## 3.0.0 Release (2025-01-30)

- Inital release of Launchpad v3. 

> [!TIP]
> See the [documentation](./README.md) for more information.
