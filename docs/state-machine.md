# State Machines

This document describes the state machines and lifecycle flows for APM transactions, spans, and error handling processes.

## Transaction Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> Inactive
    
    Inactive --> Active : ELASTIC_APM_ACTIVATE=true
    Inactive --> [*] : Module not registered
    
    Active --> Initialized : ApmModule.register()
    
    Initialized --> Ready : Agent started successfully
    Initialized --> Error : Agent start failed
    
    Ready --> TransactionStarted : startTransaction() called
    Ready --> SpanStarted : startSpan() called
    Ready --> ErrorCaptured : captureError() called
    
    TransactionStarted --> TransactionRunning : Transaction created
    TransactionStarted --> Error : Transaction creation failed
    
    TransactionRunning --> SpanStarted : startSpan() called
    TransactionRunning --> ErrorCaptured : captureError() called
    TransactionRunning --> TransactionEnded : transaction.end() called
    TransactionRunning --> Error : Transaction error
    
    SpanStarted --> SpanRunning : Span created
    SpanStarted --> Error : Span creation failed
    
    SpanRunning --> SpanEnded : span.end() called
    SpanRunning --> ErrorCaptured : captureError() called
    SpanRunning --> Error : Span error
    
    SpanEnded --> TransactionRunning : Return to parent transaction
    SpanEnded --> Ready : No parent transaction
    
    TransactionEnded --> Ready : Transaction completed
    TransactionEnded --> Error : Transaction end failed
    
    ErrorCaptured --> TransactionRunning : Error captured in transaction
    ErrorCaptured --> Ready : Error captured outside transaction
    
    Error --> Ready : Error handled
    Error --> [*] : Fatal error
```

## Span Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> SpanNotStarted
    
    SpanNotStarted --> SpanCreating : startSpan() called
    
    SpanCreating --> SpanActive : Span created successfully
    SpanCreating --> SpanFailed : Span creation failed
    
    SpanActive --> SpanExecuting : Business logic execution
    SpanActive --> SpanEnding : span.end() called
    SpanActive --> SpanError : Error occurs
    
    SpanExecuting --> SpanExecuting : Continue execution
    SpanExecuting --> SpanEnding : span.end() called
    SpanExecuting --> SpanError : Error occurs
    
    SpanEnding --> SpanCompleted : Span ended successfully
    SpanEnding --> SpanFailed : Span end failed
    
    SpanError --> SpanEnding : Error handled, ending span
    SpanError --> SpanFailed : Error not handled
    
    SpanCompleted --> [*] : Span lifecycle complete
    SpanFailed --> [*] : Span failed
```

## Error Handling State Machine

```mermaid
stateDiagram-v2
    [*] --> NoError
    
    NoError --> ErrorOccurred : Exception thrown
    
    ErrorOccurred --> ErrorIntercepted : ApmInterceptor catches error
    ErrorOccurred --> ErrorCaptured : Manual captureError() call
    
    ErrorIntercepted --> ErrorTypeCheck : Determine error type
    
    ErrorTypeCheck --> HttpExceptionHandled : HttpException detected
    ErrorTypeCheck --> GenericErrorHandled : Generic error detected
    
    HttpExceptionHandled --> ErrorCaptured : captureError(exception.message)
    GenericErrorHandled --> ErrorCaptured : captureError(error)
    
    ErrorCaptured --> ErrorSent : Error sent to APM Server
    ErrorCaptured --> ErrorFailed : Error capture failed
    
    ErrorSent --> ErrorReThrown : Error re-thrown to client
    ErrorSent --> ErrorHandled : Error handled gracefully
    
    ErrorFailed --> ErrorReThrown : Original error re-thrown
    ErrorFailed --> ErrorHandled : Error handled without APM
    
    ErrorReThrown --> [*] : Error propagated to client
    ErrorHandled --> [*] : Error handled successfully
```

## Module Initialization State Machine

```mermaid
stateDiagram-v2
    [*] --> NotInitialized
    
    NotInitialized --> EnvironmentCheck : ApmModule.register() called
    
    EnvironmentCheck --> ConfigValid : Environment variables valid
    EnvironmentCheck --> ConfigInvalid : Missing required config
    
    ConfigValid --> AgentStarting : Start Elastic APM agent
    ConfigInvalid --> AgentDisabled : Agent disabled
    
    AgentStarting --> AgentStarted : Agent started successfully
    AgentStarting --> AgentFailed : Agent start failed
    
    AgentStarted --> ServiceCreated : ApmService created
    AgentStarted --> InterceptorCreated : ApmInterceptor created
    
    ServiceCreated --> InterceptorCreated : Continue initialization
    InterceptorCreated --> ModuleReady : Module fully initialized
    
    AgentDisabled --> ModuleReady : Module ready (disabled mode)
    AgentFailed --> ModuleReady : Module ready (error mode)
    
    ModuleReady --> [*] : Module available for use
```

