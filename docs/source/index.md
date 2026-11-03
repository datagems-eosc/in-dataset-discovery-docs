# Overview

Welcome to the documentation for the **In-Dataset Discovery Service**.

This service provides an API for natural-language exploration of structured and unstructured data in a dataset. It uses LLM-backed geospatial lookup and text-to-SQL. `GET /explore` runs the meteo text-to-SQL agent, which calls text2geo and then text-to-SQL. `POST /text2sql4ap` and `POST /explore4ap` write that SQL back onto an analytical pattern graph.

## How to Use This Documentation

- **System:** Learn about the high-level [Architecture](./architecture.md), [Security](./security.md) model, and [Datastores](./datastore.md).
- **API:** Find detailed information on the [API endpoints](./api-overview.md) and [error codes](./error-codes.md).
- **Setup & Monitoring:** Get instructions on [Deployment](./deployment.md), [Configuration](./configuration.md), and [Logging](./logging.md).
