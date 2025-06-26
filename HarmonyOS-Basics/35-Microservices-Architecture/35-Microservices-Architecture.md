# 35. Microservices Architecture

## Introduction

Microservices architecture provides a scalable approach to building complex HarmonyOS applications by decomposing them into smaller, independent services. This article explores implementing microservices patterns, service communication, and distributed system design in HarmonyOS applications.

## Service Design Patterns

### Service Registry and Discovery

```typescript
export interface ServiceInfo {
  id: string;
  name: string;
  version: string;
  endpoint: string;
  port: number;
  health: ServiceHealth;
  metadata: Record<string, any>;
  timestamp: number;
}

export enum ServiceHealth {
  HEALTHY = "healthy",
  UNHEALTHY = "unhealthy",
  UNKNOWN = "unknown",
}

export class ServiceRegistry {
  private static instance: ServiceRegistry;
  private services: Map<string, ServiceInfo> = new Map();
  private healthCheckers: Map<string, HealthChecker> = new Map();

  static getInstance(): ServiceRegistry {
    if (!this.instance) {
      this.instance = new ServiceRegistry();
    }
    return this.instance;
  }

  async registerService(
    service: Omit<ServiceInfo, "timestamp">
  ): Promise<void> {
    const serviceInfo: ServiceInfo = {
      ...service,
      timestamp: Date.now(),
    };

    this.services.set(service.id, serviceInfo);

    // Start health checking
    const healthChecker = new HealthChecker(serviceInfo);
    this.healthCheckers.set(service.id, healthChecker);
    await healthChecker.start();

    console.log(`Service registered: ${service.name} (${service.id})`);
  }

  async unregisterService(serviceId: string): Promise<void> {
    const service = this.services.get(serviceId);
    if (service) {
      this.services.delete(serviceId);

      // Stop health checking
      const healthChecker = this.healthCheckers.get(serviceId);
      if (healthChecker) {
        await healthChecker.stop();
        this.healthCheckers.delete(serviceId);
      }

      console.log(`Service unregistered: ${service.name} (${serviceId})`);
    }
  }

  discoverServices(serviceName?: string): ServiceInfo[] {
    const allServices = Array.from(this.services.values());

    if (serviceName) {
      return allServices.filter(
        (service) =>
          service.name === serviceName &&
          service.health === ServiceHealth.HEALTHY
      );
    }

    return allServices.filter(
      (service) => service.health === ServiceHealth.HEALTHY
    );
  }

  async getService(serviceName: string): Promise<ServiceInfo | null> {
    const services = this.discoverServices(serviceName);

    if (services.length === 0) {
      return null;
    }

    // Load balancing - round robin
    const randomIndex = Math.floor(Math.random() * services.length);
    return services[randomIndex];
  }

  updateServiceHealth(serviceId: string, health: ServiceHealth): void {
    const service = this.services.get(serviceId);
    if (service) {
      service.health = health;
      service.timestamp = Date.now();
    }
  }

  getServicesByHealth(health: ServiceHealth): ServiceInfo[] {
    return Array.from(this.services.values()).filter(
      (service) => service.health === health
    );
  }

  getAllServices(): ServiceInfo[] {
    return Array.from(this.services.values());
  }
}

export class HealthChecker {
  private service: ServiceInfo;
  private intervalId: number = -1;
  private checkInterval: number = 30000; // 30 seconds

  constructor(service: ServiceInfo) {
    this.service = service;
  }

  async start(): Promise<void> {
    this.intervalId = setInterval(async () => {
      await this.performHealthCheck();
    }, this.checkInterval);

    // Initial health check
    await this.performHealthCheck();
  }

  async stop(): Promise<void> {
    if (this.intervalId !== -1) {
      clearInterval(this.intervalId);
      this.intervalId = -1;
    }
  }

  private async performHealthCheck(): Promise<void> {
    try {
      const healthEndpoint = `${this.service.endpoint}/health`;
      const response = await fetch(healthEndpoint, {
        method: "GET",
        timeout: 5000,
      });

      if (response.ok) {
        ServiceRegistry.getInstance().updateServiceHealth(
          this.service.id,
          ServiceHealth.HEALTHY
        );
      } else {
        ServiceRegistry.getInstance().updateServiceHealth(
          this.service.id,
          ServiceHealth.UNHEALTHY
        );
      }
    } catch (error) {
      console.error(`Health check failed for ${this.service.name}:`, error);
      ServiceRegistry.getInstance().updateServiceHealth(
        this.service.id,
        ServiceHealth.UNHEALTHY
      );
    }
  }
}
```

