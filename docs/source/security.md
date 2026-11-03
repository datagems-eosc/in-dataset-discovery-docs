# Security

## Authentication

Authentication is handled via the **OAuth 2.0** and **OpenID Connect (OIDC)** protocols. Every request to a protected endpoint must include a valid JSON Web Token (JWT) in the `Authorization` header as a Bearer token.

The service validates incoming JWTs against the OIDC provider's public keys. The token's signature, issuer, and audience are all verified to ensure its authenticity.

## Authorization

Authorization is role-based. The service checks for specific roles within the validated JWT's claims.

- **Required Roles:** All operational endpoints except `/health` require authentication. That includes `GET /text2geo`, `POST /text2sql`, `GET /explore`, `POST /text2sql4ap`, `POST /explore4ap`, and `/textQuestionAnswer`.

If a user attempts to access an endpoint without the required role, the API will respond with a `403 Forbidden` error.

## Dataset-Level Permissions

In addition to role-based access, the service can enforce dataset-level permissions when integrated with the DataGEMS Gateway.

- **Database Access:**
    - `POST /text2sql` uses the database connection in the request. The caller supplies host, port, database, schema, username, and password. The service does not store those credentials.
    - `GET /explore` uses the service ERA5-Land settings (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME_ERA5LAND`, `DB_SCHEMA_ERA5LAND`).
    - `POST /text2sql4ap` and `POST /explore4ap` take an analytical pattern graph. Host, port, and database name come from the connection node's `contentUrl`, and the service authenticates with `DB_USER` and `DB_PASSWORD`.
