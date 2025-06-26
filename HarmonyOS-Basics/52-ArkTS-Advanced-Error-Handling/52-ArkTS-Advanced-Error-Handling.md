# ArkTS Advanced Error Handling

## Introduction

Robust error handling is essential for building reliable HarmonyOS applications. This guide explores advanced error handling patterns, recovery strategies, and monitoring techniques in ArkTS applications.

## Error Handling Architecture

### Error Classification System

```typescript
enum ErrorSeverity {
  Low = "low",
  Medium = "medium",
  High = "high",
  Critical = "critical",
}

enum ErrorCategory {
  Network = "network",
  UI = "ui",
  Data = "data",
  System = "system",
  Business = "business",
}

interface AppError {
  id: string;
  message: string;
  category: ErrorCategory;
  severity: ErrorSeverity;
  timestamp: number;
  context: ErrorContext;
  stack?: string;
  recoverable: boolean;
}

interface ErrorContext {
  component?: string;
  action?: string;
  userId?: string;
  sessionId: string;
  deviceInfo: DeviceInfo;
  appVersion: string;
}

// Base error classes
abstract class BaseError extends Error {
  abstract readonly category: ErrorCategory;
  abstract readonly severity: ErrorSeverity;
  abstract readonly recoverable: boolean;

  public readonly id: string;
  public readonly timestamp: number;
  public readonly context: ErrorContext;

  constructor(message: string, context: ErrorContext) {
    super(message);
    this.id = this.generateErrorId();
    this.timestamp = Date.now();
    this.context = context;
    this.name = this.constructor.name;
  }

  private generateErrorId(): string {
    return `${this.category}_${Date.now()}_${Math.random()
      .toString(36)
      .substr(2, 9)}`;
  }

  toJSON(): AppError {
    return {
      id: this.id,
      message: this.message,
      category: this.category,
      severity: this.severity,
      timestamp: this.timestamp,
      context: this.context,
      stack: this.stack,
      recoverable: this.recoverable,
    };
  }
}

// Specific error types
class NetworkError extends BaseError {
  readonly category = ErrorCategory.Network;
  readonly severity = ErrorSeverity.Medium;
  readonly recoverable = true;

  constructor(
    message: string,
    context: ErrorContext,
    public statusCode?: number
  ) {
    super(message, context);
  }
}

class ValidationError extends BaseError {
  readonly category = ErrorCategory.Business;
  readonly severity = ErrorSeverity.Low;
  readonly recoverable = true;

  constructor(
    message: string,
    context: ErrorContext,
    public field?: string,
    public value?: any
  ) {
    super(message, context);
  }
}

class SystemError extends BaseError {
  readonly category = ErrorCategory.System;
  readonly severity = ErrorSeverity.Critical;
  readonly recoverable = false;

  constructor(
    message: string,
    context: ErrorContext,
    public systemCode?: string
  ) {
    super(message, context);
  }
}
```

### Global Error Handler

