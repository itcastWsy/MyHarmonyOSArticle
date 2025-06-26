# 46. ArkTS Concurrency and Multi-threading

## Introduction

Concurrency and multi-threading are essential for building high-performance HarmonyOS applications that can handle multiple tasks simultaneously without blocking the user interface. This article explores ArkTS's concurrency models, worker threads, and advanced synchronization techniques.

## Concurrency Fundamentals

### Thread Pool Management

```typescript
// Advanced thread pool implementation for ArkTS
export interface TaskDefinition<T = any> {
  id: string;
  execute: () => Promise<T>;
  priority: TaskPriority;
  timeout?: number;
  retries?: number;
  dependencies?: string[];
  metadata?: Record<string, any>;
}

export enum TaskPriority {
  Low = 1,
  Normal = 2,
  High = 3,
  Critical = 4,
}

export enum TaskStatus {
  Pending = "pending",
  Running = "running",
  Completed = "completed",
  Failed = "failed",
  Cancelled = "cancelled",
  Timeout = "timeout",
}

export interface TaskResult<T = any> {
  taskId: string;
  status: TaskStatus;
  result?: T;
  error?: Error;
  executionTime: number;
  attempts: number;
}

export class ThreadPoolManager {
  private static instance: ThreadPoolManager;
  private maxThreads: number;
  private activeThreads: number = 0;
  private taskQueue: TaskDefinition[] = [];
  private runningTasks: Map<string, TaskExecution> = new Map();
  private completedTasks: Map<string, TaskResult> = new Map();
  private workers: Worker[] = [];

  static getInstance(
    maxThreads: number = navigator.hardwareConcurrency || 4
  ): ThreadPoolManager {
    if (!this.instance) {
      this.instance = new ThreadPoolManager(maxThreads);
    }
    return this.instance;
  }

  constructor(maxThreads: number) {
    this.maxThreads = maxThreads;
    this.initializeWorkers();
  }

  private initializeWorkers(): void {
    for (let i = 0; i < this.maxThreads; i++) {
      try {
        // Create worker with proper error handling
        const worker = new Worker("js/worker.js");
        worker.onmessage = this.handleWorkerMessage.bind(this);
        worker.onerror = this.handleWorkerError.bind(this);
        this.workers.push(worker);
      } catch (error) {
        console.warn("Failed to create worker:", error);
      }
    }
  }

  async submitTask<T>(task: TaskDefinition<T>): Promise<TaskResult<T>> {
    return new Promise((resolve, reject) => {
      // Check for dependency resolution
      if (
        task.dependencies &&
        !this.areDependenciesResolved(task.dependencies)
      ) {
        this.taskQueue.push(task);
        // Will be resolved when dependencies complete
        this.setupDependencyWatch(task.id, resolve, reject);
        return;
      }

      // Add to queue with priority ordering
      this.insertTaskByPriority(task);
      this.setupTaskCompletion(task.id, resolve, reject);

      // Try to execute immediately if threads available
      this.tryExecuteNextTask();
    });
  }

  private insertTaskByPriority(task: TaskDefinition): void {
    let insertIndex = this.taskQueue.length;

    for (let i = 0; i < this.taskQueue.length; i++) {
      if (task.priority > this.taskQueue[i].priority) {
        insertIndex = i;
        break;
      }
    }

    this.taskQueue.splice(insertIndex, 0, task);
  }

  private areDependenciesResolved(dependencies: string[]): boolean {
    return dependencies.every((depId) => {
      const result = this.completedTasks.get(depId);
      return result?.status === TaskStatus.Completed;
    });
  }

  private setupDependencyWatch(
    taskId: string,
    resolve: (result: TaskResult) => void,
    reject: (error: Error) => void
  ): void {
    const checkDependencies = () => {
      const task = this.taskQueue.find((t) => t.id === taskId);
      if (!task) return;

      if (this.areDependenciesResolved(task.dependencies!)) {
        // Remove from queue and execute
        const index = this.taskQueue.indexOf(task);
        this.taskQueue.splice(index, 1);
        this.insertTaskByPriority(task);
        this.setupTaskCompletion(taskId, resolve, reject);
        this.tryExecuteNextTask();
      } else {
        // Check again later
        setTimeout(checkDependencies, 100);
      }
    };

    setTimeout(checkDependencies, 100);
  }

  private setupTaskCompletion(
    taskId: string,
    resolve: (result: TaskResult) => void,
    reject: (error: Error) => void
  ): void {
    const checkCompletion = () => {
      const result = this.completedTasks.get(taskId);
      if (result) {
        if (result.status === TaskStatus.Completed) {
          resolve(result);
        } else {
          reject(
            result.error ||
              new Error(`Task failed with status: ${result.status}`)
          );
        }
      } else {
        setTimeout(checkCompletion, 50);
      }
    };

    setTimeout(checkCompletion, 50);
  }

  private tryExecuteNextTask(): void {
    if (this.activeThreads >= this.maxThreads || this.taskQueue.length === 0) {
      return;
    }

    const task = this.taskQueue.shift()!;
    this.executeTask(task);
  }

  private async executeTask(task: TaskDefinition): Promise<void> {
    this.activeThreads++;

    const execution: TaskExecution = {
      task,
      startTime: Date.now(),
      attempts: 0,
      worker: this.getAvailableWorker(),
    };

    this.runningTasks.set(task.id, execution);

    try {
      const result = await this.runTaskWithTimeout(task, execution.worker);

      this.completedTasks.set(task.id, {
        taskId: task.id,
        status: TaskStatus.Completed,
        result,
        executionTime: Date.now() - execution.startTime,
        attempts: execution.attempts + 1,
      });
    } catch (error) {
      await this.handleTaskError(task, execution, error as Error);
    } finally {
      this.runningTasks.delete(task.id);
      this.activeThreads--;
      this.tryExecuteNextTask();
    }
  }

  private async runTaskWithTimeout(
    task: TaskDefinition,
    worker?: Worker
  ): Promise<any> {
    const timeout = task.timeout || 30000; // 30 seconds default

    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error("Task timeout")), timeout);
    });

    let taskPromise: Promise<any>;

    if (worker && this.isComputeIntensiveTask(task)) {
      // Run in worker thread
      taskPromise = this.executeInWorker(worker, task);
    } else {
      // Run in main thread
      taskPromise = task.execute();
    }

    return Promise.race([taskPromise, timeoutPromise]);
  }

  private isComputeIntensiveTask(task: TaskDefinition): boolean {
    // Heuristic to determine if task should run in worker
    return (
      task.metadata?.computeIntensive === true ||
      task.metadata?.estimatedDuration > 100
    );
  }

  private executeInWorker(worker: Worker, task: TaskDefinition): Promise<any> {
    return new Promise((resolve, reject) => {
      const messageId = `${task.id}_${Date.now()}`;

      const handleResponse = (event: MessageEvent) => {
        if (event.data.messageId === messageId) {
          worker.removeEventListener("message", handleResponse);

          if (event.data.error) {
            reject(new Error(event.data.error));
          } else {
            resolve(event.data.result);
          }
        }
      };

      worker.addEventListener("message", handleResponse);

      worker.postMessage({
        messageId,
        taskId: task.id,
        taskFunction: task.execute.toString(),
        metadata: task.metadata,
      });
    });
  }

  private async handleTaskError(
    task: TaskDefinition,
    execution: TaskExecution,
    error: Error
  ): Promise<void> {
    execution.attempts++;

    const maxRetries = task.retries || 0;

    if (execution.attempts <= maxRetries) {
      console.warn(
        `Task ${task.id} failed (attempt ${execution.attempts}), retrying...`
      );

      // Exponential backoff
      const delay = Math.pow(2, execution.attempts - 1) * 1000;
      await this.delay(delay);

      // Retry the task
      this.taskQueue.unshift(task);
      return;
    }

    // Task failed permanently
    this.completedTasks.set(task.id, {
      taskId: task.id,
      status: TaskStatus.Failed,
      error,
      executionTime: Date.now() - execution.startTime,
      attempts: execution.attempts,
    });
  }

  private getAvailableWorker(): Worker | undefined {
    // Simple round-robin worker selection
    return this.workers[this.activeThreads % this.workers.length];
  }

  private handleWorkerMessage(event: MessageEvent): void {
    // Handle worker responses
    console.log("Worker message:", event.data);
  }

  private handleWorkerError(error: ErrorEvent): void {
    console.error("Worker error:", error);
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }

  cancelTask(taskId: string): boolean {
    // Remove from queue
    const queueIndex = this.taskQueue.findIndex((t) => t.id === taskId);
    if (queueIndex > -1) {
      this.taskQueue.splice(queueIndex, 1);

      this.completedTasks.set(taskId, {
        taskId,
        status: TaskStatus.Cancelled,
        executionTime: 0,
        attempts: 0,
      });

      return true;
    }

    // Cancel running task
    const execution = this.runningTasks.get(taskId);
    if (execution) {
      // Signal cancellation to worker if applicable
      if (execution.worker) {
        execution.worker.postMessage({ type: "cancel", taskId });
      }

      return true;
    }

    return false;
  }

  getTaskStatus(taskId: string): TaskStatus {
    if (this.runningTasks.has(taskId)) {
      return TaskStatus.Running;
    }

    const completed = this.completedTasks.get(taskId);
    if (completed) {
      return completed.status;
    }

    const pending = this.taskQueue.find((t) => t.id === taskId);
    return pending ? TaskStatus.Pending : TaskStatus.Failed;
  }

  getPoolStats(): PoolStatistics {
    return {
      maxThreads: this.maxThreads,
      activeThreads: this.activeThreads,
      queuedTasks: this.taskQueue.length,
      completedTasks: this.completedTasks.size,
      failedTasks: Array.from(this.completedTasks.values()).filter(
        (t) => t.status === TaskStatus.Failed
      ).length,
    };
  }

  shutdown(): void {
    // Cancel all pending tasks
    this.taskQueue.forEach((task) => {
      this.completedTasks.set(task.id, {
        taskId: task.id,
        status: TaskStatus.Cancelled,
        executionTime: 0,
        attempts: 0,
      });
    });

    this.taskQueue = [];

    // Terminate workers
    this.workers.forEach((worker) => {
      worker.terminate();
    });

    this.workers = [];
  }
}

// Supporting interfaces
interface TaskExecution {
  task: TaskDefinition;
  startTime: number;
  attempts: number;
  worker?: Worker;
}

interface PoolStatistics {
  maxThreads: number;
  activeThreads: number;
  queuedTasks: number;
  completedTasks: number;
  failedTasks: number;
}
```

