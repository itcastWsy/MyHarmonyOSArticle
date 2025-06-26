# ArkUI Advanced Concurrency

## Introduction

Advanced Concurrency in ArkUI encompasses sophisticated concurrent programming patterns, parallel processing, thread management, and asynchronous coordination. This guide covers advanced concurrency models, thread safety, and performance optimization.

## Concurrency Framework

```typescript
interface ConcurrentTask<T> {
  id: string;
  priority: TaskPriority;
  dependencies: string[];
  timeout: number;
  retryPolicy: RetryPolicy;
  executor: () => Promise<T>;
  onProgress?: (progress: number) => void;
  onCancel?: () => void;
}

type TaskPriority = "low" | "normal" | "high" | "critical";

interface TaskResult<T> {
  taskId: string;
  status: TaskStatus;
  result?: T;
  error?: Error;
  startTime: number;
  endTime: number;
  attempts: number;
}

type TaskStatus = "pending" | "running" | "completed" | "failed" | "cancelled";

class AdvancedConcurrencyManager {
  private taskQueue = new PriorityQueue<ConcurrentTask<any>>();
  private runningTasks = new Map<string, TaskRunner<any>>();
  private completedTasks = new Map<string, TaskResult<any>>();
  private workerPool = new WorkerPool();
  private semaphore = new Semaphore(10); // Max concurrent tasks
  private eventBus = new EventBus();

  async submitTask<T>(task: ConcurrentTask<T>): Promise<string> {
    const taskId = `task_${Date.now()}_${Math.random()}`;
    task.id = taskId;

    // Check dependencies
    await this.waitForDependencies(task.dependencies);

    // Add to queue
    this.taskQueue.enqueue(task, this.getPriorityWeight(task.priority));

    // Process queue
    this.processQueue();

    return taskId;
  }

  async submitBatch<T>(tasks: ConcurrentTask<T>[]): Promise<string[]> {
    const taskIds: string[] = [];

    for (const task of tasks) {
      const taskId = await this.submitTask(task);
      taskIds.push(taskId);
    }

    return taskIds;
  }

  async executeParallel<T>(
    tasks: ConcurrentTask<T>[]
  ): Promise<TaskResult<T>[]> {
    const taskPromises = tasks.map((task) => this.submitTask(task));
    const taskIds = await Promise.all(taskPromises);

    return this.waitForTasks(taskIds);
  }

  async executeSequential<T>(
    tasks: ConcurrentTask<T>[]
  ): Promise<TaskResult<T>[]> {
    const results: TaskResult<T>[] = [];

    for (const task of tasks) {
      const taskId = await this.submitTask(task);
      const result = await this.waitForTask(taskId);
      results.push(result);
    }

    return results;
  }

  async executePipeline<T>(
    stages: ConcurrentTask<T>[]
  ): Promise<TaskResult<T>> {
    let previousResult: any = null;

    for (let i = 0; i < stages.length; i++) {
      const stage = { ...stages[i] };

      // Pass previous result to next stage
      if (i > 0) {
        const originalExecutor = stage.executor;
        stage.executor = () => originalExecutor.call(null, previousResult);
      }

      const taskId = await this.submitTask(stage);
      const result = await this.waitForTask(taskId);

      if (result.status === "failed") {
        throw new Error(`Pipeline failed at stage ${i}: ${result.error}`);
      }

      previousResult = result.result;
    }

    return this.completedTasks.get(stages[stages.length - 1].id)!;
  }

  async race<T>(tasks: ConcurrentTask<T>[]): Promise<TaskResult<T>> {
    const taskPromises = tasks.map((task) => this.submitTask(task));
    const taskIds = await Promise.all(taskPromises);

    return new Promise((resolve, reject) => {
      let resolved = false;

      for (const taskId of taskIds) {
        this.waitForTask(taskId).then((result) => {
          if (!resolved) {
            resolved = true;
            // Cancel other tasks
            taskIds.forEach((id) => {
              if (id !== taskId) this.cancelTask(id);
            });
            resolve(result);
          }
        });
      }
    });
  }

  createTaskGroup(): TaskGroup {
    return new TaskGroup(this);
  }

  async mapReduce<T, R>(
    data: T[],
    mapTask: (item: T) => Promise<R>,
    reduceTask: (results: R[]) => Promise<R>
  ): Promise<R> {
    // Map phase
    const mapTasks: ConcurrentTask<R>[] = data.map((item, index) => ({
      id: `map_${index}`,
      priority: "normal",
      dependencies: [],
      timeout: 30000,
      retryPolicy: { maxAttempts: 3, backoffMs: 1000 },
      executor: () => mapTask(item),
    }));

    const mapResults = await this.executeParallel(mapTasks);
    const mapValues = mapResults
      .filter((r) => r.status === "completed")
      .map((r) => r.result!);

    // Reduce phase
    const reduceTaskConfig: ConcurrentTask<R> = {
      id: "reduce",
      priority: "high",
      dependencies: [],
      timeout: 60000,
      retryPolicy: { maxAttempts: 1, backoffMs: 0 },
      executor: () => reduceTask(mapValues),
    };

    const reduceTaskId = await this.submitTask(reduceTaskConfig);
    const reduceResult = await this.waitForTask(reduceTaskId);

    if (reduceResult.status !== "completed") {
      throw new Error("Reduce phase failed");
    }

    return reduceResult.result!;
  }

  async scatter<T, R>(
    data: T[],
    workerCount: number,
    processor: (chunk: T[]) => Promise<R[]>
  ): Promise<R[]> {
    const chunkSize = Math.ceil(data.length / workerCount);
    const chunks: T[][] = [];

    for (let i = 0; i < data.length; i += chunkSize) {
      chunks.push(data.slice(i, i + chunkSize));
    }

    const processingTasks: ConcurrentTask<R[]>[] = chunks.map(
      (chunk, index) => ({
        id: `scatter_${index}`,
        priority: "normal",
        dependencies: [],
        timeout: 60000,
        retryPolicy: { maxAttempts: 2, backoffMs: 1000 },
        executor: () => processor(chunk),
      })
    );

    const results = await this.executeParallel(processingTasks);
    return results
      .filter((r) => r.status === "completed")
      .flatMap((r) => r.result!);
  }

  async waitForTask<T>(taskId: string): Promise<TaskResult<T>> {
    return new Promise((resolve) => {
      const checkResult = () => {
        const result = this.completedTasks.get(taskId);
        if (result) {
          resolve(result);
        } else {
          setTimeout(checkResult, 100);
        }
      };
      checkResult();
    });
  }

  async waitForTasks<T>(taskIds: string[]): Promise<TaskResult<T>[]> {
    const promises = taskIds.map((id) => this.waitForTask<T>(id));
    return Promise.all(promises);
  }

  cancelTask(taskId: string): boolean {
    const runner = this.runningTasks.get(taskId);
    if (runner) {
      runner.cancel();
      return true;
    }

    // Remove from queue if not started
    return this.taskQueue.remove((task) => task.id === taskId);
  }

  getTaskStatus(taskId: string): TaskStatus {
    if (this.completedTasks.has(taskId)) {
      return this.completedTasks.get(taskId)!.status;
    }
    if (this.runningTasks.has(taskId)) {
      return "running";
    }
    return "pending";
  }

  getMetrics(): ConcurrencyMetrics {
    return {
      totalTasks: this.completedTasks.size + this.runningTasks.size,
      runningTasks: this.runningTasks.size,
      completedTasks: this.completedTasks.size,
      failedTasks: Array.from(this.completedTasks.values()).filter(
        (t) => t.status === "failed"
      ).length,
      averageExecutionTime: this.calculateAverageExecutionTime(),
      throughput: this.calculateThroughput(),
      workerUtilization: this.workerPool.getUtilization(),
    };
  }

  private async processQueue(): Promise<void> {
    while (!this.taskQueue.isEmpty() && this.semaphore.tryAcquire()) {
      const task = this.taskQueue.dequeue();
      if (task) {
        const runner = new TaskRunner(task, this.semaphore, this.eventBus);
        this.runningTasks.set(task.id, runner);

        runner.execute().then((result) => {
          this.runningTasks.delete(task.id);
          this.completedTasks.set(task.id, result);
          this.eventBus.emit("taskCompleted", { taskId: task.id, result });
        });
      }
    }
  }

  private async waitForDependencies(dependencies: string[]): Promise<void> {
    if (dependencies.length === 0) return;

    const dependencyPromises = dependencies.map((depId) => {
      return new Promise<void>((resolve, reject) => {
        const checkDependency = () => {
          const result = this.completedTasks.get(depId);
          if (result) {
            if (result.status === "completed") {
              resolve();
            } else {
              reject(new Error(`Dependency ${depId} failed`));
            }
          } else {
            setTimeout(checkDependency, 100);
          }
        };
        checkDependency();
      });
    });

    await Promise.all(dependencyPromises);
  }

  private getPriorityWeight(priority: TaskPriority): number {
    const weights = { low: 1, normal: 2, high: 3, critical: 4 };
    return weights[priority];
  }

  private calculateAverageExecutionTime(): number {
    const completedTasks = Array.from(this.completedTasks.values());
    if (completedTasks.length === 0) return 0;

    const totalTime = completedTasks.reduce(
      (sum, task) => sum + (task.endTime - task.startTime),
      0
    );
    return totalTime / completedTasks.length;
  }

  private calculateThroughput(): number {
    const now = Date.now();
    const oneMinuteAgo = now - 60000;

    const recentTasks = Array.from(this.completedTasks.values()).filter(
      (task) => task.endTime > oneMinuteAgo
    );

    return recentTasks.length;
  }
}

class TaskGroup {
  private tasks: ConcurrentTask<any>[] = [];
  private manager: AdvancedConcurrencyManager;

  constructor(manager: AdvancedConcurrencyManager) {
    this.manager = manager;
  }

  addTask<T>(task: ConcurrentTask<T>): TaskGroup {
    this.tasks.push(task);
    return this;
  }

  async executeAll(): Promise<TaskResult<any>[]> {
    return this.manager.executeParallel(this.tasks);
  }

  async executeSequentially(): Promise<TaskResult<any>[]> {
    return this.manager.executeSequential(this.tasks);
  }

  async executePipeline(): Promise<TaskResult<any>> {
    return this.manager.executePipeline(this.tasks);
  }
}

class TaskRunner<T> {
  private task: ConcurrentTask<T>;
  private semaphore: Semaphore;
  private eventBus: EventBus;
  private cancelled = false;

  constructor(
    task: ConcurrentTask<T>,
    semaphore: Semaphore,
    eventBus: EventBus
  ) {
    this.task = task;
    this.semaphore = semaphore;
    this.eventBus = eventBus;
  }

  async execute(): Promise<TaskResult<T>> {
    const startTime = Date.now();
    let attempts = 0;
    let lastError: Error | undefined;

    while (attempts < (this.task.retryPolicy?.maxAttempts || 1)) {
      if (this.cancelled) {
        return this.createResult(
          "cancelled",
          startTime,
          undefined,
          undefined,
          attempts
        );
      }

      attempts++;

      try {
        this.eventBus.emit("taskStarted", {
          taskId: this.task.id,
          attempt: attempts,
        });

        const timeoutPromise = new Promise<never>((_, reject) => {
          setTimeout(
            () => reject(new Error("Task timeout")),
            this.task.timeout
          );
        });

        const taskPromise = this.task.executor();
        const result = await Promise.race([taskPromise, timeoutPromise]);

        this.semaphore.release();
        return this.createResult(
          "completed",
          startTime,
          result,
          undefined,
          attempts
        );
      } catch (error) {
        lastError = error as Error;

        if (attempts < (this.task.retryPolicy?.maxAttempts || 1)) {
          await this.delay(this.task.retryPolicy?.backoffMs || 0);
        }
      }
    }

    this.semaphore.release();
    return this.createResult(
      "failed",
      startTime,
      undefined,
      lastError,
      attempts
    );
  }

  cancel(): void {
    this.cancelled = true;
    if (this.task.onCancel) {
      this.task.onCancel();
    }
  }

  private createResult(
    status: TaskStatus,
    startTime: number,
    result?: T,
    error?: Error,
    attempts?: number
  ): TaskResult<T> {
    return {
      taskId: this.task.id,
      status,
      result,
      error,
      startTime,
      endTime: Date.now(),
      attempts: attempts || 0,
    };
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

class Semaphore {
  private permits: number;
  private waitQueue: (() => void)[] = [];

  constructor(permits: number) {
    this.permits = permits;
  }

  async acquire(): Promise<void> {
    return new Promise((resolve) => {
      if (this.permits > 0) {
        this.permits--;
        resolve();
      } else {
        this.waitQueue.push(resolve);
      }
    });
  }

  tryAcquire(): boolean {
    if (this.permits > 0) {
      this.permits--;
      return true;
    }
    return false;
  }

  release(): void {
    this.permits++;
    const next = this.waitQueue.shift();
    if (next) {
      this.permits--;
      next();
    }
  }
}

class PriorityQueue<T> {
  private items: { item: T; priority: number }[] = [];

  enqueue(item: T, priority: number): void {
    const queueItem = { item, priority };
    let added = false;

    for (let i = 0; i < this.items.length; i++) {
      if (queueItem.priority > this.items[i].priority) {
        this.items.splice(i, 0, queueItem);
        added = true;
        break;
      }
    }

    if (!added) {
      this.items.push(queueItem);
    }
  }

  dequeue(): T | undefined {
    const item = this.items.shift();
    return item?.item;
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }

  remove(predicate: (item: T) => boolean): boolean {
    const index = this.items.findIndex((queueItem) =>
      predicate(queueItem.item)
    );
    if (index >= 0) {
      this.items.splice(index, 1);
      return true;
    }
    return false;
  }
}

class WorkerPool {
  private workers: Worker[] = [];
  private availableWorkers: Worker[] = [];
  private busyWorkers: Set<Worker> = new Set();

  constructor(maxWorkers: number = 4) {
    for (let i = 0; i < maxWorkers; i++) {
      const worker = new Worker();
      this.workers.push(worker);
      this.availableWorkers.push(worker);
    }
  }

  async execute<T>(task: () => Promise<T>): Promise<T> {
    const worker = await this.acquireWorker();

    try {
      return await task();
    } finally {
      this.releaseWorker(worker);
    }
  }

  getUtilization(): number {
    return this.busyWorkers.size / this.workers.length;
  }

  private async acquireWorker(): Promise<Worker> {
    return new Promise((resolve) => {
      if (this.availableWorkers.length > 0) {
        const worker = this.availableWorkers.pop()!;
        this.busyWorkers.add(worker);
        resolve(worker);
      } else {
        // Wait for worker to become available
        setTimeout(() => this.acquireWorker().then(resolve), 10);
      }
    });
  }

  private releaseWorker(worker: Worker): void {
    this.busyWorkers.delete(worker);
    this.availableWorkers.push(worker);
  }
}

class EventBus {
  private listeners: Map<string, Function[]> = new Map();

  emit(event: string, data: any): void {
    const eventListeners = this.listeners.get(event);
    if (eventListeners) {
      eventListeners.forEach((listener) => listener(data));
    }
  }

  on(event: string, listener: Function): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(listener);
  }

  off(event: string, listener: Function): void {
    const eventListeners = this.listeners.get(event);
    if (eventListeners) {
      const index = eventListeners.indexOf(listener);
      if (index >= 0) {
        eventListeners.splice(index, 1);
      }
    }
  }
}

interface RetryPolicy {
  maxAttempts: number;
  backoffMs: number;
}

interface ConcurrencyMetrics {
  totalTasks: number;
  runningTasks: number;
  completedTasks: number;
  failedTasks: number;
  averageExecutionTime: number;
  throughput: number;
  workerUtilization: number;
}

// Mock Worker class
class Worker {
  // Implementation would depend on the actual worker system
}
```

