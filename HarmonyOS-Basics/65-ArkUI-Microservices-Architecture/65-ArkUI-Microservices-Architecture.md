# ArkUI Microservices Architecture

## Introduction

Microservices architecture enables building scalable, maintainable applications by decomposing them into small, independent services. This guide explores microservices patterns, service communication, and distributed system management in ArkUI applications.

## Service Architecture Patterns

### Service Registry and Discovery

```typescript
interface ServiceInfo {
  id: string;
  name: string;
  version: string;
  endpoint: string;
  health: "healthy" | "unhealthy";
  metadata: Record<string, any>;
}

class ServiceRegistry {
  private services = new Map<string, ServiceInfo>();
  private healthChecks = new Map<string, number>();

  register(service: ServiceInfo): void {
    this.services.set(service.id, service);
    this.startHealthCheck(service);
  }

  unregister(serviceId: string): void {
    this.services.delete(serviceId);
    this.stopHealthCheck(serviceId);
  }

  discover(serviceName: string): ServiceInfo[] {
    return Array.from(this.services.values()).filter(
      (service) => service.name === serviceName && service.health === "healthy"
    );
  }

  getService(serviceId: string): ServiceInfo | undefined {
    return this.services.get(serviceId);
  }

  private startHealthCheck(service: ServiceInfo): void {
    const intervalId = setInterval(async () => {
      try {
        const response = await fetch(`${service.endpoint}/health`);
        service.health = response.ok ? "healthy" : "unhealthy";
      } catch {
        service.health = "unhealthy";
      }
    }, 30000); // Check every 30 seconds

    this.healthChecks.set(service.id, intervalId);
  }

  private stopHealthCheck(serviceId: string): void {
    const intervalId = this.healthChecks.get(serviceId);
    if (intervalId) {
      clearInterval(intervalId);
      this.healthChecks.delete(serviceId);
    }
  }
}

// Service discovery client
class ServiceDiscovery {
  constructor(private registry: ServiceRegistry) {}

  async getService(serviceName: string): Promise<ServiceInfo> {
    const services = this.registry.discover(serviceName);
    if (services.length === 0) {
      throw new Error(`No healthy services found for: ${serviceName}`);
    }

    // Simple round-robin load balancing
    const service = services[Math.floor(Math.random() * services.length)];
    return service;
  }

  async callService<T>(
    serviceName: string,
    path: string,
    options: RequestInit = {}
  ): Promise<T> {
    const service = await this.getService(serviceName);
    const url = `${service.endpoint}${path}`;

    const response = await fetch(url, {
      ...options,
      headers: {
        "Content-Type": "application/json",
        "Service-Version": service.version,
        ...options.headers,
      },
    });

    if (!response.ok) {
      throw new Error(`Service call failed: ${response.status}`);
    }

    return response.json();
  }
}
```

### API Gateway Pattern

