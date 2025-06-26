# 38. ArkTS Asynchronous Programming and Promises

## Introduction

Asynchronous programming is fundamental to building responsive HarmonyOS applications. This article explores ArkTS's async/await patterns, Promise implementations, and advanced concurrent programming techniques for optimal user experience and performance.

## Core Asynchronous Concepts

### Promise Implementation and Usage

```typescript
// Advanced Promise utilities
export class PromiseUtils {
  static delay(ms: number): Promise<void> {
    return new Promise((resolve) => {
      setTimeout(resolve, ms);
    });
  }

  static timeout<T>(promise: Promise<T>, ms: number): Promise<T> {
    const timeoutPromise = new Promise<never>((_, reject) => {
      setTimeout(
        () => reject(new Error(`Operation timed out after ${ms}ms`)),
        ms
      );
    });

    return Promise.race([promise, timeoutPromise]);
  }

  static retry<T>(
    operation: () => Promise<T>,
    maxAttempts: number = 3,
    delayMs: number = 1000
  ): Promise<T> {
    return new Promise(async (resolve, reject) => {
      let lastError: Error;

      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          const result = await operation();
          resolve(result);
          return;
        } catch (error) {
          lastError = error as Error;

          if (attempt === maxAttempts) {
            reject(lastError);
            return;
          }

          await this.delay(delayMs * attempt);
        }
      }
    });
  }

  static allSettled<T>(promises: Promise<T>[]): Promise<
    Array<{
      status: "fulfilled" | "rejected";
      value?: T;
      reason?: any;
    }>
  > {
    return Promise.all(
      promises.map((promise) =>
        promise
          .then((value) => ({ status: "fulfilled" as const, value }))
          .catch((reason) => ({ status: "rejected" as const, reason }))
      )
    );
  }

  static sequence<T>(tasks: Array<() => Promise<T>>): Promise<T[]> {
    return tasks.reduce(
      (promiseChain, currentTask) =>
        promiseChain.then(async (results) => {
          const result = await currentTask();
          return [...results, result];
        }),
      Promise.resolve([] as T[])
    );
  }
}
```

### Async Task Management

```typescript
export interface TaskConfig {
  priority: "high" | "medium" | "low";
  timeout?: number;
  retryCount?: number;
  dependencies?: string[];
}

export interface Task<T = any> {
  id: string;
  name: string;
  execute: () => Promise<T>;
  config: TaskConfig;
  status: "pending" | "running" | "completed" | "failed" | "cancelled";
  result?: T;
  error?: Error;
  startTime?: number;
  endTime?: number;
}

export class AsyncTaskManager {
  private tasks: Map<string, Task> = new Map();
  private runningTasks: Set<string> = new Set();
  private maxConcurrency: number = 5;
  private priorityQueue: Task[] = [];

  constructor(maxConcurrency: number = 5) {
    this.maxConcurrency = maxConcurrency;
  }

  addTask<T>(
    id: string,
    name: string,
    execute: () => Promise<T>,
    config: TaskConfig = { priority: "medium" }
  ): Promise<T> {
    const task: Task<T> = {
      id,
      name,
      execute,
      config,
      status: "pending",
    };

    this.tasks.set(id, task);
    this.priorityQueue.push(task);
    this.sortPriorityQueue();

    return new Promise((resolve, reject) => {
      const checkCompletion = () => {
        const currentTask = this.tasks.get(id);
        if (currentTask?.status === "completed") {
          resolve(currentTask.result as T);
        } else if (currentTask?.status === "failed") {
          reject(currentTask.error);
        } else if (currentTask?.status === "cancelled") {
          reject(new Error("Task was cancelled"));
        } else {
          setTimeout(checkCompletion, 100);
        }
      };

      this.processTasks();
      checkCompletion();
    });
  }

  private async processTasks(): Promise<void> {
    while (
      this.priorityQueue.length > 0 &&
      this.runningTasks.size < this.maxConcurrency
    ) {
      const task = this.priorityQueue.shift()!;

      if (await this.checkDependencies(task)) {
        this.runTask(task);
      } else {
        this.priorityQueue.push(task);
      }
    }
  }

  private async checkDependencies(task: Task): Promise<boolean> {
    if (!task.config.dependencies) return true;

    return task.config.dependencies.every((depId) => {
      const depTask = this.tasks.get(depId);
      return depTask?.status === "completed";
    });
  }

  private async runTask(task: Task): Promise<void> {
    this.runningTasks.add(task.id);
    task.status = "running";
    task.startTime = Date.now();

    try {
      let result: any;

      if (task.config.timeout) {
        result = await PromiseUtils.timeout(
          task.execute(),
          task.config.timeout
        );
      } else if (task.config.retryCount) {
        result = await PromiseUtils.retry(task.execute, task.config.retryCount);
      } else {
        result = await task.execute();
      }

      task.result = result;
      task.status = "completed";
      task.endTime = Date.now();
    } catch (error) {
      task.error = error as Error;
      task.status = "failed";
      task.endTime = Date.now();
    } finally {
      this.runningTasks.delete(task.id);
      this.processTasks(); // Process next tasks
    }
  }

  private sortPriorityQueue(): void {
    const priorityOrder = { high: 3, medium: 2, low: 1 };
    this.priorityQueue.sort(
      (a, b) =>
        priorityOrder[b.config.priority] - priorityOrder[a.config.priority]
    );
  }

  cancelTask(id: string): boolean {
    const task = this.tasks.get(id);
    if (task && task.status === "pending") {
      task.status = "cancelled";
      const index = this.priorityQueue.findIndex((t) => t.id === id);
      if (index > -1) {
        this.priorityQueue.splice(index, 1);
      }
      return true;
    }
    return false;
  }

  getTaskStatus(id: string): Task | null {
    return this.tasks.get(id) || null;
  }

  getAllTasks(): Task[] {
    return Array.from(this.tasks.values());
  }
}
```

