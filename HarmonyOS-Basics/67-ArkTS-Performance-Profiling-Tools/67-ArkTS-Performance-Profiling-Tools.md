# ArkTS Performance Profiling Tools

## Introduction

Performance profiling is essential for optimizing ArkTS applications. This guide explores profiling tools, performance metrics collection, bottleneck detection, and optimization strategies for HarmonyOS applications.

## Performance Monitoring Framework

### Core Performance Metrics

```typescript
interface PerformanceMetrics {
  renderTime: number;
  memoryUsage: number;
  cpuUsage: number;
  frameRate: number;
  networkLatency: number;
  bundleSize: number;
}

interface MetricSample {
  timestamp: number;
  value: number;
  metadata?: Record<string, any>;
}

class PerformanceCollector {
  private metrics = new Map<string, MetricSample[]>();
  private observers: PerformanceObserver[] = [];
  private intervalId?: number;

  start(): void {
    this.setupObservers();
    this.startCollection();
  }

  stop(): void {
    this.observers.forEach((observer) => observer.disconnect());
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  }

  getMetrics(metricName: string, duration = 60000): MetricSample[] {
    const samples = this.metrics.get(metricName) || [];
    const cutoff = Date.now() - duration;
    return samples.filter((sample) => sample.timestamp > cutoff);
  }

  private setupObservers(): void {
    // Frame timing observer
    const frameObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        this.recordMetric("frame-timing", entry.duration);
      }
    });
    frameObserver.observe({ entryTypes: ["measure"] });
    this.observers.push(frameObserver);

    // Navigation timing observer
    const navObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        this.recordMetric(
          "navigation",
          entry.loadEventEnd - entry.navigationStart
        );
      }
    });
    navObserver.observe({ entryTypes: ["navigation"] });
    this.observers.push(navObserver);
  }

  private startCollection(): void {
    this.intervalId = setInterval(() => {
      this.collectSystemMetrics();
    }, 1000);
  }

  private collectSystemMetrics(): void {
    // Memory usage
    if ("memory" in performance) {
      const memory = (performance as any).memory;
      this.recordMetric("memory-used", memory.usedJSHeapSize);
      this.recordMetric("memory-total", memory.totalJSHeapSize);
    }

    // Frame rate estimation
    this.recordMetric("frame-rate", this.estimateFrameRate());
  }

  private recordMetric(
    name: string,
    value: number,
    metadata?: Record<string, any>
  ): void {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }

    const samples = this.metrics.get(name)!;
    samples.push({
      timestamp: Date.now(),
      value,
      metadata,
    });

    // Keep only recent samples
    const cutoff = Date.now() - 300000; // 5 minutes
    const recentSamples = samples.filter((sample) => sample.timestamp > cutoff);
    this.metrics.set(name, recentSamples);
  }

  private estimateFrameRate(): number {
    // Simplified frame rate estimation
    return 60; // Would use actual frame timing in practice
  }
}
```

### Component Performance Profiler

```typescript
interface ComponentProfile {
  name: string;
  renderCount: number;
  avgRenderTime: number;
  totalRenderTime: number;
  lastRenderTime: number;
  memoryImpact: number;
}

class ComponentProfiler {
  private profiles = new Map<string, ComponentProfile>();
  private renderTimings = new Map<string, number>();

  startRender(componentName: string): void {
    this.renderTimings.set(componentName, performance.now());
  }

  endRender(componentName: string): void {
    const startTime = this.renderTimings.get(componentName);
    if (!startTime) return;

    const renderTime = performance.now() - startTime;
    this.renderTimings.delete(componentName);

    let profile = this.profiles.get(componentName);
    if (!profile) {
      profile = {
        name: componentName,
        renderCount: 0,
        avgRenderTime: 0,
        totalRenderTime: 0,
        lastRenderTime: 0,
        memoryImpact: 0,
      };
      this.profiles.set(componentName, profile);
    }

    profile.renderCount++;
    profile.lastRenderTime = renderTime;
    profile.totalRenderTime += renderTime;
    profile.avgRenderTime = profile.totalRenderTime / profile.renderCount;
  }

  getProfile(componentName: string): ComponentProfile | undefined {
    return this.profiles.get(componentName);
  }

  getAllProfiles(): ComponentProfile[] {
    return Array.from(this.profiles.values()).sort(
      (a, b) => b.avgRenderTime - a.avgRenderTime
    );
  }

  getSlowComponents(threshold = 16): ComponentProfile[] {
    return this.getAllProfiles().filter(
      (profile) => profile.avgRenderTime > threshold
    );
  }

  reset(): void {
    this.profiles.clear();
    this.renderTimings.clear();
  }
}

// Performance-aware component decorator
function ProfiledComponent(name?: string) {
  return function <T extends new (...args: any[]) => any>(constructor: T) {
    const componentName = name || constructor.name;

    return class extends constructor {
      private static profiler = new ComponentProfiler();

      aboutToAppear() {
        ComponentProfiler.prototype.startRender.call(
          this.constructor.profiler,
          componentName
        );
        if (super.aboutToAppear) {
          super.aboutToAppear();
        }
      }

      aboutToDisappear() {
        ComponentProfiler.prototype.endRender.call(
          this.constructor.profiler,
          componentName
        );
        if (super.aboutToDisappear) {
          super.aboutToDisappear();
        }
      }

      static getProfile(): ComponentProfile | undefined {
        return this.profiler.getProfile(componentName);
      }
    };
  };
}
```