### API Gateway Implementation

```typescript
export interface Route {
  path: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
  serviceName: string;
  targetPath?: string;
  middleware?: Middleware[];
  rateLimit?: RateLimitConfig;
  authentication?: AuthConfig;
}

export interface Middleware {
  name: string;
  execute: (request: RequestContext) => Promise<RequestContext>;
}

export interface RequestContext {
  request: Request;
  response?: Response;
  params: Record<string, any>;
  user?: UserInfo;
  metadata: Record<string, any>;
}

export interface RateLimitConfig {
  windowMs: number;
  maxRequests: number;
  skipSuccessfulRequests?: boolean;
}

export interface AuthConfig {
  required: boolean;
  roles?: string[];
  permissions?: string[];
}

export class APIGateway {
  private routes: Map<string, Route> = new Map();
  private serviceRegistry: ServiceRegistry;
  private rateLimiter: RateLimiter;
  private authService: AuthenticationService;

  constructor() {
    this.serviceRegistry = ServiceRegistry.getInstance();
    this.rateLimiter = new RateLimiter();
    this.authService = new AuthenticationService();
  }

  registerRoute(route: Route): void {
    const routeKey = `${route.method}:${route.path}`;
    this.routes.set(routeKey, route);
    console.log(`Route registered: ${routeKey} -> ${route.serviceName}`);
  }

  async handleRequest(request: Request): Promise<Response> {
    const routeKey = `${request.method}:${request.url}`;
    const route = this.findMatchingRoute(request);

    if (!route) {
      return new Response('Route not found', { status: 404 });
    }

    const context: RequestContext = {
      request,
      params: this.extractParams(request, route),
      metadata: {}
    };

    try {
      // Execute middleware pipeline
      const processedContext = await this.executeMiddleware(context, route);

      // Rate limiting
      if (route.rateLimit) {
        const rateLimitResult = await this.rateLimiter.checkLimit(
          this.getClientId(request),
          route.rateLimit
        );

        if (!rateLimitResult.allowed) {
          return new Response('Rate limit exceeded', {
            status: 429,
            headers: {
              'Retry-After': rateLimitResult.retryAfter?.toString() || '60'
            }
          });
        }
      }

      // Authentication
      if (route.authentication?.required) {
        const authResult = await this.authService.authenticate(request);
        if (!authResult.success) {
          return new Response('Unauthorized', { status: 401 });
        }
        processedContext.user = authResult.user;
      }

      // Forward request to service
      const serviceResponse = await this.forwardToService(processedContext, route);
      return serviceResponse;

    } catch (error) {
      console.error('Request processing error:', error);
      return new Response('Internal Server Error', { status: 500 });
    }
  }

  private findMatchingRoute(request: Request): Route | null {
    const path = new URL(request.url).pathname;
    const method = request.method as Route['method'];

    for (const [routeKey, route] of this.routes) {
      if (route.method === method && this.pathMatches(path, route.path)) {
        return route;
      }
    }

    return null;
  }

  private pathMatches(requestPath: string, routePath: string): boolean {
    // Simple path matching - in production, use more sophisticated routing
    const requestSegments = requestPath.split('/').filter(s => s);
    const routeSegments = routePath.split('/').filter(s => s);

    if (requestSegments.length !== routeSegments.length) {
      return false;
    }

    for (let i = 0; i < routeSegments.length; i++) {
      const routeSegment = routeSegments[i];
      const requestSegment = requestSegments[i];

      if (routeSegment.startsWith(':')) {
        continue; // Parameter segment
      }

      if (routeSegment !== requestSegment) {
        return false;
      }
    }

    return true;
  }

  private extractParams(request: Request, route: Route): Record<string, any> {
    const params: Record<string, any> = {};
    const path = new URL(request.url).pathname;
    const requestSegments = path.split('/').filter(s => s);
    const routeSegments = route.path.split('/').filter(s => s);

    for (let i = 0; i < routeSegments.length; i++) {
      const routeSegment = routeSegments[i];
      if (routeSegment.startsWith(':')) {
        const paramName = routeSegment.substring(1);
        params[paramName] = requestSegments[i];
      }
    }

    return params;
  }

  private async executeMiddleware(
    context: RequestContext,
    route: Route
  ): Promise<RequestContext> {
    let currentContext = context;

    if (route.middleware) {
      for (const middleware of route.middleware) {
        currentContext = await middleware.execute(currentContext);
      }
    }

    return currentContext;
  }

  private async forwardToService(
    context: RequestContext,
    route: Route
  ): Promise<Response> {
    const service = await this.serviceRegistry.getService(route.serviceName);

    if (!service) {
      return new Response('Service unavailable', { status: 503 });
    }

    const targetPath = route.targetPath || context.request.url;
    const serviceUrl = `${service.endpoint}${targetPath}`;

    try {
      const response = await fetch(serviceUrl, {
        method: context.request.method,
        headers: context.request.headers,
        body: context.request.body
      });

      return response;
    } catch (error) {
      console.error(`Service call failed: ${route.serviceName}`, error);
      return new Response('Service error', { status: 502 });
    }
  }

  private getClientId(request: Request): string {
    // Extract client identifier (IP, user ID, API key, etc.)
    return request.headers.get('X-Client-IP') || 'anonymous';
  }
}

export class RateLimiter {
  private limits: Map<string, LimitInfo> = new Map();

  interface LimitInfo {
    requests: number;
    windowStart: number;
    windowMs: number;
    maxRequests: number;
  }

  async checkLimit(
    clientId: string,
    config: RateLimitConfig
  ): Promise<{ allowed: boolean; retryAfter?: number }> {
    const key = `${clientId}:${config.windowMs}:${config.maxRequests}`;
    const now = Date.now();
    const limitInfo = this.limits.get(key);

    if (!limitInfo || now - limitInfo.windowStart >= config.windowMs) {
      // New window
      this.limits.set(key, {
        requests: 1,
        windowStart: now,
        windowMs: config.windowMs,
        maxRequests: config.maxRequests
      });
      return { allowed: true };
    }

    if (limitInfo.requests >= config.maxRequests) {
      const retryAfter = Math.ceil(
        (limitInfo.windowStart + config.windowMs - now) / 1000
      );
      return { allowed: false, retryAfter };
    }

    limitInfo.requests++;
    return { allowed: true };
  }

  cleanup(): void {
    const now = Date.now();
    for (const [key, limitInfo] of this.limits) {
      if (now - limitInfo.windowStart >= limitInfo.windowMs) {
        this.limits.delete(key);
      }
    }
  }
}
```

