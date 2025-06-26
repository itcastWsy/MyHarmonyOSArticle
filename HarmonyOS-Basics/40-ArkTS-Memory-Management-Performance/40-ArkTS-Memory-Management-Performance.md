# 40. ArkTS Memory Management and Performance

## Introduction

Effective memory management and performance optimization are crucial for building high-quality HarmonyOS applications. This article explores ArkTS's memory management principles, performance profiling techniques, and optimization strategies for creating efficient applications.

## Memory Management Fundamentals

### Object Lifecycle Management

```typescript
// Memory-efficient object pool pattern
export class ObjectPool<T> {
  private available: T[] = [];
  private inUse: Set<T> = new Set();
  private createFn: () => T;
  private resetFn: (obj: T) => void;
  private maxSize: number;

  constructor(
    createFn: () => T,
    resetFn: (obj: T) => void,
    initialSize: number = 10,
    maxSize: number = 100
  ) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.maxSize = maxSize;

    // Pre-populate pool
    for (let i = 0; i < initialSize; i++) {
      this.available.push(this.createFn());
    }
  }

  acquire(): T {
    let obj: T;

    if (this.available.length > 0) {
      obj = this.available.pop()!;
    } else {
      obj = this.createFn();
    }

    this.inUse.add(obj);
    return obj;
  }

  release(obj: T): void {
    if (this.inUse.has(obj)) {
      this.inUse.delete(obj);
      this.resetFn(obj);

      if (this.available.length < this.maxSize) {
        this.available.push(obj);
      }
      // Else let it be garbage collected
    }
  }

  clear(): void {
    this.available.length = 0;
    this.inUse.clear();
  }

  getStats(): {
    available: number;
    inUse: number;
    total: number;
  } {
    return {
      available: this.available.length,
      inUse: this.inUse.size,
      total: this.available.length + this.inUse.size,
    };
  }
}

// Weak reference manager for memory-sensitive scenarios
export class WeakReferenceManager<T extends object> {
  private weakRefs: WeakMap<T, WeakRef<T>> = new WeakMap();
  private registry = new FinalizationRegistry((heldValue: string) => {
    console.log(`Object with id ${heldValue} was garbage collected`);
  });

  register(obj: T, id: string): WeakRef<T> {
    const weakRef = new WeakRef(obj);
    this.weakRefs.set(obj, weakRef);
    this.registry.register(obj, id);
    return weakRef;
  }

  get(obj: T): T | undefined {
    const weakRef = this.weakRefs.get(obj);
    return weakRef?.deref();
  }

  cleanup(): void {
    // Force cleanup of weak references
    this.weakRefs = new WeakMap();
  }
}
```

### Memory Monitoring and Profiling

