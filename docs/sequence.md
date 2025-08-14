# System Workflows

This document describes the key workflows and interactions between the NestJS application, Elastic APM agent, and APM server.

## HTTP Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant NestJS as NestJS Application
    participant Interceptor as ApmInterceptor
    participant Service as ApmService
    participant APM as Elastic APM Agent
    participant Server as APM Server

    Client->>NestJS: HTTP Request
    NestJS->>Interceptor: Request intercepted
    Interceptor->>APM: Automatic transaction start
    APM->>Server: Send transaction data
    
    Note over NestJS: Business logic execution
    
    alt Success
        NestJS->>Interceptor: Response
        Interceptor->>APM: Transaction end (success)
        APM->>Server: Send final transaction data
        Interceptor->>Client: HTTP Response
    else Error occurs
        NestJS->>Interceptor: Exception thrown
        Interceptor->>Service: captureError()
        Service->>APM: Send error data
        APM->>Server: Send error to APM Server
        Interceptor->>Client: HTTP Error Response
    end
```

## Custom Span Creation

```mermaid
sequenceDiagram
    participant Service as Business Service
    participant ApmService as ApmService
    participant APM as Elastic APM Agent
    participant Server as APM Server

    Service->>ApmService: startSpan('operation-name')
    ApmService->>APM: Create span
    APM->>Server: Send span start
    
    Note over Service: Execute operation
    
    Service->>ApmService: span.end()
    ApmService->>APM: End span
    APM->>Server: Send span end data
```

## Custom Transaction Flow

```mermaid
sequenceDiagram
    participant Service as Business Service
    participant ApmService as ApmService
    participant APM as Elastic APM Agent
    participant Server as APM Server

    Service->>ApmService: startTransaction('Custom Transaction')
    ApmService->>APM: Create custom transaction
    APM->>Server: Send transaction start
    
    Note over Service: Execute business logic
    
    alt Error occurs
        Service->>ApmService: captureError(error)
        ApmService->>APM: Send error data
        APM->>Server: Send error to APM Server
    end
    
    Service->>ApmService: transaction.end()
    ApmService->>APM: End transaction
    APM->>Server: Send transaction end data
```

## Error Handling Workflow

```mermaid
sequenceDiagram
    participant Controller as NestJS Controller
    participant Service as Business Service
    participant Interceptor as ApmInterceptor
    participant ApmService as ApmService
    participant APM as Elastic APM Agent
    participant Server as APM Server

    Controller->>Service: Call business method
    
    alt HttpException
        Service->>Controller: Throw HttpException
        Controller->>Interceptor: Exception caught
        Interceptor->>ApmService: captureError(exception.message)
        ApmService->>APM: Send error with message
        APM->>Server: Send error data
        Interceptor->>Controller: Re-throw exception
    else Generic Error
        Service->>Controller: Throw generic error
        Controller->>Interceptor: Exception caught
        Interceptor->>ApmService: captureError(error)
        ApmService->>APM: Send full error object
        APM->>Server: Send error data
        Interceptor->>Controller: Re-throw error
    end
```

## Module Initialization Flow

```mermaid
sequenceDiagram
    participant Main as main.ts
    participant Start as start.ts
    participant Module as ApmModule
    participant Service as ApmService
    participant Interceptor as ApmInterceptor
    participant APM as Elastic APM Agent

    Main->>Start: Import apm from 'nestjs-elastic-apm'
    Start->>APM: Initialize with environment config
    Note over Start: Configure service name, token, URL, etc.
    
    Main->>Module: ApmModule.register()
    Module->>Service: Create ApmService instance
    Module->>Interceptor: Create ApmInterceptor instance
    Module->>Module: Register as APP_INTERCEPTOR
    
    Note over Module: Module ready for use
```

## Custom Context Setting

```mermaid
sequenceDiagram
    participant Service as Business Service
    participant ApmService as ApmService
    participant APM as Elastic APM Agent
    participant Server as APM Server

    Service->>ApmService: setCustomContext({userId: '123', role: 'admin'})
    ApmService->>APM: Set custom context
    APM->>Server: Send context with next transaction/span
    
    Note over Service: Continue execution
    
    Service->>ApmService: startSpan('operation')
    ApmService->>APM: Create span with context
    APM->>Server: Send span with custom context
```

## Configuration Sources

The module uses the following environment variables for configuration:

- `ELASTIC_APM_ACTIVATE`: Enable/disable APM agent
- `ELASTIC_APM_SERVICE_NAME`: Override service name
- `ELASTIC_APM_SECRET_TOKEN`: APM Server authentication token
- `ELASTIC_APM_API_KEY`: APM Server API key authentication
- `ELASTIC_APM_SERVER_URL`: APM Server endpoint
- `ELASTIC_APM_DISABLE_INSTRUMENTATIONS`: Comma-separated list of disabled instrumentations

## Message Schemas

### Transaction Data
```json
{
  "id": "transaction-id",
  "trace_id": "trace-id",
  "name": "Transaction Name",
  "type": "request",
  "duration": 150.5,
  "timestamp": "2024-01-01T00:00:00.000Z",
  "context": {
    "user": { "id": "user-123" },
    "custom": { "business_data": "value" }
  }
}
```

### Error Data
```json
{
  "id": "error-id",
  "trace_id": "trace-id",
  "transaction_id": "transaction-id",
  "exception": {
    "message": "Error message",
    "type": "Error",
    "stacktrace": [...]
  },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Span Data
```json
{
  "id": "span-id",
  "transaction_id": "transaction-id",
  "trace_id": "trace-id",
  "name": "Database Query",
  "type": "db",
  "subtype": "mysql",
  "action": "query",
  "duration": 25.3,
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```
