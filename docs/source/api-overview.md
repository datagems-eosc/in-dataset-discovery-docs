# API Overview

The In-Dataset Discovery service exposes a RESTful API for performing in-dataset exploration operations, including geospatial queries and text-to-SQL conversion.

## Base URL

The API is served from the root of the application. All endpoint paths are relative to the base URL where the service is deployed:

**Base URL:** [https://datagems-dev.scayle.es/in-dataset-discovery/](https://datagems-dev.scayle.es/in-dataset-discovery/)

## Authentication

The `/text2geo` and `/text2sql` endpoints can accessed only with authentication. See the [Security](./security.md) page for more details. The `/health` endpoint is publicly accessible.

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
    GET /text2geo?question=What are the coordinates in Berlin?
    ```

*   **Response Body (Success):**
    ```json
    {
      "place": "berlin",
      "most_relevant_wikidata": {
        "place": "Berlin",
        "wiki_id": "Q64",
        "wiki_properties": {
          "label": "Berlin",
          "description": "capital and largest city of Germany",
          "coordinate_location": "52.517, 13.3889",
          "found_osm_json": true
        },
        "reasoning": "..."
      },
      "oql": {
        "reasoning": "Generated Overpass QL query...",
        "OQL": "area[\"ISO3166-1\"=\"DE\"]->.searchArea;(nwr[\"wikidata\"=\"Q64\"];);"
      },
      "results": {
        "points": [{"lon": "13.3889", "lat": "52.517"}],
        "geojson_data": {"type": "FeatureCollection", "features": []},
        "bounds": {
          "minlat": 52.3383,
          "minlon": 13.0883,
          "maxlat": 52.6755,
          "maxlon": 13.7611
        },
        "center": [52.517, 13.3889]
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