## Reactive Async Patterns

### Observable Streams with Async Operations

```typescript
export class AsyncObservable<T> {
  private observers: Array<{
    next: (value: T) => void;
    error?: (error: any) => void;
    complete?: () => void;
  }> = [];

  constructor(
    private asyncProducer: (observer: {
      next: (value: T) => void;
      error: (error: any) => void;
      complete: () => void;
    }) => void | (() => void)
  ) {}

  subscribe(observer: {
    next: (value: T) => void;
    error?: (error: any) => void;
    complete?: () => void;
  }): { unsubscribe: () => void } {
    this.observers.push(observer);

    const cleanup = this.asyncProducer({
      next: (value) => observer.next(value),
      error: (error) => observer.error?.(error),
      complete: () => observer.complete?.(),
    });

    return {
      unsubscribe: () => {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
          this.observers.splice(index, 1);
        }
        if (cleanup && typeof cleanup === "function") {
          cleanup();
        }
      },
    };
  }

  static fromAsyncIterable<T>(
    asyncIterable: AsyncIterable<T>
  ): AsyncObservable<T> {
    return new AsyncObservable<T>(async (observer) => {
      try {
        for await (const value of asyncIterable) {
          observer.next(value);
        }
        observer.complete();
      } catch (error) {
        observer.error(error);
      }
    });
  }

  static interval(intervalMs: number): AsyncObservable<number> {
    return new AsyncObservable<number>((observer) => {
      let counter = 0;
      const intervalId = setInterval(() => {
        observer.next(counter++);
      }, intervalMs);

      return () => clearInterval(intervalId);
    });
  }

  map<U>(transform: (value: T) => U | Promise<U>): AsyncObservable<U> {
    return new AsyncObservable<U>((observer) => {
      return this.subscribe({
        next: async (value) => {
          try {
            const transformed = await transform(value);
            observer.next(transformed);
          } catch (error) {
            observer.error(error);
          }
        },
        error: (error) => observer.error(error),
        complete: () => observer.complete(),
      });
    });
  }

  filter(
    predicate: (value: T) => boolean | Promise<boolean>
  ): AsyncObservable<T> {
    return new AsyncObservable<T>((observer) => {
      return this.subscribe({
        next: async (value) => {
          try {
            const shouldEmit = await predicate(value);
            if (shouldEmit) {
              observer.next(value);
            }
          } catch (error) {
            observer.error(error);
          }
        },
        error: (error) => observer.error(error),
        complete: () => observer.complete(),
      });
    });
  }
}
```

## HTTP Client with Advanced Async Features