## Concurrent Data Structures

```typescript
class ConcurrentMap<K, V> {
  private map = new Map<K, V>();
  private lock = new AsyncLock();

  async set(key: K, value: V): Promise<void> {
    await this.lock.acquire();
    try {
      this.map.set(key, value);
    } finally {
      this.lock.release();
    }
  }

  async get(key: K): Promise<V | undefined> {
    await this.lock.acquire();
    try {
      return this.map.get(key);
    } finally {
      this.lock.release();
    }
  }

  async delete(key: K): Promise<boolean> {
    await this.lock.acquire();
    try {
      return this.map.delete(key);
    } finally {
      this.lock.release();
    }
  }

  async size(): Promise<number> {
    await this.lock.acquire();
    try {
      return this.map.size;
    } finally {
      this.lock.release();
    }
  }
}

class ConcurrentQueue<T> {
  private queue: T[] = [];
  private lock = new AsyncLock();
  private waitingConsumers: ((item: T) => void)[] = [];

  async enqueue(item: T): Promise<void> {
    await this.lock.acquire();
    try {
      if (this.waitingConsumers.length > 0) {
        const consumer = this.waitingConsumers.shift()!;
        consumer(item);
      } else {
        this.queue.push(item);
      }
    } finally {
      this.lock.release();
    }
  }

  async dequeue(): Promise<T> {
    await this.lock.acquire();
    try {
      if (this.queue.length > 0) {
        return this.queue.shift()!;
      } else {
        return new Promise<T>((resolve) => {
          this.waitingConsumers.push(resolve);
          this.lock.release();
        });
      }
    } finally {
      if (this.queue.length > 0 || this.waitingConsumers.length === 0) {
        this.lock.release();
      }
    }
  }
}

class AsyncLock {
  private locked = false;
  private waitQueue: (() => void)[] = [];

  async acquire(): Promise<void> {
    return new Promise((resolve) => {
      if (!this.locked) {
        this.locked = true;
        resolve();
      } else {
        this.waitQueue.push(resolve);
      }
    });
  }

  release(): void {
    if (this.waitQueue.length > 0) {
      const next = this.waitQueue.shift()!;
      next();
    } else {
      this.locked = false;
    }
  }
}
```

