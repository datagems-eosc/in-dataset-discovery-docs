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
OIDC_AUDIENCE="in-data-exploration"
GATEWAY_API_URL="https://datagems-dev.scayle.es/gw"

# OpenAI Configuration
OPENAI_API_KEY="sk-your-openai-api-key"
OPENAI_MODEL_NAME="gpt-4o"

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
| OIDC_AUDIENCE | The audience claim that must be present in the JWT. | in-data-exploration |
| GATEWAY_API_URL | The base URL of the DataGEMS Gateway API, used to fetch user permissions. | https://datagems-dev.scayle.es/gw |
| IdpClientSecret | The client secret for the identity provider. | your-secret |
| OPENAI_API_KEY | The API key for accessing OpenAI services (required for LLM features). | sk-... |
| LANGCHAIN_API_KEY | The API key for accessing LangChain services, e.g., LangSmith (optional). | ls-... |
| OPENAI_MODEL_NAME | The name of the OpenAI model to use. | gpt-4o |
| OVERPASS_API_URL | The URL for the Overpass API. | https://overpass-api.de/api/ |

## Secrets

Certain configuration variables contain sensitive information and must be handled securely.

### Identified Secrets

* **OPENAI_API_KEY**: Contains the API key for OpenAI services.
* **LANGCHAIN_API_KEY**: Contains the API key for LangChain services (if used).
* **IdpClientSecret**: The client secret used for communication with the identity provider.

Note: Database credentials for the `/text2sql` endpoint are provided by users in their requests and are not stored by the service.