```typescript
export interface MemoryMetrics {
  heapUsed: number;
  heapTotal: number;
  external: number;
  arrayBuffers: number;
  timestamp: number;
}

export class MemoryProfiler {
  private static instance: MemoryProfiler;
  private metrics: MemoryMetrics[] = [];
  private isProfileling: boolean = false;
  private intervalId: number | null = null;
  private maxMetricsCount: number = 1000;

  static getInstance(): MemoryProfiler {
    if (!this.instance) {
      this.instance = new MemoryProfiler();
    }
    return this.instance;
  }

  startProfiling(intervalMs: number = 1000): void {
    if (this.isProfileling) return;

    this.isProfileling = true;
    this.intervalId = setInterval(() => {
      this.collectMetrics();
    }, intervalMs);

    console.log("Memory profiling started");
  }

  stopProfiling(): void {
    if (!this.isProfileling) return;

    this.isProfileling = false;
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }

    console.log("Memory profiling stopped");
  }

  private collectMetrics(): void {
    try {
      // Simulate memory usage collection (actual implementation would use system APIs)
      const metrics: MemoryMetrics = {
        heapUsed: this.getHeapUsed(),
        heapTotal: this.getHeapTotal(),
        external: this.getExternalMemory(),
        arrayBuffers: this.getArrayBuffers(),
        timestamp: Date.now(),
      };

      this.metrics.push(metrics);

      // Keep only recent metrics
      if (this.metrics.length > this.maxMetricsCount) {
        this.metrics.shift();
      }

      this.analyzeMemoryTrends();
    } catch (error) {
      console.error("Failed to collect memory metrics:", error);
    }
  }

  private getHeapUsed(): number {
    // Placeholder - would use actual system memory API
    return Math.random() * 100 * 1024 * 1024; // Random MB
  }

  private getHeapTotal(): number {
    return Math.random() * 200 * 1024 * 1024;
  }

  private getExternalMemory(): number {
    return Math.random() * 50 * 1024 * 1024;
  }

  private getArrayBuffers(): number {
    return Math.random() * 20 * 1024 * 1024;
  }

  private analyzeMemoryTrends(): void {
    if (this.metrics.length < 10) return;

    const recent = this.metrics.slice(-10);
    const averageHeapUsed =
      recent.reduce((sum, m) => sum + m.heapUsed, 0) / recent.length;
    const currentHeapUsed = recent[recent.length - 1].heapUsed;

    // Detect memory leaks
    if (currentHeapUsed > averageHeapUsed * 1.5) {
      console.warn("Potential memory leak detected - heap usage spike");
    }

    // Detect memory pressure
    const heapUtilization =
      currentHeapUsed / recent[recent.length - 1].heapTotal;
    if (heapUtilization > 0.9) {
      console.warn("High memory pressure detected");
      this.triggerGarbageCollection();
    }
  }

  private triggerGarbageCollection(): void {
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
      console.log("Forced garbage collection");
    }
  }

  getMetrics(): MemoryMetrics[] {
    return [...this.metrics];
  }

  getLatestMetrics(): MemoryMetrics | null {
    return this.metrics.length > 0
      ? this.metrics[this.metrics.length - 1]
      : null;
  }

  clearMetrics(): void {
    this.metrics = [];
  }

  generateReport(): {
    summary: {
      averageHeapUsed: number;
      peakHeapUsed: number;
      averageHeapTotal: number;
      memoryLeakSuspected: boolean;
    };
    recommendations: string[];
  } {
    if (this.metrics.length === 0) {
      return {
        summary: {
          averageHeapUsed: 0,
          peakHeapUsed: 0,
          averageHeapTotal: 0,
          memoryLeakSuspected: false,
        },
        recommendations: ["Start profiling to collect data"],
      };
    }

    const averageHeapUsed =
      this.metrics.reduce((sum, m) => sum + m.heapUsed, 0) /
      this.metrics.length;
    const peakHeapUsed = Math.max(...this.metrics.map((m) => m.heapUsed));
    const averageHeapTotal =
      this.metrics.reduce((sum, m) => sum + m.heapTotal, 0) /
      this.metrics.length;

    // Detect potential memory leaks
    const first10 = this.metrics.slice(0, 10);
    const last10 = this.metrics.slice(-10);
    const firstAverage =
      first10.reduce((sum, m) => sum + m.heapUsed, 0) / first10.length;
    const lastAverage =
      last10.reduce((sum, m) => sum + m.heapUsed, 0) / last10.length;
    const memoryLeakSuspected = lastAverage > firstAverage * 1.5;

    const recommendations: string[] = [];

    if (memoryLeakSuspected) {
      recommendations.push(
        "Potential memory leak detected - review object lifecycle management"
      );
    }

    if (averageHeapUsed / averageHeapTotal > 0.7) {
      recommendations.push(
        "High average memory usage - consider object pooling"
      );
    }

    if (peakHeapUsed / averageHeapTotal > 0.9) {
      recommendations.push("Memory spikes detected - implement lazy loading");
    }

    return {
      summary: {
        averageHeapUsed,
        peakHeapUsed,
        averageHeapTotal,
        memoryLeakSuspected,
      },
      recommendations,
    };
  }
}
```

## Performance Optimization Strategies

### Lazy Loading and Code Splitting

