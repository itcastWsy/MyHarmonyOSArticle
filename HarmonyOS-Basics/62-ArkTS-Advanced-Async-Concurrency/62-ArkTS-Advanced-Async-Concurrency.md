# ArkTS Advanced Async Programming and Concurrency

## Introduction

Modern applications require sophisticated asynchronous programming and concurrency management. This guide explores advanced async patterns, worker threads, task scheduling, and concurrent data structures in ArkTS.

## Advanced Promise Patterns

### Promise Utilities

```typescript
class PromiseUtils {
  static async timeout<T>(promise: Promise<T>, ms: number): Promise<T> {
    return Promise.race([
      promise,
      new Promise<never>((_, reject) => {
        setTimeout(() => reject(new Error("Timeout")), ms);
      }),
    ]);
  }

  static async retry<T>(
    fn: () => Promise<T>,
    attempts: number = 3,
    delay: number = 1000
  ): Promise<T> {
    for (let i = 0; i < attempts; i++) {
      try {
        return await fn();
      } catch (error) {
        if (i === attempts - 1) throw error;
        await this.delay(delay * Math.pow(2, i));
      }
    }
    throw new Error("Max attempts reached");
  }

  static async parallel<T>(
    tasks: (() => Promise<T>)[],
    concurrency: number = 5
  ): Promise<T[]> {
    const results: T[] = [];
    const executing: Promise<void>[] = [];

    for (let i = 0; i < tasks.length; i++) {
      const task = tasks[i];
      const promise = task().then((result) => {
        results[i] = result;
      });

      executing.push(promise);

      if (executing.length >= concurrency) {
        await Promise.race(executing);
        executing.splice(
          executing.findIndex((p) => p === promise),
          1
        );
      }
    }

    await Promise.all(executing);
    return results;
  }

  static async waterfall<T>(tasks: (() => Promise<T>)[]): Promise<T[]> {
    const results: T[] = [];
    for (const task of tasks) {
      results.push(await task());
    }
    return results;
  }

  static delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  static debounce<T extends any[]>(
    fn: (...args: T) => Promise<any>,
    delay: number
  ): (...args: T) => Promise<any> {
    let timeoutId: number | null = null;
    let resolveLatest: ((value: any) => void) | null = null;

    return (...args: T): Promise<any> => {
      return new Promise((resolve, reject) => {
        if (timeoutId) {
          clearTimeout(timeoutId);
        }

        if (resolveLatest) {
          resolveLatest(undefined);
        }

        resolveLatest = resolve;

        timeoutId = setTimeout(async () => {
          try {
            const result = await fn(...args);
            resolve(result);
          } catch (error) {
            reject(error);
          }
          resolveLatest = null;
        }, delay);
      });
    };
  }
}

// Advanced Promise-based queue
class AsyncQueue<T> {
  private queue: (() => Promise<T>)[] = [];
  private processing = false;
  private concurrency: number;
  private running = 0;

  constructor(concurrency = 1) {
    this.concurrency = concurrency;
  }

  add(task: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await task();
          resolve(result);
          return result;
        } catch (error) {
          reject(error);
          throw error;
        }
      });

      this.process();
    });
  }

  private async process(): Promise<void> {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }

    this.running++;
    const task = this.queue.shift()!;

    try {
      await task();
    } catch (error) {
      // Error already handled in task wrapper
    } finally {
      this.running--;
      this.process();
    }
  }

  clear(): void {
    this.queue = [];
  }

  size(): number {
    return this.queue.length;
  }
}
```

## Worker Thread Management

### TaskPool Implementation