```typescript
class GlobalErrorHandler {
  private static instance: GlobalErrorHandler;
  private errorListeners: Set<(error: AppError) => void> = new Set();
  private errorQueue: AppError[] = [];
  private retryStrategies = new Map<ErrorCategory, RetryStrategy>();

  static getInstance(): GlobalErrorHandler {
    if (!GlobalErrorHandler.instance) {
      GlobalErrorHandler.instance = new GlobalErrorHandler();
    }
    return GlobalErrorHandler.instance;
  }

  private constructor() {
    this.setupDefaultRetryStrategies();
    this.setupGlobalErrorCatching();
  }

  handleError(error: Error | BaseError, context?: Partial<ErrorContext>): void {
    const appError = this.normalizeError(error, context);

    // Log error
    this.logError(appError);

    // Queue for processing
    this.errorQueue.push(appError);

    // Notify listeners
    this.notifyListeners(appError);

    // Attempt recovery if possible
    if (appError.recoverable) {
      this.attemptRecovery(appError);
    }

    // Critical errors might require app restart
    if (appError.severity === ErrorSeverity.Critical) {
      this.handleCriticalError(appError);
    }
  }

  addErrorListener(listener: (error: AppError) => void): () => void {
    this.errorListeners.add(listener);
    return () => this.errorListeners.delete(listener);
  }

  setRetryStrategy(category: ErrorCategory, strategy: RetryStrategy): void {
    this.retryStrategies.set(category, strategy);
  }

  private normalizeError(
    error: Error | BaseError,
    context?: Partial<ErrorContext>
  ): AppError {
    if (error instanceof BaseError) {
      return error.toJSON();
    }

    // Convert regular Error to AppError
    const fullContext: ErrorContext = {
      sessionId: this.getSessionId(),
      deviceInfo: this.getDeviceInfo(),
      appVersion: this.getAppVersion(),
      ...context,
    };

    return {
      id: `unknown_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
      message: error.message,
      category: ErrorCategory.System,
      severity: ErrorSeverity.Medium,
      timestamp: Date.now(),
      context: fullContext,
      stack: error.stack,
      recoverable: false,
    };
  }

  private async attemptRecovery(error: AppError): Promise<void> {
    const strategy = this.retryStrategies.get(error.category);
    if (!strategy) return;

    for (let attempt = 1; attempt <= strategy.maxAttempts; attempt++) {
      try {
        await this.wait(strategy.delay * Math.pow(2, attempt - 1));

        if (await strategy.shouldRetry(error, attempt)) {
          await strategy.recover(error);
          break;
        }
      } catch (recoveryError) {
        console.error(`Recovery attempt ${attempt} failed:`, recoveryError);

        if (attempt === strategy.maxAttempts) {
          // All recovery attempts failed
          this.handleRecoveryFailure(error, recoveryError as Error);
        }
      }
    }
  }

  private setupGlobalErrorCatching(): void {
    // Catch unhandled promise rejections
    window.addEventListener("unhandledrejection", (event) => {
      this.handleError(new Error(event.reason), {
        action: "unhandled_promise_rejection",
      });
    });

    // Catch uncaught exceptions
    window.addEventListener("error", (event) => {
      this.handleError(new Error(event.message), {
        action: "uncaught_exception",
        component: event.filename,
      });
    });
  }
}

interface RetryStrategy {
  maxAttempts: number;
  delay: number;
  shouldRetry: (error: AppError, attempt: number) => Promise<boolean>;
  recover: (error: AppError) => Promise<void>;
}
```

### Component Error Boundaries

```typescript
// Error boundary decorator
function WithErrorBoundary(options: ErrorBoundaryOptions = {}) {
  return function<T extends { new(...args: any[]): any }>(constructor: T) {
    return class extends constructor {
      @State private hasError: boolean = false
      @State private error: AppError | null = null
      private errorHandler = GlobalErrorHandler.getInstance()

      build() {
        if (this.hasError && this.error) {
          return this.buildErrorFallback()
        }

        try {
          return super.build()
        } catch (error) {
          this.handleComponentError(error as Error)
          return this.buildErrorFallback()
        }
      }

      @Builder
      buildErrorFallback() {
        if (options.fallbackComponent) {
          return options.fallbackComponent(this.error!)
        }

        return this.buildDefaultErrorFallback()
      }

      @Builder
      buildDefaultErrorFallback() {
        Column() {
          Image('/resources/error_icon.png')
            .width(64)
            .height(64)
            .margin({ bottom: 16 })

          Text('Something went wrong')
            .fontSize(18)
            .fontWeight(FontWeight.Bold)
            .margin({ bottom: 8 })

          Text(this.error?.message || 'An unexpected error occurred')
            .fontSize(14)
            .fontColor('#666')
            .textAlign(TextAlign.Center)
            .margin({ bottom: 16 })

          Button('Try Again')
            .onClick(() => this.retryComponent())

          if (options.showDetails) {
            Button('Show Details')
              .backgroundColor(Color.Transparent)
              .fontColor('#007AFF')
              .onClick(() => this.showErrorDetails())
          }
        }
        .padding(32)
        .justifyContent(FlexAlign.Center)
        .alignItems(HorizontalAlign.Center)
        .width('100%')
        .height('100%')
      }

      private handleComponentError(error: Error): void {
        const context: ErrorContext = {
          component: this.constructor.name,
          sessionId: this.getSessionId(),
          deviceInfo: this.getDeviceInfo(),
          appVersion: this.getAppVersion()
        }

        this.error = new BaseError(error.message, context) as any
        this.hasError = true

        this.errorHandler.handleError(error, context)
      }

      private retryComponent(): void {
        this.hasError = false
        this.error = null
      }

      private showErrorDetails(): void {
        if (this.error) {
          // Show detailed error information
          console.log('Error details:', this.error)
        }
      }
    }
  }
}

