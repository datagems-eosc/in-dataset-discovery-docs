# Status & Error Codes

The API uses standard HTTP status codes to indicate the success or failure of a request.

| Status Code | Meaning | Description |
| :--- | :--- | :--- |
| `200 OK` | Success | The request was successful. |
| `400 Bad Request` | Validation Error | The request body is malformed or contains invalid data. The response body will contain details about the validation error. |
| `401 Unauthorized` | Authentication Error | The request lacks a valid JWT, the token is expired, or its signature is invalid. |
| `403 Forbidden` | Authorization Error | The user is authenticated but does not have the required role (`user` or `dg_user`) to perform the action. |
| `422 Unprocessable Entity` | Validation Error | The request body is well-formed but contains semantic errors (e.g., missing required fields, invalid parameter values). The response body will contain details about the validation error. |
| `424 Failed Dependency` | Downstream Service Error | The API could not communicate with a required dependency, such as the OIDC provider, OpenAI API, Overpass API, or a user-specified database. The response body will contain details about the source of the failure. |
| `500 Internal Server Error` | Unexpected Error | An unexpected error occurred on the server while processing the request. This may occur due to agent failures, LLM errors, or other internal issues. |
| `503 Service Unavailable` | Service Not Ready | A core component failed to initialize at startup. This is typically seen during health checks if the service is not fully ready. |