```typescript
interface Route {
  path: string
  method: string
  service: string
  endpoint: string
  middleware: Middleware[]
}

interface Middleware {
  name: string
  execute(request: Request, response: Response, next: () => void): Promise<void>
}

class APIGateway {
  private routes: Route[] = []
  private middleware: Middleware[] = []
  private serviceDiscovery: ServiceDiscovery

  constructor(serviceDiscovery: ServiceDiscovery) {
    this.serviceDiscovery = serviceDiscovery
  }

  addRoute(route: Route): void {
    this.routes.push(route)
  }

  use(middleware: Middleware): void {
    this.middleware.push(middleware)
  }

  async handleRequest(request: Request): Promise<Response> {
    const route = this.findRoute(request)
    if (!route) {
      return new Response('Not Found', { status: 404 })
    }

    try {
      // Execute middleware chain
      await this.executeMiddleware([...this.middleware, ...route.middleware], request)

      // Forward request to service
      const serviceResponse = await this.forwardRequest(route, request)
      return serviceResponse
    } catch (error) {
      console.error('Gateway error:', error)
      return new Response('Internal Server Error', { status: 500 })
    }
  }

  private findRoute(request: Request): Route | undefined {
    const url = new URL(request.url)
    return this.routes.find(route =>
      route.path === url.pathname && route.method === request.method
    )
  }

  private async executeMiddleware(
    middleware: Middleware[],
    request: Request
  ): Promise<void> {
    let index = 0

    const next = async () => {
      if (index < middleware.length) {
        const currentMiddleware = middleware[index++]
        await currentMiddleware.execute(request, new Response(), next)
      }
    }

    await next()
  }

  private async forwardRequest(route: Route, request: Request): Promise<Response> {
    const service = await this.serviceDiscovery.getService(route.service)
    const url = `${service.endpoint}${route.endpoint}`

    return fetch(url, {
      method: request.method,
      headers: request.headers,
      body: request.body
    })
  }
}

// Authentication middleware
class AuthMiddleware implements Middleware {
  name = 'auth'

  async execute(request: Request, response: Response, next: () => void): Promise<void> {
    const token = request.headers.get('Authorization')

    if (!token) {
      throw new Error('No authorization token')
    }

    try {
      const decoded = await this.verifyToken(token)
      // Add user info to request
      (request as any).user = decoded
      next()
    } catch {
      throw new Error('Invalid token')
    }
  }

  private async verifyToken(token: string): Promise<any> {
    // JWT verification logic
    return { userId: '123', role: 'user' }
  }
}

// Rate limiting middleware
class RateLimitMiddleware implements Middleware {
  name = 'rateLimit'
  private requests = new Map<string, number[]>()
  private limit = 100 // requests per minute
  private window = 60000 // 1 minute

  async execute(request: Request, response: Response, next: () => void): Promise<void> {
    const clientId = this.getClientId(request)
    const now = Date.now()

    if (!this.requests.has(clientId)) {
      this.requests.set(clientId, [])
    }

    const timestamps = this.requests.get(clientId)!
    const validTimestamps = timestamps.filter(t => now - t < this.window)

    if (validTimestamps.length >= this.limit) {
      throw new Error('Rate limit exceeded')
    }

    validTimestamps.push(now)
    this.requests.set(clientId, validTimestamps)

    next()
  }

  private getClientId(request: Request): string {
    return request.headers.get('x-forwarded-for') || 'unknown'
  }
}
```

## Service Communication

### Message Bus Implementation

```typescript
interface Message {
  id: string;
  type: string;
  payload: any;
  timestamp: Date;
  source: string;
  correlationId?: string;
}

interface MessageHandler {
  handle(message: Message): Promise<void>;
}

class MessageBus {
  private handlers = new Map<string, MessageHandler[]>();
  private deadLetterQueue: Message[] = [];
  private retryPolicy = { maxRetries: 3, delay: 1000 };

  subscribe(messageType: string, handler: MessageHandler): () => void {
    if (!this.handlers.has(messageType)) {
      this.handlers.set(messageType, []);
    }

    this.handlers.get(messageType)!.push(handler);

    return () => {
      const handlers = this.handlers.get(messageType);
      if (handlers) {
        const index = handlers.indexOf(handler);
        if (index >= 0) {
          handlers.splice(index, 1);
        }
      }
    };
  }

  async publish(message: Message): Promise<void> {
    const handlers = this.handlers.get(message.type) || [];

    const promises = handlers.map((handler) =>
      this.executeWithRetry(handler, message)
    );

    await Promise.allSettled(promises);
  }

  private async executeWithRetry(
    handler: MessageHandler,
    message: Message,
    attempt = 1
  ): Promise<void> {
    try {
      await handler.handle(message);
    } catch (error) {
      console.error(`Message handling failed (attempt ${attempt}):`, error);

      if (attempt < this.retryPolicy.maxRetries) {
        await this.delay(this.retryPolicy.delay * attempt);
        return this.executeWithRetry(handler, message, attempt + 1);
      } else {
        this.deadLetterQueue.push(message);
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  getDeadLetterQueue(): Message[] {
    return [...this.deadLetterQueue];
  }
}

// Event sourcing implementation
class EventStore {
  private events: Message[] = [];
  private snapshots = new Map<string, any>();

  async saveEvent(event: Message): Promise<void> {
    this.events.push(event);
    await this.notifySubscribers(event);
  }

  async getEvents(
    aggregateId: string,
    fromVersion?: number
  ): Promise<Message[]> {
    return this.events
      .filter((event) => event.correlationId === aggregateId)
      .slice(fromVersion || 0);
  }

  async saveSnapshot(
    aggregateId: string,
    version: number,
    data: any
  ): Promise<void> {
    this.snapshots.set(`${aggregateId}:${version}`, data);
  }

  async getSnapshot(aggregateId: string): Promise<any> {
    const entries = Array.from(this.snapshots.entries())
      .filter(([key]) => key.startsWith(aggregateId))
      .sort(([a], [b]) => b.localeCompare(a));

    return entries[0]?.[1];
  }

  private async notifySubscribers(event: Message): Promise<void> {
    // Notify message bus or other subscribers
  }
}
```

