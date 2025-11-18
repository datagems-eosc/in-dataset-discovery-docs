# Deployment

The In-Dataset Discovery service is designed for containerized deployment using Docker. This guide provides instructions for building the image, configuring the service, and understanding its dependencies.

## Docker

The primary method for deploying the service is via a Docker container. The repository includes a `Dockerfile` for the build process.

### Dockerfile Stages

1.  **Builder Stage:**
    - Starts from a `python:3.11-slim` base image.
    - Installs all Python dependencies from `pyproject.toml` using `uv`.

2.  **Final Stage:**
    - Starts from a fresh `python:3.11-slim` image.
    - Creates a non-root user (`appuser`) for security.
    - Copies the installed Python packages from the builder stage.
    - Copies the application source code into the container.
    - Sets `appuser` as the active user.
    - Exposes port `8080`.
    - Includes a `HEALTHCHECK` instruction that queries the `/health` endpoint to monitor container health.
    - The default command (`CMD`) starts the application using `uvicorn`, making it production-ready.

### Build and Run

To build and run the container, use the standard Docker commands:

```bash
# 1. Build the Docker image
docker build -t in-data-exploration .

# 2. Run the container
# Note: You must provide all required environment variables.
docker run -p 8080:8080 \
  --env-file .env \
  --name in-data-exploration-service \
  in-data-exploration
```

## Configuration

The service is configured entirely through environment variables. This allows for flexible deployment across different environments without changing the container image.

| Variable | Description | Example |
|----------|-------------|---------|
| OIDC_ISSUER_URL | The base URL of the OIDC identity provider for token validation. | https://datagems-dev.scayle.es/oauth/realms/dev |
| OIDC_AUDIENCE | The audience claim that must be present in the JWT. | in-data-exploration |
| GATEWAY_API_URL | The base URL of the DataGEMS Gateway API, used to fetch user permissions. | https://datagems-dev.scayle.es/gw |
| IdpClientSecret | The client secret for the identity provider (taken as secret from Vault). | your-secret |
| OPENAI_API_KEY | The API key for accessing OpenAI services (required for LLM features). | sk-... |
| LANGCHAIN_API_KEY | The API key for accessing LangChain services, e.g., LangSmith. | ls-... |
| OPENAI_MODEL_NAME | The name of the OpenAI model to use. | gpt-4o |
| OVERPASS_API_URL | The URL for the Overpass API. | https://overpass-api.de/api/ |

## Dependencies

The service requires several external systems and resources to be available at runtime to function correctly.

### Runtime Dependencies

**OpenAI API:**
- **Description:** Required for LLM-powered features including text-to-SQL conversion.
- **Requirement:** A valid `OPENAI_API_KEY` must be configured.

**Overpass API:**
- **Description:** Used for retrieving geospatial data from OpenStreetMap for the `/geospatial` endpoint.
- **Requirement:** Network access to the Overpass API (default: `https://overpass-api.de/api/`). Can be configured via `OVERPASS_API_URL`.

**OIDC Provider:**
- **Description:** An OpenID Connect compliant identity provider (e.g., Keycloak) is necessary for authenticating users by validating their JWTs (for protected endpoints).
- **Requirement:** The `OIDC_ISSUER_URL` and `OIDC_AUDIENCE` must be correctly configured to point to the identity provider.

**DataGEMS Gateway (Optional):**
- **Description:** The service may communicate with the DataGEMS Gateway API to fetch dataset-level permissions for users.
- **Requirement:** The `GATEWAY_API_URL` must be configured if gateway integration is needed.

**User Databases:**
- **Description:** For the `/text2sql` endpoint, users provide database connection information in their requests. The service connects to these databases to execute queries.
- **Requirement:** Network access to user-specified databases. The service uses `psycopg2` for PostgreSQL connections.