```typescript
interface TaskConfig {
  timeout?: number;
  priority?: "low" | "normal" | "high";
  retries?: number;
}

interface TaskResult<T> {
  success: boolean;
  data?: T;
  error?: Error;
  duration: number;
}

class TaskPool {
  private workers: Worker[] = [];
  private taskQueue: PendingTask[] = [];
  private activeTasks = new Map<string, ActiveTask>();
  private maxWorkers: number;
  private workerScript: string;

  constructor(
    maxWorkers = navigator.hardwareConcurrency || 4,
    workerScript: string
  ) {
    this.maxWorkers = maxWorkers;
    this.workerScript = workerScript;
  }

  async execute<T>(
    taskData: any,
    config: TaskConfig = {}
  ): Promise<TaskResult<T>> {
    return new Promise((resolve, reject) => {
      const taskId = this.generateTaskId();
      const task: PendingTask = {
        id: taskId,
        data: taskData,
        config: {
          timeout: 30000,
          priority: "normal",
          retries: 0,
          ...config,
        },
        resolve,
        reject,
        startTime: Date.now(),
      };

      this.queueTask(task);
      this.processQueue();
    });
  }

  private queueTask(task: PendingTask): void {
    // Insert based on priority
    const priorityOrder = { high: 0, normal: 1, low: 2 };
    const insertIndex = this.taskQueue.findIndex(
      (t) =>
        priorityOrder[t.config.priority!] > priorityOrder[task.config.priority!]
    );

    if (insertIndex === -1) {
      this.taskQueue.push(task);
    } else {
      this.taskQueue.splice(insertIndex, 0, task);
    }
  }

  private async processQueue(): Promise<void> {
    if (this.taskQueue.length === 0 || this.workers.length >= this.maxWorkers) {
      return;
    }

    const task = this.taskQueue.shift()!;
    const worker = await this.getOrCreateWorker();

    this.executeTask(worker, task);
  }

  private async getOrCreateWorker(): Promise<Worker> {
    // Try to find idle worker
    const idleWorker = this.workers.find((w) => !this.isWorkerBusy(w));
    if (idleWorker) {
      return idleWorker;
    }

    // Create new worker if under limit
    if (this.workers.length < this.maxWorkers) {
      const worker = new Worker(this.workerScript);
      this.workers.push(worker);
      this.setupWorkerListeners(worker);
      return worker;
    }

    // Wait for worker to become available
    return new Promise((resolve) => {
      const checkForWorker = () => {
        const availableWorker = this.workers.find((w) => !this.isWorkerBusy(w));
        if (availableWorker) {
          resolve(availableWorker);
        } else {
          setTimeout(checkForWorker, 10);
        }
      };
      checkForWorker();
    });
  }

  private executeTask(worker: Worker, task: PendingTask): void {
    const activeTask: ActiveTask = {
      ...task,
      worker,
      timeout: setTimeout(() => {
        this.handleTaskTimeout(task.id);
      }, task.config.timeout!),
    };

    this.activeTasks.set(task.id, activeTask);

    worker.postMessage({
      taskId: task.id,
      data: task.data,
    });
  }

  private setupWorkerListeners(worker: Worker): void {
    worker.onmessage = (event) => {
      const { taskId, success, data, error } = event.data;
      this.handleTaskComplete(taskId, success, data, error);
    };

    worker.onerror = (error) => {
      console.error("Worker error:", error);
      this.handleWorkerError(worker, error);
    };
  }

  private handleTaskComplete(
    taskId: string,
    success: boolean,
    data: any,
    error: any
  ): void {
    const task = this.activeTasks.get(taskId);
    if (!task) return;

    clearTimeout(task.timeout);
    this.activeTasks.delete(taskId);

    const result: TaskResult<any> = {
      success,
      data,
      error: error ? new Error(error) : undefined,
      duration: Date.now() - task.startTime,
    };

    if (success) {
      task.resolve(result);
    } else if (task.config.retries! > 0) {
      // Retry task
      task.config.retries!--;
      this.queueTask(task);
      this.processQueue();
    } else {
      task.reject(result);
    }

    // Process next task
    this.processQueue();
  }

  private handleTaskTimeout(taskId: string): void {
    const task = this.activeTasks.get(taskId);
    if (!task) return;

    this.activeTasks.delete(taskId);
    task.worker.terminate();

    // Remove terminated worker
    const workerIndex = this.workers.indexOf(task.worker);
    if (workerIndex >= 0) {
      this.workers.splice(workerIndex, 1);
    }

    const result: TaskResult<any> = {
      success: false,
      error: new Error("Task timeout"),
      duration: Date.now() - task.startTime,
    };

    task.reject(result);
    this.processQueue();
  }

  private handleWorkerError(worker: Worker, error: any): void {
    // Find and fail all tasks for this worker
    for (const [taskId, task] of this.activeTasks) {
      if (task.worker === worker) {
        clearTimeout(task.timeout);
        this.activeTasks.delete(taskId);

        const result: TaskResult<any> = {
          success: false,
          error: new Error(error.message || "Worker error"),
          duration: Date.now() - task.startTime,
        };

        task.reject(result);
      }
    }

    // Remove failed worker
    const workerIndex = this.workers.indexOf(worker);
    if (workerIndex >= 0) {
      this.workers.splice(workerIndex, 1);
    }

    this.processQueue();
  }

  private isWorkerBusy(worker: Worker): boolean {
    return Array.from(this.activeTasks.values()).some(
      (task) => task.worker === worker
    );
  }

  private generateTaskId(): string {
    return `task_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  terminate(): void {
    this.workers.forEach((worker) => worker.terminate());
    this.workers = [];
    this.taskQueue = [];
    this.activeTasks.clear();
  }
}

