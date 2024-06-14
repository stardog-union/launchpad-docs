# Launchpad Release Notes

Stardog Launchpad is distributed as a Docker image, which includes the Launchpad login web app together with the Stardog Applications (Studio, Explorer, and Designer). The Stardog Platform server is not included in the Launchpad Docker image.

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

## Version Scheme

Stardog Launchpad releases are versioned as `vX.Y.Z` where:

   - `X` represents the (minimum) version of the Stardog platform required to use all current Launchpad features; this value will increment when the minimum required version of Stardog changes
   - `Y` represents the current version of Launchpad features; this value will increment when new features or bug fixes are added to the Launchpad login web app
   - `Z` represents the versions of the Stardog Applications included in the Docker image (see the table below); this value will increment when one or more of the Stardog Applications are updated

# Launchpad Releases

| Release | Stardog Version | Designer Version | Explorer Version | Studio Version |
|         |    (minimum)    |                  |                  |                |
| ------- | --------------- | ---------------- | ---------------- | -------------- |
| 2.0.0   | [9.1.0](https://docs.stardog.com/release-notes/stardog-platform#910-release-2023-07-06) | [2.24.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2240-release) | [1.26.3](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v1263-release) | [4.1.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v411-release) |
| 2.1.1   | [9.1.0](https://docs.stardog.com/release-notes/stardog-platform#910-release-2023-07-06) | [2.28.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2280-release) | [1.28.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v1280-release) | [4.3.3](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v433-release) |
| 2.2.2   | [9.1.0](https://docs.stardog.com/release-notes/stardog-platform#910-release-2023-07-06) | [2.30.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2301-release) | [1.32.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v1320-release) | [4.4.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v440-release) |
| 2.2.3   | [9.1.0](https://docs.stardog.com/release-notes/stardog-platform#910-release-2023-07-06) | [2.32.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2321-release) | [2.3.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v230-release) | [5.2.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v520-release) |
| 2.3.4   | [9.1.0](https://docs.stardog.com/release-notes/stardog-platform#910-release-2023-07-06) | [2.33.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2330-release) | [2.5.1](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v251-release) | [5.4.2](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v542-release) |

## 2.3.4 Release (2024-06-14)

- Explorer: Show selected graphs first in list (#VET-4571)
- Remove unneeded messages logged to the Launchpad console (#VET-4771)
- Remove unused components to mitigate critical CVEs (#CLOUD-2322)

## 2.2.3 Release (2024-05-08)
- Update Studio, Explorer and Designer to latest versions

## 2.2.2 Release (2024-02-09)

- Fix for visualize disabled in Explorer for select queries (#VET-4291)
- Upgrade to Python 3.11 (#CLOUD-1463)

## 2.1.1 Release (2023-12-13)

- Add option to show/hide password auth inputs (#CLOUD-1276); see [config doc](./?tab=readme-ov-file#configuration-options)
- Ensure access token for AzureAD auth is refreshed (#CLOUD-1371)

## 2.0.0 Release (2023-09-27)

### Launchpad Updates
- Support using a client certificate when authenticating to Azure (#CLOUD-1332); see [config doc](./azure/client-certificate-config.md)
- Support access token passthrough mode with Azure (#CLOUD-1278); see [config doc](./azure/access-token-passthrough-mode.md)
- Disable Django Admin (#CLOUD-1489)

### Stardog Notes
- Stardog Platform version 9.1.0 (or later) is required to use the access token passthrough feature. Other Launchpad features require at least version 9.0.0 of Stardog.
