# Datastores

The In-Dataset Discovery service interacts with various data sources and external services to provide its functionality.

## External Data Sources

### Overpass API

The service uses the Overpass API to retrieve geospatial data from OpenStreetMap.

- **Technology:** Overpass API (OpenStreetMap)
- **Purpose:** Supplies OpenStreetMap data for `GET /text2geo`, and for the text2geo step of `GET /explore` and `POST /explore4ap`.
- **Configuration:** The Overpass API URL can be configured via the `OVERPASS_API_URL` environment variable (default: `https://overpass-api.de/api/`).

### PostgreSQL

Generated SQL is executed on PostgreSQL through `psycopg2`.

- **`POST /text2sql`:** Host, port, database, schema, username, and password come from `parameters.db_info` on each request.
- **`GET /explore`:** Uses the service ERA5-Land settings: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME_ERA5LAND`, and `DB_SCHEMA_ERA5LAND`.
- **`POST /text2sql4ap` and `POST /explore4ap`:** These take an analytical pattern graph. Host, port, and database name come from the connection node's `contentUrl`. Username and password are `DB_USER` and `DB_PASSWORD`. The schema is `public`.

For the `era5land` database, the table allowlist is `landcover_upscaled`, `landcover_type`, and `meteo_*`. `meteo_ssrd.ssrd` is stored as J/m².

### Wikidata

The service uses Wikidata to identify and enrich place information for geospatial queries.

- **Technology:** Wikidata REST API
- **Purpose:** Identifies places mentioned in natural language queries and retrieves their properties (coordinates, country codes, etc.) for generating OverpassQL queries.

### Qdrant

Few-shot examples for text-to-SQL are retrieved from in-cluster Qdrant.

- **Purpose:** When `RETRIEVAL_ENABLED` is on and the target database is `era5land`, a few-shot block from Qdrant is added to the DAIL-SQL prompt if retrieval returns text. Other databases use the live schema only.
- **Configuration:** Set `QDRANT_URL`. The local default is `http://127.0.0.1:6335`. The in-cluster address is supplied in that variable (for example `http://qdrant-service.shared:6333`).
- **Startup:** If the expected collections are missing, `verify_retrieval` logs `retrieval_degraded` and text-to-SQL continues without few-shot text.
