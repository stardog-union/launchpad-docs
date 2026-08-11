# Configuring the Voicebox Service

How to run and configure the Voicebox Service: network routing to Stardog, the environment variables it accepts, the `vbx-config.json` file and its supported LLM providers, and per-request customization.

> [!NOTE]
> Except where a step is marked otherwise, everything on this page applies to both the current Voicebox Service (`v1.0.0+`) and the previous-generation `0.x` service — the configuration file, the LLM providers, custom headers, and JWT authentication are the same in both. Storage backends and operational tuning are specific to `v1.0.0+` and are covered in [Deploying the Voicebox Service](./voicebox-deployment.md).

For what Voicebox is and how to use it, see [Voicebox](../voicebox.md).

## Running the Voicebox Service

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
   -v voicebox-frames:/var/lib/voicebox/frames \
   stardog-stardog-apps.jfrog.io/voicebox-service:current
   ```

   - `.env.voicebox-service` can be named anything but contains the configuration for the Voicebox Service. See [Configuration](#configuration) for more details.
   - `/host/path/to/vbx-config.json` is mounted from the host to `/voicebox-config/vbx-config.json` in the container. This configuration has more LLM specific configuration.
      - `.env.voicebox-service` should have `VBX_CONFIG_FILE=/voicebox-config/vbx-config.json` in it.
   - `voicebox-frames` is a named volume mounted at the frame store path, where the service persists query results. This is required as of `v1.0.0` when using the default local-disk frame store: without it, frames are written to the container's writable layer and lost on restart. Omit it if you configure an [S3 or Azure Blob frame store](./voicebox-deployment.md#frame-stores) instead. See [Deploying the Voicebox Service](./voicebox-deployment.md) for sizing, tuning, and the Kubernetes equivalent.
   - The Voicebox Service HTTP server is exposed on port 8000 so Launchpad can communicate with it.

3. Update Launchpad configuration to point to URL of the Voicebox Service.

   ```bash
   VOICEBOX_SERVICE_ENDPOINT=http://<host>:8000
   ```

4. Assuming you have restarted Launchpad with the Voicebox Service endpoint configured, you should be able to access Voicebox from the Launchpad UI.

https://github.com/user-attachments/assets/d49f3ddd-a62b-448d-9d03-733d32d9a2eb

## Configuration

The following options should be provided as environment variables to the Voicebox Service. Additional environment variables might be needed based on the LLM provider configured. See the Voicebox Configuration File section below for more details.

| Environment Variable | **Required** | **Description** | **Available Options** |
| --- | --- | --- | --- |
| `LOG_TYPE` | `N` | The format of the logs sent to STDOUT. Default is `JSON` | `TEXT`, `JSON`. |
| `LOG_LEVEL` | `N` | Modifies the log level. Default is `INFO`.  | `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `VBX_CONFIG_FILE` | `Y` | The **absolute path** to the Voicebox configuration file. You will need to mount a directory with the file in it. The Voicebox configuration file contains information like which LLM model provider and model you want to use. Example: `/config/vbx-azure-config.json` |  |

## Voicebox Configuration File

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

### LLM Configuration

The LLM configuration is specified as a JSON object in the Voicebox configuration file with the following fields.