interface ErrorBoundaryOptions {
  fallbackComponent?: (error: AppError) => void
  showDetails?: boolean
  onError?: (error: AppError) => void
}

// Usage example
@WithErrorBoundary({
  showDetails: true,
  onError: (error) => console.log('Component error:', error)
})
@Component
struct SafeProductList {
  @State products: Product[] = []

  build() {
    List() {
      ForEach(this.products, (product: Product) => {
        ListItem() {
          ProductCard({ product: product })
        }
      })
    }
  }
}
```

## Async Error Handling

### Promise-based Error Handling

```typescript
class AsyncErrorHandler {
  static async withRetry<T>(
    operation: () => Promise<T>,
    options: RetryOptions = {}
  ): Promise<T> {
    const {
      maxAttempts = 3,
      delay = 1000,
      backoff = "exponential",
      shouldRetry = () => true,
    } = options;

    let lastError: Error;

    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error as Error;

        if (attempt === maxAttempts || !shouldRetry(error as Error, attempt)) {
          throw lastError;
        }

        const waitTime = this.calculateDelay(delay, attempt, backoff);
        await this.wait(waitTime);
      }
    }

    throw lastError!;
  }

  static async withTimeout<T>(
    operation: () => Promise<T>,
    timeoutMs: number,
    timeoutMessage = "Operation timed out"
  ): Promise<T> {
    return Promise.race([
      operation(),
      new Promise<never>((_, reject) => {
        setTimeout(() => {
          reject(new Error(timeoutMessage));
        }, timeoutMs);
      }),
    ]);
  }

  static async withCircuitBreaker<T>(
    operation: () => Promise<T>,
    circuitBreaker: CircuitBreaker
  ): Promise<T> {
    if (circuitBreaker.isOpen()) {
      throw new Error("Circuit breaker is open");
    }

    try {
      const result = await operation();
      circuitBreaker.recordSuccess();
      return result;
    } catch (error) {
      circuitBreaker.recordFailure();
      throw error;
    }
  }

  private static calculateDelay(
    baseDelay: number,
    attempt: number,
    backoff: "linear" | "exponential"
  ): number {
    switch (backoff) {
      case "linear":
        return baseDelay * attempt;
      case "exponential":
        return baseDelay * Math.pow(2, attempt - 1);
      default:
        return baseDelay;
    }
  }

  private static wait(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

interface RetryOptions {
  maxAttempts?: number;
  delay?: number;
  backoff?: "linear" | "exponential";
  shouldRetry?: (error: Error, attempt: number) => boolean;
}

// Circuit breaker implementation
class CircuitBreaker {
  private failureCount = 0;
  private lastFailureTime = 0;
  private state: "closed" | "open" | "half-open" = "closed";

  constructor(private threshold: number = 5, private timeout: number = 60000) {}

  isOpen(): boolean {
    if (this.state === "open") {
      if (Date.now() - this.lastFailureTime >= this.timeout) {
        this.state = "half-open";
        return false;
      }
      return true;
    }
    return false;
  }

  recordSuccess(): void {
    this.failureCount = 0;
    this.state = "closed";
  }

  recordFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.threshold) {
      this.state = "open";
    }
  }
}
```

### Safe API Service

```typescript
class SafeApiService {
  private circuitBreaker = new CircuitBreaker(5, 60000);
  private errorHandler = GlobalErrorHandler.getInstance();

