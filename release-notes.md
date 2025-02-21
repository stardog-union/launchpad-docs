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
| 3.0.0 | `v3.0.0` | [2.41.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2410-release) | [2.9.3](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v293-release) | [5.7.5](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v575-release) | [1.4.10](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1410-release) |
| 3.0.1 | `v3.0.1` | [2.42.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-designer#v2420-release) | [2.10.0](https://docs.stardog.com/release-notes/stardog-cloud/stardog-explorer#v2100-release) | [5.7.5](https://docs.stardog.com/release-notes/stardog-cloud/stardog-studio#v575-release) | [1.4.11](https://docs.stardog.com/release-notes/stardog-cloud/stardog-knowledge-catalog#v1411-release) |

## 3.0.0 Release (2025-01-30)

- Inital release of Launchpad v3. 

> [!TIP]
> See the [documentation](./README.md) for more information.


## 3.0.1 Release (2025-02-21)

- [CLOUD-3018]: Fixed an issue where the Azure Login provider was requesting unnecessary `Directory.Read.All` scope permissions.