## Service Communication Patterns

### Message Queue System

```typescript
export interface Message {
  id: string;
  topic: string;
  payload: any;
  timestamp: number;
  priority: MessagePriority;
  retryCount: number;
  maxRetries: number;
}

export enum MessagePriority {
  HIGH = 1,
  MEDIUM = 2,
  LOW = 3,
}

export interface MessageHandler {
  handle(message: Message): Promise<void>;
  onError?(error: Error, message: Message): Promise<void>;
}

export class MessageQueue {
  private static instance: MessageQueue;
  private queues: Map<string, Message[]> = new Map();
  private subscribers: Map<string, MessageHandler[]> = new Map();
  private processing: Map<string, boolean> = new Map();
  private deadLetterQueue: Message[] = [];

  static getInstance(): MessageQueue {
    if (!this.instance) {
      this.instance = new MessageQueue();
    }
    return this.instance;
  }

  async publish(
    topic: string,
    payload: any,
    priority = MessagePriority.MEDIUM
  ): Promise<void> {
    const message: Message = {
      id: this.generateId(),
      topic,
      payload,
      timestamp: Date.now(),
      priority,
      retryCount: 0,
      maxRetries: 3,
    };

    await this.enqueue(topic, message);
    await this.processQueue(topic);
  }

  subscribe(topic: string, handler: MessageHandler): void {
    if (!this.subscribers.has(topic)) {
      this.subscribers.set(topic, []);
    }

    this.subscribers.get(topic)!.push(handler);
    console.log(`Subscribed to topic: ${topic}`);
  }

  unsubscribe(topic: string, handler: MessageHandler): void {
    const handlers = this.subscribers.get(topic);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index > -1) {
        handlers.splice(index, 1);
      }
    }
  }

  private async enqueue(topic: string, message: Message): Promise<void> {
    if (!this.queues.has(topic)) {
      this.queues.set(topic, []);
    }

    const queue = this.queues.get(topic)!;

    // Insert message based on priority
    let insertIndex = queue.length;
    for (let i = 0; i < queue.length; i++) {
      if (message.priority < queue[i].priority) {
        insertIndex = i;
        break;
      }
    }

    queue.splice(insertIndex, 0, message);
  }

  private async processQueue(topic: string): Promise<void> {
    if (this.processing.get(topic)) {
      return; // Already processing
    }

    this.processing.set(topic, true);

    try {
      const queue = this.queues.get(topic);
      const handlers = this.subscribers.get(topic);

      if (!queue || !handlers || queue.length === 0 || handlers.length === 0) {
        return;
      }

      while (queue.length > 0) {
        const message = queue.shift()!;

        for (const handler of handlers) {
          try {
            await handler.handle(message);
          } catch (error) {
            console.error(`Message handling error for topic ${topic}:`, error);

            if (handler.onError) {
              await handler.onError(error, message);
            }

            await this.handleFailedMessage(message, error);
          }
        }
      }
    } finally {
      this.processing.set(topic, false);
    }
  }

  private async handleFailedMessage(
    message: Message,
    error: Error
  ): Promise<void> {
    message.retryCount++;

    if (message.retryCount <= message.maxRetries) {
      // Retry with exponential backoff
      const delay = Math.pow(2, message.retryCount) * 1000;
      setTimeout(async () => {
        await this.enqueue(message.topic, message);
        await this.processQueue(message.topic);
      }, delay);
    } else {
      // Move to dead letter queue
      this.deadLetterQueue.push(message);
      console.error(`Message moved to dead letter queue: ${message.id}`);
    }
  }

  getDeadLetterMessages(): Message[] {
    return [...this.deadLetterQueue];
  }

  private generateId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  getQueueStats(): Record<string, { pending: number; processing: boolean }> {
    const stats: Record<string, { pending: number; processing: boolean }> = {};

    for (const [topic, queue] of this.queues) {
      stats[topic] = {
        pending: queue.length,
        processing: this.processing.get(topic) || false,
      };
    }

    return stats;
  }
}

// Event-driven service communication
export class EventBus {
  private static instance: EventBus;
  private messageQueue: MessageQueue;

  static getInstance(): EventBus {
    if (!this.instance) {
      this.instance = new EventBus();
    }
    return this.instance;
  }

  constructor() {
    this.messageQueue = MessageQueue.getInstance();
  }

  async emit(eventName: string, data: any): Promise<void> {
    await this.messageQueue.publish(`event:${eventName}`, {
      eventName,
      data,
      source: "EventBus",
      timestamp: Date.now(),
    });
  }

  on(eventName: string, handler: (data: any) => Promise<void>): void {
    this.messageQueue.subscribe(`event:${eventName}`, {
      handle: async (message) => {
        await handler(message.payload.data);
      },
    });
  }

  off(eventName: string, handler: (data: any) => Promise<void>): void {
    // Implementation would track handlers to remove them
  }
}
```

