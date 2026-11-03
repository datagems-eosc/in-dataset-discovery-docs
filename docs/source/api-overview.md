# API Overview

The In-Dataset Discovery service exposes a RESTful API for performing in-dataset exploration operations, including geospatial queries, text-to-SQL conversion, and RAG-based question answering. `POST /text2sql4ap` and `POST /explore4ap` apply the same SQL result to an analytical pattern graph.

## Base URL

The API is served from the root of the application. All endpoint paths are relative to the base URL where the service is deployed:

**Base URL:** [https://datagems-dev.scayle.es/in-dataset-discovery/](https://datagems-dev.scayle.es/in-dataset-discovery/)

## Authentication

All operational endpoints require authentication except `/health`. See the [Security](./security.md) page for more details.

---

## Endpoints

### Text to Geospatial Objects Query
<p id="geospatial-query"></p>
Performs geospatial queries using natural language to find locations, generate OverpassQL queries and return geospatial objects.

*   **Endpoint:** `GET /text2geo`
*   **Description:** Processes a natural language geospatial question, identifies the place using Wikidata, generates an OverpassQL query, and returns geospatial objects, resulting in coordinates, GeoJSON data, and bounding boxes.
*   **Query Parameters:**
    - `question` (required): The natural language geospatial query question.

*   **Example Request:**
    ```
    GET /text2geo?question=What are the coordinates in Zurich?
    ```

*   **Response Body (Success):**
    ```json
    {
      "most_relevant_wikidata": {
        "place": "Zurich",
        "reasoning": "The question asks for the city of Zurich, which is best represented by the entity with ID Q72. This entity is described as the capital of the canton of Zurich and is a municipality and city in Switzerland. It also has 'found_osm_json: True', which gives it priority if multiple similar entities were found. Other entities like universities or sports clubs are not relevant to the context of the question, which specifically asks for the city.",
        "wiki_id": "Q72",
        "wiki_properties": {
          "aliases": "City of Zurich, ZH, Stadt Zürich, Zurich, Switzerland, Zürich",
          "coordinate location": "47.37444444444444, 8.54111111111111",
          "country": "Switzerland, Old Swiss Confederacy, Helvetic Republic, Switzerland",
          "description": "capital of the canton of Zurich, Switzerland",
          "found_osm_json": true,
          "instance of": "municipality of Switzerland, city of Switzerland, cantonal capital of Switzerland, college town, largest city, big city",
          "label": "Zurich",
          "located in the administrative territorial entity": "Zürich District",
          "part of": "Greater Zurich Area, Zurich metropolitan area, Canton of Zürich"
        }
      },
      "oql": {
        "OQL": "area[\"ISO3166-1\"=\"CH\"]->.searchArea;(nwr[\"wikidata\"=\"Q72\"];);",
        "reasoning": "Generated Overpass QL query using Wikidata ID Q72 and country ISO alpha-2 code CH."
      },
      "place": "zurich",
      "results": {
        "bounds": {
          "maxlat": 47.4346662,
          "maxlon": 8.6254413,
          "minlat": 47.3202187,
          "minlon": 8.4480061
        },
        "center": [
          47.4,
          8.5
        ],
        "geojson_data": {
          "features": [],
          "type": "FeatureCollection"
        },
        "points": [
          {
            "lat": "47.4",
            "lon": "8.5"
          }
        ]
      }
    }
    ```

#### Implementation

`results.points` entries are `{ "lon", "lat" }` strings, and `results.center` is `[lat, lon]`. A missing place returns `404`. If Wikidata or Overpass cannot be reached, the response is `503`. Other failures return `500`. When LLM authentication fails, the client message can be "No place name found"; the log event is `geospatial_place_name_extraction_failed`. `GET /explore` and `POST /explore4ap` call this same step before text-to-SQL.

### Text-to-SQL Query
<p id="text-to-sql-query"></p>
Converts natural language questions into SQL queries and executes them against a specified database.

*   **Endpoint:** `POST /text2sql`
*   **Description:** Takes a natural language question and database connection information, generates a SQL query using LLM capabilities, and executes it to return results.
    The parameters are:
    - `db_info`: The database connection information.
    - `results`: The results containing points for SQL query.
    The `results` parameter is optional. If not provided, the SQL query will be generated without parameters.
    - The `db_info` parameter is required. The `db_info` parameter is a dictionary with the following keys:
      - `db_host`: The database host.
      - `db_username`: The database username.
      - `db_pass`: The database password.
      - `db_database`: The database name.
      - `db_schema`: The database schema.
      - `db_port`: The database port.

*   **Request Body with parameters:**

    ```json
    {
      "question": "What are the average mean temperatures in the coordinates (lon, lat) in year 2020?",
      "parameters": {
        "results": {
          "points": [[8.4, 47.4], [8.5, 47.4]]
        },
        "db_info": {
          "db_host": "the_db_host_ip",
          "db_username": "db_username",
          "db_pass": "db_password",
          "db_database": "database_name",
          "db_schema": "database_schema",
          "db_port": 5432
        }
      }
    }
    ```

*   **Response Body with parameters (Success):**

    ```json
    {
      "status": "success",
      "message": "Successfully generated SQL query on the first attempt.",
      "question": "What are the average mean temperatures...",
      "model_name": "gpt-4o",
      "sql_pattern": "SELECT AVG(tmean - 273.15)...",
      "sql_query": "SELECT AVG(tmean - 273.15) AS average_mean_temperature_celsius...",
      "sql_results": {
        "status": "success",
        "data": [{"average_mean_temperature_celsius": 10.5}]
      }
    }
    ```

*   **Request Body without parameters:**

    ```json
    {
      "question": "What are the average mean temperatures at lat 47.4, lon 8.4 in year 2020?",
      "parameters": {
        "db_info": {
          "db_host": "the_db_host_ip",
          "db_username": "db_username",
          "db_pass": "db_password",
          "db_database": "database_name",
          "db_schema": "database_schema",
          "db_port": 5432
        }
      }
    }
    ```

*   **Response Body without parameters (Success):**

    ```json
    {
    "status": "success",
    "message": "Successfully generated SQL query on the first attempt.",
    "params": null,
    "question": "Show the minimum temperature at lat 47.4, lon 8.5 in 2005",
    "model_name": "gpt-4o",
    "sql_pattern": null,
    "input_params": null,
    "output_params": null,
    "reasoning": "The query is designed to find the minimum temperature recorded in the year 2005 at the specified latitude and longitude coordinates (47.4, 8.5). The temperatures in the 'meteo_tmin' table are stored in Kelvin, so they are converted to Celsius by subtracting 273.15. We use the WHERE clause to filter records that match the given coordinates and year, and then SELECT the MIN value of these converted temperatures.",
    "sql_query": "SELECT MIN(tmin - 273.15) AS min_temp_celsius FROM era5_land2.meteo_tmin WHERE round(latitude::numeric, 1) = 47.4 AND round(longitude::numeric,1) = 8.5 AND EXTRACT(YEAR FROM time) = 2005;",
    "sql_results": {
      "status": "success",
      "data": [
        {
          "min_temp_celsius": -12.898565673828102
        }
        ]
      }
    }
    ```

#### Implementation

The response keys above are unchanged. The default generator is DAIL-SQL (`TEXT2SQL_FRAMEWORK=dail`), one candidate at temperature 0. On that path the body matches the "without parameters" example: `sql_pattern`, `input_params`, and `output_params` are null even when `results.points` was sent. Clients use `sql_query`. `model_name` is the configured model (`LLM_PROVIDER`, default `scayle`, model `SCAYLE_MODEL_NAME`).

`TEXT2SQL_FRAMEWORK=opensearch` selects the OpenSearch-SQL graph. `TEXT2SQL_LEGACY=true` (or `TEXT2SQL_PIPELINE=false`) keeps the single-shot pattern path, which can fill `sql_pattern`, `input_params`, and `output_params` as in the "with parameters" example.

Coordinate pairs in `results.points` accept either order. When both values are within ±90, the larger magnitude is latitude, so `[8.5, 47.5]` and `[47.5, 8.5]` both resolve.

When `db_database` is `era5land`, generation uses `METEO_DDL`, `UNIT_CONTRACTS` (`meteo_ssrd.ssrd` is stored as J/m²), hints, the constraint validator, and a query rewrite. The table allowlist is `landcover_upscaled`, `landcover_type`, and `meteo_*`. Other databases use the live schema only. Few-shot text is taken from in-cluster Qdrant only when `RETRIEVAL_ENABLED` is on, the database is `era5land`, and retrieval returns text.

The query rewrite runs after SQL generation, and only for ERA5-Land. A coordinate predicate of the form `ROUND(CAST(latitude ...), 1), ROUND(CAST(longitude ...), 1) IN ((...), ...)` with 20 or more pairs is rewritten to `IN (SELECT rlat, rlon FROM (VALUES ...) AS _geo_pts(rlat, rlon))`. The coordinates are unchanged. PostgreSQL executes this `VALUES` join much faster, and its query optimizer does not apply the rewrite on its own. Lists with fewer than 20 pairs are left unchanged.

If generation or execution fails, the response can still be HTTP `200` with `"status": "error"` in the body. See the soft-failure note under Explore.

### Explore
<p id="explore-endpoint"></p>
Runs text2geo and then text-to-SQL for one natural-language question.

*   **Endpoint:** `GET /explore`
*   **Authentication:** Required.
*   **Query Parameters:**
    - `question` (required): The exploration question.

*   **Example Request:**
    ```
    GET /explore?question=What is the average mean temperature in Zurich in 2020?
    ```

*   **Response Body:** Same keys as `POST /text2sql` (`status`, `message`, `question`, `model_name`, `sql_pattern`, `input_params`, `output_params`, `reasoning`, `sql_query`, `sql_results`), plus `inference_time_s` and `db_fetch_time_s`.

#### Implementation

The meteo text-to-SQL agent (`meteo_agent`) calls text2geo and then text-to-SQL. It uses the service ERA5-Land connection (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME_ERA5LAND`, `DB_SCHEMA_ERA5LAND`). The caller does not send `db_info`. On the default DAIL-SQL path, `sql_pattern`, `input_params`, and `output_params` are null. `inference_time_s` is total pipeline time minus database time.

A soft failure still returns HTTP `200`. The body has `"status": "error"` when SQL generation fails, or `sql_results.status` of `"error"` when execution fails. The request was accepted, but the SQL step did not succeed. `/explore` logs `meteo_agent_soft_failed`. `/text2sql` logs `text2sql_soft_failed`. An unexpected exception is HTTP `500`.

### Text-to-SQL for an analytical pattern
<p id="text2sql4ap"></p>
Generates SQL for the natural language operator (`NLQ_Operator`) already present on an analytical pattern graph.

*   **Endpoint:** `POST /text2sql4ap`
*   **Authentication:** Required.
*   **Description:** Accepts an analytical pattern graph (`ap`, with an `Analytical_Pattern` node) and returns the same graph enriched with the generated SQL, plus a `metadata` object that uses the `POST /text2sql` fields. `metadata.sql_results` is omitted. `metadata.sql_query_with_args` is set when the SQL names dataset tables.

*   **Request Body:**
    ```json
    {
      "ap": {
        "nodes": [],
        "edges": []
      }
    }
    ```

The graph must include a natural language operator (`NLQ_Operator`), a relational-database node, its upstream dataset, and a database connection node.

*   **Response Body (Success):**
    ```json
    {
      "ap": {"nodes": [], "edges": []},
      "metadata": {
        "status": "success",
        "question": "Average mean temperature in 2020",
        "model_name": "Qwen3.8-Flash-Next",
        "sql_pattern": null,
        "input_params": null,
        "output_params": null,
        "sql_query": "SELECT ...",
        "sql_query_with_args": "SELECT ..."
      }
    }
    ```

#### Implementation

The question is read from the natural language operator. Host, port, and database name come from the database connection node's `contentUrl`. Username and password are the service `DB_USER` and `DB_PASSWORD` (schema `public`). Generation is the same text-to-SQL path as `POST /text2sql`, with no caller-supplied points. On the default DAIL-SQL path, `sql_pattern`, `input_params`, and `output_params` are null. Prepared SQL is written onto the SQL operator, and dataset-table nodes are added for referenced tables. An invalid graph returns `422`. DMM or database connection failures return `424`.

### Explore for an analytical pattern
<p id="explore4ap"></p>
Runs the meteo text-to-SQL agent and writes the result back onto an analytical pattern graph.

*   **Endpoint:** `POST /explore4ap`
*   **Authentication:** Required.
*   **Description:** Same request body as `POST /text2sql4ap`. The response graph is enriched the same way. `metadata` keeps the `GET /explore` fields, including `sql_results`, `inference_time_s`, and `db_fetch_time_s`, and adds `params.text2geo` and `params.text2sql`.

*   **Request Body:** Same `{ "ap": { "nodes", "edges" } }` body as `POST /text2sql4ap`.

*   **Response Body (Success):**
    ```json
    {
      "ap": {"nodes": [], "edges": []},
      "metadata": {
        "status": "success",
        "question": "Average mean temperature in Zurich in 2020",
        "sql_query": "SELECT ...",
        "sql_results": {"status": "success", "data": []},
        "inference_time_s": 4.2,
        "db_fetch_time_s": 0.8,
        "params": {
          "text2geo": {},
          "text2sql": {}
        }
      }
    }
    ```

#### Implementation

The meteo text-to-SQL agent runs text2geo and then text-to-SQL, and writes the SQL onto the graph. Credentials come from the same source as `POST /text2sql4ap`. `metadata.sql_results` is kept. On the default DAIL-SQL path, `sql_pattern`, `input_params`, and `output_params` are null. An invalid graph returns `422`. DMM or database connection failures return `424`.

### Text Question Answer Query
<p id="text-question-answer-query"></p>
Summarizes retrieved RAG context into an answer for a question.

*   **Endpoint:** `POST /textQuestionAnswer`
*   **Description:** Takes a natural language question together with retrieved RAG output and returns a short answer, a structured reason code, and the chunks used to produce the summary.
*   **Authentication:** Required.

*   **Request Body:** The request body must include `question` and `rag_output`. The `rag_output` object contains `results` and an optional `query_time` value.

*   **Example Request:**
    ```json
    {
      "question": "What is the chief town of the island of COPRAJA?",
      "rag_output": {
        "query_time": 3263.56,
        "results": [
          {
            "content": "COPRAJA, an island in the Mediterranean Sea... The chief town is Tratozzio.",
            "dataset_id": "d84d1a2e-127d-4393-91d0-afb7e4fd9c68",
            "object_id": "kp-eb0707-031203-1328-v2.xml",
            "similarity": 1.0
          }
        ]
      }
    }
    ```

*   **Response Body (Success):**
    ```json
    {
      "answerable": true,
      "reason_code": "ANSWERED",
      "summary": "Based on the provided context, the chief town of COPRAJA is Tratozzio.",
      "used_chunks": [
        {
          "dataset_id": "d84d1a2e-127d-4393-91d0-afb7e4fd9c68",
          "object_id": "kp-eb0707-031203-1328-v2.xml",
          "similarity": 1.0
        }
      ]
    }
    ```

### Linguistic Features
<p id="linguistic-features"></p>

[todo] Author: complete this section.

### Decompose
<p id="decompose"></p>

[todo] Author: complete this section.

### Extend Knowledge
<p id="extend-knowledge"></p>

[todo] Author: complete this section.

### NLG Evaluate
<p id="nlg-evaluate"></p>

[todo] Author: complete this section.

### Keyword Matching
<p id="keyword-matching"></p>

[todo] Author: complete this section.

### Health Check
<p id="health"></p>
Verifies the operational status of the API.

*   **Endpoint:** `GET /health`
*   **Description:** Checks the availability of the API service. Returns a `200 OK` if the service is healthy. This endpoint does not require authentication.
*   **Response Body (Success):**

    ```json
    {
      "status": "ok"
    }
    ```