### Advanced Synchronization Primitives

```typescript
// Mutex implementation for critical sections
export class Mutex {
  private locked: boolean = false;
  private waitQueue: Array<() => void> = [];

  async acquire(): Promise<() => void> {
    return new Promise<() => void>((resolve) => {
      const tryAcquire = () => {
        if (!this.locked) {
          this.locked = true;
          resolve(() => this.release());
        } else {
          this.waitQueue.push(tryAcquire);
        }
      };

      tryAcquire();
    });
  }

  private release(): void {
    this.locked = false;
    const next = this.waitQueue.shift();
    if (next) {
      // Use setTimeout to avoid stack overflow
      setTimeout(next, 0);
    }
  }

  async withLock<T>(fn: () => Promise<T>): Promise<T> {
    const release = await this.acquire();
    try {
      return await fn();
    } finally {
      release();
    }
  }
}

// Semaphore for resource counting
export class Semaphore {
  private count: number;
  private waitQueue: Array<() => void> = [];

  constructor(initialCount: number) {
    this.count = initialCount;
  }

  async acquire(): Promise<() => void> {
    return new Promise<() => void>((resolve) => {
      const tryAcquire = () => {
        if (this.count > 0) {
          this.count--;
          resolve(() => this.release());
        } else {
          this.waitQueue.push(tryAcquire);
        }
      };

      tryAcquire();
    });
  }

  private release(): void {
    this.count++;
    const next = this.waitQueue.shift();
    if (next) {
      setTimeout(next, 0);
    }
  }

  async withResource<T>(fn: () => Promise<T>): Promise<T> {
    const release = await this.acquire();
    try {
      return await fn();
    } finally {
      release();
    }
  }

  getAvailableCount(): number {
    return this.count;
  }
}

// Condition variable for thread coordination
export class ConditionVariable {
  private waitQueue: Array<() => void> = [];
  private mutex: Mutex;

  constructor(mutex: Mutex) {
    this.mutex = mutex;
  }

  async wait(): Promise<void> {
    return new Promise<void>((resolve) => {
      this.waitQueue.push(resolve);
    });
  }

  notify(): void {
    const waiter = this.waitQueue.shift();
    if (waiter) {
      setTimeout(waiter, 0);
    }
  }

  notifyAll(): void {
    const waiters = [...this.waitQueue];
    this.waitQueue = [];
    waiters.forEach((waiter) => setTimeout(waiter, 0));
  }
}

// ReadWrite lock for concurrent reading
export class ReadWriteLock {
  private readers: number = 0;
  private writer: boolean = false;
  private readQueue: Array<() => void> = [];
  private writeQueue: Array<() => void> = [];

  async acquireRead(): Promise<() => void> {
    return new Promise<() => void>((resolve) => {
      const tryAcquireRead = () => {
        if (!this.writer && this.writeQueue.length === 0) {
          this.readers++;
          resolve(() => this.releaseRead());
        } else {
          this.readQueue.push(tryAcquireRead);
        }
      };

      tryAcquireRead();
    });
  }

  async acquireWrite(): Promise<() => void> {
    return new Promise<() => void>((resolve) => {
      const tryAcquireWrite = () => {
        if (!this.writer && this.readers === 0) {
          this.writer = true;
          resolve(() => this.releaseWrite());
        } else {
          this.writeQueue.push(tryAcquireWrite);
        }
      };

      tryAcquireWrite();
    });
  }

  private releaseRead(): void {
    this.readers--;
    this.processQueues();
  }

  private releaseWrite(): void {
    this.writer = false;
    this.processQueues();
  }

  private processQueues(): void {
    // Process write queue first (writers have priority)
    if (!this.writer && this.readers === 0 && this.writeQueue.length > 0) {
      const nextWriter = this.writeQueue.shift()!;
      setTimeout(nextWriter, 0);
    }
    // Process read queue if no writers waiting
    else if (
      !this.writer &&
      this.writeQueue.length === 0 &&
      this.readQueue.length > 0
    ) {
      const readers = [...this.readQueue];
      this.readQueue = [];
      readers.forEach((reader) => setTimeout(reader, 0));
    }
  }
}
```