### Circuit Breaker Pattern

```typescript
export enum CircuitState {
  CLOSED = "closed",
  OPEN = "open",
  HALF_OPEN = "half_open",
}

export interface CircuitBreakerConfig {
  failureThreshold: number;
  resetTimeout: number;
  monitoringWindow: number;
  successThreshold: number;
}

export class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failureCount: number = 0;
  private successCount: number = 0;
  private lastFailureTime: number = 0;
  private config: CircuitBreakerConfig;
  private name: string;

  constructor(name: string, config: CircuitBreakerConfig) {
    this.name = name;
    this.config = config;
  }

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (this.shouldAttemptReset()) {
        this.state = CircuitState.HALF_OPEN;
        console.log(`Circuit breaker ${this.name} transitioning to HALF_OPEN`);
      } else {
        throw new Error(`Circuit breaker ${this.name} is OPEN`);
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

    if (this.state === CircuitState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.config.successThreshold) {
        this.state = CircuitState.CLOSED;
        this.successCount = 0;
        console.log(`Circuit breaker ${this.name} transitioning to CLOSED`);
      }
    }
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.state === CircuitState.HALF_OPEN) {
      this.state = CircuitState.OPEN;
      this.successCount = 0;
      console.log(
        `Circuit breaker ${this.name} transitioning to OPEN from HALF_OPEN`
      );
    } else if (this.failureCount >= this.config.failureThreshold) {
      this.state = CircuitState.OPEN;
      console.log(`Circuit breaker ${this.name} transitioning to OPEN`);
    }
  }

  private shouldAttemptReset(): boolean {
    return Date.now() - this.lastFailureTime >= this.config.resetTimeout;
  }

  getState(): CircuitState {
    return this.state;
  }

  getStats(): {
    state: CircuitState;
    failureCount: number;
    successCount: number;
  } {
    return {
      state: this.state,
      failureCount: this.failureCount,
      successCount: this.successCount,
    };
  }
}

export class CircuitBreakerRegistry {
  private static instance: CircuitBreakerRegistry;
  private breakers: Map<string, CircuitBreaker> = new Map();

  static getInstance(): CircuitBreakerRegistry {
    if (!this.instance) {
      this.instance = new CircuitBreakerRegistry();
    }
    return this.instance;
  }

  getOrCreate(name: string, config: CircuitBreakerConfig): CircuitBreaker {
    if (!this.breakers.has(name)) {
      this.breakers.set(name, new CircuitBreaker(name, config));
    }
    return this.breakers.get(name)!;
  }

  getBreaker(name: string): CircuitBreaker | undefined {
    return this.breakers.get(name);
  }

  getAllBreakers(): Map<string, CircuitBreaker> {
    return new Map(this.breakers);
  }
}
```

