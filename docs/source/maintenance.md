# Maintenance

This document outlines key maintenance tasks for the In-Dataset Discovery Service.

## Regular Maintenance Tasks

### Monitoring Service Health

Regularly monitor the service health using the `/health` endpoint to ensure all components are functioning correctly.

### API Key Management

- **OpenAI API Key**: Monitor usage and ensure the API key remains valid. Rotate keys periodically for security.
- **LangChain API Key**: If using LangSmith, monitor usage and ensure the key is valid.

### Dependency Updates

Regularly update dependencies to:
- Fix security vulnerabilities
- Get bug fixes and performance improvements
- Stay compatible with external services (Overpass API, OpenAI API, etc.)

### Log Monitoring

Monitor application logs for:
- Error patterns that may indicate issues
- Performance degradation
- Authentication/authorization failures
- Downstream service failures (OpenAI, Overpass API, etc.)

### Database Connection Management

For the `/text2sql` endpoint:
- Monitor database connection performance
- Ensure proper connection pooling if implemented
- Validate that user-provided database credentials are handled securely

## Troubleshooting

### Common Issues

1. **OpenAI API Errors**: Check API key validity, rate limits, and service status.
2. **Overpass API Timeouts**: The Overpass API may be slow or unavailable. Consider implementing retries or fallback mechanisms.
3. **Database Connection Failures**: Verify network connectivity and database credentials provided by users.
4. **Authentication Failures**: Verify OIDC provider configuration and token validity.

### Performance Optimization

- Monitor response times for each endpoint
- Consider caching frequently accessed data (e.g., Wikidata lookups)
- Optimize LLM prompts to reduce token usage and improve response times
- Monitor and optimize database query performance for text-to-SQL operations