interface PendingTask {
  id: string;
  data: any;
  config: Required<TaskConfig>;
  resolve: (result: TaskResult<any>) => void;
  reject: (result: TaskResult<any>) => void;
  startTime: number;
}

interface ActiveTask extends PendingTask {
  worker: Worker;
  timeout: number;
}
```

## Concurrent Data Structures

### Thread-Safe Collections

```typescript
class ConcurrentMap<K, V> {
  private map = new Map<K, V>();
  private lock = new AsyncLock();

  async set(key: K, value: V): Promise<void> {
    await this.lock.acquire(async () => {
      this.map.set(key, value);
    });
  }

  async get(key: K): Promise<V | undefined> {
    return this.lock.acquire(async () => {
      return this.map.get(key);
    });
  }

  async has(key: K): Promise<boolean> {
    return this.lock.acquire(async () => {
      return this.map.has(key);
    });
  }

  async delete(key: K): Promise<boolean> {
    return this.lock.acquire(async () => {
      return this.map.delete(key);
    });
  }

  async clear(): Promise<void> {
    await this.lock.acquire(async () => {
      this.map.clear();
    });
  }

  async size(): Promise<number> {
    return this.lock.acquire(async () => {
      return this.map.size;
    });
  }

  async entries(): Promise<[K, V][]> {
    return this.lock.acquire(async () => {
      return Array.from(this.map.entries());
    });
  }
}

class AsyncLock {
  private locked = false;
  private queue: (() => void)[] = [];

  async acquire<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise(async (resolve, reject) => {
      const execute = async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        } finally {
          this.release();
        }
      };

      if (this.locked) {
        this.queue.push(execute);
      } else {
        this.locked = true;
        execute();
      }
    });
  }

  private release(): void {
    if (this.queue.length > 0) {
      const next = this.queue.shift()!;
      next();
    } else {
      this.locked = false;
    }
  }
}

// Producer-Consumer pattern
class AsyncChannel<T> {
  private buffer: T[] = [];
  private producers: ((value: T | undefined) => void)[] = [];
  private consumers: ((value: T) => void)[] = [];
  private closed = false;
  private maxSize: number;

  constructor(maxSize = Infinity) {
    this.maxSize = maxSize;
  }

  async send(value: T): Promise<void> {
    if (this.closed) {
      throw new Error("Channel is closed");
    }

    return new Promise((resolve) => {
      if (this.consumers.length > 0) {
        const consumer = this.consumers.shift()!;
        consumer(value);
        resolve();
      } else if (this.buffer.length < this.maxSize) {
        this.buffer.push(value);
        resolve();
      } else {
        this.producers.push(resolve);
        this.buffer.push(value);
      }
    });
  }

  async receive(): Promise<T> {
    if (this.closed && this.buffer.length === 0) {
      throw new Error("Channel is closed and empty");
    }

    return new Promise((resolve) => {
      if (this.buffer.length > 0) {
        const value = this.buffer.shift()!;
        if (this.producers.length > 0) {
          const producer = this.producers.shift()!;
          producer(undefined);
        }
        resolve(value);
      } else if (!this.closed) {
        this.consumers.push(resolve);
      }
    });
  }

  close(): void {
    this.closed = true;
    this.consumers.forEach((consumer) => {
      if (this.buffer.length > 0) {
        consumer(this.buffer.shift()!);
      }
    });
    this.consumers = [];
  }

