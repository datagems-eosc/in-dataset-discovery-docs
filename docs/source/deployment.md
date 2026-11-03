# Deployment

The In-Dataset Discovery service is designed for containerized deployment using Docker. This guide provides instructions for building the image, configuring the service, and understanding its dependencies.

## Docker

The primary method for deploying the service is via a Docker container. The repository includes a `Dockerfile` for the build process.

### Image

The image is a single stage based on `python:3.12-slim-bookworm`. It installs dependencies from `pyproject.toml` with `uv`, exposes port `8080`, and starts `uvicorn src.app.main:app`. `RETRIEVAL_ENABLED` defaults to `false` in the image. `LLM_PROVIDER` defaults to `scayle`.

### Build and Run

To build and run the container, use the standard Docker commands:

```bash
# 1. Build the Docker image
docker build -t in-dataset-discovery .

# 2. Run the container
# Note: You must provide all required environment variables.
docker run -p 8080:8080 \
  --env-file .env \
  --name in-dataset-discovery-service \
  in-dataset-discovery
```

## Configuration

The service is configured entirely through environment variables. This allows for flexible deployment across different environments without changing the container image.

| Variable | Description | Example |
|----------|-------------|---------|
| OIDC_ISSUER_URL | The base URL of the OIDC identity provider for token validation. | https://datagems-dev.scayle.es/oauth/realms/dev |
| OIDC_AUDIENCE | The audience claim that must be present in the JWT. | in-dataset-discovery |
| GATEWAY_API_URL | The base URL of the DataGEMS Gateway API, used to fetch user permissions. | https://datagems-dev.scayle.es/gw |
| IdpClientSecret | The client secret for the identity provider (taken as secret from Vault). | your-secret |
| LLM_PROVIDER | LLM provider. Default `scayle`. | scayle |
| SCAYLE_BASE_URL | Scayle chat API base URL. | https://chat.scayle.es/api |
| SCAYLE_USERNAME | Scayle username. | scayle-user |
| SCAYLE_PASSWORD | Scayle password. | scayle-password |
| SCAYLE_MODEL_NAME | Default Scayle model. | Qwen3.8-Flash-Next |
| OPENAI_API_KEY | API key used when `LLM_PROVIDER=openai`. | sk-... |
| LANGCHAIN_API_KEY | The API key for accessing LangChain services, e.g., LangSmith. | ls-... |
| OPENAI_MODEL_NAME | OpenAI model name when that provider is selected. | gpt-4o |
| TEXT2SQL_FRAMEWORK | `dail` (default) or `opensearch`. | dail |
| TEXT2SQL_LEGACY | `true` keeps the single-shot SQL pattern path. | false |
| DB_HOST | PostgreSQL host for `GET /explore`. | the_db_host |
| DB_PORT | PostgreSQL port for `GET /explore`. | 5432 |
| DB_USER | PostgreSQL user for `GET /explore` and the analytical-pattern endpoints. | db_user |
| DB_PASSWORD | PostgreSQL password for `GET /explore` and the analytical-pattern endpoints. | db_password |
| DB_NAME_ERA5LAND | Database name for `GET /explore`. | era5land |
| DB_SCHEMA_ERA5LAND | Schema for `GET /explore`. | public |
| RETRIEVAL_ENABLED | Enable Qdrant few-shot retrieval for `era5land`. | false |
| QDRANT_URL | URL for retrieval from in-cluster Qdrant. | http://qdrant-service.shared:6333 |
| OVERPASS_API_URL | The URL for the Overpass API. | https://overpass-api.de/api/ |

## Dependencies

The service requires several external systems and resources to be available at runtime to function correctly.

### Runtime Dependencies

**Scayle chat API:**
- **Description:** Default LLM for text-to-SQL and place-name extraction (`LLM_PROVIDER=scayle`).
- **Requirement:** `SCAYLE_BASE_URL`, `SCAYLE_USERNAME`, `SCAYLE_PASSWORD`, and `SCAYLE_MODEL_NAME`. OpenAI remains available when `LLM_PROVIDER=openai`.

**Overpass API:**
- **Description:** Used for retrieving geospatial data from OpenStreetMap for `GET /text2geo` and the explore pipeline.
- **Requirement:** Network access to the Overpass API (default: `https://overpass-api.de/api/`). Can be configured via `OVERPASS_API_URL`.

**Qdrant (optional):**
- **Description:** Few-shot examples for `era5land` text-to-SQL, retrieved from in-cluster Qdrant when `RETRIEVAL_ENABLED=true`.
- **Requirement:** `QDRANT_URL`. If the expected collections are missing, startup logs `retrieval_degraded` and SQL generation continues without few-shot text.

**OIDC Provider:**
- **Description:** An OpenID Connect compliant identity provider (e.g., Keycloak) is necessary for authenticating users by validating their JWTs (for protected endpoints).
- **Requirement:** The `OIDC_ISSUER_URL` and `OIDC_AUDIENCE` must be correctly configured to point to the identity provider.

**DataGEMS Gateway (Optional):**
- **Description:** The service may communicate with the DataGEMS Gateway API to fetch dataset-level permissions for users.
- **Requirement:** The `GATEWAY_API_URL` must be configured if gateway integration is needed.

**PostgreSQL:**
- **Description:** `POST /text2sql` connects with the request `db_info`. `GET /explore` uses the ERA5-Land env vars. `POST /text2sql4ap` and `POST /explore4ap` use `contentUrl` plus `DB_USER` / `DB_PASSWORD`.
- **Requirement:** Network access to those databases. The service uses `psycopg2`.
