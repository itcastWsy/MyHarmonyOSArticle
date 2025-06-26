# ArkTS Memory Management and Optimization

## Introduction

Effective memory management is crucial for building high-performance HarmonyOS applications. This guide explores advanced memory management techniques, garbage collection optimization, and memory leak prevention strategies in ArkTS applications.

## Memory Management Architecture

### Memory Pool Management

```typescript
interface MemoryPoolConfig {
  initialSize: number;
  maxSize: number;
  growthFactor: number;
  cleanupThreshold: number;
}

class MemoryPool<T> {
  private pool: T[] = [];
  private inUse = new Set<T>();
  private factory: () => T;
  private reset: (item: T) => void;
  private config: MemoryPoolConfig;

  constructor(
    factory: () => T,
    reset: (item: T) => void,
    config: MemoryPoolConfig
  ) {
    this.factory = factory;
    this.reset = reset;
    this.config = config;
    this.initializePool();
  }

  acquire(): T {
    let item: T;

    if (this.pool.length > 0) {
      item = this.pool.pop()!;
    } else {
      item = this.factory();
    }

    this.inUse.add(item);
    return item;
  }

  release(item: T): void {
    if (!this.inUse.has(item)) {
      console.warn("Attempting to release item not in use");
      return;
    }

    this.inUse.delete(item);
    this.reset(item);

    if (this.pool.length < this.config.maxSize) {
      this.pool.push(item);
    }

    this.checkCleanup();
  }

  getStats(): PoolStats {
    return {
      poolSize: this.pool.length,
      inUseCount: this.inUse.size,
      totalAllocated: this.pool.length + this.inUse.size,
      memoryUsage: this.estimateMemoryUsage(),
    };
  }

  private initializePool(): void {
    for (let i = 0; i < this.config.initialSize; i++) {
      this.pool.push(this.factory());
    }
  }

  private checkCleanup(): void {
    if (this.pool.length > this.config.cleanupThreshold) {
      const excessItems = this.pool.length - this.config.cleanupThreshold;
      this.pool.splice(0, excessItems);
    }
  }

  private estimateMemoryUsage(): number {
    // Estimate memory usage in bytes
    return (this.pool.length + this.inUse.size) * this.estimateItemSize();
  }

  private estimateItemSize(): number {
    // Override in specific implementations
    return 64; // Default estimate
  }
}

interface PoolStats {
  poolSize: number;
  inUseCount: number;
  totalAllocated: number;
  memoryUsage: number;
}

// Example usage for UI components
class ComponentPool extends MemoryPool<ComponentInstance> {
  constructor() {
    super(
      () => new ComponentInstance(),
      (item) => item.reset(),
      {
        initialSize: 10,
        maxSize: 100,
        growthFactor: 1.5,
        cleanupThreshold: 50,
      }
    );
  }

  private estimateItemSize(): number {
    return 1024; // Estimated component size in bytes
  }
}

class ComponentInstance {
  private data: any = {};
  private listeners: Function[] = [];

  reset(): void {
    this.data = {};
    this.listeners = [];
  }

  setData(data: any): void {
    this.data = data;
  }

  addListener(listener: Function): void {
    this.listeners.push(listener);
  }
}
```

### Smart Reference Management