```typescript
// Dynamic import manager for code splitting
export class DynamicImportManager {
  private static instance: DynamicImportManager;
  private loadedModules: Map<string, any> = new Map();
  private loadingPromises: Map<string, Promise<any>> = new Map();

  static getInstance(): DynamicImportManager {
    if (!this.instance) {
      this.instance = new DynamicImportManager();
    }
    return this.instance;
  }

  async loadModule<T>(
    moduleId: string,
    importFn: () => Promise<T>
  ): Promise<T> {
    // Return cached module if already loaded
    if (this.loadedModules.has(moduleId)) {
      return this.loadedModules.get(moduleId);
    }

    // Return existing promise if currently loading
    if (this.loadingPromises.has(moduleId)) {
      return this.loadingPromises.get(moduleId);
    }

    // Start loading
    const loadingPromise = this.performLoad(moduleId, importFn);
    this.loadingPromises.set(moduleId, loadingPromise);

    try {
      const module = await loadingPromise;
      this.loadedModules.set(moduleId, module);
      return module;
    } finally {
      this.loadingPromises.delete(moduleId);
    }
  }

  private async performLoad<T>(
    moduleId: string,
    importFn: () => Promise<T>
  ): Promise<T> {
    console.log(`Loading module: ${moduleId}`);
    const startTime = Date.now();

    try {
      const module = await importFn();
      const loadTime = Date.now() - startTime;
      console.log(`Module ${moduleId} loaded in ${loadTime}ms`);
      return module;
    } catch (error) {
      console.error(`Failed to load module ${moduleId}:`, error);
      throw error;
    }
  }

  preloadModule<T>(moduleId: string, importFn: () => Promise<T>): Promise<T> {
    return this.loadModule(moduleId, importFn);
  }

  isModuleLoaded(moduleId: string): boolean {
    return this.loadedModules.has(moduleId);
  }

  unloadModule(moduleId: string): boolean {
    return this.loadedModules.delete(moduleId);
  }

  getLoadedModules(): string[] {
    return Array.from(this.loadedModules.keys());
  }
}

// Lazy component loader
export class LazyComponentLoader {
  private componentCache: Map<string, any> = new Map();

  async loadComponent(
    componentName: string,
    loader: () => Promise<any>
  ): Promise<any> {
    if (this.componentCache.has(componentName)) {
      return this.componentCache.get(componentName);
    }

    try {
      const component = await loader();
      this.componentCache.set(componentName, component);
      return component;
    } catch (error) {
      console.error(`Failed to load component ${componentName}:`, error);
      throw error;
    }
  }

  preloadComponents(
    components: Array<{
      name: string;
      loader: () => Promise<any>;
    }>
  ): Promise<void[]> {
    return Promise.all(
      components.map(({ name, loader }) =>
        this.loadComponent(name, loader).catch((error) => {
          console.warn(`Failed to preload component ${name}:`, error);
        })
      )
    );
  }
}
```

### Data Structure Optimization