## Advanced Patterns

```typescript
class ProducerConsumerPattern<T> {
  private buffer: ConcurrentQueue<T>;
  private producers: Set<() => Promise<T>> = new Set();
  private consumers: Set<(item: T) => Promise<void>> = new Set();
  private running = false;

  constructor(bufferSize: number = 100) {
    this.buffer = new ConcurrentQueue<T>();
  }

  addProducer(producer: () => Promise<T>): void {
    this.producers.add(producer);
  }

  addConsumer(consumer: (item: T) => Promise<void>): void {
    this.consumers.add(consumer);
  }

  async start(): Promise<void> {
    this.running = true;

    // Start producers
    for (const producer of this.producers) {
      this.runProducer(producer);
    }

    // Start consumers
    for (const consumer of this.consumers) {
      this.runConsumer(consumer);
    }
  }

  stop(): void {
    this.running = false;
  }

  private async runProducer(producer: () => Promise<T>): Promise<void> {
    while (this.running) {
      try {
        const item = await producer();
        await this.buffer.enqueue(item);
      } catch (error) {
        console.error("Producer error:", error);
        await this.delay(1000); // Backoff on error
      }
    }
  }

  private async runConsumer(
    consumer: (item: T) => Promise<void>
  ): Promise<void> {
    while (this.running) {
      try {
        const item = await this.buffer.dequeue();
        await consumer(item);
      } catch (error) {
        console.error("Consumer error:", error);
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

class RateLimiter {
  private tokens: number;
  private maxTokens: number;
  private refillRate: number;
  private lastRefill: number;
  private waitQueue: (() => void)[] = [];

  constructor(maxTokens: number, refillRate: number) {
    this.maxTokens = maxTokens;
    this.tokens = maxTokens;
    this.refillRate = refillRate;
    this.lastRefill = Date.now();
  }

  async acquire(tokens: number = 1): Promise<void> {
    this.refill();

    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return;
    }

    return new Promise<void>((resolve) => {
      this.waitQueue.push(() => {
        if (this.tokens >= tokens) {
          this.tokens -= tokens;
          resolve();
        } else {
          // Re-queue if still not enough tokens
          this.waitQueue.unshift(() => resolve());
        }
      });
    });
  }

  private refill(): void {
    const now = Date.now();
    const timePassed = now - this.lastRefill;
    const tokensToAdd = Math.floor((timePassed / 1000) * this.refillRate);

    if (tokensToAdd > 0) {
      this.tokens = Math.min(this.maxTokens, this.tokens + tokensToAdd);
      this.lastRefill = now;

      // Process waiting requests
      while (this.waitQueue.length > 0 && this.tokens > 0) {
        const next = this.waitQueue.shift();
        if (next) next();
      }
    }
  }
}

class CircuitBreaker {
  private state: "closed" | "open" | "half-open" = "closed";
  private failureCount = 0;
  private lastFailureTime = 0;
  private successCount = 0;

  constructor(
    private failureThreshold: number = 5,
    private recoveryTimeout: number = 60000,
    private successThreshold: number = 3
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === "open") {
      if (Date.now() - this.lastFailureTime > this.recoveryTimeout) {
        this.state = "half-open";
        this.successCount = 0;
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
    if (this.state === "half-open") {
      this.successCount++;
      if (this.successCount >= this.successThreshold) {
        this.state = "closed";
        this.failureCount = 0;
      }
    } else {
      this.failureCount = 0;
    }
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.failureThreshold) {
      this.state = "open";
    }
  }

  getState(): string {
    return this.state;
  }
}
```