### Producer-Consumer Pattern

```typescript
// Thread-safe producer-consumer implementation
export class ConcurrentQueue<T> {
  private queue: T[] = [];
  private maxSize: number;
  private producers: Array<() => void> = [];
  private consumers: Array<(item: T) => void> = [];
  private mutex = new Mutex();

  constructor(maxSize: number = 1000) {
    this.maxSize = maxSize;
  }

  async produce(item: T): Promise<void> {
    await this.mutex.withLock(async () => {
      if (this.queue.length >= this.maxSize) {
        // Wait for space
        await new Promise<void>((resolve) => {
          this.producers.push(resolve);
        });
      }

      this.queue.push(item);

      // Notify waiting consumers
      const consumer = this.consumers.shift();
      if (consumer) {
        setTimeout(() => consumer(item), 0);
      }
    });
  }

  async consume(): Promise<T> {
    return await this.mutex.withLock(async () => {
      if (this.queue.length === 0) {
        // Wait for items
        return new Promise<T>((resolve) => {
          this.consumers.push(resolve);
        });
      }

      const item = this.queue.shift()!;

      // Notify waiting producers
      const producer = this.producers.shift();
      if (producer) {
        setTimeout(producer, 0);
      }

      return item;
    });
  }

  async peek(): Promise<T | undefined> {
    return await this.mutex.withLock(async () => {
      return this.queue[0];
    });
  }

  async size(): Promise<number> {
    return await this.mutex.withLock(async () => {
      return this.queue.length;
    });
  }

  async isEmpty(): Promise<boolean> {
    return await this.mutex.withLock(async () => {
      return this.queue.length === 0;
    });
  }

  async isFull(): Promise<boolean> {
    return await this.mutex.withLock(async () => {
      return this.queue.length >= this.maxSize;
    });
  }
}

// Producer-Consumer coordinator
export class ProducerConsumerSystem<T> {
  private queue: ConcurrentQueue<T>;
  private producers: Worker[] = [];
  private consumers: Worker[] = [];
  private isRunning: boolean = false;
  private stats = {
    produced: 0,
    consumed: 0,
    errors: 0,
  };

  constructor(queueSize: number = 1000) {
    this.queue = new ConcurrentQueue<T>(queueSize);
  }

  async startProducer(
    producerFn: () => Promise<T>,
    interval: number = 1000
  ): Promise<() => void> {
    const worker = new Worker("js/producer-worker.js");
    this.producers.push(worker);

    const runProducer = async () => {
      while (this.isRunning) {
        try {
          const item = await producerFn();
          await this.queue.produce(item);
          this.stats.produced++;
        } catch (error) {
          console.error("Producer error:", error);
          this.stats.errors++;
        }

        await this.delay(interval);
      }
    };

    // Start producer
    runProducer();

    // Return stop function
    return () => {
      const index = this.producers.indexOf(worker);
      if (index > -1) {
        this.producers.splice(index, 1);
        worker.terminate();
      }
    };
  }

  async startConsumer(
    consumerFn: (item: T) => Promise<void>,
    batchSize: number = 1
  ): Promise<() => void> {
    const worker = new Worker("js/consumer-worker.js");
    this.consumers.push(worker);

    const runConsumer = async () => {
      while (this.isRunning) {
        try {
          const items: T[] = [];

          // Consume batch
          for (let i = 0; i < batchSize; i++) {
            if (await this.queue.isEmpty()) break;
            const item = await this.queue.consume();
            items.push(item);
          }

          // Process batch
          if (items.length > 0) {
            await Promise.all(items.map((item) => consumerFn(item)));
            this.stats.consumed += items.length;
          } else {
            // No items, wait a bit
            await this.delay(100);
          }
        } catch (error) {
          console.error("Consumer error:", error);
          this.stats.errors++;
        }
      }
    };

    // Start consumer
    runConsumer();

    // Return stop function
    return () => {
      const index = this.consumers.indexOf(worker);
      if (index > -1) {
        this.consumers.splice(index, 1);
        worker.terminate();
      }
    };
  }

  start(): void {
    this.isRunning = true;
  }

  stop(): void {
    this.isRunning = false;

    // Terminate all workers
    [...this.producers, ...this.consumers].forEach((worker) => {
      worker.terminate();
    });

    this.producers = [];
    this.consumers = [];
  }

  getStats(): {
    produced: number;
    consumed: number;
    errors: number;
    queueSize: number;
    backlog: number;
  } {
    return {
      ...this.stats,
      queueSize: this.queue.size(),
      backlog: this.stats.produced - this.stats.consumed,
    };
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}
```