```typescript
enum ReferenceType {
  Strong = "strong",
  Weak = "weak",
  Soft = "soft",
}

interface ReferenceConfig {
  type: ReferenceType;
  autoCleanup: boolean;
  timeoutMs?: number;
}

class SmartReference<T extends object> {
  private target: T | null;
  private config: ReferenceConfig;
  private cleanupTimer: number | null = null;
  private accessCount: number = 0;
  private lastAccess: number = Date.now();

  constructor(target: T, config: ReferenceConfig) {
    this.target = target;
    this.config = config;
    this.setupCleanup();
  }

  get(): T | null {
    if (this.target) {
      this.accessCount++;
      this.lastAccess = Date.now();
      this.resetCleanupTimer();
    }
    return this.target;
  }

  clear(): void {
    this.target = null;
    this.clearCleanupTimer();
  }

  isValid(): boolean {
    return this.target !== null;
  }

  getStats(): ReferenceStats {
    return {
      isValid: this.isValid(),
      accessCount: this.accessCount,
      lastAccess: this.lastAccess,
      type: this.config.type,
    };
  }

  private setupCleanup(): void {
    if (this.config.autoCleanup && this.config.timeoutMs) {
      this.resetCleanupTimer();
    }
  }

  private resetCleanupTimer(): void {
    this.clearCleanupTimer();

    if (this.config.autoCleanup && this.config.timeoutMs) {
      this.cleanupTimer = setTimeout(() => {
        this.clear();
      }, this.config.timeoutMs);
    }
  }

  private clearCleanupTimer(): void {
    if (this.cleanupTimer) {
      clearTimeout(this.cleanupTimer);
      this.cleanupTimer = null;
    }
  }
}

interface ReferenceStats {
  isValid: boolean;
  accessCount: number;
  lastAccess: number;
  type: ReferenceType;
}

// Reference manager for complex object graphs
class ReferenceManager {
  private references = new Map<string, SmartReference<any>>();
  private dependencyGraph = new Map<string, Set<string>>();

  createReference<T extends object>(
    id: string,
    target: T,
    config: ReferenceConfig
  ): SmartReference<T> {
    const reference = new SmartReference(target, config);
    this.references.set(id, reference);
    return reference;
  }

  getReference<T extends object>(id: string): SmartReference<T> | null {
    return this.references.get(id) || null;
  }

  addDependency(parentId: string, childId: string): void {
    if (!this.dependencyGraph.has(parentId)) {
      this.dependencyGraph.set(parentId, new Set());
    }
    this.dependencyGraph.get(parentId)!.add(childId);
  }

  cleanup(id: string): void {
    const reference = this.references.get(id);
    if (reference) {
      reference.clear();
      this.references.delete(id);

      // Cleanup dependent references
      const dependencies = this.dependencyGraph.get(id);
      if (dependencies) {
        dependencies.forEach((depId) => this.cleanup(depId));
        this.dependencyGraph.delete(id);
      }
    }
  }

  performGarbageCollection(): GCStats {
    const startTime = Date.now();
    let cleanedCount = 0;
    let totalMemoryFreed = 0;

    for (const [id, reference] of this.references) {
      if (!reference.isValid()) {
        this.cleanup(id);
        cleanedCount++;
        totalMemoryFreed += this.estimateReferenceSize(reference);
      }
    }

    return {
      duration: Date.now() - startTime,
      cleanedReferences: cleanedCount,
      memoryFreed: totalMemoryFreed,
      remainingReferences: this.references.size,
    };
  }

  private estimateReferenceSize(reference: SmartReference<any>): number {
    // Estimate memory usage of reference
    return 256; // Default estimate
  }
}

interface GCStats {
  duration: number;
  cleanedReferences: number;
  memoryFreed: number;
  remainingReferences: number;
}
```

## Memory Leak Detection

### Advanced Leak Detector

