# Datastores

The In-Dataset Discovery service interacts with various data sources and external services to provide its functionality.

## External Data Sources

### Overpass API

The service uses the Overpass API to retrieve geospatial data from OpenStreetMap.

- **Technology:** Overpass API (OpenStreetMap)
- **Purpose:** Provides geospatial data retrieval for the `/geospatial` endpoint.
- **Configuration:** The Overpass API URL can be configured via the `OVERPASS_API_URL` environment variable (default: `https://overpass-api.de/api/`).

### User-Provided Databases

For the `/text2sql` endpoint, the service connects to databases specified by users in their requests.

- **Technology:** PostgreSQL (and potentially other SQL databases)
- **Purpose:** Executes generated SQL queries to retrieve data from user-specified databases.
- **Connection:** Database connection information (host, port, username, password, database name, schema) is provided in each request. The service uses `psycopg2` for PostgreSQL connections.

### Wikidata

The service uses Wikidata to identify and enrich place information for geospatial queries.

- **Technology:** Wikidata REST API
- **Purpose:** Identifies places mentioned in natural language queries and retrieves their properties (coordinates, country codes, etc.) for generating OverpassQL queries.