### Circuit Breaker Pattern

```typescript
enum CircuitState {
  Closed = "closed",
  Open = "open",
  HalfOpen = "half-open",
}

interface CircuitBreakerConfig {
  failureThreshold: number;
  resetTimeout: number;
  monitoringPeriod: number;
}

class CircuitBreaker {
  private state = CircuitState.Closed;
  private failureCount = 0;
  private lastFailureTime = 0;
  private successes = 0;

  constructor(private config: CircuitBreakerConfig) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.Open) {
      if (this.shouldAttemptReset()) {
        this.state = CircuitState.HalfOpen;
      } else {
        throw new Error("Circuit breaker is OPEN");
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failureCount = 0;

    if (this.state === CircuitState.HalfOpen) {
      this.successes++;
      if (this.successes >= 3) {
        // Require 3 successes to close
        this.state = CircuitState.Closed;
        this.successes = 0;
      }
    }
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.config.failureThreshold) {
      this.state = CircuitState.Open;
    }
  }

  private shouldAttemptReset(): boolean {
    return Date.now() - this.lastFailureTime >= this.config.resetTimeout;
  }

  getState(): CircuitState {
    return this.state;
  }
}

// Service client with circuit breaker
class ServiceClient {
  private circuitBreaker: CircuitBreaker;

  constructor(
    private serviceName: string,
    private discovery: ServiceDiscovery
  ) {
    this.circuitBreaker = new CircuitBreaker({
      failureThreshold: 5,
      resetTimeout: 60000,
      monitoringPeriod: 10000,
    });
  }

  async call<T>(path: string, options: RequestInit = {}): Promise<T> {
    return this.circuitBreaker.execute(async () => {
      return this.discovery.callService<T>(this.serviceName, path, options);
    });
  }

  getCircuitState(): CircuitState {
    return this.circuitBreaker.getState();
  }
}
```

## Distributed System Patterns

### Saga Pattern

```typescript
interface SagaStep {
  execute(): Promise<any>;
  compensate(): Promise<void>;
}

class Saga {
  private steps: SagaStep[] = [];
  private executedSteps: SagaStep[] = [];

  addStep(step: SagaStep): Saga {
    this.steps.push(step);
    return this;
  }

  async execute(): Promise<void> {
    try {
      for (const step of this.steps) {
        await step.execute();
        this.executedSteps.push(step);
      }
    } catch (error) {
      await this.compensate();
      throw error;
    }
  }

  private async compensate(): Promise<void> {
    for (const step of this.executedSteps.reverse()) {
      try {
        await step.compensate();
      } catch (error) {
        console.error("Compensation failed:", error);
      }
    }
  }
}

// Example: Order processing saga
class CreateOrderStep implements SagaStep {
  constructor(private orderData: any) {}

  async execute(): Promise<any> {
    // Create order
    const order = await orderService.create(this.orderData);
    this.orderId = order.id;
    return order;
  }

  async compensate(): Promise<void> {
    if (this.orderId) {
      await orderService.cancel(this.orderId);
    }
  }

  private orderId?: string;
}

class ReserveInventoryStep implements SagaStep {
  constructor(private items: any[]) {}

  async execute(): Promise<any> {
    this.reservationId = await inventoryService.reserve(this.items);
    return this.reservationId;
  }

  async compensate(): Promise<void> {
    if (this.reservationId) {
      await inventoryService.release(this.reservationId);
    }
  }

  private reservationId?: string;
}

class ProcessPaymentStep implements SagaStep {
  constructor(private paymentData: any) {}

  async execute(): Promise<any> {
    this.transactionId = await paymentService.charge(this.paymentData);
    return this.transactionId;
  }

  async compensate(): Promise<void> {
    if (this.transactionId) {
      await paymentService.refund(this.transactionId);
    }
  }

  private transactionId?: string;
}

// Usage
async function processOrder(orderData: any): Promise<void> {
  const saga = new Saga()
    .addStep(new CreateOrderStep(orderData))
    .addStep(new ReserveInventoryStep(orderData.items))
    .addStep(new ProcessPaymentStep(orderData.payment));

  await saga.execute();
}
```