## Memory Profiling Tools

### Memory Leak Detector

```typescript
interface ObjectRef {
  id: string;
  type: string;
  size: number;
  timestamp: number;
  stackTrace?: string;
}

class MemoryLeakDetector {
  private objects = new Map<string, ObjectRef>();
  private leakThreshold = 1000; // Max objects of same type
  private checkInterval = 30000; // 30 seconds

  start(): void {
    setInterval(() => {
      this.checkForLeaks();
    }, this.checkInterval);
  }

  trackObject(obj: any, type: string): string {
    const id = this.generateId();
    const ref: ObjectRef = {
      id,
      type,
      size: this.estimateSize(obj),
      timestamp: Date.now(),
      stackTrace: this.captureStackTrace(),
    };

    this.objects.set(id, ref);
    return id;
  }

  releaseObject(id: string): void {
    this.objects.delete(id);
  }

  private checkForLeaks(): void {
    const typeGroups = new Map<string, ObjectRef[]>();

    this.objects.forEach((ref) => {
      if (!typeGroups.has(ref.type)) {
        typeGroups.set(ref.type, []);
      }
      typeGroups.get(ref.type)!.push(ref);
    });

    typeGroups.forEach((refs, type) => {
      if (refs.length > this.leakThreshold) {
        this.reportLeak(type, refs);
      }
    });
  }

  private reportLeak(type: string, refs: ObjectRef[]): void {
    console.warn(
      `Potential memory leak detected: ${refs.length} instances of ${type}`
    );

    // Report oldest instances
    const oldest = refs.sort((a, b) => a.timestamp - b.timestamp).slice(0, 5);

    oldest.forEach((ref) => {
      console.log(
        `Leaked object: ${ref.id}, age: ${Date.now() - ref.timestamp}ms`
      );
      if (ref.stackTrace) {
        console.log(ref.stackTrace);
      }
    });
  }

  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }

  private estimateSize(obj: any): number {
    return JSON.stringify(obj).length * 2; // Rough estimate
  }

  private captureStackTrace(): string {
    return new Error().stack || "";
  }
}
```

### Bundle Size Analyzer

```typescript
interface BundleChunk {
  name: string;
  size: number;
  gzipSize: number;
  modules: ModuleInfo[];
}

interface ModuleInfo {
  name: string;
  size: number;
  imports: string[];
  exports: string[];
}

class BundleAnalyzer {
  async analyzeBundles(): Promise<BundleChunk[]> {
    // This would analyze the actual bundle files
    return [
      {
        name: "main.js",
        size: 245760,
        gzipSize: 89432,
        modules: [
          {
            name: "@ohos/utils",
            size: 45280,
            imports: [],
            exports: ["formatDate", "validateEmail"],
          },
        ],
      },
    ];
  }

  generateReport(chunks: BundleChunk[]): BundleReport {
    const totalSize = chunks.reduce((sum, chunk) => sum + chunk.size, 0);
    const totalGzipSize = chunks.reduce(
      (sum, chunk) => sum + chunk.gzipSize,
      0
    );

    const largestChunk = chunks.reduce((largest, chunk) =>
      chunk.size > largest.size ? chunk : largest
    );

    return {
      totalSize,
      totalGzipSize,
      compressionRatio: totalGzipSize / totalSize,
      chunks: chunks.sort((a, b) => b.size - a.size),
      largestChunk,
      recommendations: this.generateRecommendations(chunks),
    };
  }

  private generateRecommendations(chunks: BundleChunk[]): string[] {
    const recommendations: string[] = [];

    chunks.forEach((chunk) => {
      if (chunk.size > 500000) {
        // > 500KB
        recommendations.push(`Consider code splitting for ${chunk.name}`);
      }

      if (chunk.gzipSize / chunk.size > 0.8) {
        // Poor compression
        recommendations.push(
          `${chunk.name} may contain binary data or already compressed content`
        );
      }
    });

    return recommendations;
  }
}

interface BundleReport {
  totalSize: number;
  totalGzipSize: number;
  compressionRatio: number;
  chunks: BundleChunk[];
  largestChunk: BundleChunk;
  recommendations: string[];
}
```

