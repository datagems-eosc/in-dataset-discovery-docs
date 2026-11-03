# Maintenance

This document outlines key maintenance tasks for the In-Dataset Discovery Service.

## Regular Maintenance Tasks

### Monitoring Service Health

Regularly monitor the service health using the `/health` endpoint to ensure all components are functioning correctly.

### API Key Management

- **Scayle credentials**: The default LLM is Scayle (`LLM_PROVIDER=scayle`). A chat API `401` means the credentials were rejected. Rotate `SCAYLE_PASSWORD` and restart the pods that should pick it up.
- **OpenAI API Key**: Required only when `LLM_PROVIDER=openai`.
- **LangChain API Key**: If using LangSmith, monitor usage and ensure the key is valid.

### Dependency Updates

Regularly update dependencies to:
- Fix security vulnerabilities
- Get bug fixes and performance improvements
- Stay compatible with external services (Overpass API, Scayle, Qdrant, etc.)

### Log Monitoring

Monitor application logs for:
- Error patterns that may indicate issues
- Performance degradation
- Authentication/authorization failures
- Downstream service failures (Scayle, Overpass, Wikidata, Qdrant `retrieval_degraded`, PostgreSQL)

### Database Connection Management

For `POST /text2sql`, `GET /explore`, `POST /text2sql4ap`, and `POST /explore4ap`:
- Monitor database connection performance
- `POST /text2sql` uses credentials from the request. `GET /explore` and the analytical-pattern endpoints use `DB_USER` and `DB_PASSWORD`.
- Watch for `retrieval_degraded` when `RETRIEVAL_ENABLED` is on and the Qdrant collections are missing.

## Troubleshooting

### Common Issues

1. **Scayle errors**: A `401` is a credential failure. Timeouts, DNS errors, and `5xx` mean the chat API is unreachable. Check `SCAYLE_BASE_URL`.
2. **Overpass or Wikidata failures**: `GET /text2geo` returns `503` when those services cannot be reached, and `404` when no place is found.
3. **Database connection failures**: For `POST /text2sql`, check the request `db_info`. For `GET /explore` and the analytical-pattern endpoints, check `DB_USER`, `DB_PASSWORD`, and, on those endpoints, `contentUrl`. Those connection failures return `424`.
4. **Authentication Failures**: Verify OIDC provider configuration and token validity.
5. **Retrieval**: `retrieval_degraded` at startup means text-to-SQL is running without few-shot examples from in-cluster Qdrant.

### Performance Optimization

- Monitor response times for each endpoint
- Consider caching frequently accessed data (e.g., Wikidata lookups)
- Optimize LLM prompts to reduce token usage and improve response times
- Monitor and optimize database query performance for text-to-SQL operations