## Distributed Data Management

### Saga Pattern Implementation

```typescript
export interface SagaStep {
  id: string;
  action: () => Promise<any>;
  compensation: () => Promise<any>;
  retryable: boolean;
  maxRetries: number;
}

export interface SagaExecution {
  id: string;
  steps: SagaStep[];
  currentStep: number;
  state: SagaState;
  results: any[];
  error?: Error;
  startTime: number;
  endTime?: number;
}

export enum SagaState {
  PENDING = "pending",
  RUNNING = "running",
  COMPLETED = "completed",
  COMPENSATING = "compensating",
  FAILED = "failed",
}

export class SagaOrchestrator {
  private executions: Map<string, SagaExecution> = new Map();

  async execute(steps: SagaStep[]): Promise<SagaExecution> {
    const execution: SagaExecution = {
      id: this.generateId(),
      steps: [...steps],
      currentStep: 0,
      state: SagaState.PENDING,
      results: [],
      startTime: Date.now(),
    };

    this.executions.set(execution.id, execution);

    try {
      execution.state = SagaState.RUNNING;
      await this.executeSteps(execution);
      execution.state = SagaState.COMPLETED;
      execution.endTime = Date.now();
    } catch (error) {
      execution.error = error;
      execution.state = SagaState.COMPENSATING;
      await this.compensate(execution);
      execution.state = SagaState.FAILED;
      execution.endTime = Date.now();
      throw error;
    }

    return execution;
  }

  private async executeSteps(execution: SagaExecution): Promise<void> {
    for (let i = 0; i < execution.steps.length; i++) {
      execution.currentStep = i;
      const step = execution.steps[i];

      let retries = 0;
      while (retries <= step.maxRetries) {
        try {
          const result = await step.action();
          execution.results.push(result);
          console.log(`Saga step ${step.id} completed successfully`);
          break;
        } catch (error) {
          retries++;

          if (retries > step.maxRetries || !step.retryable) {
            throw error;
          }

          const delay = Math.pow(2, retries) * 1000;
          await this.delay(delay);
          console.log(`Retrying saga step ${step.id}, attempt ${retries}`);
        }
      }
    }
  }

  private async compensate(execution: SagaExecution): Promise<void> {
    console.log(`Starting compensation for saga ${execution.id}`);

    // Compensate in reverse order
    for (let i = execution.currentStep; i >= 0; i--) {
      const step = execution.steps[i];

      try {
        await step.compensation();
        console.log(`Compensation for step ${step.id} completed`);
      } catch (error) {
        console.error(`Compensation failed for step ${step.id}:`, error);
        // Continue with other compensations
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  private generateId(): string {
    return `saga_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  getExecution(id: string): SagaExecution | undefined {
    return this.executions.get(id);
  }

  getAllExecutions(): SagaExecution[] {
    return Array.from(this.executions.values());
  }
}

