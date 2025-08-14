# API Documentation

This document describes the public API interfaces provided by the `@wellcare/nestjs-elastic-apm` module.

## ApmService

The main service class that provides APM functionality.

### Constructor

```typescript
constructor()
```

Creates a new instance of ApmService. No parameters required.

### Methods

#### captureError

Captures and reports errors to the APM server.

```typescript
captureError(data: any): void
```

**Parameters:**
- `data` (any): Error object, string, or any data to capture

**Example:**
```typescript
@Injectable()
export class UserService {
  constructor(private readonly apmService: ApmService) {}

  async getUser(id: string) {
    try {
      return await this.userRepository.findById(id);
    } catch (error) {
      this.apmService.captureError(error);
      throw error;
    }
  }
}
```

#### startTransaction

Starts a new custom transaction.

```typescript
startTransaction(
  name?: string, 
  options?: apm.TransactionOptions
): apm.Transaction | null
```

**Parameters:**
- `name` (string, optional): Transaction name
- `options` (TransactionOptions, optional): Transaction configuration options

**Returns:**
- `apm.Transaction | null`: Transaction object or null if failed

**Example:**
```typescript
@Injectable()
export class OrderService {
  constructor(private readonly apmService: ApmService) {}

  async processOrder(orderData: any) {
    const transaction = this.apmService.startTransaction('Process Order');
    
    try {
      // Business logic here
      return await this.processOrderLogic(orderData);
    } finally {
      transaction?.end();
    }
  }
}
```

#### setTransactionName

Sets the name of the current transaction.

```typescript
setTransactionName(name: string): void
```

**Parameters:**
- `name` (string): New transaction name

**Example:**
```typescript
@Injectable()
export class PaymentService {
  constructor(private readonly apmService: ApmService) {}

  async processPayment(paymentData: any) {
    this.apmService.setTransactionName(`Payment ${paymentData.id}`);
    // Process payment logic
  }
}
```

#### startSpan

Starts a new span within the current transaction.

```typescript
startSpan(
  name?: string, 
  options?: apm.SpanOptions
): apm.Span | null
```

**Parameters:**
- `name` (string, optional): Span name
- `options` (SpanOptions, optional): Span configuration options

**Returns:**
- `apm.Span | null`: Span object or null if failed

**Example:**
```typescript
@Injectable()
export class DatabaseService {
  constructor(private readonly apmService: ApmService) {}

  async query(sql: string) {
    const span = this.apmService.startSpan('Database Query');
    
    try {
      return await this.database.execute(sql);
    } finally {
      span?.end();
    }
  }
}
```

#### setCustomContext

Sets custom context data for the current transaction.

```typescript
setCustomContext(context: Record<string, unknown>): void
```

**Parameters:**
- `context` (Record<string, unknown>): Custom context object

**Example:**
```typescript
@Injectable()
export class UserService {
  constructor(private readonly apmService: ApmService) {}

  async getUserProfile(userId: string) {
    this.apmService.setCustomContext({
      userId: userId,
      operation: 'getUserProfile',
      timestamp: new Date().toISOString()
    });
    
    // Business logic here
  }
}
```

#### currentTraceparent

Gets the current trace parent string.

```typescript
currentTraceparent(): string | null
```

**Returns:**
- `string | null`: Current trace parent string or null

**Example:**
```typescript
@Injectable()
export class LoggingService {
  constructor(private readonly apmService: ApmService) {}

  log(message: string) {
    const traceparent = this.apmService.currentTraceparent();
    console.log(`[${traceparent}] ${message}`);
  }
}
```

## ApmModule

The NestJS module that provides APM functionality.

### Static Methods

#### register

Registers the APM module with NestJS.

```typescript
static register(): DynamicModule
```

**Returns:**
- `DynamicModule`: NestJS dynamic module configuration

**Example:**
```typescript
import { Module } from '@nestjs/common';
import { ApmModule } from 'nestjs-elastic-apm';

@Module({
  imports: [
    ApmModule.register(),
    // Other modules...
  ],
})
export class AppModule {}
```

## ApmInterceptor

Automatic interceptor that captures HTTP exceptions.

### Constructor

```typescript
constructor(private readonly apmService: ApmService)
```

**Parameters:**
- `apmService` (ApmService): Instance of ApmService

### Methods

#### intercept

Intercepts HTTP requests and captures errors.

```typescript
intercept(
  context: ExecutionContext, 
  next: CallHandler
): Observable<Response>
```

**Parameters:**
- `context` (ExecutionContext): NestJS execution context
- `next` (CallHandler): Next handler in the chain

**Returns:**
- `Observable<Response>`: Observable of the response

**Example:**
```typescript
// Automatically registered when using ApmModule.register()
// No manual usage required
```

## Configuration

### Environment Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `ELASTIC_APM_ACTIVATE` | string | - | Set to 'true' to enable APM |
| `ELASTIC_APM_SERVICE_NAME` | string | package.json name | Service name |
| `ELASTIC_APM_SECRET_TOKEN` | string | - | APM Server secret token |
| `ELASTIC_APM_API_KEY` | string | - | APM Server API key |
| `ELASTIC_APM_SERVER_URL` | string | http://localhost:8200 | APM Server URL |
| `ELASTIC_APM_DISABLE_INSTRUMENTATIONS` | string | - | Comma-separated list of disabled instrumentations |

### Example Configuration

```bash
# .env file
ELASTIC_APM_ACTIVATE=true
ELASTIC_APM_SERVICE_NAME=my-nestjs-app
ELASTIC_APM_SECRET_TOKEN=your-secret-token
ELASTIC_APM_SERVER_URL=http://apm-server:8200
ELASTIC_APM_DISABLE_INSTRUMENTATIONS=http,https
```

## Error Handling

The module provides automatic error capture through the `ApmInterceptor`. Errors are categorized as:

### HttpException
- Captures the error message
- Preserves HTTP status codes
- Maintains request context

### Generic Errors
- Captures the full error object
- Includes stack traces
- Preserves error context

## Performance Monitoring

### Automatic Instrumentation
- HTTP requests and responses
- Database queries (if supported)
- External HTTP calls
- Express.js middleware

### Custom Instrumentation
- Custom transactions
- Custom spans
- Custom context
- Manual error capture

## Type Definitions

### TransactionOptions
```typescript
interface TransactionOptions {
  type?: string;
  subtype?: string;
  action?: string;
  timestamp?: number;
  duration?: number;
  result?: string;
  context?: any;
}
```

### SpanOptions
```typescript
interface SpanOptions {
  type?: string;
  subtype?: string;
  action?: string;
  timestamp?: number;
  duration?: number;
  context?: any;
}
```

## Best Practices

1. **Always end spans and transactions** in finally blocks
2. **Use descriptive names** for transactions and spans
3. **Set custom context** for better debugging
4. **Handle errors appropriately** before capturing them
5. **Use environment variables** for configuration
6. **Monitor APM server connectivity** in production
