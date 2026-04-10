# API Overview

The In-Dataset Discovery service exposes a RESTful API for performing in-dataset exploration operations, including geospatial queries, text-to-SQL conversion, RAG-based question answering, and API exploration.

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

### Explore
<p id="explore-endpoint"></p>
Provides a generic authenticated endpoint for API exploration.

*   **Endpoint:** `GET /explore`
*   **Description:** Accepts a required `question` query parameter and returns an open-ended JSON object describing the exploration result.
*   **Authentication:** Required.

*   **Query Parameters:**
    - `question` (required): The exploration question to send to the service.

*   **Example Request:**
    ```
    GET /explore?question=What endpoints are available for text-to-SQL workflows?
    ```

*   **Response Body:** The response schema is intentionally flexible and may vary by deployment or query type.

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