```typescript
interface LeakDetectionConfig {
  sampleRate: number;
  threshold: number;
  trackingDuration: number;
  reportingEnabled: boolean;
}

interface MemorySnapshot {
  timestamp: number;
  heapUsed: number;
  heapTotal: number;
  external: number;
  objectCounts: Map<string, number>;
}

class MemoryLeakDetector {
  private config: LeakDetectionConfig;
  private snapshots: MemorySnapshot[] = [];
  private objectTracker = new Map<string, WeakSet<object>>();
  private leakReports: LeakReport[] = [];
  private isMonitoring = false;

  constructor(config: LeakDetectionConfig) {
    this.config = config;
  }

  startMonitoring(): void {
    if (this.isMonitoring) return;

    this.isMonitoring = true;
    this.scheduleSnapshot();
  }

  stopMonitoring(): void {
    this.isMonitoring = false;
  }

  trackObject(object: object, type: string): void {
    if (!this.objectTracker.has(type)) {
      this.objectTracker.set(type, new WeakSet());
    }
    this.objectTracker.get(type)!.add(object);
  }

  takeSnapshot(): MemorySnapshot {
    const snapshot: MemorySnapshot = {
      timestamp: Date.now(),
      heapUsed: this.getHeapUsed(),
      heapTotal: this.getHeapTotal(),
      external: this.getExternalMemory(),
      objectCounts: this.getObjectCounts(),
    };

    this.snapshots.push(snapshot);
    this.trimSnapshots();

    return snapshot;
  }

  detectLeaks(): LeakReport[] {
    if (this.snapshots.length < 3) {
      return [];
    }

    const reports: LeakReport[] = [];
    const recentSnapshots = this.snapshots.slice(-3);

    // Analyze memory growth
    const memoryGrowth = this.analyzeMemoryGrowth(recentSnapshots);
    if (memoryGrowth.isLeaking) {
      reports.push({
        type: "memory_growth",
        severity: "high",
        description: "Continuous memory growth detected",
        data: memoryGrowth,
        timestamp: Date.now(),
      });
    }

    // Analyze object count growth
    const objectLeaks = this.analyzeObjectGrowth(recentSnapshots);
    reports.push(...objectLeaks);

    this.leakReports.push(...reports);
    return reports;
  }

  getLeakHistory(): LeakReport[] {
    return [...this.leakReports];
  }

  private scheduleSnapshot(): void {
    if (!this.isMonitoring) return;

    setTimeout(() => {
      this.takeSnapshot();
      this.detectLeaks();
      this.scheduleSnapshot();
    }, this.config.sampleRate);
  }

  private analyzeMemoryGrowth(
    snapshots: MemorySnapshot[]
  ): MemoryGrowthAnalysis {
    const growthRates = [];

    for (let i = 1; i < snapshots.length; i++) {
      const prev = snapshots[i - 1];
      const curr = snapshots[i];
      const growth = (curr.heapUsed - prev.heapUsed) / prev.heapUsed;
      growthRates.push(growth);
    }

    const avgGrowthRate =
      growthRates.reduce((sum, rate) => sum + rate, 0) / growthRates.length;
    const isLeaking = avgGrowthRate > this.config.threshold;

    return {
      isLeaking,
      growthRate: avgGrowthRate,
      totalGrowth:
        snapshots[snapshots.length - 1].heapUsed - snapshots[0].heapUsed,
      snapshots: snapshots.length,
    };
  }

  private analyzeObjectGrowth(snapshots: MemorySnapshot[]): LeakReport[] {
    const reports: LeakReport[] = [];
    const objectTypes = new Set<string>();

    snapshots.forEach((snapshot) => {
      snapshot.objectCounts.forEach((_, type) => objectTypes.add(type));
    });

    objectTypes.forEach((type) => {
      const counts = snapshots.map((s) => s.objectCounts.get(type) || 0);
      const growth = counts[counts.length - 1] - counts[0];

      if (growth > 100) {
        // Threshold for object count growth
        reports.push({
          type: "object_leak",
          severity: "medium",
          description: `Object type '${type}' shows potential leak`,
          data: { type, growth, counts },
          timestamp: Date.now(),
        });
      }
    });

    return reports;
  }

  private getHeapUsed(): number {
    if ("memory" in performance) {
      return (performance as any).memory.usedJSHeapSize;
    }
    return 0;
  }

  private getHeapTotal(): number {
    if ("memory" in performance) {
      return (performance as any).memory.totalJSHeapSize;
    }
    return 0;
  }

  private getExternalMemory(): number {
    // Platform specific implementation
    return 0;
  }

  private getObjectCounts(): Map<string, number> {
    const counts = new Map<string, number>();

    this.objectTracker.forEach((weakSet, type) => {
      // Note: WeakSet doesn't provide size, this is conceptual
      counts.set(type, this.estimateWeakSetSize(weakSet));
    });

    return counts;
  }

  private estimateWeakSetSize(weakSet: WeakSet<object>): number {
    // This would require platform-specific implementation
    return 0;
  }

  private trimSnapshots(): void {
    const maxSnapshots = Math.floor(
      this.config.trackingDuration / this.config.sampleRate
    );
    if (this.snapshots.length > maxSnapshots) {
      this.snapshots.splice(0, this.snapshots.length - maxSnapshots);
    }
  }
}

interface MemoryGrowthAnalysis {
  isLeaking: boolean;
  growthRate: number;
  totalGrowth: number;
  snapshots: number;
}

interface LeakReport {
  type: string;
  severity: "low" | "medium" | "high";
  description: string;
  data: any;
  timestamp: number;
}
```