```typescript
export interface RequestConfig {
  method: "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
  url: string;
  headers?: Record<string, string>;
  params?: Record<string, any>;
  data?: any;
  timeout?: number;
  retries?: number;
  retryDelay?: number;
}

export interface RequestInterceptor {
  request?: (config: RequestConfig) => RequestConfig | Promise<RequestConfig>;
  response?: (response: any) => any | Promise<any>;
  error?: (error: Error) => Error | Promise<Error>;
}

export class AsyncHttpClient {
  private baseURL: string = "";
  private defaultHeaders: Record<string, string> = {};
  private interceptors: RequestInterceptor[] = [];
  private taskManager: AsyncTaskManager;

  constructor(baseURL?: string) {
    if (baseURL) this.baseURL = baseURL;
    this.taskManager = new AsyncTaskManager(10);
  }

  addInterceptor(interceptor: RequestInterceptor): void {
    this.interceptors.push(interceptor);
  }

  setDefaultHeader(key: string, value: string): void {
    this.defaultHeaders[key] = value;
  }

  async request<T>(config: RequestConfig): Promise<T> {
    const requestId = `${config.method}-${config.url}-${Date.now()}`;

    return this.taskManager.addTask(
      requestId,
      `HTTP ${config.method} ${config.url}`,
      () => this.executeRequest<T>(config),
      {
        priority: "medium",
        timeout: config.timeout,
        retryCount: config.retries,
      }
    );
  }

  private async executeRequest<T>(config: RequestConfig): Promise<T> {
    // Apply request interceptors
    let processedConfig = config;
    for (const interceptor of this.interceptors) {
      if (interceptor.request) {
        processedConfig = await interceptor.request(processedConfig);
      }
    }

    // Build full URL
    const fullUrl = this.baseURL + processedConfig.url;

    // Prepare headers
    const headers = {
      ...this.defaultHeaders,
      ...processedConfig.headers,
      "Content-Type": "application/json",
    };

    // Prepare request options
    const requestOptions: any = {
      method: processedConfig.method,
      headers,
    };

    if (processedConfig.data) {
      requestOptions.body = JSON.stringify(processedConfig.data);
    }

    try {
      const response = await fetch(fullUrl, requestOptions);

      if (!response.ok) {
        throw new Error(
          `HTTP Error: ${response.status} ${response.statusText}`
        );
      }

      let result = await response.json();

      // Apply response interceptors
      for (const interceptor of this.interceptors) {
        if (interceptor.response) {
          result = await interceptor.response(result);
        }
      }

      return result;
    } catch (error) {
      // Apply error interceptors
      let processedError = error as Error;
      for (const interceptor of this.interceptors) {
        if (interceptor.error) {
          processedError = await interceptor.error(processedError);
        }
      }
      throw processedError;
    }
  }

  // Convenience methods
  async get<T>(url: string, config?: Partial<RequestConfig>): Promise<T> {
    return this.request<T>({ method: "GET", url, ...config });
  }

  async post<T>(
    url: string,
    data?: any,
    config?: Partial<RequestConfig>
  ): Promise<T> {
    return this.request<T>({ method: "POST", url, data, ...config });
  }

  async put<T>(
    url: string,
    data?: any,
    config?: Partial<RequestConfig>
  ): Promise<T> {
    return this.request<T>({ method: "PUT", url, data, ...config });
  }

  async delete<T>(url: string, config?: Partial<RequestConfig>): Promise<T> {
    return this.request<T>({ method: "DELETE", url, ...config });
  }
}
```

## ArkUI Integration Examples

### Async Data Loading Component