  get size(): number {
    return this.buffer.length;
  }

  get isClosed(): boolean {
    return this.closed;
  }
}
```

## Advanced Async Patterns

### Event-Driven Architecture

```typescript
interface AsyncEventHandler<T> {
  (data: T): Promise<void> | void;
}

class AsyncEventEmitter<T = any> {
  private listeners = new Map<string, AsyncEventHandler<T>[]>();
  private onceListeners = new Map<string, AsyncEventHandler<T>[]>();
  private maxListeners = 10;

  on(event: string, handler: AsyncEventHandler<T>): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }

    const handlers = this.listeners.get(event)!;
    if (handlers.length >= this.maxListeners) {
      console.warn(
        `Max listeners (${this.maxListeners}) exceeded for event: ${event}`
      );
    }

    handlers.push(handler);

    return () => this.off(event, handler);
  }

  once(event: string, handler: AsyncEventHandler<T>): () => void {
    if (!this.onceListeners.has(event)) {
      this.onceListeners.set(event, []);
    }

    this.onceListeners.get(event)!.push(handler);

    return () => {
      const handlers = this.onceListeners.get(event);
      if (handlers) {
        const index = handlers.indexOf(handler);
        if (index >= 0) {
          handlers.splice(index, 1);
        }
      }
    };
  }

  off(event: string, handler: AsyncEventHandler<T>): void {
    const handlers = this.listeners.get(event);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index >= 0) {
        handlers.splice(index, 1);
      }
    }
  }

  async emit(event: string, data: T): Promise<void> {
    const handlers: AsyncEventHandler<T>[] = [];

    // Regular listeners
    const regularHandlers = this.listeners.get(event);
    if (regularHandlers) {
      handlers.push(...regularHandlers);
    }

    // Once listeners
    const onceHandlers = this.onceListeners.get(event);
    if (onceHandlers) {
      handlers.push(...onceHandlers);
      this.onceListeners.delete(event); // Clear once listeners
    }

    // Execute all handlers
    const promises = handlers.map(async (handler) => {
      try {
        await handler(data);
      } catch (error) {
        console.error(`Error in event handler for '${event}':`, error);
      }
    });

    await Promise.all(promises);
  }

  removeAllListeners(event?: string): void {
    if (event) {
      this.listeners.delete(event);
      this.onceListeners.delete(event);
    } else {
      this.listeners.clear();
      this.onceListeners.clear();
    }
  }

  listenerCount(event: string): number {
    const regular = this.listeners.get(event)?.length || 0;
    const once = this.onceListeners.get(event)?.length || 0;
    return regular + once;
  }

  setMaxListeners(max: number): void {
    this.maxListeners = max;
  }
}

// Usage example
interface AppEvents {
  "user-login": { userId: string; timestamp: Date };
  "data-updated": { table: string; records: number };
  error: { message: string; stack?: string };
}

class AppEventBus extends AsyncEventEmitter<AppEvents[keyof AppEvents]> {
  async emitUserLogin(userId: string): Promise<void> {
    await this.emit("user-login", { userId, timestamp: new Date() });
  }

  async emitDataUpdated(table: string, records: number): Promise<void> {
    await this.emit("data-updated", { table, records });
  }

  async emitError(message: string, stack?: string): Promise<void> {
    await this.emit("error", { message, stack });
  }
}

const eventBus = new AppEventBus();

// Event handlers
eventBus.on("user-login", async (data) => {
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
  // Update user analytics
  await updateUserAnalytics(data.userId);
});

eventBus.on("data-updated", async (data) => {
  console.log(`${data.records} records updated in ${data.table}`);
  // Invalidate cache
  await invalidateCache(data.table);
});

eventBus.on("error", async (data) => {
  console.error("Application error:", data.message);
  // Log to error tracking service
  await logError(data);
});
```

## Conclusion

Advanced async programming and concurrency in ArkTS enables:

- Sophisticated promise-based control flow
- Efficient worker thread management
- Thread-safe concurrent data structures
- Event-driven asynchronous architectures
- Scalable task processing systems
- Resource-efficient parallel execution

These patterns provide the foundation for building high-performance, responsive applications that handle complex asynchronous operations efficiently.