// Example: Order processing saga
export class OrderProcessingSaga {
  private sagaOrchestrator: SagaOrchestrator;
  private paymentService: PaymentService;
  private inventoryService: InventoryService;
  private shippingService: ShippingService;

  constructor() {
    this.sagaOrchestrator = new SagaOrchestrator();
    this.paymentService = new PaymentService();
    this.inventoryService = new InventoryService();
    this.shippingService = new ShippingService();
  }

  async processOrder(order: Order): Promise<SagaExecution> {
    const steps: SagaStep[] = [
      {
        id: "reserve_inventory",
        action: async () => {
          return await this.inventoryService.reserveItems(order.items);
        },
        compensation: async () => {
          await this.inventoryService.releaseReservation(order.items);
        },
        retryable: true,
        maxRetries: 3,
      },
      {
        id: "process_payment",
        action: async () => {
          return await this.paymentService.processPayment({
            amount: order.total,
            paymentMethod: order.paymentMethod,
          });
        },
        compensation: async () => {
          await this.paymentService.refundPayment(order.paymentId);
        },
        retryable: false,
        maxRetries: 0,
      },
      {
        id: "create_shipment",
        action: async () => {
          return await this.shippingService.createShipment({
            orderId: order.id,
            address: order.shippingAddress,
            items: order.items,
          });
        },
        compensation: async () => {
          await this.shippingService.cancelShipment(order.shipmentId);
        },
        retryable: true,
        maxRetries: 2,
      },
    ];

    return await this.sagaOrchestrator.execute(steps);
  }
}
```

## Service Monitoring and Observability

### Distributed Tracing

```typescript
export interface Span {
  traceId: string;
  spanId: string;
  parentSpanId?: string;
  serviceName: string;
  operationName: string;
  startTime: number;
  endTime?: number;
  duration?: number;
  tags: Record<string, any>;
  logs: SpanLog[];
  status: SpanStatus;
}

export interface SpanLog {
  timestamp: number;
  level: "info" | "warn" | "error";
  message: string;
  fields?: Record<string, any>;
}

export enum SpanStatus {
  OK = "ok",
  ERROR = "error",
  TIMEOUT = "timeout",
}

export class DistributedTracer {
  private static instance: DistributedTracer;
  private spans: Map<string, Span> = new Map();
  private activeSpans: Map<string, Span> = new Map();

  static getInstance(): DistributedTracer {
    if (!this.instance) {
      this.instance = new DistributedTracer();
    }
    return this.instance;
  }

  startSpan(
    operationName: string,
    serviceName: string,
    parentSpanId?: string,
    traceId?: string
  ): Span {
    const span: Span = {
      traceId: traceId || this.generateTraceId(),
      spanId: this.generateSpanId(),
      parentSpanId,
      serviceName,
      operationName,
      startTime: Date.now(),
      tags: {},
      logs: [],
      status: SpanStatus.OK,
    };

    this.spans.set(span.spanId, span);
    this.activeSpans.set(span.spanId, span);

    return span;
  }

  finishSpan(spanId: string, status: SpanStatus = SpanStatus.OK): void {
    const span = this.spans.get(spanId);
    if (span) {
      span.endTime = Date.now();
      span.duration = span.endTime - span.startTime;
      span.status = status;
      this.activeSpans.delete(spanId);
    }
  }

  addTag(spanId: string, key: string, value: any): void {
    const span = this.spans.get(spanId);
    if (span) {
      span.tags[key] = value;
    }
  }

  addLog(
    spanId: string,
    level: SpanLog["level"],
    message: string,
    fields?: Record<string, any>
  ): void {
    const span = this.spans.get(spanId);
    if (span) {
      span.logs.push({
        timestamp: Date.now(),
        level,
        message,
        fields,
      });
    }
  }

  getSpan(spanId: string): Span | undefined {
    return this.spans.get(spanId);
  }

  getTrace(traceId: string): Span[] {
    return Array.from(this.spans.values())
      .filter((span) => span.traceId === traceId)
      .sort((a, b) => a.startTime - b.startTime);
  }

  getActiveSpans(): Span[] {
    return Array.from(this.activeSpans.values());
  }

  private generateTraceId(): string {
    return `trace_${Date.now()}_${Math.random().toString(36).substr(2, 16)}`;
  }