```typescript
// Efficient data structures for large datasets
export class OptimizedDataStructures {
  // Sparse array for memory-efficient storage
  static createSparseArray<T>(): SparseArray<T> {
    return new SparseArray<T>();
  }

  // LRU cache with memory constraints
  static createLRUCache<K, V>(
    maxSize: number,
    maxMemoryMB?: number
  ): LRUCache<K, V> {
    return new LRUCache<K, V>(maxSize, maxMemoryMB);
  }

  // Immutable data structure for state management
  static createImmutableRecord<T extends Record<string, any>>(
    data: T
  ): ImmutableRecord<T> {
    return new ImmutableRecord(data);
  }
}

class SparseArray<T> {
  private data: Map<number, T> = new Map();
  private _length: number = 0;

  get length(): number {
    return this._length;
  }

  set(index: number, value: T): void {
    if (index < 0) throw new Error("Index cannot be negative");

    this.data.set(index, value);
    this._length = Math.max(this._length, index + 1);
  }

  get(index: number): T | undefined {
    return this.data.get(index);
  }

  has(index: number): boolean {
    return this.data.has(index);
  }

  delete(index: number): boolean {
    const result = this.data.delete(index);
    if (result && index === this._length - 1) {
      // Recalculate length if we deleted the last element
      this._length = this.data.size > 0 ? Math.max(...this.data.keys()) + 1 : 0;
    }
    return result;
  }

  forEach(callback: (value: T, index: number) => void): void {
    this.data.forEach((value, index) => callback(value, index));
  }

  getMemoryUsage(): number {
    return this.data.size * 8; // Approximate bytes
  }
}

class LRUCache<K, V> {
  private cache: Map<K, V> = new Map();
  private maxSize: number;
  private maxMemoryBytes: number;
  private currentMemoryBytes: number = 0;

  constructor(maxSize: number, maxMemoryMB?: number) {
    this.maxSize = maxSize;
    this.maxMemoryBytes = maxMemoryMB ? maxMemoryMB * 1024 * 1024 : Infinity;
  }

  get(key: K): V | undefined {
    const value = this.cache.get(key);
    if (value !== undefined) {
      // Move to end (most recently used)
      this.cache.delete(key);
      this.cache.set(key, value);
    }
    return value;
  }

  set(key: K, value: V): void {
    const estimatedSize = this.estimateSize(value);

    // Remove if already exists
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    // Check memory constraints
    while (
      (this.cache.size >= this.maxSize ||
        this.currentMemoryBytes + estimatedSize > this.maxMemoryBytes) &&
      this.cache.size > 0
    ) {
      this.evictLRU();
    }

    this.cache.set(key, value);
    this.currentMemoryBytes += estimatedSize;
  }

  private evictLRU(): void {
    const firstKey = this.cache.keys().next().value;
    if (firstKey !== undefined) {
      const value = this.cache.get(firstKey);
      this.cache.delete(firstKey);
      this.currentMemoryBytes -= this.estimateSize(value);
    }
  }

  private estimateSize(value: any): number {
    // Simple size estimation
    try {
      return JSON.stringify(value).length * 2; // Rough Unicode estimation
    } catch {
      return 100; // Default size for non-serializable objects
    }
  }

  clear(): void {
    this.cache.clear();
    this.currentMemoryBytes = 0;
  }

  size(): number {
    return this.cache.size;
  }

  getMemoryUsage(): number {
    return this.currentMemoryBytes;
  }
}

class ImmutableRecord<T extends Record<string, any>> {
  private data: T;
  private frozenData: T;

  constructor(data: T) {
    this.data = { ...data };
    this.frozenData = Object.freeze({ ...data });
  }

  get<K extends keyof T>(key: K): T[K] {
    return this.frozenData[key];
  }

  set<K extends keyof T>(key: K, value: T[K]): ImmutableRecord<T> {
    const newData = { ...this.data, [key]: value };
    return new ImmutableRecord(newData);
  }

  update(updateFn: (data: T) => Partial<T>): ImmutableRecord<T> {
    const updates = updateFn(this.frozenData);
    const newData = { ...this.data, ...updates };
    return new ImmutableRecord(newData);
  }

  toObject(): T {
    return { ...this.frozenData };
  }
}
```

## Performance Monitoring Component