## Performance Dashboard

### Real-time Performance Monitor

```typescript
@Component
struct PerformanceDashboard {
  @State private metrics: PerformanceMetrics = {
    renderTime: 0,
    memoryUsage: 0,
    cpuUsage: 0,
    frameRate: 60,
    networkLatency: 0,
    bundleSize: 0
  }

  @State private isVisible: boolean = false
  private collector = new PerformanceCollector()
  private updateTimer?: number

  build() {
    if (this.isVisible) {
      Column() {
        this.buildHeader()
        this.buildMetricsGrid()
        this.buildCharts()
        this.buildRecommendations()
      }
      .width('100%')
      .height('100%')
      .backgroundColor('#f5f5f5')
      .padding(16)
    }
  }

  aboutToAppear() {
    this.collector.start()
    this.startUpdates()
  }

  aboutToDisappear() {
    this.collector.stop()
    if (this.updateTimer) {
      clearInterval(this.updateTimer)
    }
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Performance Dashboard')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)

      Blank()

      Button(this.isVisible ? 'Hide' : 'Show')
        .onClick(() => this.isVisible = !this.isVisible)
    }
    .width('100%')
    .margin({ bottom: 16 })
  }

  @Builder
  private buildMetricsGrid() {
    Grid() {
      GridItem() {
        this.buildMetricCard('Frame Rate', `${this.metrics.frameRate.toFixed(1)} fps`, '#4CAF50')
      }

      GridItem() {
        this.buildMetricCard('Memory', `${(this.metrics.memoryUsage / 1024 / 1024).toFixed(1)} MB`, '#2196F3')
      }

      GridItem() {
        this.buildMetricCard('Render Time', `${this.metrics.renderTime.toFixed(1)} ms`, '#FF9800')
      }

      GridItem() {
        this.buildMetricCard('Network', `${this.metrics.networkLatency.toFixed(0)} ms`, '#9C27B0')
      }
    }
    .columnsTemplate('1fr 1fr')
    .rowsTemplate('1fr 1fr')
    .columnsGap(12)
    .rowsGap(12)
    .height(200)
    .margin({ bottom: 16 })
  }

  @Builder
  private buildMetricCard(title: string, value: string, color: string) {
    Column() {
      Text(title)
        .fontSize(12)
        .fontColor('#666')

      Text(value)
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .fontColor(color)
        .margin({ top: 4 })
    }
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
    .width('100%')
    .height('100%')
    .backgroundColor(Color.White)
    .borderRadius(8)
    .border({ width: 1, color: '#e0e0e0' })
  }

  @Builder
  private buildCharts() {
    Column() {
      Text('Performance Trends')
        .fontSize(16)
        .fontWeight(FontWeight.Medium)
        .margin({ bottom: 8 })

      // Simplified chart representation
      Row() {
        ForEach(Array.from({ length: 20 }, (_, i) => i), (index: number) => {
          Column()
            .width(8)
            .height(`${Math.random() * 100}%`)
            .backgroundColor('#2196F3')
            .margin({ right: 2 })
        })
      }
      .height(100)
      .width('100%')
      .backgroundColor(Color.White)
      .borderRadius(8)
      .padding(8)
    }
    .margin({ bottom: 16 })
  }

  @Builder
  private buildRecommendations() {
    Column() {
      Text('Performance Recommendations')
        .fontSize(16)
        .fontWeight(FontWeight.Medium)
        .margin({ bottom: 8 })

      Column() {
        this.buildRecommendationItem('⚡', 'Consider lazy loading for large components')
        this.buildRecommendationItem('🎯', 'Optimize image sizes and formats')
        this.buildRecommendationItem('📦', 'Enable code splitting for better load times')
      }
      .backgroundColor(Color.White)
      .borderRadius(8)
      .padding(12)
    }
  }

  @Builder
  private buildRecommendationItem(icon: string, text: string) {
    Row() {
      Text(icon)
        .fontSize(16)
        .margin({ right: 8 })

      Text(text)
        .fontSize(14)
        .flexGrow(1)
    }
    .margin({ bottom: 8 })
  }

  private startUpdates(): void {
    this.updateTimer = setInterval(() => {
      this.updateMetrics()
    }, 1000)
  }

  private updateMetrics(): void {
    // Get latest metrics from collector
    const frameRateData = this.collector.getMetrics('frame-rate', 5000)
    const memoryData = this.collector.getMetrics('memory-used', 5000)

    if (frameRateData.length > 0) {
      this.metrics.frameRate = frameRateData[frameRateData.length - 1].value
    }

    if (memoryData.length > 0) {
      this.metrics.memoryUsage = memoryData[memoryData.length - 1].value
    }

    // Update other metrics...
    this.metrics.renderTime = Math.random() * 20 // Simulated
    this.metrics.networkLatency = Math.random() * 200 // Simulated
  }

  toggleVisibility(): void {
    this.isVisible = !this.isVisible
  }
}
```