  async request<T>(config: ApiRequestConfig): Promise<ApiResponse<T>> {
    try {
      return await AsyncErrorHandler.withCircuitBreaker(
        () =>
          AsyncErrorHandler.withRetry(
            () =>
              AsyncErrorHandler.withTimeout(
                () => this.makeRequest<T>(config),
                config.timeout || 10000
              ),
            {
              maxAttempts: config.retryAttempts || 3,
              shouldRetry: (error) => this.shouldRetryRequest(error),
            }
          ),
        this.circuitBreaker
      );
    } catch (error) {
      const apiError = new NetworkError(
        error.message,
        {
          action: "api_request",
          sessionId: this.getSessionId(),
          deviceInfo: this.getDeviceInfo(),
          appVersion: this.getAppVersion(),
        },
        error.status
      );

      this.errorHandler.handleError(apiError);
      throw apiError;
    }
  }

  private async makeRequest<T>(
    config: ApiRequestConfig
  ): Promise<ApiResponse<T>> {
    const response = await fetch(config.url, {
      method: config.method || "GET",
      headers: config.headers,
      body: config.body ? JSON.stringify(config.body) : undefined,
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const data = await response.json();
    return {
      data,
      status: response.status,
      headers: response.headers,
    };
  }

  private shouldRetryRequest(error: Error): boolean {
    // Retry on network errors and certain HTTP status codes
    if (error.message.includes("network")) return true;
    if (error.message.includes("timeout")) return true;
    if (error.message.includes("HTTP 5")) return true; // 5xx errors
    if (error.message.includes("HTTP 429")) return true; // Rate limiting

    return false;
  }
}

interface ApiRequestConfig {
  url: string;
  method?: string;
  headers?: Record<string, string>;
  body?: any;
  timeout?: number;
  retryAttempts?: number;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  headers: Headers;
}
```

## Error Recovery Strategies

### Graceful Degradation

```typescript
class FeatureManager {
  private features = new Map<string, FeatureState>();
  private fallbacks = new Map<string, () => any>();

  registerFeature(
    name: string,
    implementation: () => any,
    fallback?: () => any
  ): void {
    this.features.set(name, {
      name,
      implementation,
      status: "enabled",
      errorCount: 0,
      lastError: null,
    });

    if (fallback) {
      this.fallbacks.set(name, fallback);
    }
  }

  async executeFeature<T>(name: string, ...args: any[]): Promise<T> {
    const feature = this.features.get(name);
    if (!feature) {
      throw new Error(`Feature '${name}' not registered`);
    }

    if (feature.status === "disabled") {
      return this.executeFallback(name, ...args);
    }

    try {
      const result = await feature.implementation(...args);

      // Reset error count on success
      feature.errorCount = 0;
      feature.status = "enabled";

      return result;
    } catch (error) {
      feature.errorCount++;
      feature.lastError = error as Error;

      // Disable feature if too many errors
      if (feature.errorCount >= 3) {
        feature.status = "disabled";
        console.warn(`Feature '${name}' disabled due to repeated errors`);
      }

      // Try fallback
      if (this.fallbacks.has(name)) {
        return this.executeFallback(name, ...args);
      }

      throw error;
    }
  }

  private async executeFallback<T>(name: string, ...args: any[]): Promise<T> {
    const fallback = this.fallbacks.get(name);
    if (!fallback) {
      throw new Error(`No fallback available for feature '${name}'`);
    }

    return fallback(...args);
  }

  getFeatureStatus(name: string): FeatureState | undefined {
    return this.features.get(name);
  }

  enableFeature(name: string): void {
    const feature = this.features.get(name);
    if (feature) {
      feature.status = "enabled";
      feature.errorCount = 0;
    }
  }
}

interface FeatureState {
  name: string;
  implementation: () => any;
  status: "enabled" | "disabled";
  errorCount: number;
  lastError: Error | null;
}
```

## Conclusion

Advanced error handling in ArkTS applications requires:

- Comprehensive error classification and context capture
- Global error handling with recovery strategies
- Component-level error boundaries
- Robust async error handling patterns
- Circuit breaker and retry mechanisms
- Graceful degradation strategies
- Comprehensive error monitoring and reporting

These patterns ensure applications remain stable and user-friendly even when unexpected errors occur, providing better user experiences and easier debugging for developers.