  private generateSpanId(): string {
    return `span_${Date.now()}_${Math.random().toString(36).substr(2, 12)}`;
  }

  // Export traces in Jaeger format
  exportTraces(): any[] {
    const traces: Record<string, Span[]> = {};

    for (const span of this.spans.values()) {
      if (!traces[span.traceId]) {
        traces[span.traceId] = [];
      }
      traces[span.traceId].push(span);
    }

    return Object.entries(traces).map(([traceId, spans]) => ({
      traceID: traceId,
      spans: spans.map((span) => ({
        traceID: span.traceId,
        spanID: span.spanId,
        parentSpanID: span.parentSpanId,
        operationName: span.operationName,
        startTime: span.startTime * 1000, // Convert to microseconds
        duration: (span.duration || 0) * 1000,
        tags: Object.entries(span.tags).map(([key, value]) => ({
          key,
          value: String(value),
          type: typeof value,
        })),
        logs: span.logs.map((log) => ({
          timestamp: log.timestamp * 1000,
          fields: [
            { key: "level", value: log.level },
            { key: "message", value: log.message },
            ...(log.fields
              ? Object.entries(log.fields).map(([k, v]) => ({
                  key: k,
                  value: String(v),
                }))
              : []),
          ],
        })),
        process: {
          serviceName: span.serviceName,
          tags: [],
        },
      })),
    }));
  }
}

// Instrumented HTTP client
export class TracedHttpClient {
  private tracer: DistributedTracer;

  constructor() {
    this.tracer = DistributedTracer.getInstance();
  }

  async request(options: {
    url: string;
    method: string;
    headers?: Record<string, string>;
    body?: any;
    parentSpanId?: string;
    traceId?: string;
  }): Promise<Response> {
    const span = this.tracer.startSpan(
      `HTTP ${options.method} ${options.url}`,
      "http-client",
      options.parentSpanId,
      options.traceId
    );

    // Add HTTP-specific tags
    this.tracer.addTag(span.spanId, "http.method", options.method);
    this.tracer.addTag(span.spanId, "http.url", options.url);

    // Inject trace context into headers
    const headers = {
      ...options.headers,
      "x-trace-id": span.traceId,
      "x-span-id": span.spanId,
    };

    try {
      const response = await fetch(options.url, {
        method: options.method,
        headers,
        body: options.body,
      });

      this.tracer.addTag(span.spanId, "http.status_code", response.status);

      if (response.ok) {
        this.tracer.finishSpan(span.spanId, SpanStatus.OK);
      } else {
        this.tracer.addLog(
          span.spanId,
          "error",
          `HTTP error: ${response.status}`
        );
        this.tracer.finishSpan(span.spanId, SpanStatus.ERROR);
      }

      return response;
    } catch (error) {
      this.tracer.addLog(span.spanId, "error", error.message);
      this.tracer.finishSpan(span.spanId, SpanStatus.ERROR);
      throw error;
    }
  }
}
```

## Security in Microservices

### JWT Authentication Service

```typescript
export interface JWTPayload {
  sub: string; // Subject (user ID)
  iat: number; // Issued at
  exp: number; // Expiration time
  iss: string; // Issuer
  aud: string; // Audience
  scope: string[]; // Permissions/scopes
  roles: string[];
}

export interface UserInfo {
  id: string;
  username: string;
  email: string;
  roles: string[];
  permissions: string[];
}

export class AuthenticationService {
  private secretKey: string;
  private issuer: string;
  private audience: string;
  private tokenExpiry: number = 3600; // 1 hour

  constructor(secretKey: string, issuer: string, audience: string) {
    this.secretKey = secretKey;
    this.issuer = issuer;
    this.audience = audience;
  }

  async generateToken(user: UserInfo): Promise<string> {
    const payload: JWTPayload = {
      sub: user.id,
      iat: Math.floor(Date.now() / 1000),
      exp: Math.floor(Date.now() / 1000) + this.tokenExpiry,
      iss: this.issuer,
      aud: this.audience,
      scope: user.permissions,
      roles: user.roles,
    };

    // In a real implementation, use a proper JWT library
    const header = this.base64UrlEncode(
      JSON.stringify({ alg: "HS256", typ: "JWT" })
    );
    const payloadEncoded = this.base64UrlEncode(JSON.stringify(payload));
    const signature = await this.sign(`${header}.${payloadEncoded}`);

    return `${header}.${payloadEncoded}.${signature}`;
  }