## ArkUI Concurrency Component

```typescript
@Component
export struct ConcurrencyDashboard {
  @State private concurrencyManager: AdvancedConcurrencyManager = new AdvancedConcurrencyManager()
  @State private metrics: ConcurrencyMetrics | null = null
  @State private tasks: TaskResult<any>[] = []
  @State private isRunning: boolean = false

  aboutToAppear() {
    this.startMetricsCollection()
  }

  build() {
    Scroll() {
      Column({ space: 16 }) {
        this.buildHeader()
        this.buildMetricsSection()
        this.buildTaskControls()
        this.buildTaskList()
      }
      .width('100%')
      .padding(16)
    }
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Concurrency Dashboard')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Spacer()

      Button(this.isRunning ? 'Stop' : 'Start')
        .type(ButtonType.Normal)
        .backgroundColor(this.isRunning ? '#FF3B30' : '#34C759')
        .onClick(() => {
          this.isRunning = !this.isRunning
          if (this.isRunning) {
            this.startDemoTasks()
          }
        })
    }
    .width('100%')
    .justifyContent(FlexAlign.SpaceBetween)
  }

  @Builder
  private buildMetricsSection() {
    if (!this.metrics) return

    Column() {
      Text('Performance Metrics')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        this.buildMetricCard('Total Tasks', this.metrics.totalTasks.toString())
        this.buildMetricCard('Running', this.metrics.runningTasks.toString())
        this.buildMetricCard('Completed', this.metrics.completedTasks.toString())
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)

      Row() {
        this.buildMetricCard('Failed', this.metrics.failedTasks.toString())
        this.buildMetricCard('Avg Time', `${this.metrics.averageExecutionTime.toFixed(0)}ms`)
        this.buildMetricCard('Throughput', `${this.metrics.throughput}/min`)
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)
      .margin({ top: 8 })

      Progress({
        value: this.metrics.workerUtilization * 100,
        total: 100,
        type: ProgressType.Linear
      })
      .width('100%')
      .margin({ top: 12 })

      Text(`Worker Utilization: ${(this.metrics.workerUtilization * 100).toFixed(1)}%`)
        .fontSize(12)
        .fontColor('#666666')
        .margin({ top: 4 })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildMetricCard(title: string, value: string) {
    Column() {
      Text(value)
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .fontColor('#34C759')

      Text(title)
        .fontSize(12)
        .fontColor('#666666')
    }
    .width('30%')
    .padding(12)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
    .alignItems(HorizontalAlign.Center)
  }

  @Builder
  private buildTaskControls() {
    Row() {
      Button('Submit Single Task')
        .flexGrow(1)
        .margin({ right: 8 })
        .onClick(async () => {
          await this.submitSampleTask()
        })

      Button('Submit Batch')
        .flexGrow(1)
        .margin({ left: 8 })
        .onClick(async () => {
          await this.submitBatchTasks()
        })
    }
    .width('100%')

    Row() {
      Button('Map-Reduce Demo')
        .flexGrow(1)
        .margin({ right: 8 })
        .onClick(async () => {
          await this.runMapReduceDemo()
        })

      Button('Pipeline Demo')
        .flexGrow(1)
        .margin({ left: 8 })
        .onClick(async () => {
          await this.runPipelineDemo()
        })
    }
    .width('100%')
    .margin({ top: 8 })
  }

  @Builder
  private buildTaskList() {
    Column() {
      Text('Recent Tasks')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      ForEach(this.tasks.slice(-10), (task: TaskResult<any>) => {
        this.buildTaskItem(task)
      })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildTaskItem(task: TaskResult<any>) {
    Row() {
      Column() {
        Text(task.taskId)
          .fontSize(14)
          .fontWeight(FontWeight.Bold)

        Text(`Duration: ${task.endTime - task.startTime}ms`)
          .fontSize(12)
          .fontColor('#666666')

        Text(`Attempts: ${task.attempts}`)
          .fontSize(12)
          .fontColor('#666666')
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)

      Text(task.status)
        .fontSize(12)
        .fontColor('#FFFFFF')
        .backgroundColor(this.getStatusColor(task.status))
        .padding({ horizontal: 8, vertical: 4 })
        .borderRadius(4)
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
    .margin({ bottom: 8 })
  }

  private startMetricsCollection(): void {
    setInterval(() => {
      this.metrics = this.concurrencyManager.getMetrics()
    }, 1000)
  }

  private async startDemoTasks(): Promise<void> {
    while (this.isRunning) {
      await this.submitSampleTask()
      await this.delay(2000)
    }
  }

  private async submitSampleTask(): Promise<void> {
    const task: ConcurrentTask<number> = {
      id: '',
      priority: 'normal',
      dependencies: [],
      timeout: 5000,
      retryPolicy: { maxAttempts: 2, backoffMs: 1000 },
      executor: async () => {
        await this.delay(Math.random() * 2000 + 500)
        if (Math.random() < 0.1) throw new Error('Random failure')
        return Math.floor(Math.random() * 100)
      }
    }

    const taskId = await this.concurrencyManager.submitTask(task)
    const result = await this.concurrencyManager.waitForTask(taskId)
    this.tasks.push(result)
  }

  private async submitBatchTasks(): Promise<void> {
    const tasks: ConcurrentTask<number>[] = Array.from({ length: 5 }, (_, i) => ({
      id: '',
      priority: 'normal',
      dependencies: [],
      timeout: 5000,
      retryPolicy: { maxAttempts: 2, backoffMs: 1000 },
      executor: async () => {
        await this.delay(Math.random() * 1000 + 200)
        return i * 10
      }
    }))

    const results = await this.concurrencyManager.executeParallel(tasks)
    this.tasks.push(...results)
  }

  private async runMapReduceDemo(): Promise<void> {
    const data = Array.from({ length: 10 }, (_, i) => i + 1)

    const result = await this.concurrencyManager.mapReduce(
      data,
      async (item) => {
        await this.delay(100)
        return item * item
      },
      async (results) => {
        await this.delay(50)
        return results.reduce((sum, val) => sum + val, 0)
      }
    )

    console.log('Map-Reduce result:', result)
  }

  private async runPipelineDemo(): Promise<void> {
    const stages: ConcurrentTask<number>[] = [
      {
        id: '',
        priority: 'normal',
        dependencies: [],
        timeout: 5000,
        retryPolicy: { maxAttempts: 1, backoffMs: 0 },
        executor: async () => {
          await this.delay(200)
          return 10
        }
      },
      {
        id: '',
        priority: 'normal',
        dependencies: [],
        timeout: 5000,
        retryPolicy: { maxAttempts: 1, backoffMs: 0 },
        executor: async (input: number) => {
          await this.delay(300)
          return input * 2
        }
      },
      {
        id: '',
        priority: 'normal',
        dependencies: [],
        timeout: 5000,
        retryPolicy: { maxAttempts: 1, backoffMs: 0 },
        executor: async (input: number) => {
          await this.delay(100)
          return input + 5
        }
      }
    ]

    const result = await this.concurrencyManager.executePipeline(stages)
    console.log('Pipeline result:', result)
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }

  private getStatusColor(status: TaskStatus): string {
    const colors = {
      pending: '#FF9500',
      running: '#007AFF',
      completed: '#34C759',
      failed: '#FF3B30',
      cancelled: '#8E8E93'
    }
    return colors[status] || '#8E8E93'
  }
}
```

## Conclusion

Advanced Concurrency in ArkUI provides:

- Sophisticated task management and execution
- Parallel processing and coordination patterns
- Thread-safe data structures and operations
- Performance monitoring and optimization
- Fault tolerance and error recovery

These capabilities enable efficient concurrent programming for complex applications requiring high performance and reliability.