| **Configuration Option** | **Required** | **Description** | Type |
| --- | --- | --- | --- |
| `llm_provider` | `Y` | Name of the LLM provider | String |
| `llm_name` | `Y` | Name of the LLM | String |
| `server_url` | `N` | Optional server URL for the LLM provider. May be required based on the provider configured. | URL |
| `max_tokens` | `N` | Maximum number of tokens for LLM requests. This limit can be used to control LLM costs to prevent LLM from returning very long responses. | Integer |
| `provider_args` | `N` | Provider-specific arguments. See [Custom Headers](#custom-headers) for details. | Object |

The following LLM providers are supported:
- [Anthropic](#anthropic-configuration)
- [Azure AI](#azure-ai-foundry-configuration)
- [AWS Bedrock](#aws-bedrock-configuration)
- [Databricks](#databricks-configuration)
- [Fireworks](#fireworks-configuration)
- [Google Vertex](#google-vertex-configuration)
- [OpenAI](#openai-configuration)

### Anthropic Configuration

Voicebox can use [Anthropic](https://www.anthropic.com/) as an LLM provider, either directly via the Anthropic API or through [AWS Bedrock](#aws-bedrock-configuration) and Azure AI Foundry endpoints that host Anthropic models.

The following configuration options are used with Anthropic in the Voicebox configuration file.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `anthropic` |
| `llm_name` | `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` |
| `server_url` | (Optional — set to an Azure AI Foundry endpoint to use Azure-hosted Anthropic models, e.g. `https://AZURE_AI_ENDPOINT.services.ai.azure.com/anthropic/`) |

The following environment variables are used with Anthropic.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `ANTHROPIC_API_KEY` | `Y` (unless using Azure AI Foundry with `AZURE_API_KEY`) | Anthropic API key |

When `server_url` points to an Azure AI Foundry endpoint, you can use either `ANTHROPIC_API_KEY` or `AZURE_API_KEY` for authentication. If both are set, `ANTHROPIC_API_KEY` takes priority.

Anthropic models can also be used via [AWS Bedrock](#aws-bedrock-configuration) using the `bedrock` provider.

### Azure AI Foundry Configuration

Voicebox can use an Azure AI endpoint deployed within [Azure AI Foundry](https://azure.microsoft.com/en-us/products/ai-foundry).

The following configuration options are used with Azure LLM in the Voicebox configuration file. Update the `AZURE_AI_ENDPOINT` in the server URL to point to your endpoint.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `azure` |
| `llm_name` | `Meta-Llama-3.1-70B-Instruct` , `Meta-Llama-3.3-70B-Instruct`, `Llama-4-Maverick-17B-128E-Instruct-FP8` |
| `server_url` | `https://AZURE_AI_ENDPOINT.services.ai.azure.com/models` |

Azure AI also supports [custom headers](#custom-headers) via `provider_args.headers`.

#### API Key Authentication

The simplest way to authenticate with Azure AI is using an API key.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `AZURE_API_KEY` | `Y` | Azure API key |

#### Service Principal (SPN) Authentication

As an alternative to API key authentication, Voicebox supports authenticating to Azure AI Services using a Service Principal via the OAuth 2.0 client credentials flow. When SPN variables are configured, the API key is not required.

**Client Secret Authentication**

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `AZURE_TENANT_ID` | `Y` | Azure Entra ID tenant ID |
| `AZURE_CLIENT_ID` | `Y` | Service Principal application (client) ID |
| `AZURE_CLIENT_SECRET` | `Y` | Service Principal client secret |

**Certificate Authentication**

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `AZURE_TENANT_ID` | `Y` | Azure Entra ID tenant ID |
| `AZURE_CLIENT_ID` | `Y` | Service Principal application (client) ID |
| `AZURE_CLIENT_CERTIFICATE_PATH` | `Y` | Path to PEM file containing private key and certificate |
| `AZURE_CLIENT_CERTIFICATE_PASSWORD` | `N` | Password for the certificate private key, if encrypted |

> [!NOTE]
> If both certificate and client secret are configured, certificate authentication takes priority. If neither SPN method is configured, Voicebox falls back to `AZURE_API_KEY`.

**Additional SPN Configuration**

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `AZURE_CREDENTIAL_SCOPES` | `N` | Token audience scope(s) for the OAuth 2.0 client credentials request, comma-separated. Defaults to `https://cognitiveservices.azure.com/.default`. |
| `AZURE_AUTHORITY_HOST` | `N` | Authority host for sovereign clouds. Defaults to `login.microsoftonline.com`. |

**Azure Setup for SPN Authentication**

1. **Create a Service Principal**: Go to **Microsoft Entra ID** → **App registrations** → **New registration**. Name it (e.g. `voicebox-service`) and register. Note the **Application (client) ID** and **Directory (tenant) ID** from the Overview page.

2. **Create credentials**:
   - **Option A: Client Secret** — Go to **Certificates & secrets** → **Client secrets** → **New client secret**. Set a description and expiration, then copy the secret **Value** (shown only once).
   - **Option B: Certificate** — Generate a certificate, upload the public cert to the app registration, and mount the combined private key + cert PEM file into the Voicebox container. Set `AZURE_CLIENT_CERTIFICATE_PATH` to the mount path.

3. **Assign the [Cognitive Services User](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/ai-machine-learning#cognitive-services-user) role**: Go to your **Azure AI Services resource** → **Access control (IAM)** → **Add role assignment**. Search for **Cognitive Services User**, select it, and assign it to your Service Principal.

### AWS Bedrock Configuration

Voicebox can use a [Bedrock endpoint](https://aws.amazon.com/bedrock/) deployed within AWS. 

The following configuration options are used with Bedrock LLM in the Voicebox configuration file.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `bedrock` |
| `llm_name` | `meta.llama3-1-70b-instruct-v1:0` , `us.meta.llama3-1-70b-instruct-v1:0`, `meta.llama4-maverick-17b-instruct-v1:0`, `us.meta.llama4-maverick-17b-instruct-v1:0`, `us.anthropic.claude-haiku-4-5-20251001-v1:0`, (application inference profile name) |

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

### Databricks Configuration

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

### Fireworks Configuration

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

### Google Vertex Configuration

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

### OpenAI Configuration

Voicebox can use [OpenAI](https://openai.com/api/) as an LLM endpoint.

The following configuration options are used with OpenAI in the Voicebox configuration file.

| **Configuration Option** | **Available Options** |
| --- | --- |
| `llm_provider` | `openai` |
| `llm_name` | `gpt-4o`, `gpt-4o-mini` |
| `server_url` | (Optional - can be set if OpenAI endpoint is access via proxy)  |

OpenAI also supports [custom headers](#custom-headers) via `provider_args.headers`.

The following environment variables are used with OpenAI.

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `OPENAI_API_KEY` | `Y` (unless `basic_auth` is configured) | OpenAI API key |

#### Basic Authentication

For OpenAI-compatible proxies that require HTTP Basic authentication instead of an API key, you can configure `basic_auth` in `provider_args`. When `basic_auth` is configured, `OPENAI_API_KEY` is not required.

```json
{
    "default_llm_config": {
        "llm_provider": "openai",
        "llm_name": "gpt-4o",
        "server_url": "https://your-llm-proxy.example.com/v1/",
        "provider_args": {
            "basic_auth": {
                "client_id": "my-client-id",
                "client_secret": "$LLM_CLIENT_SECRET"
            }
        }
    }
}
```

The `client_id` and `client_secret` are Base64-encoded at runtime to produce an `Authorization: Basic <token>` header. Values support the same `$ENV_VAR` template substitution used by `provider_args.headers` — use environment variable references (e.g. `$LLM_CLIENT_SECRET`) for sensitive values and hardcoded strings for non-sensitive values like `client_id`.

An optional `auth_scheme` field can be set within `basic_auth` to change the authorization scheme (defaults to `"Basic"`).

## Custom Headers

The OpenAI and Azure AI providers support custom HTTP headers included in LLM requests. Custom headers are configured via `provider_args.headers` in the LLM configuration. Here is an example:

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

Header values support [Python string template](https://docs.python.org/3/library/string.html#template-strings) substitution. Variables referenced in the template string (e.g. `$PROJECT_ID`) should be defined as environment variables on the Voicebox Service. Environment variables are a better choice for sensitive values whereas non-sensitive values can be directly included in the configuration file.

### `$VOICEBOX_USER` Variable

In addition to environment variables, the special variable `$VOICEBOX_USER` is automatically resolved to the identity of the user making the request. For web users this is the authenticated username. For [Public API](../voicebox.md#using-voicebox-programmatically) requests this is the `X-Client-Id` header value.

This allows LLM requests to carry user identity, which can be useful for auditing and access control at the LLM provider level:

```json
{  
    "default_llm_config": {    
        "llm_provider": "azure",
        "llm_name": "Meta-Llama-3.3-70B-Instruct",
        "server_url": "https://my-endpoint.services.ai.azure.com/models",
        "provider_args": {
          "headers" : {
            "X-User-Identity": "$VOICEBOX_USER"
          }
        } 
    }
}
```

## Advanced Customization

For deployments with multiple Stardog servers or databases, you can provide different Voicebox configurations for each endpoint and database combination. This allows you to customize LLM settings, enable/disable features, or use different models based on the target Stardog server.

### Configuration Directory

Instead of (or in addition to) a single configuration file, you can specify a directory containing multiple configuration files:

| Environment Variable | **Required** | **Description** |
| --- | --- | --- |
| `VBX_CONFIG_DIR` | `N` | The **absolute path** to a directory containing Voicebox configuration files. All `.json` files in this directory will be loaded. |

### Endpoint and Database Matching

Each configuration file can include `endpoint` and `database` fields to specify which Stardog connections it applies to:

```json
{
  "endpoint": "https://stardog-prod.example.com:5820",
  "database": "my-database",
  "default_llm_config": {
    "llm_provider": "bedrock",
    "llm_name": "us.meta.llama3-1-70b-instruct-v1:0"
  }
}
```

| **Field** | **Description** |
| --- | --- |
| `endpoint` | The Stardog server URL to match. Use `*` to match any endpoint. |
| `database` | The database name to match. Use `*` to match any database. |

### Matching Priority

When a request is made, Voicebox selects the most specific matching configuration:

1. **Exact match** - Config with matching endpoint AND database
2. **Endpoint wildcard** - Config with matching endpoint and `database: "*"`
3. **Global wildcard** - Config with `endpoint: "*"` and `database: "*"`

### Example Setup

```
/voicebox-config/
├── default.json           # endpoint: "*", database: "*"
├── prod-server.json       # endpoint: "https://prod.example.com:5820", database: "*"
└── prod-analytics.json    # endpoint: "https://prod.example.com:5820", database: "analytics"
```

With this setup:
- Requests to `prod.example.com` with database `analytics` use `prod-analytics.json`
- Requests to `prod.example.com` with any other database use `prod-server.json`
- All other requests use `default.json`

> [!NOTE]
> If both `VBX_CONFIG_FILE` and `VBX_CONFIG_DIR` are specified, configurations from both sources are loaded. No two configuration files can have the same endpoint and database combination.

> [!IMPORTANT]
> If no matching configuration is found for a request, Voicebox will return an error. You should provide a global default configuration (`endpoint: "*"`, `database: "*"`) unless you intentionally want to disable Voicebox for specific endpoints or databases.

## Internal Stardog Endpoint Support

When Voicebox makes requests to Stardog servers, it uses server-side connections that may require different network routing than browser-based requests. To support architectures where the Voicebox service container cannot access Stardog on the public endpoint, Launchpad v3.5.0+ allows you to configure an additional internal endpoint for connections.

When an internal endpoint is configured for a connection, Voicebox automatically uses the internal endpoint while browser-based applications (Studio, Explorer, Designer, Knowledge Catalog) continue using the public endpoint. This is particularly useful in scenarios where:
- Stardog is behind a firewall accessible only within a private network
- Different DNS resolution is needed for internal vs. external access
- Network policies restrict container-to-container communication to internal networks

See the [SSO Connection Configuration](../README.md#sso-connection-configuration) section and individual provider documentation for details on configuring internal endpoints.

## JWT Authentication & Token Exchange

For enterprise deployments requiring OAuth-based authentication between Launchpad, Voicebox, and your LLM Gateway, see the dedicated guide:

**[JWT Authentication with Okta](./jwt-authentication-okta.md)**

This guide covers:
- Okta authorization server setup
- On-Behalf-Of (OBO) token exchange configuration
- Launchpad and Voicebox Service environment variables
- Public API JWT authentication

