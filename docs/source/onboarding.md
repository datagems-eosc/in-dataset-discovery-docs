# Developer Onboarding Guide

Welcome to the developer guide for the **In-Dataset Discovery Service**.

This document provides a comprehensive overview of the service's architecture and a step-by-step guide for extending it with new features, agents, and tools.

## Table of Contents

1. [Service Overview](#1-service-overview)
   * [Core Principles](#core-principles)
   * [Directory Structure](#directory-structure)
   * [Key Components](#key-components)
2. [System Architecture](#2-system-architecture)
   * [API Layer](#api-layer)
   * [Agent Layer](#agent-layer)
   * [Tool Layer](#tool-layer)
3. [Adding New Features](#3-adding-new-features)
   * [Adding a New API Endpoint](#adding-a-new-api-endpoint)
   * [Adding a New Agent](#adding-a-new-agent)
   * [Adding a New Tool](#adding-a-new-tool)
4. [Development Workflow](#4-development-workflow)
   * [Local Setup](#local-setup)
   * [Testing](#testing)
   * [Code Style](#code-style)

---

## 1. Service Overview

The In-Dataset Discovery Service is a FastAPI-based application that provides natural language interfaces for exploring datasets. It uses LLM capabilities to convert user questions into executable queries and agents to orchestrate complex multi-step tasks.

### Core Principles

The service is built on a few key principles:

- **Modularity:** Each component (agents, tools, models) is self-contained and can be extended independently.
- **LLM-Powered:** Leverages LLM and LangChain for intelligent query generation and processing.
- **Agent-Based:** Uses LangGraph to orchestrate complex workflows that combine multiple tools.
- **Type Safety:** Uses Pydantic models for request/response validation and type safety.

### Directory Structure

```
src/
├── app/                    # FastAPI application
│   ├── main.py            # API endpoints and routing
│   ├── config.py          # Configuration management
│   ├── security.py        # Authentication and authorization
│   ├── exceptions.py      # Custom exception classes
│   └── logging_config.py  # Logging setup
├── agents/                 # Agent implementations
│   ├── geospatial.py      # Geospatial query agent
│   ├── text2sql.py        # Text-to-SQL agent
│   ├── graph.py           # LangGraph workflow definitions
│   └── tools/             # Agent tools
│       ├── geospatial_tools.py
│       └── text2sql_tool.py
└── models/                 # Pydantic models
    └── geospatial_models.py
```

### Key Components

1. **FastAPI Application** (`src/app/main.py`): Defines all API endpoints and request/response models.
2. **Agents** (`src/agents/`): Implement the core logic for processing queries:
   - `geospatial.py`: Handles geospatial queries using Wikidata and Overpass API
   - `text2sql.py`: Converts natural language to SQL queries
3. **Tools** (`src/agents/tools/`): Reusable functions that agents can call:
   - `geospatial_tools.py`: Tools for geospatial operations
   - `text2sql_tool.py`: Tools for SQL generation and execution
4. **Models** (`src/agents/models/`): Pydantic models for data validation

---

## 2. System Architecture

### API Layer

The API layer (`src/app/main.py`) handles HTTP requests and responses. Each endpoint:
- Validates input using Pydantic models
- Handles authentication (if required)
- Calls the appropriate agent or tool
- Returns structured responses

### Agent Layer

Agents are the core processing units that handle specific types of queries:

- **Geospatial Agent**: Processes location-based queries
- **Text-to-SQL Agent**: Converts questions to SQL queries

### Tool Layer

Tools are reusable functions that agents can invoke. They encapsulate specific operations like:
- Querying external APIs (Overpass, Wikidata)
- Generating SQL queries
- Executing database queries

---

## 3. Adding New Features

### Adding a New API Endpoint

1. **Define Request/Response Models** in `src/app/main.py`:

```python
class MyQueryRequest(BaseModel):
    question: str = Field(..., description="The question to process")

class MyQueryResponse(BaseModel):
    answer: str
    metadata: Dict[str, Any]
```

2. **Create the Endpoint**:

```python
@app.post("/my-endpoint", response_model=MyQueryResponse)
async def my_endpoint(query: MyQueryRequest):
    """Process a query and return results."""
    # Your logic here
    result = process_query(query.question)
    return MyQueryResponse(answer=result, metadata={})
```

3. **Add Error Handling**:

```python
try:
    result = process_query(query.question)
    return MyQueryResponse(answer=result, metadata={})
except Exception as e:
    logger.error("query_failed", error=str(e))
    raise HTTPException(status_code=500, detail=str(e))
```

### Adding a New Agent

1. **Create Agent File** in `src/agents/`:

```python
# src/agents/my_agent.py
import structlog
from typing import Dict, Any

logger = structlog.get_logger(__name__)

def my_agent(question: str, log: structlog.BoundLogger) -> Dict[str, Any]:
    """Process a question using the my_agent."""
    log.info("my_agent_started", question=question)

    # Your agent logic here
    result = process_question(question)

    log.info("my_agent_completed", result=result)
    return result
```

2. **Add Tools** (if needed) in `src/agents/tools/`:

```python
# src/agents/tools/my_tool.py
def my_tool(param: str) -> Dict[str, Any]:
    """A tool that does something useful."""
    # Tool implementation
    return {"result": "value"}
```

3. **Integrate with API** in `src/app/main.py`:

```python
from src.agents.my_agent import my_agent

@app.get("/my-endpoint")
async def my_endpoint(question: str, log: structlog.BoundLogger = Depends(get_logger)):
    return my_agent(question, log)
```

### Adding a New Tool

1. **Create Tool File** in `src/agents/tools/`:

```python
# src/agents/tools/my_tool.py
from typing import Dict, Any

def my_tool(param1: str, param2: int) -> Dict[str, Any]:
    """
    Tool description.

    Args:
        param1: Description of param1
        param2: Description of param2

    Returns:
        Dictionary with results
    """
    # Tool implementation
    return {"result": "value"}
```

2. **Use in Agent**:

```python
from src.agents.tools.my_tool import my_tool

def my_agent(question: str) -> Dict[str, Any]:
    result = my_tool(param1="value", param2=42)
    return result
```


---

## 4. Development Workflow

### Local Setup

1. **Clone the repository**:
```bash
git clone git@github.com:datagems-eosc/in-data-exploration.git
cd in-data-exploration
```

2. **Install dependencies**:
```bash
# Using uv (recommended)
uv sync

# Or using pip
pip install -e .
```

3. **Set up environment variables**:
```bash
cp .env.template .env
# Edit .env with your configuration
```

4. **Run the service**:
```bash
# Using uvicorn directly
uvicorn src.app.main:app --reload --port 8080

# Or using Docker
docker build -t in-data-exploration .
docker run --env-file .env -p 8080:8080 in-data-exploration
```

### Testing

1. **Run health check**:
```bash
curl http://localhost:8080/health
```

2. **Test an endpoint**:
```bash
curl "http://localhost:8080/geospatial?question=What are the coordinates in Berlin?"
```

3. **View API documentation**:
- Swagger UI: http://localhost:8080/swagger
- ReDoc: http://localhost:8080/redoc

### Code Style

- Follow PEP 8 style guidelines
- Use type hints for all function parameters and return values
- Use Pydantic models for data validation
- Add docstrings to all public functions and classes
- Use structured logging with `structlog`
- Handle errors gracefully and return appropriate HTTP status codes

### Best Practices

1. **Error Handling**: Always wrap agent calls in try-except blocks and log errors appropriately.
2. **Logging**: Use structured logging with context (user ID, correlation ID, etc.).
3. **Validation**: Use Pydantic models for all request/response validation.
4. **Security**: Follow the security patterns established in `src/app/security.py` for protected endpoints.
5. **Testing**: Write tests for new features and ensure they pass before submitting PRs.

---

## Next Steps

- Review the [Architecture](./architecture.md) documentation for more details on system design
- Check the [API Overview](./api-overview.md) to understand existing endpoints
- Read the [Configuration](./configuration.md) guide for environment setup
- Explore the source code in `src/agents/` to see examples of agent implementations