### Microservice Component

```typescript
@Component
struct MicroserviceApp {
  @State private services: ServiceInfo[] = []
  @State private selectedService: ServiceInfo | null = null
  private serviceRegistry = new ServiceRegistry()
  private discovery = new ServiceDiscovery(this.serviceRegistry)

  build() {
    Column() {
      this.buildServiceList()
      this.buildServiceDetails()
    }
    .padding(16)
  }

  aboutToAppear() {
    this.initializeServices()
  }

  @Builder
  private buildServiceList() {
    Text('Available Services')
      .fontSize(20)
      .fontWeight(FontWeight.Bold)
      .margin({ bottom: 16 })

    List() {
      ForEach(this.services, (service: ServiceInfo) => {
        ListItem() {
          Row() {
            Column() {
              Text(service.name)
                .fontSize(16)
                .fontWeight(FontWeight.Medium)

              Text(service.endpoint)
                .fontSize(12)
                .fontColor('#666')
            }
            .alignItems(HorizontalAlign.Start)
            .flexGrow(1)

            Text(service.health)
              .fontSize(12)
              .fontColor(service.health === 'healthy' ? '#4CAF50' : '#F44336')
              .backgroundColor(service.health === 'healthy' ? '#E8F5E8' : '#FFEBEE')
              .padding({ horizontal: 8, vertical: 4 })
              .borderRadius(4)
          }
          .padding(12)
          .backgroundColor('#f5f5f5')
          .borderRadius(8)
          .onClick(() => this.selectedService = service)
        }
        .margin({ bottom: 8 })
      })
    }
  }

  @Builder
  private buildServiceDetails() {
    if (this.selectedService) {
      Column() {
        Text('Service Details')
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .margin({ top: 24, bottom: 16 })

        this.buildDetailRow('Name', this.selectedService.name)
        this.buildDetailRow('Version', this.selectedService.version)
        this.buildDetailRow('Endpoint', this.selectedService.endpoint)
        this.buildDetailRow('Status', this.selectedService.health)

        Button('Test Service')
          .margin({ top: 16 })
          .onClick(() => this.testService())
      }
    }
  }

  @Builder
  private buildDetailRow(label: string, value: string) {
    Row() {
      Text(label)
        .fontSize(14)
        .fontWeight(FontWeight.Medium)
        .width(80)

      Text(value)
        .fontSize(14)
        .flexGrow(1)
    }
    .margin({ bottom: 8 })
  }

  private async initializeServices(): Promise<void> {
    // Register sample services
    this.serviceRegistry.register({
      id: 'user-service-1',
      name: 'user-service',
      version: '1.0.0',
      endpoint: 'http://localhost:3001',
      health: 'healthy',
      metadata: { region: 'us-east-1' }
    })

    this.serviceRegistry.register({
      id: 'order-service-1',
      name: 'order-service',
      version: '1.2.0',
      endpoint: 'http://localhost:3002',
      health: 'healthy',
      metadata: { region: 'us-east-1' }
    })

    this.services = Array.from(this.serviceRegistry['services'].values())
  }

  private async testService(): Promise<void> {
    if (!this.selectedService) return

    try {
      const response = await this.discovery.callService(
        this.selectedService.name,
        '/health'
      )
      console.log('Service response:', response)
    } catch (error) {
      console.error('Service test failed:', error)
    }
  }
}
```

## Conclusion

Microservices architecture in ArkUI enables:

- Service discovery and registry management
- API gateway routing and middleware
- Circuit breaker resilience patterns
- Event-driven communication via message bus
- Distributed transaction management with sagas
- Scalable, maintainable service architectures

These patterns provide the foundation for building robust distributed systems with independent, scalable microservices.