## Automated Performance Testing

```typescript
interface PerformanceTest {
  name: string;
  run(): Promise<PerformanceTestResult>;
}

interface PerformanceTestResult {
  passed: boolean;
  duration: number;
  metrics: Record<string, number>;
  errors: string[];
}

class PerformanceTestSuite {
  private tests: PerformanceTest[] = [];

  addTest(test: PerformanceTest): void {
    this.tests.push(test);
  }

  async runAll(): Promise<PerformanceTestResult[]> {
    const results: PerformanceTestResult[] = [];

    for (const test of this.tests) {
      console.log(`Running test: ${test.name}`);
      const result = await test.run();
      results.push(result);
    }

    return results;
  }
}

// Example performance tests
class RenderPerformanceTest implements PerformanceTest {
  name = "Component Render Performance";

  async run(): Promise<PerformanceTestResult> {
    const startTime = performance.now();
    const errors: string[] = [];

    try {
      // Simulate component rendering
      for (let i = 0; i < 100; i++) {
        await this.renderComponent();
      }
    } catch (error) {
      errors.push(error.message);
    }

    const duration = performance.now() - startTime;
    const avgRenderTime = duration / 100;

    return {
      passed: avgRenderTime < 16 && errors.length === 0,
      duration,
      metrics: {
        avgRenderTime,
        totalRenders: 100,
      },
      errors,
    };
  }

  private async renderComponent(): Promise<void> {
    // Simulate component render
    await new Promise((resolve) => setTimeout(resolve, Math.random() * 10));
  }
}

class MemoryLeakTest implements PerformanceTest {
  name = "Memory Leak Detection";

  async run(): Promise<PerformanceTestResult> {
    const startTime = performance.now();
    const initialMemory = this.getMemoryUsage();
    const errors: string[] = [];

    try {
      // Simulate memory-intensive operations
      for (let i = 0; i < 1000; i++) {
        this.createObjects();
      }

      // Force garbage collection (if available)
      if (global.gc) {
        global.gc();
      }

      await new Promise((resolve) => setTimeout(resolve, 1000));
    } catch (error) {
      errors.push(error.message);
    }

    const finalMemory = this.getMemoryUsage();
    const memoryIncrease = finalMemory - initialMemory;
    const duration = performance.now() - startTime;

    return {
      passed: memoryIncrease < 10 * 1024 * 1024, // < 10MB
      duration,
      metrics: {
        initialMemory,
        finalMemory,
        memoryIncrease,
      },
      errors,
    };
  }

  private createObjects(): void {
    // Create temporary objects
    const objects = Array.from({ length: 100 }, () => ({
      data: new Array(1000).fill(Math.random()),
    }));
    // Objects should be garbage collected
  }

  private getMemoryUsage(): number {
    if ("memory" in performance) {
      return (performance as any).memory.usedJSHeapSize;
    }
    return 0;
  }
}

// Usage
const testSuite = new PerformanceTestSuite();
testSuite.addTest(new RenderPerformanceTest());
testSuite.addTest(new MemoryLeakTest());

// Run tests
testSuite.runAll().then((results) => {
  results.forEach((result) => {
    console.log(`Test: ${result.passed ? "PASSED" : "FAILED"}`);
    console.log(`Duration: ${result.duration.toFixed(2)}ms`);
    console.log(`Metrics:`, result.metrics);
    if (result.errors.length > 0) {
      console.log(`Errors:`, result.errors);
    }
  });
});
```

## Conclusion

Performance profiling tools in ArkTS enable:

- Comprehensive performance metrics collection
- Component-level render profiling
- Memory leak detection and monitoring
- Bundle size analysis and optimization
- Real-time performance dashboards
- Automated performance testing

These tools provide the insights needed to identify performance bottlenecks and optimize application performance across all aspects of HarmonyOS development.
