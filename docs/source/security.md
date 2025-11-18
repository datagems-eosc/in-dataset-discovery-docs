# Security

## Authentication

Authentication is handled via the **OAuth 2.0** and **OpenID Connect (OIDC)** protocols. Every request to a protected endpoint must include a valid JSON Web Token (JWT) in the `Authorization` header as a Bearer token.

The service validates incoming JWTs against the OIDC provider's public keys. The token's signature, issuer, and audience are all verified to ensure its authenticity.

## Authorization

Authorization is role-based. The service checks for specific roles within the validated JWT's claims.

- **Required Roles:** The `/geospatial` and `/text2sql` endpoints may have different authentication requirements depending on the deployment configuration. The `/health` endpoint is publicly accessible.

If a user attempts to access an endpoint without the required role, the API will respond with a `403 Forbidden` error.

## Dataset-Level Permissions

In addition to role-based access, the service can enforce dataset-level permissions when integrated with the DataGEMS Gateway.

- **Database Access:**
    - For `/text2sql` endpoints, database access is controlled through the database connection information provided in the request. Users must have valid credentials for the databases they wish to query.
    - The service does not store or manage database credentials; they must be provided with each request.
