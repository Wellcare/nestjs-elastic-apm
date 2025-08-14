# Database Schema

This document describes the data structures and relationships used by the Elastic APM system. The actual data is stored in Elasticsearch by the APM Server, but this schema represents the conceptual structure that the NestJS module interacts with.

```dbml
// Elastic APM Data Schema
// This represents the conceptual data structures used by the APM system
// Actual storage is handled by Elasticsearch via the APM Server

Table transactions {
  id varchar [primary key, note: "Unique transaction identifier"]
  trace_id varchar [not null, note: "Distributed tracing identifier"]
  parent_id varchar [note: "Parent span/transaction ID"]
  name varchar [not null, note: "Transaction name"]
  type varchar [not null, note: "Transaction type (request, custom, etc.)"]
  result varchar [note: "Transaction result (success, error, etc.)"]
  duration decimal [not null, note: "Transaction duration in milliseconds"]
  timestamp timestamp [not null, note: "Transaction start timestamp"]
  context json [note: "Transaction context (user, custom, etc.)"]
  sampled boolean [note: "Whether transaction was sampled"]
  service_name varchar [not null, note: "Service identifier"]
  service_version varchar [note: "Service version"]
  service_environment varchar [note: "Environment name"]
}

Table spans {
  id varchar [primary key, note: "Unique span identifier"]
  transaction_id varchar [not null, note: "Parent transaction ID"]
  trace_id varchar [not null, note: "Distributed tracing identifier"]
  parent_id varchar [note: "Parent span ID"]
  name varchar [not null, note: "Span name"]
  type varchar [not null, note: "Span type (db, external, custom, etc.)"]
  subtype varchar [note: "Span subtype (mysql, http, etc.)"]
  action varchar [note: "Span action (query, request, etc.)"]
  duration decimal [not null, note: "Span duration in milliseconds"]
  timestamp timestamp [not null, note: "Span start timestamp"]
  context json [note: "Span context (db, http, etc.)"]
  stacktrace json [note: "Stack trace information"]
  sampled boolean [note: "Whether span was sampled"]
  service_name varchar [not null, note: "Service identifier"]
}

Table errors {
  id varchar [primary key, note: "Unique error identifier"]
  transaction_id varchar [note: "Associated transaction ID"]
  trace_id varchar [not null, note: "Distributed tracing identifier"]
  span_id varchar [note: "Associated span ID"]
  error_id varchar [not null, note: "Error grouping identifier"]
  parent_id varchar [note: "Parent error ID"]
  culprit varchar [note: "Error culprit (function/file)"]
  exception json [not null, note: "Exception details"]
  log json [note: "Log message details"]
  timestamp timestamp [not null, note: "Error timestamp"]
  context json [note: "Error context"]
  sampled boolean [note: "Whether error was sampled"]
  service_name varchar [not null, note: "Service identifier"]
  service_version varchar [note: "Service version"]
  service_environment varchar [note: "Environment name"]
}

Table metrics {
  id varchar [primary key, note: "Unique metric identifier"]
  transaction_id varchar [note: "Associated transaction ID"]
  trace_id varchar [not null, note: "Distributed tracing identifier"]
  span_id varchar [note: "Associated span ID"]
  name varchar [not null, note: "Metric name"]
  value decimal [not null, note: "Metric value"]
  unit varchar [note: "Metric unit"]
  timestamp timestamp [not null, note: "Metric timestamp"]
  context json [note: "Metric context"]
  service_name varchar [not null, note: "Service identifier"]
  service_version varchar [note: "Service version"]
  service_environment varchar [note: "Environment name"]
}

Table services {
  name varchar [primary key, note: "Service name"]
  version varchar [note: "Service version"]
  environment varchar [note: "Environment name"]
  agent_name varchar [note: "Agent name (nodejs)"]
  agent_version varchar [note: "Agent version"]
  framework_name varchar [note: "Framework name (NestJS)"]
  framework_version varchar [note: "Framework version"]
  language_name varchar [note: "Language name (javascript)"]
  language_version varchar [note: "Language version"]
  runtime_name varchar [note: "Runtime name (node)"]
  runtime_version varchar [note: "Runtime version"]
  created_at timestamp [not null, note: "Service registration timestamp"]
  updated_at timestamp [not null, note: "Last update timestamp"]
}

// Relationships
Ref: spans.transaction_id > transactions.id
Ref: spans.parent_id > spans.id
Ref: errors.transaction_id > transactions.id
Ref: errors.span_id > spans.id
Ref: errors.parent_id > errors.id
Ref: metrics.transaction_id > transactions.id
Ref: metrics.span_id > spans.id

// Indexes for performance
// Note: These are conceptual indexes that would be created in Elasticsearch
// transactions.trace_id [note: "Index for distributed tracing queries"]
// transactions.timestamp [note: "Index for time-based queries"]
// spans.trace_id [note: "Index for trace span queries"]
// spans.timestamp [note: "Index for time-based span queries"]
// errors.trace_id [note: "Index for error tracing"]
// errors.timestamp [note: "Index for error time queries"]
// metrics.timestamp [note: "Index for metric time series"]
```

## Data Flow

1. **Transaction Creation**: When a request comes in, a transaction is created with a unique trace ID
2. **Span Creation**: Within transactions, spans are created for specific operations (DB queries, HTTP calls, etc.)
3. **Error Capture**: Errors are captured and linked to transactions/spans
4. **Metric Collection**: Performance metrics are collected throughout the process
5. **Data Transmission**: All data is sent to the APM Server, which stores it in Elasticsearch

## Key Concepts

### Distributed Tracing
- **Trace ID**: Unique identifier for a distributed request
- **Transaction ID**: Identifier for a single service's handling of a request
- **Span ID**: Identifier for a specific operation within a transaction

### Sampling
- Not all transactions/spans are stored to reduce storage costs
- Sampling decisions are made at the transaction level
- Child spans inherit the sampling decision

### Context
- **User Context**: Information about the user making the request
- **Custom Context**: Application-specific data
- **System Context**: Information about the service and environment