  async verifyToken(
    token: string
  ): Promise<{ success: boolean; payload?: JWTPayload; error?: string }> {
    try {
      const parts = token.split(".");
      if (parts.length !== 3) {
        return { success: false, error: "Invalid token format" };
      }

      const [header, payload, signature] = parts;

      // Verify signature
      const expectedSignature = await this.sign(`${header}.${payload}`);
      if (signature !== expectedSignature) {
        return { success: false, error: "Invalid token signature" };
      }

      // Decode payload
      const decodedPayload: JWTPayload = JSON.parse(
        this.base64UrlDecode(payload)
      );

      // Check expiration
      if (decodedPayload.exp < Math.floor(Date.now() / 1000)) {
        return { success: false, error: "Token expired" };
      }

      // Check issuer and audience
      if (
        decodedPayload.iss !== this.issuer ||
        decodedPayload.aud !== this.audience
      ) {
        return { success: false, error: "Invalid token issuer or audience" };
      }

      return { success: true, payload: decodedPayload };
    } catch (error) {
      return { success: false, error: "Token verification failed" };
    }
  }

  async authenticate(
    request: Request
  ): Promise<{ success: boolean; user?: UserInfo; error?: string }> {
    const authHeader = request.headers.get("Authorization");
    if (!authHeader || !authHeader.startsWith("Bearer ")) {
      return {
        success: false,
        error: "Missing or invalid authorization header",
      };
    }

    const token = authHeader.substring(7);
    const verificationResult = await this.verifyToken(token);

    if (!verificationResult.success) {
      return { success: false, error: verificationResult.error };
    }

    const payload = verificationResult.payload!;
    const user: UserInfo = {
      id: payload.sub,
      username: "", // Would be fetched from user service
      email: "", // Would be fetched from user service
      roles: payload.roles,
      permissions: payload.scope,
    };

    return { success: true, user };
  }

  private async sign(data: string): Promise<string> {
    // Simplified HMAC-SHA256 implementation
    // In production, use crypto APIs or libraries
    const encoder = new TextEncoder();
    const key = await crypto.subtle.importKey(
      "raw",
      encoder.encode(this.secretKey),
      { name: "HMAC", hash: "SHA-256" },
      false,
      ["sign"]
    );

    const signature = await crypto.subtle.sign(
      "HMAC",
      key,
      encoder.encode(data)
    );
    return this.base64UrlEncode(new Uint8Array(signature));
  }

  private base64UrlEncode(data: string | Uint8Array): string {
    const bytes =
      typeof data === "string" ? new TextEncoder().encode(data) : data;
    const base64 = btoa(String.fromCharCode(...bytes));
    return base64.replace(/\+/g, "-").replace(/\//g, "_").replace(/=/g, "");
  }

  private base64UrlDecode(encoded: string): string {
    const base64 = encoded.replace(/-/g, "+").replace(/_/g, "/");
    const padded = base64.padEnd(
      base64.length + ((4 - (base64.length % 4)) % 4),
      "="
    );
    return atob(padded);
  }
}
```

## Best Practices

### 1. Service Design

- Keep services small and focused on single business capabilities
- Design services around business domains, not technical layers
- Ensure services are independently deployable and scalable
- Implement proper service boundaries and contracts

### 2. Communication Patterns

- Use asynchronous messaging for loose coupling
- Implement circuit breakers for resilience
- Design for eventual consistency
- Handle partial failures gracefully

### 3. Data Management

- Each service should own its data
- Use saga patterns for distributed transactions
- Implement proper data synchronization strategies
- Design for data consistency requirements

### 4. Monitoring and Observability

- Implement distributed tracing across all services
- Use structured logging with correlation IDs
- Monitor service health and performance metrics
- Set up proper alerting and monitoring dashboards

## Conclusion

Implementing microservices architecture in HarmonyOS applications requires careful consideration of service design, communication patterns, and distributed system challenges. By following the patterns and practices outlined in this article, you can build scalable, resilient, and maintainable microservices-based applications.

The key to successful microservices implementation is starting simple, focusing on business value, and gradually evolving the architecture based on real-world requirements and constraints. Remember to invest in proper tooling, monitoring, and team practices to support the increased complexity that comes with distributed systems.
