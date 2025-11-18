# System Architecture

The In-Dataset Discovery Service is a self-contained application designed to act as a specialized query layer within the DataGEMS ecosystem.

## Core Components

1.  **FastAPI Application:** The heart of the service is a Python web application built with FastAPI. It exposes the REST API, handles incoming HTTP requests, and orchestrates all internal operations.

2.  **Geospatial Component:** A module that processes natural language geospatial queries, identifies places using Wikidata, generates OverpassQL queries, and retrieves geospatial data from OpenStreetMap via the Overpass API.

3.  **Text-to-SQL Component:** An intelligent module that converts natural language questions into SQL queries using LLM capabilities. It handles database schema understanding, query generation, and execution.

4.  **Security Layer:** This layer intercepts all incoming requests to perform authentication and authorization. It integrates with an external OIDC provider to validate JWTs and with the DataGEMS Gateway to fetch user-specific permissions.

## Request Flow

### Geospatial Query Flow

1.  A user sends a `GET /geospatial` request with a natural language question about a location.
2.  The service uses Wikidata to identify the most relevant place entity.
3.  An OverpassQL query is generated based on the Wikidata information.
4.  The Overpass API is queried to retrieve geospatial data.
5.  Results are formatted and returned, including coordinates, GeoJSON data, and bounding boxes.

### Text-to-SQL Query Flow

1.  A user sends a `POST /text2sql` request with a natural language question and database connection information.
2.  The Security Layer validates the token and checks for the required user roles (if authentication is enabled).
3.  The Text-to-SQL Component uses an LLM to analyze the question and database schema.
4.  A SQL query pattern is generated and parameterized with the provided coordinates or other parameters.
5.  The SQL query is executed against the specified database.
6.  Results are returned with the generated SQL and execution results.