## Parallel Processing Examples

### Parallel Data Processing Component

```typescript
@Component
export struct ParallelProcessingDemo {
  @State private isProcessing: boolean = false;
  @State private progress: number = 0;
  @State private results: ProcessingResult[] = [];
  @State private stats: PoolStatistics | null = null;

  private threadPool = ThreadPoolManager.getInstance(4);
  private processingData: number[] = [];

  aboutToAppear() {
    this.generateTestData();
  }

  private generateTestData(): void {
    this.processingData = Array.from({ length: 10000 }, () => Math.random() * 1000);
  }

  private async startParallelProcessing(): Promise<void> {
    this.isProcessing = true;
    this.progress = 0;
    this.results = [];

    const chunkSize = 1000;
    const chunks = this.chunkArray(this.processingData, chunkSize);
    const totalChunks = chunks.length;

    try {
      // Submit all tasks
      const taskPromises = chunks.map((chunk, index) => {
        return this.threadPool.submitTask({
          id: `process-chunk-${index}`,
          execute: async () => {
            return await this.processChunk(chunk);
          },
          priority: TaskPriority.Normal,
          timeout: 10000,
          metadata: {
            computeIntensive: true,
            estimatedDuration: 200,
            chunkIndex: index,
            chunkSize: chunk.length
          }
        });
      });

      // Process results as they complete
      for (let i = 0; i < taskPromises.length; i++) {
        const result = await taskPromises[i];
        this.results.push({
          chunkIndex: i,
          result: result.result,
          executionTime: result.executionTime,
          status: result.status
        });

        this.progress = ((i + 1) / totalChunks) * 100;
        this.stats = this.threadPool.getPoolStats();
      }

      console.log('Parallel processing completed');
    } catch (error) {
      console.error('Parallel processing failed:', error);
    } finally {
      this.isProcessing = false;
    }
  }

  private async processChunk(chunk: number[]): Promise<ChunkResult> {
    // Simulate compute-intensive processing
    await this.delay(Math.random() * 500 + 100);

    const sum = chunk.reduce((acc, val) => acc + val, 0);
    const average = sum / chunk.length;
    const min = Math.min(...chunk);
    const max = Math.max(...chunk);

    // Simulate more processing
    const sorted = [...chunk].sort((a, b) => a - b);
    const median = sorted[Math.floor(sorted.length / 2)];

    return {
      sum,
      average,
      min,
      max,
      median,
      count: chunk.length
    };
  }

  private chunkArray<T>(array: T[], chunkSize: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += chunkSize) {
      chunks.push(array.slice(i, i + chunkSize));
    }
    return chunks;
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  build() {
    ScrollView() {
      Column({ space: 16 }) {
        Text('Parallel Processing Demo')
          .fontSize(24)
          .fontWeight(FontWeight.Bold);

        Text(`Data size: ${this.processingData.length.toLocaleString()} items`)
          .fontSize(16)
          .fontColor(Color.Gray);

        // Processing controls
        Row({ space: 12 }) {
          Button('Start Processing')
            .onClick(() => this.startParallelProcessing())
            .enabled(!this.isProcessing);

          Button('Generate New Data')
            .onClick(() => this.generateTestData())
            .enabled(!this.isProcessing);
        }

        // Progress indicator
        if (this.isProcessing) {
          Column({ space: 8 }) {
            Text(`Progress: ${this.progress.toFixed(1)}%`)
              .fontSize(16);

            Progress({
              value: this.progress,
              total: 100,
              type: ProgressType.Linear
            })
              .width('100%')
              .color(Color.Blue);
          }
          .width('100%');
        }

        // Thread pool statistics
        if (this.stats) {
          Card() {
            Column({ space: 8 }) {
              Text('Thread Pool Statistics')
                .fontSize(18)
                .fontWeight(FontWeight.Bold);

              Row() {
                Text('Active Threads:')
                  .width('50%');
                Text(`${this.stats.activeThreads}/${this.stats.maxThreads}`)
                  .fontColor(Color.Blue);
              }
              .width('100%');

              Row() {
                Text('Queued Tasks:')
                  .width('50%');
                Text(`${this.stats.queuedTasks}`)
                  .fontColor(Color.Orange);
              }
              .width('100%');

              Row() {
                Text('Completed:')
                  .width('50%');
                Text(`${this.stats.completedTasks}`)
                  .fontColor(Color.Green);
              }
              .width('100%');

              Row() {
                Text('Failed:')
                  .width('50%');
                Text(`${this.stats.failedTasks}`)
                  .fontColor(Color.Red);
              }
              .width('100%');
            }
            .alignItems(HorizontalAlign.Start)
            .padding(16);
          }
        }

        // Results display
        if (this.results.length > 0) {
          Column({ space: 12 }) {
            Text(`Results (${this.results.length} chunks processed)`)
              .fontSize(18)
              .fontWeight(FontWeight.Bold);

            List({ space: 8 }) {
              ForEach(this.results.slice(0, 10), (result: ProcessingResult, index: number) => {
                ListItem() {
                  Card() {
                    Column({ space: 8 }) {
                      Row() {
                        Text(`Chunk ${result.chunkIndex}`)
                          .fontSize(16)
                          .fontWeight(FontWeight.Medium)
                          .layoutWeight(1);

                        Text(`${result.executionTime}ms`)
                          .fontSize(12)
                          .fontColor(Color.Gray);
                      }
                      .width('100%');

                      if (result.result) {
                        Column({ space: 4 }) {
                          Text(`Sum: ${result.result.sum.toFixed(2)}`)
                            .fontSize(12);
                          Text(`Avg: ${result.result.average.toFixed(2)}`)
                            .fontSize(12);
                          Text(`Range: ${result.result.min.toFixed(2)} - ${result.result.max.toFixed(2)}`)
                            .fontSize(12);
                        }
                        .alignItems(HorizontalAlign.Start);
                      }
                    }
                    .alignItems(HorizontalAlign.Start)
                    .padding(12);
                  }
                }
              }, (result: ProcessingResult) => `result-${result.chunkIndex}`)
            }
            .height(300);

            if (this.results.length > 10) {
              Text(`... and ${this.results.length - 10} more results`)
                .fontSize(12)
                .fontColor(Color.Gray)
                .textAlign(TextAlign.Center)
                .width('100%');
            }
          }
        }
      }
    }
    .width('100%')
    .height('100%')
    .padding(16);
  }
}

// Supporting interfaces
interface ChunkResult {
  sum: number;
  average: number;
  min: number;
  max: number;
  median: number;
  count: number;
}

interface ProcessingResult {
  chunkIndex: number;
  result?: ChunkResult;
  executionTime: number;
  status: TaskStatus;
}
```

## Best Practices

1. **Thread Safety**: Always use proper synchronization when accessing shared resources
2. **Resource Management**: Properly manage thread pools and avoid creating excessive threads
3. **Error Handling**: Implement robust error handling for concurrent operations
4. **Performance Monitoring**: Monitor thread pool utilization and task completion times
5. **Deadlock Prevention**: Design synchronization carefully to avoid deadlocks
6. **Memory Management**: Be aware of memory usage in multi-threaded scenarios
7. **Testing**: Thoroughly test concurrent code with race condition detection tools

## Conclusion

Effective concurrency and multi-threading in ArkTS enable building high-performance HarmonyOS applications that can handle multiple tasks efficiently. By implementing proper thread pool management, synchronization primitives, and parallel processing patterns, developers can create responsive applications that make optimal use of device resources.

The key to successful concurrent programming lies in understanding synchronization requirements, proper resource management, and thorough testing to ensure thread safety and optimal performance across different scenarios and device capabilities.
