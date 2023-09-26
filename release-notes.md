# Launchpad Release Notes

Stardog Launchpad is distributed as a Docker container, which includes the Launchpad login web app together with the Stardog Applications (Studio, Explorer, and Designer). The Stardog Platform server is not included in the Launchpad Docker container.

## Getting the Current Version of Launchpad

The latest release of Launchpad is available from Stardog's Docker registry:

1. Log in to the Docker registry:

```bash
docker login stardog-stardog-apps.jfrog.io
```

2. Pull the latest image:

```bash
docker pull stardog-stardog-apps.jfrog.io/cloud-login:launchpad-current
```

## Version Scheme

Stardog Launchpad container releases are versioned as `vX.Y.Z` where:

   - `X` represents the (minimum) version of the Stardog platform required to use all current Launchpad features; this value will increment when the minimum required version of Stardog changes
   - `Y` represents the current version of Launchpad features; this value will increment when new features or bug fixes are added to the Launchpad login web app
   - `Z` represents the versions of the Stardog Applications included in the Docker container (see the table below); this value will increment when one or more of the Stardog Applications are updated

# Launchpad Container Releases

| Release | Stardog Version | Designer Version | Explorer Version | Studio Version |
| ------- | --------------- | ---------------- | ---------------- | -------------- |
| 2.0.0   | [9.1.0](https://docs.stardog.com/release-notes/stardog-platform#910-release-2023-07-06) | [2.24.0](https://docs.stardog.com/release-notes/stardog-designer#v2240-release) | [1.26.3](https://docs.stardog.com/release-notes/stardog-explorer#v1263-release) | [4.1.1](https://docs.stardog.com/release-notes/stardog-studio#v411-release) |

## 2.0.0 Release (2023-09-27)

### Launchpad Updates
- Support using a client certificate when authenticating to Azure (#CLOUD-1332); see [config doc](./azure/client-certificate-config.md)
- Support access token passthrough mode with Azure (#CLOUD-1278); see [config doc](./azure/access-token-passthrough-mode.md)
- Disable Django Admin (#CLOUD-1489)

### Stardog Notes
- Stardog Platform version 9.1.0 (or later) is required to use the access token passthrough feature. Other Launchpad features require at least version 9.0.0 of Stardog.
