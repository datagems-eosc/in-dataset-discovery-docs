# Configuration

The In-Dataset Discovery service is configured using a combination of a local configuration file and environment variables.

## Configuration files

For local development, the service can be configured using a `.env` file placed in the root of the repository. The application will automatically load variables from this file on startup.

> **Warning**
> The `.env` file is intended for **local development only** and should be added to your `.gitignore` file to prevent committing secrets to version control.

### Example `.env` file

```env
# .env

# OIDC Authentication
OIDC_ISSUER_URL="https://datagems-dev.scayle.es/oauth/realms/dev"
OIDC_AUDIENCE="in-dataset-discovery"
GATEWAY_API_URL="https://datagems-dev.scayle.es/gw"

# LLM (default provider is Scayle)
LLM_PROVIDER="scayle"
SCAYLE_BASE_URL="https://chat.scayle.es/api"
SCAYLE_USERNAME="scayle-user"
SCAYLE_PASSWORD="scayle-password"
SCAYLE_MODEL_NAME="Qwen3.8-Flash-Next"

# Optional OpenAI provider (LLM_PROVIDER=openai)
OPENAI_API_KEY="sk-your-openai-api-key"
OPENAI_MODEL_NAME="gpt-4o"

# Text-to-SQL
TEXT2SQL_FRAMEWORK="dail"
TEXT2SQL_LEGACY="false"

# ERA5-Land database used by GET /explore
DB_HOST="the_db_host"
DB_PORT="5432"
DB_USER="db_user"
DB_PASSWORD="db_password"
DB_NAME_ERA5LAND="era5land"
DB_SCHEMA_ERA5LAND="public"

# Optional Qdrant few-shot retrieval (era5land only)
RETRIEVAL_ENABLED="false"
QDRANT_URL="http://127.0.0.1:6335"

# LangChain Configuration (optional, for LangSmith)
LANGCHAIN_API_KEY="ls-your-langchain-api-key"

# Overpass API Configuration
OVERPASS_API_URL="https://overpass-api.de/api/"

# Secrets (for local use only)
IdpClientSecret="your-local-dev-secret"
```

## Environment Overrides

In any environment, especially in production, environment variables will always override values set in the `.env` file. This is the standard and recommended way to configure the service when deploying it as a container.

The following table lists all available configuration variables:

| Variable | Description | Example |
|----------|-------------|---------|
| OIDC_ISSUER_URL | The base URL of the OIDC identity provider for token validation. | https://datagems-dev.scayle.es/oauth/realms/dev |
| OIDC_AUDIENCE | The audience claim that must be present in the JWT. | in-dataset-discovery |
| GATEWAY_API_URL | The base URL of the DataGEMS Gateway API, used to fetch user permissions. | https://datagems-dev.scayle.es/gw |
| IdpClientSecret | The client secret for the identity provider. | your-secret |
| LLM_PROVIDER | LLM provider. Default `scayle`. Also `openai` or `bedrock`. | scayle |
| LLM_MODEL | Optional model override for every provider. | Qwen3.8-Flash-Next |
| SCAYLE_BASE_URL | Scayle chat API base URL. | https://chat.scayle.es/api |
| SCAYLE_USERNAME | Scayle username. | scayle-user |
| SCAYLE_PASSWORD | Scayle password. | scayle-password |
| SCAYLE_MODEL_NAME | Scayle model when `LLM_MODEL` is unset. | Qwen3.8-Flash-Next |
| OPENAI_API_KEY | API key used when `LLM_PROVIDER=openai`. | sk-... |
| OPENAI_MODEL_NAME | OpenAI model when `LLM_PROVIDER=openai` and `LLM_MODEL` is unset. | gpt-4o |
| LANGCHAIN_API_KEY | The API key for accessing LangChain services, e.g., LangSmith (optional). | ls-... |
| TEXT2SQL_FRAMEWORK | SQL generator. Default `dail`. `opensearch` selects the OpenSearch-SQL graph. | dail |
| TEXT2SQL_LEGACY | `true` keeps the single-shot SQL pattern path. `TEXT2SQL_PIPELINE=false` does the same. | false |
| DB_HOST | PostgreSQL host for `GET /explore`. | the_db_host |
| DB_PORT | PostgreSQL port for `GET /explore`. | 5432 |
| DB_USER | PostgreSQL user for `GET /explore` and the analytical-pattern endpoints. | db_user |
| DB_PASSWORD | PostgreSQL password for `GET /explore` and the analytical-pattern endpoints. | db_password |
| DB_NAME_ERA5LAND | Database name for `GET /explore`. | era5land |
| DB_SCHEMA_ERA5LAND | Schema for `GET /explore`. | public |
| RETRIEVAL_ENABLED | Attach Qdrant few-shot text for `era5land` when retrieval returns text. Default off. | false |
| QDRANT_URL | URL for retrieval from in-cluster Qdrant. Local default `http://127.0.0.1:6335`. | http://127.0.0.1:6335 |
| OVERPASS_API_URL | The URL for the Overpass API. | https://overpass-api.de/api/ |

## Secrets

Certain configuration variables contain sensitive information and must be handled securely.

### Identified Secrets

* **SCAYLE_PASSWORD**: Password for the default Scayle LLM.
* **OPENAI_API_KEY**: API key used when `LLM_PROVIDER=openai`.
* **DB_PASSWORD**: Password for `GET /explore` and the analytical-pattern endpoints.
* **LANGCHAIN_API_KEY**: Contains the API key for LangChain services (if used).
* **IdpClientSecret**: The client secret used for communication with the identity provider.

`POST /text2sql` takes database credentials in the request and does not store them. `GET /explore` and `POST /text2sql4ap` / `POST /explore4ap` use the service `DB_USER` and `DB_PASSWORD`.
