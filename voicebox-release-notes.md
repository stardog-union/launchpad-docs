# Voicebox Service Release Notes

The Voicebox Service is released independently of Launchpad. 

> [!NOTE] 
> All available releases of the Voicebox Service are listed below. The image tag for a release is simply the release name prepended with `v` as in `v0.1.1`.

## 1.0.0 Release (August 21, 2026)

> [!IMPORTANT]
> First generally available release of the next-generation Voicebox Service, distributed under the `v1.0.0` tag. It supersedes the `0.x` line and serves all Voicebox traffic in [Launchpad 4.0.0](./release-notes.md#400-release-2026-08-21) and later.
>
> Upgrading to Launchpad 4.0.0 requires a configuration change in every deployment, whether or not you took part in the Voicebox Service beta. See the [4.0.0 release notes](./release-notes.md#400-release-2026-08-21) for the steps.

<!-- TODO: changes since 1.0.0-beta.3. -->

## 1.0.0-beta.3 Release (July 29, 2026)

> [!NOTE]
> This is a beta build of the next-generation Voicebox Service, distributed under the `v1.0.0-beta.3` tag. It powers the Launchpad public API beta and runs alongside the stable `0.x` service. See [Deploying the Voicebox Service for the Beta](./guides/voicebox-deployment.md) for deployment details.

* Answer questions faster by loading matching example queries up front, and answer immediately when a question exactly matches a saved query
* Return a summary of what was attempted when a question is too complex to finish, instead of a generic "cannot find an answer" response
* Fix intermittent authentication errors when answering questions against a knowledge graph whose cached schema outlived the token that loaded it
* Add [`VOICEBOX_QUERY_EXEC_TIMEOUT_SECONDS`](./guides/voicebox-deployment.md#tuning-the-agent) (default: `60`), [`VOICEBOX_QUERY_MAX_RESULTS`](./guides/voicebox-deployment.md#tuning-the-agent) (default: `100000`), and [`VOICEBOX_RECURSION_LIMIT`](./guides/voicebox-deployment.md#tuning-the-agent) (default: unset) to tune query execution limits and the agent step cap
* Update dependencies to address reported CVEs

## 0.30.1 Release (Jul 21, 2026)

* Do not store expired tokens in the schema cache


## 1.0.0-beta.2 Release (July 8, 2026)

> [!NOTE]
> This is a beta build of the next-generation Voicebox Service, distributed under the `v1.0.0-beta.2` tag. It powers the Launchpad public API beta and runs alongside the stable `0.x` service. See [Deploying the Voicebox Service for the Beta](./guides/voicebox-deployment.md) for deployment details.

* Fix an error when answering questions against knowledge graphs where a relationship is mapped from multiple source columns
* Pick up republished or edited knowledge graph schemas immediately instead of serving a cached schema for up to an hour
* Fix an error when a question returned results with too many columns to summarize; these questions now complete successfully
* Add [`VOICEBOX_CODE_EXEC_TIMEOUT_SECONDS`](./guides/voicebox-deployment.md#tuning-the-agent) (default: `30`) to configure how long the service may spend analyzing query results while answering a question, and add per-run telemetry to the [`code_executed`](./guides/voicebox-deployment.md#log-events) log event
* Emit error tracebacks as structured log events instead of plain-text stack traces

## 1.0.0-beta.1 Release (June 30, 2026)

> [!NOTE]
> This is a beta build of the next-generation Voicebox Service, distributed under the `v1.0.0-beta.1` tag. It powers the Launchpad public API beta and runs alongside the stable `0.x` service. See [Deploying the Voicebox Service for the Beta](./guides/voicebox-deployment.md) for deployment details.

* First beta of the next-generation Voicebox Service.

## 0.29.0 Release (May 14, 2026)

* Add support for [Anthropic](./guides/voicebox-configuration.md#anthropic-configuration) as an LLM provider
* Support Anthropic models hosted on [AWS Bedrock](./guides/voicebox-configuration.md#aws-bedrock-configuration) and [Azure AI Foundry](./guides/voicebox-configuration.md#anthropic-configuration)
* Always return query lineage even when metadata collection is disabled

## 0.28.0 Release (Apr 16, 2026)

* Support [custom HTTP headers](./guides/voicebox-configuration.md#custom-headers) for Azure AI Foundry LLM provider
* Add [`$VOICEBOX_USER`](./guides/voicebox-configuration.md#voicebox_user-variable) special variable for custom headers that resolves to the authenticated username

## 0.27.0 Release (Apr 8, 2026)

* Fix streaming memory accumulation for improved memory efficiency

## 0.26.0 Release (Mar 19, 2026)

* Add [HTTP basic authentication support for OpenAI](./guides/voicebox-configuration.md#basic-authentication)
* Support [Azure Service Principal (SPN) authentication](./guides/voicebox-configuration.md#service-principal-spn-authentication) for Azure AI
* Include timing and LLM usage information in #debug output
* Update dependencies to address security advisories

## 0.25.0 Release (Feb 5, 2026)

* Add endpoint/database-specific Voicebox configuration support
* Add #debug command to show query generation diagnostics
* Support executing queries with GRAPH keyword
* Sanitize var names that start or end with underscore
* Slimmer base image with reduced vulnerabilities

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