## Component Memory Optimization

### Memory-Aware Component System

```typescript
interface ComponentMemoryConfig {
  maxCacheSize: number;
  cleanupInterval: number;
  lazyCleanup: boolean;
  trackLifecycle: boolean;
}

class MemoryAwareComponentManager {
  private componentCache = new Map<string, CachedComponent>();
  private memoryTracker = new Map<string, ComponentMemoryInfo>();
  private config: ComponentMemoryConfig;
  private cleanupTimer: number | null = null;

  constructor(config: ComponentMemoryConfig) {
    this.config = config;
    this.startCleanupTimer();
  }

  createComponent(id: string, factory: () => any): any {
    // Check if component exists in cache
    const cached = this.componentCache.get(id);
    if (cached && this.isComponentValid(cached)) {
      cached.lastAccess = Date.now();
      cached.accessCount++;
      return cached.component;
    }

    // Create new component
    const component = factory();
    const memoryInfo = this.measureComponentMemory(component);

    this.cacheComponent(id, component, memoryInfo);
    this.trackComponent(id, memoryInfo);

    return component;
  }

  destroyComponent(id: string): void {
    const cached = this.componentCache.get(id);
    if (cached) {
      this.cleanupComponent(cached.component);
      this.componentCache.delete(id);
      this.memoryTracker.delete(id);
    }
  }

  optimizeMemory(): MemoryOptimizationResult {
    const before = this.getCurrentMemoryUsage();
    let freedComponents = 0;
    let freedMemory = 0;

    // Remove least recently used components
    const sortedComponents = Array.from(this.componentCache.entries()).sort(
      ([, a], [, b]) => a.lastAccess - b.lastAccess
    );

    while (
      this.getTotalCacheSize() > this.config.maxCacheSize &&
      sortedComponents.length > 0
    ) {
      const [id, component] = sortedComponents.shift()!;
      const memoryInfo = this.memoryTracker.get(id);

      this.destroyComponent(id);
      freedComponents++;
      freedMemory += memoryInfo?.estimatedSize || 0;
    }

    const after = this.getCurrentMemoryUsage();

    return {
      freedComponents,
      freedMemory,
      beforeOptimization: before,
      afterOptimization: after,
      improvement: before - after,
    };
  }

  getMemoryReport(): ComponentMemoryReport {
    const components = Array.from(this.memoryTracker.entries()).map(
      ([id, info]) => ({
        id,
        ...info,
      })
    );

    return {
      totalComponents: components.length,
      totalMemoryUsage: this.getCurrentMemoryUsage(),
      components: components.sort((a, b) => b.estimatedSize - a.estimatedSize),
      cacheHitRate: this.calculateCacheHitRate(),
    };
  }

  private cacheComponent(
    id: string,
    component: any,
    memoryInfo: ComponentMemoryInfo
  ): void {
    this.componentCache.set(id, {
      component,
      createdAt: Date.now(),
      lastAccess: Date.now(),
      accessCount: 1,
      memoryFootprint: memoryInfo.estimatedSize,
    });
  }

  private trackComponent(id: string, memoryInfo: ComponentMemoryInfo): void {
    this.memoryTracker.set(id, memoryInfo);
  }

  private measureComponentMemory(component: any): ComponentMemoryInfo {
    // Estimate component memory usage
    const estimatedSize = this.estimateObjectSize(component);

    return {
      estimatedSize,
      createdAt: Date.now(),
      type: component.constructor.name,
      properties: Object.keys(component).length,
    };
  }

  private estimateObjectSize(obj: any): number {
    let size = 0;

    const calculateSize = (value: any, visited = new Set()): number => {
      if (visited.has(value)) return 0;
      visited.add(value);

      if (value === null || value === undefined) return 0;

      switch (typeof value) {
        case "boolean":
          return 4;
        case "number":
          return 8;
        case "string":
          return value.length * 2;
        case "object":
          if (Array.isArray(value)) {
            return value.reduce(
              (sum, item) => sum + calculateSize(item, visited),
              0
            );
          }
          return Object.values(value).reduce(
            (sum, prop) => sum + calculateSize(prop, visited),
            0
          );
        default:
          return 0;
      }
    };

    return calculateSize(obj);
  }

  private isComponentValid(cached: CachedComponent): boolean {
    // Check if component is still valid (not disposed, etc.)
    return cached.component && typeof cached.component === "object";
  }

  private cleanupComponent(component: any): void {
    // Perform cleanup operations
    if (component && typeof component.dispose === "function") {
      component.dispose();
    }
  }

  private startCleanupTimer(): void {
    if (this.config.cleanupInterval > 0) {
      this.cleanupTimer = setInterval(() => {
        this.performScheduledCleanup();
      }, this.config.cleanupInterval);
    }
  }

  private performScheduledCleanup(): void {
    if (this.config.lazyCleanup) {
      const now = Date.now();
      const staleThreshold = 5 * 60 * 1000; // 5 minutes

      for (const [id, cached] of this.componentCache) {
        if (now - cached.lastAccess > staleThreshold) {
          this.destroyComponent(id);
        }
      }
    }
  }

  private getTotalCacheSize(): number {
    return Array.from(this.componentCache.values()).reduce(
      (sum, cached) => sum + cached.memoryFootprint,
      0
    );
  }

  private getCurrentMemoryUsage(): number {
    return this.getTotalCacheSize();
  }

  private calculateCacheHitRate(): number {
    const totalAccesses = Array.from(this.componentCache.values()).reduce(
      (sum, cached) => sum + cached.accessCount,
      0
    );

    const cacheHits = Array.from(this.componentCache.values()).reduce(
      (sum, cached) => sum + (cached.accessCount - 1),
      0
    );

    return totalAccesses > 0 ? cacheHits / totalAccesses : 0;
  }
}

interface CachedComponent {
  component: any;
  createdAt: number;
  lastAccess: number;
  accessCount: number;
  memoryFootprint: number;
}

interface ComponentMemoryInfo {
  estimatedSize: number;
  createdAt: number;
  type: string;
  properties: number;
}

interface MemoryOptimizationResult {
  freedComponents: number;
  freedMemory: number;
  beforeOptimization: number;
  afterOptimization: number;
  improvement: number;
}

interface ComponentMemoryReport {
  totalComponents: number;
  totalMemoryUsage: number;
  components: (ComponentMemoryInfo & { id: string })[];
  cacheHitRate: number;
}
```

## Conclusion

Effective memory management in ArkTS applications requires:

- Advanced memory pool management for object reuse
- Smart reference management with automatic cleanup
- Proactive memory leak detection and monitoring
- Memory-aware component lifecycle management
- Regular garbage collection optimization
- Performance monitoring and memory profiling

These techniques ensure applications maintain optimal memory usage while providing smooth user experiences across all HarmonyOS devices.
