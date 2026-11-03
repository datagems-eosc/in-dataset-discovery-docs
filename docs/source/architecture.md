# System Architecture

The In-Dataset Discovery Service is a self-contained application designed to act as a specialized query layer within the DataGEMS ecosystem.

## Core Components

1.  **FastAPI Application:** The heart of the service is a Python web application built with FastAPI. It exposes the REST API, handles incoming HTTP requests, and orchestrates all internal operations.

2.  **Geospatial Component:** A module that processes natural language geospatial queries, identifies places using Wikidata, generates OverpassQL queries, and retrieves geospatial data from OpenStreetMap via the Overpass API. `GET /text2geo` and the first step of `GET /explore` both use this module.

3.  **Text-to-SQL Component:** Converts a natural-language question into SQL and executes it. The default generator is DAIL-SQL (`TEXT2SQL_FRAMEWORK=dail`), with one candidate at temperature 0. `TEXT2SQL_FRAMEWORK=opensearch` selects the OpenSearch-SQL graph. `TEXT2SQL_LEGACY=true` (or `TEXT2SQL_PIPELINE=false`) keeps the single-shot pattern generator. On the DAIL-SQL path, `sql_pattern`, `input_params`, and `output_params` are null, and clients use `sql_query`. ERA5-Land receives `METEO_DDL`, `UNIT_CONTRACTS`, hints, the constraint validator, and a query rewrite. A `ROUND(CAST(latitude/longitude)) IN (...)` list of 20 or more pairs is rewritten as a `VALUES` join with the same coordinates. PostgreSQL executes that form much faster, and its query optimizer does not apply the rewrite automatically. Shorter lists are left unchanged. Other databases use the live schema only. Few-shot text comes from in-cluster Qdrant only when `RETRIEVAL_ENABLED` is on, the database is `era5land`, and retrieval returns text.

4.  **Meteo text-to-SQL agent:** `GET /explore` runs this LangGraph workflow (`meteo_agent`). It calls text2geo, then text-to-SQL, against the ERA5-Land database set by `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME_ERA5LAND`, and `DB_SCHEMA_ERA5LAND`. `POST /explore4ap` runs the same chain and writes the result onto an analytical pattern graph, including `metadata.params.text2geo` and `metadata.params.text2sql`.

5.  **Analytical-pattern-specific endpoints:** `POST /text2sql4ap` and `POST /explore4ap` are the analytical-pattern-specific forms of `/text2sql` and `/explore`. They accept an analytical pattern graph (node label `Analytical_Pattern`), read the question from the natural language operator (`NLQ_Operator`), and connect with `DB_USER` and `DB_PASSWORD` plus the database connection node's `contentUrl`. They return the enriched graph and text-to-SQL metadata. `/text2sql4ap` omits `sql_results` from metadata.

6.  **Security Layer:** This layer intercepts protected requests to perform authentication and authorization. It integrates with an external OIDC provider to validate JWTs and with the DataGEMS Gateway to fetch user-specific permissions.

## Request Flow

### Geospatial Query Flow

1.  A user sends a `GET /text2geo` request with a natural language question about a location.
2.  The Security Layer validates the token and checks for the required user roles.
3.  The service uses Wikidata to identify the most relevant place entity.
4.  An OverpassQL query is generated based on the Wikidata information.
5.  The Overpass API is queried to retrieve geospatial data.
6.  Results are returned, including `{lon, lat}` points, a `[lat, lon]` center, GeoJSON, and a bounding box. A missing place is `404`. Wikidata or Overpass connectivity failure is `503`.

### Text-to-SQL Query Flow

1.  A user sends a `POST /text2sql` request with a natural language question and database connection information. Coordinate pairs, when present, are disambiguated so the larger magnitude within ±90 is latitude.
2.  The Security Layer validates the token and checks for the required user roles.
3.  The text-to-SQL component generates SQL with DAIL-SQL. `TEXT2SQL_FRAMEWORK=opensearch` selects the OpenSearch-SQL graph, and `TEXT2SQL_LEGACY=true` keeps the single-shot pattern path.
4.  For `era5land`, the prompt includes `METEO_DDL` and `UNIT_CONTRACTS`. Few-shot text from in-cluster Qdrant is added only when retrieval is enabled and returns text.
5.  The SQL query is executed against the database in the request.
6.  Results are returned with `sql_query` and `sql_results`. If generation or execution fails, the HTTP status can still be `200` and the body `"status"` is `"error"`.

### Explore Flow

1.  A user sends `GET /explore?question=...`.
2.  text2geo returns coordinates for the place named in the question.
3.  Text-to-SQL runs against `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME_ERA5LAND`, and `DB_SCHEMA_ERA5LAND`.
4.  The response includes `sql_query`, `sql_results`, `inference_time_s`, and `db_fetch_time_s`.

### Analytical-pattern-specific flow

1.  A user sends `POST /text2sql4ap` or `POST /explore4ap` with `{ "ap": { "nodes", "edges" } }`.
2.  The service reads the question from the natural language operator and the relational-database node from the graph.
3.  Database host, port, and name come from the database connection node's `contentUrl`. The password is the service `DB_PASSWORD`.
4.  `/text2sql4ap` generates SQL directly. `/explore4ap` runs the meteo text-to-SQL agent first.
5.  The enriched graph is returned with metadata. An invalid graph returns `422`. DMM or database connection failures return `424`.
