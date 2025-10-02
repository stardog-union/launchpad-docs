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
| 3.5.0 | `v3.5.0` | TBD| TBD | TBD | TBD |
| 3.4.0 | `v3.4.0` | [2.48.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2480-release) | [2.13.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2131-release) | [5.7.15](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v5715-release) | [1.4.19](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1419-release) |
| 3.3.1 | `v3.3.1` | [2.46.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2462-release) | [2.11.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2112-release) | [5.7.12](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v5712-release) | [1.4.17](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1417-release) |
| 3.3.0 | `v3.3.0` | [2.46.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2462-release) | [2.11.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2112-release) | [5.7.12](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v5712-release) | [1.4.17](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1417-release) |
| 3.2.0 | `v3.2.0` | [2.45.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2451-release) | [2.10.4](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2104-release) | [5.7.9](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v579-release) | [1.4.15](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1415-release) |
| 3.1.0 | `v3.1.0` | [2.43.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2432-release) | [2.10.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2102-release) | [5.7.7](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v577-release) | [1.4.13](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1413-release) |
| 3.0.1 | `v3.0.1` | [2.42.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2420-release) | [2.10.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2100-release) | [5.7.5](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v575-release) | [1.4.11](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1411-release) |
| 3.0.0 | `v3.0.0` | [2.41.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2410-release) | [2.9.3](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v293-release) | [5.7.5](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v575-release) | [1.4.10](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1410-release) |

## 3.5.0 Release (2025-10-02)

- Adds the Launchpad version to header of dashboard

<img width="600" height="524" alt="launchpad-version" src="https://github.com/user-attachments/assets/c27aeef2-d91d-4a52-b807-b24bffa21945" />

- Adds support for saving an additional internal/private endpoint for Stardog connections. The internal endpoint enables Launchpad to use a separate endpoint for server-side operations. This is particularly beneficial for the Voicebox service container, which may not be able to access Stardog on the public endpoint but can communicate using the internal endpoint. This supports architectures where different network routes are required for backend services versus browser-based access (Studio, Explorer, etc).

> [!NOTE]
> When both endpoints are configured, Voicebox requests will automatically use the internal endpoint while browser-based requests continue using the public endpoint.

  - **SSO Connections** - The internal endpoint can be pre-set similar Stardog endpoint or display name for the sso connection by using the following environment variable `SSOCONNECTION_<IDENTIFIER>_<PROVIDER>_STARDOG_INTERNAL_ENDPOINT`. It can always be overridden by the user creating the connection under "Advanced Options" in the SSO connection dialog.
    
  https://github.com/user-attachments/assets/85ea8c27-0222-4800-bc8c-19931b0317e6

  - **Username/Password Connections** - The internal endpoint can be set under "Advanced Options" in the SSO connection dialog. 
    
    <img width="403" height="637" alt="username-password-connection-internal-endpoint" src="https://github.com/user-attachments/assets/e124fc75-7b38-4d39-9429-f342bf36cd10" />


## 3.4.0 Release (2025-07-29)

### New Features

- Adds support for using Duo as a login provider to authenticate users into Launchpad using OpenID Connect (OIDC). See [Duo Login Provider](./providers/duo.md) for more information.
- Adds support for configuring a secondary authentication provider for Microsoft Entra users based on app roles. Users with specific app roles assigned in Microsoft Entra will be required to authenticate with an additional provider (such as Duo) after successful Microsoft Entra authentication, providing an extra layer of security. See [Microsoft Entra Secondary Authentication Provider](./providers/microsoft-entra.md#secondary-authentication-provider) for more information.
- Adds support for Microsoft Entra On-Behalf-Of (OBO) flow for seamless authentication to SSO connections. When enabled, users authenticate once via Azure login and gain seamless access to all connected Stardog instances without interactive sign-in prompts for individual connections. See [On-Behalf-Of (OBO) Flow SSO Connections](./providers/microsoft-entra.md#on-behalf-of-obo-flow-sso-connections) for more information.
- Adds [button to copy connection JWT token](./README.md#copy_connection_token_button_enabled) and button to copy the endpoint from the connection details in the Launchpad UI for improved user experience.

## 3.3.1 Release (2025-06-20)

### Bug Fixes

- Ensure all users are able to use directives (`#llm`, `#chart`, `#analyze`) when asking Voicebox a question.

## 3.3.0 Release (2025-06-12)

### New Features

- Adds support for using Kerberos to log users into Launchpad. See [Kerberos Login Provider](./README.md#kerberos-login-provider) for more information.
- Adds a "public" REST API to Launchpad to programatically interact with Voicebox. See [Using Voicebox Programmatically](./voicebox.md#using-voicebox-programmatically) for more information.

### Modifications

- Reduce noise in Launchpad logs by not logging expected unauthorized errors when user visits the login page before logging in.

### Security

- Update internal packages and dependencies to eliminate all fixable CVEs (CVEs that do not have an upstream fix or patch).

## 3.2.0 Release (2025-05-06)

### New Features

- Adds support for using shared user authentication to log users into to Launchpad, bypassing the requirement for using an SSO login provider. This is **not** intended for production use. See [Shared User Authentication](./README.md#shared-user-authentication) for more information.
- Adds support for saving Designer projects to the Launchpad database instead of the user's browser's local storage. See [`DESIGNER_STORAGE_ENABLED`](./README.md#designer_storage_enabled) for more information. You must set the `DESIGNER_STORAGE_ENABLED` to `true` in Launchpad configuration to enable this feature. All Designer projects will continue to be saved to browser local storage by default.

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