```typescript
@Component
export struct AsyncDataComponent {
  @State loading: boolean = false;
  @State error: string = '';
  @State data: any[] = [];
  private httpClient: AsyncHttpClient = new AsyncHttpClient('https://api.example.com');
  private taskManager: AsyncTaskManager = new AsyncTaskManager();

  aboutToAppear() {
    this.loadData();
  }

  private async loadData(): Promise<void> {
    this.loading = true;
    this.error = '';

    try {
      // Load multiple data sources concurrently
      const [users, posts, comments] = await Promise.all([
        this.httpClient.get<User[]>('/users'),
        this.httpClient.get<Post[]>('/posts'),
        this.httpClient.get<Comment[]>('/comments')
      ]);

      // Process data asynchronously
      const processedData = await this.taskManager.addTask(
        'process-data',
        'Process loaded data',
        () => this.processData(users, posts, comments),
        { priority: 'high' }
      );

      this.data = processedData;
    } catch (error) {
      this.error = (error as Error).message;
    } finally {
      this.loading = false;
    }
  }

  private async processData(
    users: User[],
    posts: Post[],
    comments: Comment[]
  ): Promise<any[]> {
    // Simulate complex async processing
    await PromiseUtils.delay(500);

    return users.map(user => ({
      ...user,
      posts: posts.filter(post => post.userId === user.id),
      commentCount: comments.filter(comment =>
        posts.some(post => post.id === comment.postId && post.userId === user.id)
      ).length
    }));
  }

  private async refreshData(): Promise<void> {
    await this.loadData();
  }

  build() {
    Column({ space: 16 }) {
      Row() {
        Text('Async Data Example')
          .fontSize(24)
          .fontWeight(FontWeight.Bold)

        if (this.loading) {
          LoadingProgress()
            .width(24)
            .height(24)
        }

        Button('Refresh')
          .onClick(() => this.refreshData())
          .enabled(!this.loading)
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)

      if (this.error) {
        Text(this.error)
          .fontColor(Color.Red)
          .fontSize(14)
      }

      List({ space: 12 }) {
        ForEach(this.data, (item: any) => {
          ListItem() {
            Card() {
              Column({ space: 8 }) {
                Text(item.name)
                  .fontSize(18)
                  .fontWeight(FontWeight.Bold)

                Text(`Posts: ${item.posts.length}`)
                  .fontSize(14)
                  .fontColor(Color.Gray)

                Text(`Comments: ${item.commentCount}`)
                  .fontSize(14)
                  .fontColor(Color.Gray)
              }
              .alignItems(HorizontalAlign.Start)
              .width('100%')
              .padding(16)
            }
          }
        }, (item: any) => item.id.toString())
      }
      .layoutWeight(1)
    }
    .width('100%')
    .height('100%')
    .padding(16)
  }
}

interface User {
  id: number;
  name: string;
  email: string;
}

interface Post {
  id: number;
  userId: number;
  title: string;
  body: string;
}

interface Comment {
  id: number;
  postId: number;
  name: string;
  body: string;
}
```

## Error Handling and Recovery

### Robust Error Management

```typescript
export class AsyncErrorHandler {
  private static errorHandlers: Map<string, (error: Error) => void> = new Map();

  static registerErrorHandler(
    errorType: string,
    handler: (error: Error) => void
  ): void {
    this.errorHandlers.set(errorType, handler);
  }

  static async handleAsync<T>(
    operation: () => Promise<T>,
    fallback?: T,
    errorType?: string
  ): Promise<T> {
    try {
      return await operation();
    } catch (error) {
      const err = error as Error;

      if (errorType && this.errorHandlers.has(errorType)) {
        this.errorHandlers.get(errorType)!(err);
      }

      console.error("Async operation failed:", err);

      if (fallback !== undefined) {
        return fallback;
      }

      throw error;
    }
  }

  static createCircuitBreaker<T>(
    operation: () => Promise<T>,
    failureThreshold: number = 5,
    resetTimeoutMs: number = 60000
  ): () => Promise<T> {
    let failureCount = 0;
    let lastFailureTime = 0;
    let state: "closed" | "open" | "half-open" = "closed";

    return async (): Promise<T> => {
      const now = Date.now();

      if (state === "open") {
        if (now - lastFailureTime > resetTimeoutMs) {
          state = "half-open";
        } else {
          throw new Error("Circuit breaker is open");
        }
      }

      try {
        const result = await operation();

        if (state === "half-open") {
          state = "closed";
          failureCount = 0;
        }

        return result;
      } catch (error) {
        failureCount++;
        lastFailureTime = now;

        if (failureCount >= failureThreshold) {
          state = "open";
        }

        throw error;
      }
    };
  }
}
```

## Best Practices

1. **Promise Management**: Always handle Promise rejection with proper error handling
2. **Concurrent Operations**: Use Promise.all() for independent parallel operations
3. **Task Prioritization**: Implement priority queues for critical async operations
4. **Memory Management**: Clean up subscriptions and cancel pending operations
5. **Error Recovery**: Implement circuit breakers and retry mechanisms
6. **Performance Monitoring**: Track async operation performance and bottlenecks

## Conclusion

Mastering asynchronous programming in ArkTS enables developers to build highly responsive and efficient HarmonyOS applications. By leveraging Promises, async/await patterns, and advanced concurrent programming techniques, applications can handle complex data flows while maintaining excellent user experience and system performance.

The key to successful async programming lies in proper error handling, resource management, and understanding the trade-offs between different async patterns for specific use cases.
