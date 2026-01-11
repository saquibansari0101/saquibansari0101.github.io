---
layout: post
title: Python for Data Engineering - My Experience with Pandas and FastAPI
---

While Java and Spring Boot are my primary tools for building microservices, Python has become an invaluable part of my toolkit for data engineering tasks. Here's what I've learned using Python, Pandas, and FastAPI in production.

## The Use Case

At Infosys, I was tasked with building a data validation and transformation framework that would:
- Validate data across multiple heterogeneous databases
- Transform data between different formats and schemas
- Provide REST APIs for data aggregation and quality checks
- Generate reports and analytics on data consistency

## Why Python?

For this use case, Python was the perfect choice:

**Pandas for Data Manipulation**  
Pandas made it incredibly easy to:
- Load data from multiple database sources
- Perform complex transformations and aggregations
- Handle missing or inconsistent data
- Generate statistical summaries

**NumPy for Numerical Operations**  
NumPy provided efficient array operations for:
- Mathematical transformations
- Statistical calculations
- Performance-critical computations

**FastAPI for Building APIs**  
FastAPI became my go-to framework for building REST APIs because:
- Automatic API documentation with Swagger/OpenAPI
- Built-in data validation using Pydantic models
- Async support for high-performance endpoints
- Type hints for better code quality and IDE support

## Real-World Example: Data Validation Suite

Here's how I structured the Data Validation Suite:

### 1. Data Extraction Layer
```python
# Extract data from Oracle, PostgreSQL, SQL Server
import pandas as pd
from sqlalchemy import create_engine

def extract_data(connection_string, query):
    engine = create_engine(connection_string)
    df = pd.read_sql(query, engine)
    return df
```

### 2. Validation Rules Engine
```python
# Define customizable validation rules
class ValidationRule:
    def __init__(self, name, condition, severity):
        self.name = name
        self.condition = condition
        self.severity = severity
    
    def validate(self, df):
        violations = df[~df.apply(self.condition, axis=1)]
        return violations
```

### 3. FastAPI Endpoints
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class ValidationRequest(BaseModel):
    source: str
    rules: List[str]
    
@app.post("/validate")
async def validate_data(request: ValidationRequest):
    # Extract, validate, and return results
    results = run_validation(request.source, request.rules)
    return {"status": "success", "violations": results}
```

### 4. Deployment
Containerized with Docker and deployed on Kubernetes alongside our Java microservices.

## Key Learnings

**1. Pandas is Powerful but Memory-Intensive**  
For large datasets (millions of rows), I learned to:
- Use chunking to process data in batches
- Optimize data types (e.g., use `category` for strings with few unique values)
- Use `dask` for datasets that don't fit in memory

**2. FastAPI's Async Capabilities are Game-Changing**  
Async endpoints allowed me to:
- Handle concurrent requests efficiently
- Perform non-blocking database operations
- Improve overall API throughput

**3. Type Hints Improve Code Quality**  
Using Pydantic models and type hints:
- Caught errors at development time
- Provided automatic request/response validation
- Made the code self-documenting

**4. Testing is Essential**  
I wrote comprehensive tests using pytest:
- Unit tests for validation rules
- Integration tests for database operations
- API tests using FastAPI's TestClient

## Performance Optimization

For production workloads, I implemented:

**Caching**  
Used Redis to cache frequently accessed data and validation results.

**Database Connection Pooling**  
Configured SQLAlchemy with connection pools to avoid connection overhead.

**Parallel Processing**  
Used Python's `concurrent.futures` for parallel validation across multiple data sources.

**Profiling**  
Used `cProfile` and `memory_profiler` to identify and optimize bottlenecks.

## Integration with Java Microservices

The Python services integrated seamlessly with our Java ecosystem:
- REST APIs for synchronous communication
- Kafka for async event streaming
- Shared PostgreSQL database for persistence
- Common monitoring and logging infrastructure

## Impact

The Data Validation Suite:
- Improved data accuracy by catching inconsistencies early
- Reduced manual validation effort by 80%
- Provided real-time visibility into data quality
- Became a critical tool for data governance

## Conclusion

Python, Pandas, and FastAPI are excellent tools for data engineering tasks. They complement Java/Spring Boot beautifully, allowing you to use the right tool for the right job.

If you're building data-intensive applications, I highly recommend adding Python to your toolkit. The ecosystem is mature, the libraries are powerful, and the developer experience is fantastic.