```typescript
@Component
export struct PerformanceMonitor {
  @State memoryMetrics: MemoryMetrics | null = null;
  @State performanceReport: any = null;
  @State isMonitoring: boolean = false;

  private memoryProfiler: MemoryProfiler = MemoryProfiler.getInstance();
  private updateInterval: number | null = null;

  aboutToAppear() {
    this.startMonitoring();
  }

  aboutToDisappear() {
    this.stopMonitoring();
  }

  private startMonitoring(): void {
    this.isMonitoring = true;
    this.memoryProfiler.startProfiling(1000);

    this.updateInterval = setInterval(() => {
      this.memoryMetrics = this.memoryProfiler.getLatestMetrics();
      this.performanceReport = this.memoryProfiler.generateReport();
    }, 2000);
  }

  private stopMonitoring(): void {
    this.isMonitoring = false;
    this.memoryProfiler.stopProfiling();

    if (this.updateInterval) {
      clearInterval(this.updateInterval);
      this.updateInterval = null;
    }
  }

  private formatBytes(bytes: number): string {
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    if (bytes === 0) return '0 Bytes';
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    return Math.round(bytes / Math.pow(1024, i) * 100) / 100 + ' ' + sizes[i];
  }

  build() {
    ScrollView() {
      Column({ space: 16 }) {
        // Header
        Row() {
          Text('Performance Monitor')
            .fontSize(24)
            .fontWeight(FontWeight.Bold)

          Button(this.isMonitoring ? 'Stop' : 'Start')
            .onClick(() => {
              if (this.isMonitoring) {
                this.stopMonitoring();
              } else {
                this.startMonitoring();
              }
            })
            .backgroundColor(this.isMonitoring ? Color.Red : Color.Green)
        }
        .width('100%')
        .justifyContent(FlexAlign.SpaceBetween)

        // Current metrics
        if (this.memoryMetrics) {
          Card() {
            Column({ space: 12 }) {
              Text('Current Memory Usage')
                .fontSize(18)
                .fontWeight(FontWeight.Bold)

              Row() {
                Text('Heap Used:')
                  .width('50%')
                Text(this.formatBytes(this.memoryMetrics.heapUsed))
                  .fontColor(Color.Blue)
              }
              .width('100%')

              Row() {
                Text('Heap Total:')
                  .width('50%')
                Text(this.formatBytes(this.memoryMetrics.heapTotal))
                  .fontColor(Color.Gray)
              }
              .width('100%')

              Row() {
                Text('External:')
                  .width('50%')
                Text(this.formatBytes(this.memoryMetrics.external))
                  .fontColor(Color.Orange)
              }
              .width('100%')

              // Memory usage bar
              Progress({
                value: (this.memoryMetrics.heapUsed / this.memoryMetrics.heapTotal) * 100,
                total: 100,
                type: ProgressType.Linear
              })
                .width('100%')
                .color(
                  this.memoryMetrics.heapUsed / this.memoryMetrics.heapTotal > 0.8
                    ? Color.Red
                    : Color.Blue
                )
            }
            .padding(16)
            .width('100%')
          }
        }

        // Performance report
        if (this.performanceReport) {
          Card() {
            Column({ space: 12 }) {
              Text('Performance Summary')
                .fontSize(18)
                .fontWeight(FontWeight.Bold)

              Row() {
                Text('Average Heap:')
                  .width('50%')
                Text(this.formatBytes(this.performanceReport.summary.averageHeapUsed))
                  .fontColor(Color.Blue)
              }
              .width('100%')

              Row() {
                Text('Peak Heap:')
                  .width('50%')
                Text(this.formatBytes(this.performanceReport.summary.peakHeapUsed))
                  .fontColor(Color.Red)
              }
              .width('100%')

              if (this.performanceReport.summary.memoryLeakSuspected) {
                Text('⚠️ Memory leak suspected')
                  .fontColor(Color.Red)
                  .fontSize(14)
                  .fontWeight(FontWeight.Bold)
              }

              // Recommendations
              if (this.performanceReport.recommendations.length > 0) {
                Text('Recommendations:')
                  .fontSize(16)
                  .fontWeight(FontWeight.Bold)
                  .margin({ top: 8 })

                ForEach(this.performanceReport.recommendations, (rec: string) => {
                  Text(`• ${rec}`)
                    .fontSize(14)
                    .fontColor(Color.Gray)
                    .margin({ left: 8 })
                }, (rec: string, index: number) => `rec-${index}`)
              }
            }
            .padding(16)
            .width('100%')
          }
        }

        // Control buttons
        Row({ space: 10 }) {
          Button('Clear Metrics')
            .onClick(() => {
              this.memoryProfiler.clearMetrics();
              this.memoryMetrics = null;
              this.performanceReport = null;
            })
            .backgroundColor(Color.Orange)

          Button('Force GC')
            .onClick(() => {
              // Trigger garbage collection if available
              if (global.gc) {
                global.gc();
              }
            })
            .backgroundColor(Color.Purple)
        }
      }
    }
    .width('100%')
    .height('100%')
    .padding(16)
  }
}
```

## Best Practices

1. **Object Pooling**: Reuse objects to reduce garbage collection pressure
2. **Weak References**: Use weak references for large objects that can be garbage collected
3. **Lazy Loading**: Load resources only when needed to reduce initial memory footprint
4. **Memory Profiling**: Regularly monitor memory usage to detect leaks early
5. **Data Structure Selection**: Choose appropriate data structures based on access patterns
6. **Immutable Data**: Use immutable structures to prevent accidental mutations and enable optimizations

## Conclusion

Effective memory management and performance optimization in ArkTS require understanding of object lifecycles, garbage collection patterns, and system constraints. By implementing proper monitoring, using efficient data structures, and following best practices, developers can create HarmonyOS applications that perform well across different device capabilities while providing excellent user experiences.

Regular performance auditing and proactive memory management are essential for maintaining application quality as complexity and user base grow.
