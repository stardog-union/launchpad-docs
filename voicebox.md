# Voicebox

[Stardog Voicebox](https://docs.stardog.com/voicebox/) is a conversational AI chat interface for your Enterprise Data. This document provides instructions for using Voicebox within Launchpad.

Launchpad can operate with or without the Voicebox. To enable Voicebox, you'll need to run an additional Docker image ([Voicebox Service](#voicebox-service)) and provide its address to Launchpad. The two services communicate over HTTP.

## Voicebox Features

### Think Mode

Think Mode is powered by a multi-agent architecture and Voicebox 3, enabling chain-of-thought reasoning to handle complex, multi-step questions. When enabled, users will see a "Think Mode" button in the Voicebox input interface.

![Think Mode Screenshot](https://github.com/user-attachments/assets/1fcc15db-b711-44e7-9c3f-c0c53baa3aca)

To enable Think Mode in your Launchpad deployment, add the following environment variable to your Launchpad configuration:

```bash
VOICEBOX_THREE_ENABLED=true
```

> [!IMPORTANT]
> Think Mode requires Voicebox Service version `v0.22.0+` and Stardog version `v11.2.0+`. See the [Voicebox Release Notes](#0220-release-oct-16-2025) and [Stardog Release Notes](https://docs.stardog.com/release-notes/stardog-platform#1120-release-2025-10-01) for more information.

### Voicebox Suggestions for Designer

Voicebox Suggestions makes it possible to create a Voicebox-enabled Knowledge Graph, complete with spotlight questions, from just a project description, input data, and a few clicks. When enabled, users will see the new Voicebox-assisted project creation flow when creating a new project in Designer.

![Voicebox Suggestions Screenshot](https://github.com/user-attachments/assets/6c3d78ac-8129-42ef-9dd5-4228501f3653)

To enable Voicebox Suggestions in your Launchpad deployment, add the following environment variable to your Launchpad configuration:

```bash
VOICEBOX_SUGGESTIONS_ENABLED=true
```

> [!IMPORTANT]
> Voicebox Suggestions requires Voicebox Service version `v0.22.0+` and Stardog version `v11.2.0+`. See the [Voicebox Release Notes](#0220-release-oct-16-2025) and [Stardog Release Notes](https://docs.stardog.com/release-notes/stardog-platform#1120-release-2025-10-01) for more information.

## Using Voicebox Programmatically

You can interact with Voicebox programmatically using the Launchpad REST API. This allows you to send queries to Voicebox and receive responses in a structured format, which can be useful for integrating Voicebox functionality into other applications or workflows.

In order to use the Voicebox REST APIs, you *must* have the [Voicebox Service](#voicebox-service) running and configured with Launchpad. You can access the REST API documentation served by Launchpad at `/api/v1/docs` endpoint. There's a Swagger UI that provides an interactive interface to explore the API endpoints. Additional documentation about how to use the API is contained there.

![Launchpad REST API Swagger Documentation](./assets/rest-api-docs.png)

### Creating a Voicebox Application and Associated API Key

To use the Voicebox REST API, you need to create a Voicebox application in Launchpad and generate an API key that is scoped to that application. This API key will be used to authenticate your requests to the Voicebox REST API:

To do so, follow these steps:

1. Create a Voicebox application. In the bottom left corner of the Launchpad UI, click on your username opening the user menu, then click on **Manage API Keys**.

![Manage API Keys](./assets/user-voicebox-api-keys.png)

2. Click on **New Voicebox App** and fill in the details for your application. It will ask you to scope the application with details such as the connection to use, database, model, etc.

![Create Voicebox App Dialog](./assets/create-voicebox-app-dialog.png)

3. Once the application is created, you will need to create an API key that is scoped to the Voicebox application. Click on **New App Key** in the Voicebox application you just created. You can provide a name for the API key, and adjust the expiration date if desired.

![Create Voicebox App Key Dialog](./assets/create-voicebox-app-dialog.png)

4. After creating the API key, you will be able to copy it only once. This API key will be used to authenticate your requests to the Voicebox REST API.

![Copy Voicebox App Key Dialog](./assets/copy-voicebox-app-api-key-dialog.png)

The API key should be provided in the `Authorization` header of your requests as a *Bearer* token. You must also include a `X-Client-Id` header with an identifier < 255 characters long that identifies a user of the Voicebox application. This can be any alphanumeric string.


```
# Voicebox App token
token=sdc_xyz...

curl --silent -X 'POST' \
  'http://launchpad.local:8080/api/v1/voicebox/ask' \
  -H 'Accept: application/json' \
  -H 'X-Client-Id: someones-id' \
  -H "Authorization: Bearer $token" \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "Who is Cersei Lannister married to?"
}' | jq '{ "Result": .result, "SPARQL Query Used": (.actions[] | select(.type == "sparql") | .value)}'
{
  "Result": "Cersei Lannister is married to [Robert Baratheon](urn:stardog:marketplace:demos:got:characters:901).",
  "SPARQL Query Used": "# Who is Cersei Lannister married to?\n\nSELECT DISTINCT ?spouse0 \nWHERE {\n  ?character0 a got:Character . \n  ?character0 stardog:label \"Cersei Lannister\" . \n  ?character0 got:spouse ?spouse0 . \n  ?spouse0 a got:Character . \n  FILTER ( ?character0 != ?spouse0)\n}"
}
```

### Stardog Authentication for API Requests

When making API requests to the Voicebox REST API, the Voicebox Service makes requests to Stardog in order to answer your question. There is a Stardog credential associated with the Voicebox application by means of the Launchpad connection you selected when creating the Voicebox application. In some cases, you may need to override the default authentication method used by Launchpad.

The authentication method depends on your Stardog connection type:

#### Username/Password Connections

When you authenticate through the Launchpad interface (create a new connection), Launchpad obtains a JWT from the Stardog server on behalf of the authenticated user and persists it beyond the Launchpad session for subsequent requests. If you try and use the connection in Launchpad and your token is invalid or expired, Launchpad will prompt you to re-authenticate, obtaining a new JWT and persisting it for future use. This means that it is possible for a JWT associated with the connection to expire when using it programmatically and not through the Launchpad UI since you will not be prompted to re-authenticate.

If you encounter authentication errors or want to ensure you always have a valid token for programmatic requests, you can manually retrieve a fresh token from the Stardog server associated with the Voicebox app's connection using the `/admin/token` endpoint ([as described below](#manual-token-retrieval-from-stardog)) and include it in your requests using the `X-SD-Auth-Token` header to override the stored credential. You could also log 
into Launchpad and re-authenticate the connection to obtain a new JWT, but this is not always practical for programmatic use.

#### SSO Connections

SSO connections (such as Microsoft Entra) require manual token management since Launchpad does not persist JWTs from external identity providers beyond the session. For these connections, you must:

1. **Obtain a valid token** from your SSO provider that Stardog is configured to accept
2. **Include the token** in your API requests using the `X-SD-Auth-Token` header

##### Token Override Capability

The `X-SD-Auth-Token` header can also override any JWT that Launchpad has persisted for a connection. This is particularly useful for ensuring you always have a valid, unexpired token.

###### Manual Token Retrieval from Stardog

As noted earlier, if Stardog is configured to issue JWTs, you can manually retrieve a token using the `/admin/token` endpoint. This is useful for obtaining a fresh token when needed.

**Example:**
```bash
curl -u <username>:<password> https://<stardog-server-url>/admin/token \
```

The returned JWT issued by Stardog can then be included in your Voicebox API requests:

```bash
X-SD-Auth-Token: <your-jwt-token>
```

> [!TIP]
> If using the Stardog CLI, you can also use the [`stardog-admin user token`](https://docs.stardog.com/stardog-admin-cli-reference/user/user-token) command to obtain a JWT from Stardog for a user.

## Voicebox Service

The Voicebox Service is distributed as a Docker image. The Voicebox Service is a stateless HTTP server that packages up Stardog Voicebox functionality. It is intended to be run in conjunction with Stardog Launchpad. The Voicebox Service will communicate directly with whichever Stardog servers you are interacting with in Launchpad, so they should be accessible to this image when run as a container.

![Voicebox Service with Launchpad Architecuture](./assets/voicebox-service-architecture.png)

### Internal Stardog Endpoint Support

When Voicebox makes requests to Stardog servers, it uses server-side connections that may require different network routing than browser-based requests. To support architectures where the Voicebox service container cannot access Stardog on the public endpoint, Launchpad v3.5.0+ allows you to configure an additional internal endpoint for connections.

When an internal endpoint is configured for a connection, Voicebox automatically uses the internal endpoint while browser-based applications (Studio, Explorer, Designer, Knowledge Catalog) continue using the public endpoint. This is particularly useful in scenarios where:
- Stardog is behind a firewall accessible only within a private network
- Different DNS resolution is needed for internal vs. external access
- Network policies restrict container-to-container communication to internal networks

See the [SSO Connection Configuration](./README.md#sso-connection-configuration) section and individual provider documentation for details on configuring internal endpoints.

### Running the Voicebox Service

1. Similar to Launchpad, the Voicebox Service image can be pulled from Stardog's private Docker registry.

   ```bash
   docker pull stardog-stardog-apps.jfrog.io/voicebox-service:current
   ```

   - The `current` tag will always point to the latest version of the Voicebox Service. You can also specify a specific version tag to avoid accidental upgrades.

2. To run the Voicebox Service with Docker, you can use the following command:

   ```bash
   docker run \
   --env-file .env.voicebox-service \
   -p 8000:8000 \
   -v /host/path/to/vbx-config.json:/voicebox-config/vbx-config.json \
   stardog-stardog-apps.jfrog.io/voicebox-service:current
   ```

   - `.env.voicebox-service` can be named anything but contains the configuration for the Voicebox Service. See [Configuration](#configuration) for more details.
   - `/host/path/to/vbx-config.json` is mounted from the host to `/voicebox-config/vbx-config.json` in the container. This configuration has more LLM specific configuration.
      - `.env.voicebox-service` should have `VBX_CONFIG_FILE=/voicebox-config/vbx-config.json` in it.
   - The Voicebox Service HTTP server is exposed on port 8000 so Launchpad can communicate with it.

3. Update Launchpad configuration to point to URL of the Voicebox Service.

   ```bash
   VOICEBOX_SERVICE_ENDPOINT=http://<host>:8000
   ```

4. Assuming you have restarted Launchpad with the Voicebox Service endpoint configured, you should be able to access Voicebox from the Launchpad UI.

https://github.com/user-attachments/assets/d49f3ddd-a62b-448d-9d03-733d32d9a2eb

### Configuration

The following options should be provided as environment variables to the Voicebox Service. Additional environment variables might be needed based on the LLM provider configured. See the Voicebox Configuration File section below for more details.

| Environment Variable | **Required** | **Description** | **Available Options** |
| --- | --- | --- | --- |
| `LOG_TYPE` | `N` | The format of the logs sent to STDOUT. Default is `JSON` | `TEXT`, `JSON`. |
| `LOG_LEVEL` | `N` | Modifies the log level. Default is `INFO`.  | `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `VBX_CONFIG_FILE` | `Y` | The **absolute path** to the Voicebox configuration file. You will need to mount a directory with the file in it. The Voicebox configuration file contains information like which LLM model provider and model you want to use. Example: `/config/vbx-azure-config.json` |  |

### Voicebox Configuration File

In addition to the environment variables provided to the Voicebox Service, the Voicebox Service also requires you to give it a configuration file in JSON.

Here's an example configuration file for using Voicebox with AWS Bedrock:

```json
{  
	"enable_external_llm": true,  
	"enable_analytics": true, 
	"enable_charts": true,  
	"default_llm_config": {    
		"llm_provider": "bedrock",    
		"llm_name": "us.meta.llama3-1-70b-instruct-v1:0"  
	}
}
```

The following table explains the fields that can be specified in this configuration file. LLM Configuration is explained in the following section.

| **Configuration Option** | **Required** | **Description** | Type |
| --- | --- | --- | --- |
| `default_llm_config` | `Y` | Configuration for the LLM provider. | **LLM Configuration** |
| `enable_analytics` | `N` | Enable the analytics agents that can perform further analysis over results returned from the Knowledge Graph | boolean |
| `enable_charts` | `N` | Enable the capability to turn tabular results in answers to charts | boolean |
| `enable_external_llm` | `N` | Enable the ability to use the LLM to answer questions with its background knowledge instead of the Knowledge Graph | boolean |
| `external_llm_config` | `N` | Configuration for an alternate LLM provider to use for background knowledge if `enable_external_llm` is enabled. If this configuration is not provided the `default_llm_config` will be used. | **LLM Configuration** |

#### LLM Configuration

The LLM configuration is specified as a JSON object in the Voicebox configuration file with the following fields.

| **Configuration Option** | **Required** | **Description** | Type |
| --- | --- | --- | --- |
| `llm_provider` | `Y` | Name of the LLM provider | String |
| `llm_name` | `Y` | Name of the LLM | String |
| `server_url` | `N` | Optional server URL for the LLM provider. May be required based on the provider configured. | URL |
| `max_tokens` | `N` | Maximum number of tokens for LLM requests. This limit can be used to control LLM costs to prevent LLM from returning very long responses. | Integer |

The following LLM providers are supported:
- [Azure AI](#azure-ai-configuration)
- [AWS Bedrock](#aws-bedrock-configuration)
- [Databricks](#databricks-configuration)
- [Fireworks](#fireworks-configuration)
- [Google Vertex](#google-vertex-configuration)
- [OpenAI](#openai-configuration)

#### Azure AI Configuration

Voicebox can use an Azure AI endpoint deployed within [Azure AI Foundry](https://azure.microsoft.com/en-us/products/ai-foundry). 

The following configuration options are used with Azure LLM in the Voicebox configuration file. Update the `AZURE_AI_ENDPOINT` in the server URL to point to your endpoint.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `azure` |
| `llm_name` | `Meta-Llama-3.1-70B-Instruct` , `Meta-Llama-3.3-70B-Instruct`, `Llama-4-Maverick-17B-128E-Instruct-FP8` |
| `server_url` | `https://AZURE_AI_ENDPOINT.services.ai.azure.com/models` |

The following environment variables are used with Azure.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `AZURE_API_KEY` | `Y` | Azure API key |

#### AWS Bedrock Configuration

Voicebox can use a [Bedrock endpoint](https://aws.amazon.com/bedrock/) deployed within AWS. 

The following configuration options are used with Bedrock LLM in the Voicebox configuration file.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `bedrock` |
| `llm_name` | `meta.llama3-1-70b-instruct-v1:0` , `us.meta.llama3-1-70b-instruct-v1:0`, `meta.llama4-maverick-17b-instruct-v1:0`, `us.meta.llama4-maverick-17b-instruct-v1:0`, (application inference profile name) |

AWS Bedrock allows users to create [application inference profiles](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-create.html) to track usage and costs when invoking a model. The ARN associated with an inference profile can be used as the `llm_name` in Voicebox configuration. All LLM calls initiated by Voicebox will be done using this inference profile.

The following environment variables are used with Bedrock.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `AWS_ACCESS_KEY_ID` | `Y` | AWS access key ID |
| `AWS_SECRET_ACCESS_KEY` | `Y` | AWS secret access key |
| `BEDROCK_PROFILE` | `N` | AWS profile that can be specified instead of the access key ID and secret access key |
| `BEDROCK_REGION` | `Y` | Name of the AWS region where the Bedrock LLM is deployed, e.g. `us-west-1` |

It is also possible to use IAM roles for accessing Bedrock models instead of specifying access keys if Voicebox service is running on an EC2 instance. The IAM role should have the permissions `bedrock:Get*`, `bedrock:List*`, `bedrock:InvokeModel`, `bedrock:InvokeModelWithResponseStream`. The AWS built-in policy `AmazonBedrockLimitedAccess` includes these permissions and can be used directly or a new role can be defined with these permissions. 

Once the IAM role containing correct permissions is defined, the role can be attached to the EC2 instance where the Voicebox service is running. For the IAM role to take effect none of the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, or `BEDROCK_PROFILE` should be set. When everything is configured correctly, in the Voicebox service logs, you should see a message as follows for the role you have defined:

```
Found credentials from IAM Role: VoiceboxBedrockRole
```

#### Databricks Configuration

Voicebox can use an LLM endpoint deployed within a [Databricks workspace](https://www.databricks.com/product/model-serving).

The following configuration options are used with Databricks LLM in the Voicebox configuration file. Update the `DATABRICKS_WORKSPACE` in the server URL to point to your workspace.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `databricks` |
| `llm_name` | `databricks-meta-llama-3-1-70b-instruct`, `databricks-meta-llama-3-3-70b-instruct`, `databricks-llama-4-maverick`  |
| `server_url` | `https://DATABRICKS_WORKSPACE.cloud.databricks.com/serving-endpoints` |

The following environment variables are used with Databricks.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `DATABRICKS_API_KEY` | `Y` | Databricks API key |

#### Fireworks Configuration

Voicebox can use [Fireworks.ai](http://Fireworks.ai) as an LLM endpoint.

The following configuration options are used with Fireworks in the Voicebox configuration file. 

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `fireworks` |
| `llm_name` | `accounts/fireworks/models/llama-v3p1-70b-instruct`, `accounts/fireworks/models/llama-v3p3-70b-instruct`, `accounts/fireworks/models/llama4-maverick-instruct-basic` |

The following environment variables are used with Fireworks.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `FIREWORKS_API_KEY` | `Y` | Fireworks API key |

#### Google Vertex Configuration

Voicebox can use Llama models hosted at [Google Vertex Model Garden](https://cloud.google.com/model-garden) as an LLM endpoint.

The following configuration options are used with Google Vertex in the Voicebox configuration file. Replace the `Google_Vertex_AI_Project_Name` value in the example with your project name.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `vertex` |
| `llm_name` | `meta/llama-3.1-70b-instruct-maas`, `meta/llama-3.3-70b-instruct-maas` |
| `provider_args` | { "project": "Google_Vertex_AI_Project_Name" } |

Note that, `provider_args` is a JSON object itself. An example LLM configuration for Google Vertex looks like this:
```json
{  
    "default_llm_config": {    
        "llm_provider": "vertex",
        "llm_name": "meta/llama-3.3-70b-instruct-maas",
        "provider_args": {
          "project" : "My Project Name"
        } 
    }
}
```

The following environment variables are used with Google Vertex.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `GOOGLE_APPLICATION_CREDENTIALS` | `Y` |  The location of a credential JSON file |

See [Google documentation](https://cloud.google.com/docs/authentication/application-default-credentials) for the details of creating credential files.

#### OpenAI Configuration

Voicebox can use [OpenAI](https://openai.com/api/) as an LLM endpoint.

The following configuration options are used with OpenAI in the Voicebox configuration file.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `openai` |
| `llm_name` | `gpt-4o`, `gpt-4o-mini` |
| `server_url` | (Optional - can be set if OpenAI endpoint is access via proxy)  |

It is also possible to provide optional custom HTTP headers included in the OpenAI requests. Here is an example configuration file showing how these custom headers can be configured:

```json
{  
    "default_llm_config": {    
        "llm_provider": "openai",
        "llm_name": "gpt-4o-mini",
        "server_url": "https://api.openai.com/v1/",
        "provider_args": {
          "headers" : {
        "OpenAI-Organization": "org-gnSjNrpIz0bb7V1modfLrNof",
        "OpenAI-Project": "$PROJECT_ID"
          }
        } 
    }
}
```

The values for custom headers should be valid [Python string templates](https://docs.python.org/3/library/string.html#template-strings). The variables mentioned in template string should be defined as environment variables, e.g. in the above example there should be an environment variable named `PROJECT_ID`. Environment variables is a better choice for including sensitive values in the configuration file whereas non-sensitive values can be directly included in the configuration file, e.g. organization value in the above example.

The following environment variables are used with OpenAI.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `OPENAI_API_KEY` | `Y` | OpenAI API key |

## JWT Authentication & Token Exchange

> [!IMPORTANT]
> This JWT authentication flow is currently only supported with **Okta** as the identity provider.

When deploying Voicebox in enterprise environments, you can enable JWT-based authentication to secure communication between services. This establishes a chain of trust from the user through to the LLM provider:

```
User → Launchpad → Voicebox Service → LLM Gateway (proxy to LLM provider)
```

**Authentication Flow Overview:**

1. **User authenticates** with Launchpad via Okta
2. **Launchpad exchanges** the user's token for a Voicebox service token using OAuth 2.0 On-Behalf-Of (OBO) flow
3. **Voicebox Service validates** the incoming token and exchanges it for an LLM Gateway token
4. **LLM Gateway** (a proxy that routes requests to your LLM provider like Azure OpenAI) **receives** a scoped token that can be traced back to the original user

#### UI Flow (User Logged into Launchpad)

```mermaid
sequenceDiagram
    participant User
    participant Launchpad
    participant Backend as Launchpad Backend App
    participant AuthzA as Okta Authz Server A
    participant Voicebox as Voicebox Service
    participant AuthzB as Okta Authz Server B
    participant LLM Gateway

    User->>Launchpad: Login request
    Launchpad->>AuthzA: OIDC authentication
    AuthzA-->>Launchpad: User access token
    Launchpad-->>User: Session established

    User->>Launchpad: Voicebox request
    Backend->>AuthzA: Token exchange (OBO)<br/>user token → voicebox token
    AuthzA-->>Backend: Voicebox service token
    Launchpad->>Voicebox: Request (with voicebox-scoped JWT)
    Voicebox->>AuthzA: Validate token
    Voicebox->>AuthzB: Token exchange (OBO)<br/>voicebox token → llm-gateway token
    AuthzB-->>Voicebox: LLM Gateway token
    Voicebox->>LLM Gateway: Request (with llm-gateway-scoped JWT)
    LLM Gateway-->>Voicebox: LLM response
    Voicebox-->>Launchpad: Response
    Launchpad-->>User: Response
```

> [!NOTE]
> This diagram shows the LLM Gateway expecting tokens issued by a different authorization server (Authz Server B, configured as a trusted server in Authz Server A). The Voicebox Service validates incoming tokens against Authz Server A, then exchanges them with Authz Server B to obtain a token with the scope required by the LLM Gateway. When all components share the same authorization server, the Voicebox Service performs both validation and exchange against Authz Server A.

#### Public API Flow (External Client)

```mermaid
sequenceDiagram
    participant Client as Public API Client
    participant AuthzC as Okta Authz Server C
    participant Launchpad
    participant Backend as Launchpad Backend App
    participant AuthzA as Okta Authz Server A
    participant Voicebox as Voicebox Service
    participant AuthzB as Okta Authz Server B
    participant LLM Gateway

    Client->>AuthzC: Authorization code flow
    AuthzC-->>Client: Access token

    Client->>Launchpad: API request<br/>Authorization: Bearer {token}<br/>X-Voicebox-App-Key: {key}
    Launchpad->>AuthzC: Validate token
    Backend->>AuthzA: Token exchange (OBO)<br/>client token → voicebox token
    AuthzA-->>Backend: Voicebox service token
    Launchpad->>Voicebox: Request (with voicebox-scoped JWT)
    Voicebox->>AuthzA: Validate token
    Voicebox->>AuthzB: Token exchange (OBO)<br/>voicebox token → llm-gateway token
    AuthzB-->>Voicebox: LLM Gateway token
    Voicebox->>LLM Gateway: Request (with llm-gateway-scoped JWT)
    LLM Gateway-->>Voicebox: LLM response
    Voicebox-->>Launchpad: Response
    Launchpad-->>Client: Response
```

> [!NOTE]
> This diagram shows three different authorization servers: Authz Server C issues tokens for the Public API Client, Authz Server A is used by Launchpad and Voicebox Service, and Authz Server B issues tokens for the LLM Gateway. Launchpad validates the client's token against Authz Server C (configured as a trusted server in Authz Server A), then the Backend App exchanges it with Authz Server A for a voicebox-scoped token. When all components share the same authorization server, all validation and exchange operations use that single server.

This architecture provides:
- **Scoped Authorization**: Each service receives only the permissions it needs
- **Audit Trail**: User identity flows through the entire chain for compliance

> [!IMPORTANT]
> This authentication flow requires using an **OpenAI-compatible LLM provider** (`llm_provider: openai` in your Voicebox configuration). Token exchange for the LLM Gateway is not supported with other providers (Bedrock, Databricks, etc.).

### Okta Authorization Server Setup

#### Authorization Server Architecture

The authentication chain involves multiple components. Launchpad (login and backend) and Voicebox Service **must** use the same authorization server. Other components can optionally use different authorization servers.

| Component | Can Use Different Auth Server? | Notes |
| --- | --- | --- |
| Launchpad Login App | Yes (base) | This is the primary authorization server |
| Launchpad Backend App | **No** | Must use the same authorization server as the Login App |
| Voicebox Service | **No** | Must use the same authorization server as Launchpad |
| Public API Client | Yes | If using a different auth server for the Public API, add Launchpad's auth server as a trusted server |
| LLM Gateway | Yes | If using a different auth server, add Voicebox's auth server as a trusted server |

> [!IMPORTANT]
> When using different authorization servers for external components (Public API Client or LLM Gateway), you must configure **trusted server** relationships in Okta. The authorization server that issues tokens for a downstream service must trust the authorization server that issued the incoming tokens.
>
> To add a trusted server: In your authorization server settings, navigate to the trusted servers configuration and add the issuer URL of the upstream authorization server.

#### Step 1: Create or Configure a Custom Authorization Server

1. In the Okta Admin Console, navigate to **Security** → **API** → **Authorization Servers**
2. Click **Add Authorization Server** (or use an existing custom server)
3. Configure the server:
   - **Name**: e.g., "Stardog Services"
   - **Audience**: e.g., `stardog-services` (note this value - it will be used in multiple places)
4. After saving, note the **Authorization Server ID** from the Metadata URI:
   ```
   https://{domain}/oauth2/{AUTHORIZATION_SERVER_ID}/.well-known/oauth-authorization-server
   ```

#### Step 2: Create Required Scopes

Create custom scopes for accessing the Voicebox service and LLM Gateway.

1. In your authorization server, go to the **Scopes** tab
2. Click **Add Scope** and create the following scopes:

| Scope Name | Description |
| --- | --- |
| `voicebox` | Access to Voicebox service (used by Launchpad → Voicebox). You can name this whatever you prefer. |
| `ai-gateway` | Access to LLM Gateway (used by Voicebox → LLM Gateway). This must match the scope your LLM Gateway/proxy expects for authentication. It can be named whatever you prefer, but ensure it matches the LLM Gateway configuration. |

> [!NOTE]
> If your LLM Gateway uses a **different authorization server** than Launchpad/Voicebox, the `ai-gateway` scope must be created on (or already exist in) **that** authorization server, not the one documented here. The scope name must match what the LLM Gateway expects for authentication.

#### Step 3: Create a Backend API Services Application for Launchpad

Okta requires a separate API Services application to perform token exchange operations. This is different from the web application used for user login.

1. Navigate to **Applications** → **Applications**
2. Click **Create App Integration**
3. Select **API Services** as the application type
4. Configure:
   - **App integration name**: e.g., "Launchpad Backend"
5. After creation, under **General**:
   - Note the **Client ID** (for `OKTA_BACKEND_CLIENT_ID`)
   - Note the **Client Secret** (for `OKTA_BACKEND_CLIENT_SECRET`), or configure public/private key authentication
   - Do NOT check "Require Demonstrating Proof of Possession (DPoP) header in token requests"
6. Under **General** → **Advanced** → **Non-interactive grants**:
   - Check **Token Exchange**

#### Step 4: Create an API Services Application for Voicebox Service

Create another API Services application for the Voicebox Service to exchange tokens for LLM Gateway access.

1. Navigate to **Applications** → **Applications**
2. Click **Create App Integration** → **API Services**
3. Configure:
   - **App integration name**: e.g., "Voicebox Service"
4. Under **General**:
   - Note the **Client ID** (for Voicebox's `OKTA_CLIENT_ID`)
   - Configure client secret or private key authentication
   - Do NOT check "Require Demonstrating Proof of Possession (DPoP) header in token requests"
5. Under **General** → **Advanced** → **Non-interactive grants**:
   - Check **Token Exchange**

#### Step 5: Configure Access Policies

Create access policies in your authorization server to control token issuance for each application.

**Policy 1: Launchpad Login Policy**

This policy allows users to log into Launchpad via the web application.

1. In your authorization server, go to **Access Policies** tab
2. Click **Add Policy**
3. Configure:
   - **Name**: "Launchpad Login Policy"
   - **Assign to**: Select your Launchpad login web application
4. Click **Create Policy**, then **Add Rule**
5. Configure the rule:
   - **Rule Name**: "Allow user login"
   - **Scopes**: Add `openid`, `profile`, `email`, `address`, `phone`, `offline_access` (or click "OIDC default scopes")
6. Click **Create Rule**

**Policy 2: Launchpad Token Exchange Policy**

This policy allows the Launchpad backend to exchange user tokens for Voicebox service tokens.

1. Click **Add Policy**
2. Configure:
   - **Name**: "Launchpad Token Exchange Policy"
   - **Description**: "Allows Launchpad to exchange user tokens for service-scoped tokens"
   - **Assign to**: Select your Launchpad Backend API Services application (from Step 3)
3. Click **Create Policy**, then **Add Rule**
4. Configure the rule:
   - **Rule Name**: "Allow Voicebox Token Exchange"
   - Under **Advanced** → **Core grants**: Check **Token Exchange**
   - **Scopes requested**: Add your Voicebox scope (e.g., `voicebox`)
5. Click **Create Rule**

**Policy 3: Voicebox to LLM Gateway Token Exchange Policy**

This policy allows the Voicebox Service to exchange tokens for LLM Gateway access.

1. Click **Add Policy**
2. Configure:
   - **Name**: "Voicebox to LLM Gateway Token Exchange"
   - **Assign to**: Select the Voicebox Service application (from Step 4)
3. Click **Create Policy**, then **Add Rule**
4. Configure the rule:
   - **Rule Name**: "Exchange for LLM Gateway"
   - Under **Advanced** → **Core grants**: Check **Token Exchange**
   - **Scopes requested**: Add your LLM Gateway scope (e.g., `ai-gateway`)
5. Click **Create Rule**

### Launchpad Configuration

Add the following environment variables to your Launchpad configuration to enable OBO token exchange with the Voicebox Service:

```bash
# Custom Authorization Server
OKTA_AUTHORIZATION_SERVER_ID=<your-authorization-server-id>
OKTA_AUTHORIZATION_SERVER_AUDIENCE=<your-audience-value>

# Backend API Services App (for token exchange)
OKTA_BACKEND_CLIENT_ID=<your-api-services-client-id>
OKTA_BACKEND_CLIENT_SECRET=<your-api-services-client-secret>
# OR use private key authentication:
# OKTA_BACKEND_CLIENT_PRIVATE_KEY_FILE=/path/to/private-key.pem

# Voicebox Service Scope
VOICEBOX_SERVICE_SCOPE=<your-voicebox-scope>
```

| Environment Variable | Required | Description |
| --- | --- | --- |
| `OKTA_AUTHORIZATION_SERVER_ID` | Yes | ID of the custom authorization server |
| `OKTA_AUTHORIZATION_SERVER_AUDIENCE` | Yes | Audience configured in the authorization server |
| `OKTA_BACKEND_CLIENT_ID` | Yes | Client ID of the Launchpad Backend API Services app |
| `OKTA_BACKEND_CLIENT_SECRET` | Conditional | Client secret (if not using private key) |
| `OKTA_BACKEND_CLIENT_PRIVATE_KEY_FILE` | Conditional | Path to private key file (if not using client secret) |
| `VOICEBOX_SERVICE_SCOPE` | Yes | Scope for Voicebox service access |

> [!NOTE]
> If both `OKTA_BACKEND_CLIENT_SECRET` and `OKTA_BACKEND_CLIENT_PRIVATE_KEY_FILE` are configured, private key authentication takes precedence.

#### Complete Launchpad Configuration Example

```bash
# Login Web App
OKTA_AUTH_ENABLED=true
OKTA_DOMAIN=your-domain.okta.com
OKTA_CLIENT_ID=0oasvuwo2ktPBT6Jb697
OKTA_CLIENT_SECRET=<login-app-client-secret>
OKTA_POST_LOGOUT_REDIRECT_URI=http://localhost:8080
OKTA_REQUIRE_PKCE=true

# Custom Authorization Server (shared across Launchpad and Voicebox Service)
OKTA_AUTHORIZATION_SERVER_ID=ausx5bsy1p2Oy2RVm697
OKTA_AUTHORIZATION_SERVER_AUDIENCE=stardog-services

# Backend API Services App (for token exchange)
OKTA_BACKEND_CLIENT_ID=0oax6j3ssnoYAPeo2697
OKTA_BACKEND_CLIENT_SECRET=<backend-client-secret>

# Voicebox Service
VOICEBOX_SERVICE_ENDPOINT=http://voicebox-service:8000
VOICEBOX_SERVICE_SCOPE=voicebox
```

### Voicebox Service Configuration

Configure the Voicebox Service to validate incoming JWTs from Launchpad and exchange them for LLM Gateway tokens.

#### JWT Authentication Settings

| Environment Variable | Required | Description |
| --- | --- | --- |
| `REQUIRE_JWT_AUTH` | Yes | Set to `true` to require JWT authentication |
| `JWT_ISSUER` | Yes | Issuer URL: `https://{domain}/oauth2/{authorization-server-id}` |
| `JWT_JWKS_URI` | Yes | JWKS URL: `https://{domain}/oauth2/{authorization-server-id}/v1/keys` |
| `JWT_AUDIENCE` | Yes | Expected audience, must match Launchpad's `OKTA_AUTHORIZATION_SERVER_AUDIENCE` |
| `JWT_REQUIRED_SCOPE` | Yes | Required scope, must match Launchpad's `VOICEBOX_SERVICE_SCOPE` |

Optional JWT settings:

| Environment Variable | Default | Description |
| --- | --- | --- |
| `JWT_SCOPE_CLAIM_NAME` | `scp` | Name of the scope claim in the JWT |
| `JWT_SUBJECT_CLAIM_NAME` | `sub` | Name of the subject claim in the JWT |
| `JWT_ALLOWED_ALGORITHMS` | `RS256` | Allowed signing algorithms |
| `JWT_LEEWAY_SECONDS` | `60` | Clock skew tolerance in seconds |
| `JWKS_CACHE_TTL` | `3600` | How long to cache JWKS keys (seconds) |

#### Token Exchange Settings (for LLM Gateway)

| Environment Variable | Required | Description |
| --- | --- | --- |
| `TOKEN_EXCHANGE_PROVIDER` | Yes | Set to `okta` |
| `OKTA_CLIENT_ID` | Yes | Voicebox Service application client ID |
| `OKTA_DISCOVERY_URL` | Yes | OpenID Connect discovery URL for the authorization server |
| `OKTA_LLM_GATEWAY_AUDIENCE` | Yes | Audience for the LLM Gateway tokens |
| `OKTA_LLM_GATEWAY_SCOPE` | Yes | Scope to request for LLM Gateway access (must match what the LLM Gateway expects) |
| `OKTA_CLIENT_SECRET` | Conditional | Client secret (if not using private key) |
| `OKTA_PRIVATE_KEY_PATH` | Conditional | Path to private key file (if not using client secret) |

> [!NOTE]
> **Using a different authorization server for LLM Gateway:** If the LLM Gateway uses a separate authorization server, set `OKTA_DISCOVERY_URL` to point to that server's discovery endpoint and `OKTA_LLM_GATEWAY_AUDIENCE` to match its audience. You must also add Voicebox's authorization server as a trusted server in the LLM Gateway's authorization server configuration.

Optional token exchange settings:

| Environment Variable | Default | Description |
| --- | --- | --- |
| `TOKEN_CACHE_TTL_SECONDS` | `3600` | Maximum time to cache exchanged tokens |
| `TOKEN_CACHE_EXPIRY_BUFFER_SECONDS` | `300` | Refresh tokens this many seconds before expiry |

#### Voicebox Configuration File

When using JWT authentication with token exchange, you must configure the Voicebox Service to use the `openai` LLM provider. The exchanged LLM Gateway token will be automatically injected into the `Authorization` header for requests to the LLM Gateway.

See [Voicebox Configuration File](#voicebox-configuration-file) for the full configuration reference. Here's an example configuration for use with JWT authentication:

```json
{
  "enable_external_llm": true,
  "enable_analytics": true,
  "enable_charts": true,
  "default_llm_config": {
    "llm_provider": "openai",
    "llm_name": "gpt-4o",
    "server_url": "https://your-llm-gateway.example.com/v1/",
    "provider_args": {
      "headers": {
        "X-LLM-Gateway-Id": "your-gateway-id"
      }
    }
  }
}
```

> [!IMPORTANT]
> The `llm_provider` must be set to `openai` for token exchange to work. The `server_url` should point to your LLM Gateway/proxy endpoint. You can include additional custom headers your LLM Gateway requires via `provider_args.headers`.

#### Complete Voicebox Service Configuration Example

```bash
# JWT Authentication (validates tokens from Launchpad)
REQUIRE_JWT_AUTH=true
JWT_ISSUER=https://your-domain.okta.com/oauth2/ausx5bsy1p2Oy2RVm697
JWT_JWKS_URI=https://your-domain.okta.com/oauth2/ausx5bsy1p2Oy2RVm697/v1/keys
JWT_AUDIENCE=stardog-services
JWT_REQUIRED_SCOPE=voicebox

# Token Exchange for LLM Gateway (using same authorization server)
TOKEN_EXCHANGE_PROVIDER=okta
OKTA_CLIENT_ID=0oa456def
OKTA_CLIENT_SECRET=<voicebox-service-client-secret>
OKTA_DISCOVERY_URL=https://your-domain.okta.com/oauth2/ausx5bsy1p2Oy2RVm697/.well-known/openid-configuration
OKTA_LLM_GATEWAY_AUDIENCE=stardog-services
OKTA_LLM_GATEWAY_SCOPE=ai-gateway

# Voicebox configuration file (must use openai provider)
VBX_CONFIG_FILE=/voicebox-config/vbx-config.json
LOG_LEVEL=INFO
```

### Public API Authentication (JWT)

In addition to session-based authentication via the Launchpad UI, external applications can access the Voicebox API using JWT-based authentication. This allows programmatic access where the calling application authenticates users with the identity provider and passes their access tokens to Launchpad.

#### Enabling Public API JWT Authentication

Add the following environment variables to Launchpad:

```bash
# Enable JWT authentication for public API
API_AUTH_JWT_ENABLED=true

# Token validation settings
API_AUTH_JWKS_URI=https://your-domain.okta.com/oauth2/{auth-server-id}/v1/keys
API_AUTH_ISSUER=https://your-domain.okta.com/oauth2/{auth-server-id}
API_AUTH_AUDIENCE=<your-auth-server-audience>

# Optional settings
API_AUTH_REQUIRED_SCOPES=openid,profile  # Comma-separated list of required scopes
API_AUTH_SCOPE_CLAIM_NAME=scp            # Claim name for scopes (default: checks "scp" then "scope")
API_AUTH_JWT_ALGORITHMS=RS256            # Allowed algorithms (default: RS256)

# Required when multiple login providers are enabled
API_AUTH_ACCESS_TOKEN_IDP=okta           # Which IDP to use for token exchange
```

| Environment Variable | Required | Description |
| --- | --- | --- |
| `API_AUTH_JWT_ENABLED` | Yes | Set to `true` to enable JWT authentication for public API |
| `API_AUTH_JWKS_URI` | Yes | URL to fetch public keys for validating access tokens |
| `API_AUTH_ISSUER` | Yes | Expected issuer claim in access tokens |
| `API_AUTH_AUDIENCE` | Yes | Expected audience claim in access tokens |
| `API_AUTH_REQUIRED_SCOPES` | No | Comma-separated list of scopes that must be present |
| `API_AUTH_SCOPE_CLAIM_NAME` | No | Name of the scope claim (default: `scp`) |
| `API_AUTH_JWT_ALGORITHMS` | No | Allowed signing algorithms (default: `RS256`) |
| `API_AUTH_ACCESS_TOKEN_IDP` | Conditional | Required when multiple login providers are enabled |

#### Making API Requests with JWT Authentication

When `API_AUTH_JWT_ENABLED=true`, API clients must provide:

1. **`Authorization: Bearer <access_token>`** - User's access token from the identity provider
2. **`X-Voicebox-App-Key: <app_key>`** - The Voicebox application API key
3. **`X-Client-Id: <client_id>`** - Client identifier for the request
4. **`X-SD-Auth-Token: <stardog_token>`** (optional) - Override the Stardog authentication token. See [Stardog Authentication for API Requests](#stardog-authentication-for-api-requests) for details.

```bash
# Get an access token from your IDP (e.g., via OAuth authorization code flow)
ACCESS_TOKEN="eyJhbGciOiJSUzI1NiIsInR5cCI6..."

# Make a Voicebox request
curl -X POST "https://launchpad.example.com/api/v1/voicebox/ask" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "X-Voicebox-App-Key: your-voicebox-app-key" \
  -H "X-Client-Id: my-client-123" \
  -H "Content-Type: application/json" \
  -d '{"message": "Who is Bob?"}'
```

> [!NOTE]
> The `X-Voicebox-App-Key` header separates the Voicebox application API key from the user's access token. When `API_AUTH_JWT_ENABLED=false`, you can continue using the `Authorization` header for the Voicebox App Key as documented in [Using Voicebox Programmatically](#using-voicebox-programmatically).

## Release Notes

The Voicebox Service is released independently of Launchpad. 

> [!NOTE] 
> All available releases of the Voicebox Service are listed below. The image tag for a release is simply the release name prepended with `v` as in `v0.1.1`.


## 0.24.0 Release (Dec 11, 2025)

* Sanitize XSD IRIs in example queries
* Handle default prefix in schema serialization
* Prevent dataset description errors when statistics is missing
* Fix date/time reference errors in result summarization
* Use more robust LLM formatting during query linting
* Do not include BITES schema when querying the knowledge graph
* Backend support for on behalf of flow token exchange for LLM providers

## 0.23.0 Release (Nov 19, 2025)

* Several improvements to Think Mode
  * Better handle large outputs
  * Improve responses when an answer is not found
  * Support user-configured LLMs for powering Think Mode
* Enhancements to model and mapping creation in Designer
  * Create more detailed project summaries using markdown
  * Improve evaluation of competency questions
  * Generate synthetic data
* Increase default max token configuration for query generation
* Handle escaped characters that are included in generated queries
* Consider inferences when computing query lineage
* Update dependencies to address vulnerabilities

## 0.22.0 Release (Oct 16, 2025)

* Support for generating SPARQL queries for competenecy questions in Designer
* Include Voicebox core version in the diagnostics report

## 0.21.0 Release (Oct 2, 2025)

* Support for competency question evaluation functionality in Designer
* Use entity summarization from the KG with RAG
* Add chunk IRIs to the RAG response
* Handle incomplete tags in LLM output during query generation
* Support for virtual graphs in Think mode
* Extend support for local prefixes for plain query-generation
* Upgrade Docker image to use Debian 13 and pull in OS patches
  
## 0.20.2 Release (Aug 21, 2025)

* Fix compatibility issues with AMD processors
* Improve handling of binary data, date/time fields, and token limits when generating mappings for Designer
* Return labels of instances from virtual graphs

## 0.20.1 Release (Jul 21, 2025)
## 0.20.0 Release (Jul 10, 2025)
## 0.19.0 Release (Jun 20, 2025)
## 0.18.10 Release (May 12, 2025)
## 0.18.9 Release (Apr 17, 2025)
