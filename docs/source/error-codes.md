# Status & Error Codes

The API uses standard HTTP status codes to indicate the success or failure of a request.

| Status Code | Meaning | Description |
| :--- | :--- | :--- |
| `200 OK` | Success | The request was successful. `POST /text2sql`, `GET /explore`, and the analytical-pattern endpoints can also return `200` with `"status": "error"` in the body. This soft failure means the call was accepted, but SQL generation or execution did not succeed. The log event is `text2sql_soft_failed` or `meteo_agent_soft_failed`. |
| `400 Bad Request` | Validation Error | The request body is malformed or contains invalid data. The response body will contain details about the validation error. |
| `401 Unauthorized` | Authentication Error | The request lacks a valid JWT, the token is expired, or its signature is invalid. |
| `403 Forbidden` | Authorization Error | The user is authenticated but does not have the required role (`user` or `dg_user`) to perform the action. |
| `404 Not Found` | Place Not Found | `GET /text2geo`, and the text2geo step of explore, found no matching place. If LLM authentication fails, the client can receive the same "No place name found" message. The log event is `geospatial_place_name_extraction_failed`. |
| `422 Unprocessable Entity` | Validation Error | The request is well-formed but missing required fields, or an analytical pattern graph is invalid (missing the natural language operator, the relational-database node, the upstream dataset, or the database connection node `contentUrl`). |
| `424 Failed Dependency` | Downstream Service Error | A required dependency failed. For `POST /text2sql4ap` and `POST /explore4ap` this includes DMM schema fetch failures and PostgreSQL connection failures (`DB_USER` / `DB_PASSWORD` against the host in `contentUrl`). |
| `500 Internal Server Error` | Unexpected Error | An unexpected error occurred on the server while processing the request. This may occur due to agent failures, LLM errors, or other internal issues. |
| `503 Service Unavailable` | Dependency Unavailable | Wikidata or Overpass could not be reached during `GET /text2geo` or explore. The same status is used when a core component is not ready. |