## Request Processing State Machine

```mermaid
stateDiagram-v2
    [*] --> RequestReceived
    
    RequestReceived --> InterceptorActive : ApmInterceptor activated
    
    InterceptorActive --> TransactionCreated : Automatic transaction start
    InterceptorActive --> ErrorOccurred : Interceptor error
    
    TransactionCreated --> RequestProcessing : Business logic execution
    TransactionCreated --> ErrorOccurred : Transaction creation failed
    
    RequestProcessing --> RequestSuccess : Processing completed
    RequestProcessing --> RequestError : Processing failed
    
    RequestSuccess --> TransactionEnded : Transaction ended successfully
    RequestSuccess --> ErrorOccurred : Transaction end failed
    
    RequestError --> ErrorCaptured : Error captured by interceptor
    RequestError --> TransactionEnded : Error handled, transaction ended
    
    ErrorCaptured --> TransactionEnded : Error sent to APM, transaction ended
    
    TransactionEnded --> ResponseSent : Response sent to client
    TransactionEnded --> ErrorOccurred : Response sending failed
    
    ResponseSent --> [*] : Request complete
    ErrorOccurred --> [*] : Request failed
```

## State Transitions Table

| Trigger | From State | To State | Conditions |
|---------|------------|----------|------------|
| `ELASTIC_APM_ACTIVATE=true` | Inactive | Active | Environment variable set |
| `ApmModule.register()` | Active | Initialized | Module registration |
| Agent start success | Initialized | Ready | APM agent started |
| `startTransaction()` | Ready | TransactionStarted | Valid transaction name |
| `startSpan()` | Ready/TransactionRunning | SpanStarted | Valid span name |
| `captureError()` | Any | ErrorCaptured | Error object provided |
| `transaction.end()` | TransactionRunning | TransactionEnded | Transaction exists |
| `span.end()` | SpanRunning | SpanEnded | Span exists |
| Exception thrown | Any | ErrorOccurred | Unhandled exception |
| Interceptor catch | ErrorOccurred | ErrorIntercepted | HTTP request context |
| Manual capture | ErrorOccurred | ErrorCaptured | Direct service call |

## State Metadata

### Transaction States
- **Inactive**: APM agent not enabled
- **Active**: APM agent enabled but not initialized
- **Initialized**: Module registered, agent starting
- **Ready**: Agent ready to handle requests
- **TransactionStarted**: Custom transaction created
- **TransactionRunning**: Transaction active and collecting data
- **TransactionEnded**: Transaction completed and sent to APM server

### Span States
- **SpanNotStarted**: No span active
- **SpanCreating**: Span creation in progress
- **SpanActive**: Span created and ready
- **SpanExecuting**: Business logic running within span
- **SpanEnding**: Span ending process
- **SpanCompleted**: Span ended successfully
- **SpanFailed**: Span creation or execution failed

### Error States
- **NoError**: Normal operation
- **ErrorOccurred**: Exception or error detected
- **ErrorIntercepted**: Error caught by interceptor
- **ErrorTypeCheck**: Determining error type
- **ErrorCaptured**: Error captured by APM service
- **ErrorSent**: Error sent to APM server
- **ErrorReThrown**: Error propagated to client
- **ErrorHandled**: Error handled gracefully

## Error Handling Patterns

### Automatic Error Capture
1. Exception thrown in business logic
2. ApmInterceptor catches the exception
3. Error type determined (HttpException vs generic)
4. Error captured with appropriate context
5. Error sent to APM server
6. Original exception re-thrown

### Manual Error Capture
1. Developer calls `captureError()` directly
2. Error immediately sent to APM server
3. No automatic re-throwing
4. Developer controls error flow

### Transaction Error Handling
1. Error occurs within transaction
2. Error captured with transaction context
3. Transaction continues or ends based on business logic
4. Error data included in transaction

## Performance Considerations

- **State Transitions**: Minimal overhead for state changes
- **Error Capture**: Asynchronous to avoid blocking
- **Transaction Management**: Automatic cleanup on errors
- **Span Management**: Parent-child relationship preservation
- **Context Preservation**: Maintained across state transitions
