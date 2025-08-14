# @wellcare/nestjs-elastic-apm

A NestJS module for seamless integration with Elastic APM (Application Performance Monitoring). This module provides automatic instrumentation, error tracking, and performance monitoring for NestJS applications.

## 🚀 Features

- **Automatic Instrumentation**: Automatically tracks HTTP requests, database queries, and external service calls
- **Error Tracking**: Captures and reports errors with full stack traces
- **Custom Spans**: Create custom performance spans for business logic
- **Transaction Management**: Automatic transaction naming and custom transaction support
- **Context Management**: Set custom context for better debugging
- **Interceptor Integration**: Built-in interceptor for automatic error capture
- **Environment Configuration**: Flexible configuration through environment variables

## 🛠 Tech Stack

- **NestJS**: Modern Node.js framework for building scalable applications
- **Elastic APM Node.js Agent**: Official Elastic APM agent for Node.js
- **TypeScript**: Type-safe JavaScript development
- **RxJS**: Reactive programming for handling observables

## 📦 Installation

```bash
npm install @wellcare/nestjs-elastic-apm
```

## 🔧 Configuration

### 1. Initialize APM Agent (main.ts)

```typescript
import * as dotenv from 'dotenv';
dotenv.config();

import apm from 'nestjs-elastic-apm';
// ... rest of your main.ts code
```

### 2. Register Module (app.module.ts)

```typescript
import { Module } from '@nestjs/common';
import { ApmModule } from 'nestjs-elastic-apm';

@Module({
  imports: [
    ApmModule.register(),
    // ... other modules
  ],
})
export class AppModule {}
```

### 3. Environment Variables

```bash
# Enable/disable APM agent
ELASTIC_APM_ACTIVATE=true

# Override service name from package.json
ELASTIC_APM_SERVICE_NAME=my-service

# APM Server authentication
ELASTIC_APM_SECRET_TOKEN=your-secret-token
ELASTIC_APM_API_KEY=your-api-key

# APM Server URL (default: http://localhost:8200)
ELASTIC_APM_SERVER_URL=http://your-apm-server:8200

# Disable specific instrumentations
ELASTIC_APM_DISABLE_INSTRUMENTATIONS=http,https,express
```

## 📁 Folder Structure

```
nestjs-elastic-apm/
├── lib/                    # Source code
│   ├── index.ts           # Main exports
│   ├── start.ts           # APM agent initialization
│   ├── apm.module.ts      # NestJS module definition
│   ├── apm.service.ts     # APM service wrapper
│   └── apm.interceptor.ts # Error interceptor
├── dist/                  # Compiled JavaScript (generated)
├── package.json           # Package configuration
├── tsconfig.json          # TypeScript configuration
└── README.md             # This file
```

## 🔑 Key Components

### ApmModule
The main NestJS module that registers the APM service and interceptor.

### ApmService
Service wrapper around the Elastic APM agent providing:
- Error capture
- Transaction management
- Span creation
- Custom context setting
- Trace parent access

### ApmInterceptor
Automatic interceptor that captures HTTP exceptions and reports them to APM.

## 💻 Usage Examples

### Basic Service Usage

```typescript
import { Injectable } from '@nestjs/common';
import { ApmService } from 'nestjs-elastic-apm';

@Injectable()
export class UserService {
  constructor(private readonly apmService: ApmService) {}

  async getUser(id: string) {
    // Start a custom span
    const span = this.apmService.startSpan('getUser');
    
    try {
      // Your business logic here
      const user = await this.userRepository.findById(id);
      
      // Set custom context
      this.apmService.setCustomContext({
        userId: id,
        userRole: user.role
      });
      
      return user;
    } finally {
      span?.end();
    }
  }
}
```

### Custom Transaction

```typescript
import { Injectable } from '@nestjs/common';
import { ApmService } from 'nestjs-elastic-apm';

@Injectable()
export class OrderService {
  constructor(private readonly apmService: ApmService) {}

  async processOrder(orderData: any) {
    // Start a custom transaction
    const transaction = this.apmService.startTransaction('Process Order');
    
    try {
      // Process order steps
      await this.validateOrder(orderData);
      await this.processPayment(orderData);
      await this.sendConfirmation(orderData);
      
      return { success: true };
    } catch (error) {
      // Capture error
      this.apmService.captureError(error);
      throw error;
    } finally {
      transaction?.end();
    }
  }
}
```

### Error Handling

```typescript
import { Injectable, HttpException } from '@nestjs/common';
import { ApmService } from 'nestjs-elastic-apm';

@Injectable()
export class PaymentService {
  constructor(private readonly apmService: ApmService) {}

  async processPayment(paymentData: any) {
    try {
      // Payment processing logic
      const result = await this.paymentGateway.process(paymentData);
      return result;
    } catch (error) {
      // Custom error context
      this.apmService.setCustomContext({
        paymentId: paymentData.id,
        amount: paymentData.amount,
        currency: paymentData.currency
      });
      
      // Capture and re-throw
      this.apmService.captureError(error);
      throw new HttpException('Payment failed', 400);
    }
  }
}
```

## 🔍 Monitoring

Once configured, your application will automatically send performance data to your Elastic APM server. You can monitor:

- **Transactions**: HTTP requests, custom transactions
- **Spans**: Database queries, external API calls, custom spans
- **Errors**: Exceptions with stack traces and context
- **Metrics**: Response times, throughput, error rates

## 📚 API Reference

### ApmService Methods

- `captureError(data: any)`: Capture and report errors
- `startTransaction(name?: string, options?: TransactionOptions)`: Start a custom transaction
- `setTransactionName(name: string)`: Set the current transaction name
- `startSpan(name?: string, options?: SpanOptions)`: Start a custom span
- `setCustomContext(context: Record<string, unknown>)`: Set custom context
- `currentTraceparent(): string | null`: Get current trace parent

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Links

- [Elastic APM Documentation](https://www.elastic.co/guide/en/apm/agent/nodejs/current/index.html)
- [NestJS Documentation](https://nestjs.com/)
- [GitHub Repository](https://github.com/Wellcare/nestjs-elastic-apm)
