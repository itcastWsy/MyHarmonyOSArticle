# ArkUI Performance Monitoring and Optimization

## Introduction

Performance monitoring and optimization are crucial for delivering smooth user experiences in HarmonyOS applications. This guide covers advanced performance monitoring techniques, optimization strategies, and tools for building high-performance ArkUI applications.

## Performance Monitoring Framework

### Real-time Performance Metrics

```typescript
interface PerformanceMetrics {
  frameRate: number;
  renderTime: number;
  memoryUsage: number;
  cpuUsage: number;
  networkLatency: number;
  loadTime: number;
}

class PerformanceMonitor {
  private metrics: PerformanceMetrics = {
    frameRate: 0,
    renderTime: 0,
    memoryUsage: 0,
    cpuUsage: 0,
    networkLatency: 0,
    loadTime: 0,
  };

  private frameCount = 0;
  private lastFrameTime = 0;
  private observers: Set<(metrics: PerformanceMetrics) => void> = new Set();

  startMonitoring(): void {
    this.monitorFrameRate();
    this.monitorMemoryUsage();
    this.monitorRenderTime();
    this.setupPerformanceObserver();
  }

  getMetrics(): PerformanceMetrics {
    return { ...this.metrics };
  }

  subscribe(observer: (metrics: PerformanceMetrics) => void): () => void {
    this.observers.add(observer);
    return () => this.observers.delete(observer);
  }

  private monitorFrameRate(): void {
    const measureFrame = () => {
      const currentTime = performance.now();
      if (this.lastFrameTime > 0) {
        const frameDuration = currentTime - this.lastFrameTime;
        this.metrics.frameRate = 1000 / frameDuration;
      }
      this.lastFrameTime = currentTime;
      this.frameCount++;

      requestAnimationFrame(measureFrame);
    };

    requestAnimationFrame(measureFrame);
  }

  private monitorMemoryUsage(): void {
    setInterval(() => {
      if ("memory" in performance) {
        const memory = (performance as any).memory;
        this.metrics.memoryUsage = memory.usedJSHeapSize / 1024 / 1024; // MB
      }
      this.notifyObservers();
    }, 1000);
  }

  private setupPerformanceObserver(): void {
    if ("PerformanceObserver" in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          this.processPerformanceEntry(entry);
        }
      });

      observer.observe({ entryTypes: ["measure", "navigation", "paint"] });
    }
  }

  private processPerformanceEntry(entry: PerformanceEntry): void {
    switch (entry.entryType) {
      case "measure":
        if (entry.name.includes("render")) {
          this.metrics.renderTime = entry.duration;
        }
        break;
      case "navigation":
        this.metrics.loadTime = entry.duration;
        break;
    }
  }

  private notifyObservers(): void {
    this.observers.forEach((observer) => observer(this.metrics));
  }
}

// Component performance profiler
class ComponentProfiler {
  private componentMetrics = new Map<string, ComponentMetrics>();

  profileComponent(componentName: string): PropertyDecorator {
    return (target: any, propertyKey: string | symbol) => {
      const originalMethod = target[propertyKey];

      target[propertyKey] = function (...args: any[]) {
        const startTime = performance.now();
        const result = originalMethod.apply(this, args);
        const endTime = performance.now();

        this.recordComponentMetric(componentName, endTime - startTime);
        return result;
      };
    };
  }

  private recordComponentMetric(componentName: string, duration: number): void {
    if (!this.componentMetrics.has(componentName)) {
      this.componentMetrics.set(componentName, {
        totalRenderTime: 0,
        renderCount: 0,
        averageRenderTime: 0,
      });
    }

    const metrics = this.componentMetrics.get(componentName)!;
    metrics.totalRenderTime += duration;
    metrics.renderCount++;
    metrics.averageRenderTime = metrics.totalRenderTime / metrics.renderCount;
  }

  getComponentMetrics(componentName: string): ComponentMetrics | undefined {
    return this.componentMetrics.get(componentName);
  }
}

interface ComponentMetrics {
  totalRenderTime: number;
  renderCount: number;
  averageRenderTime: number;
}
```

### Performance Bottleneck Detection

```typescript
class BottleneckDetector {
  private thresholds = {
    frameRate: 55, // FPS
    renderTime: 16, // ms
    memoryUsage: 100, // MB
    loadTime: 3000, // ms
  };

  private alerts: PerformanceAlert[] = [];

  analyzePerformance(metrics: PerformanceMetrics): PerformanceAlert[] {
    const alerts: PerformanceAlert[] = [];

    if (metrics.frameRate < this.thresholds.frameRate) {
      alerts.push({
        type: "framerate",
        severity: "high",
        message: `Low frame rate detected: ${metrics.frameRate.toFixed(1)} FPS`,
        suggestions: [
          "Reduce complex animations",
          "Optimize list rendering",
          "Use virtual scrolling for large lists",
        ],
      });
    }

    if (metrics.renderTime > this.thresholds.renderTime) {
      alerts.push({
        type: "rendertime",
        severity: "medium",
        message: `High render time: ${metrics.renderTime.toFixed(2)}ms`,
        suggestions: [
          "Optimize component hierarchy",
          "Use @Builder for repeated components",
          "Implement lazy loading",
        ],
      });
    }

    if (metrics.memoryUsage > this.thresholds.memoryUsage) {
      alerts.push({
        type: "memory",
        severity: "high",
        message: `High memory usage: ${metrics.memoryUsage.toFixed(1)}MB`,
        suggestions: [
          "Implement object pooling",
          "Clear unused references",
          "Optimize image loading",
        ],
      });
    }

    return alerts;
  }
}

interface PerformanceAlert {
  type: string;
  severity: "low" | "medium" | "high";
  message: string;
  suggestions: string[];
}
```

## Optimization Strategies

### Component Optimization

```typescript
// Memoized component for expensive renders
class MemoizedComponent {
  private static cache = new Map<string, any>()

  static create<T>(key: string, factory: () => T, dependencies: any[]): T {
    const cacheKey = this.generateCacheKey(key, dependencies)

    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey)
    }

    const component = factory()
    this.cache.set(cacheKey, component)
    return component
  }

  private static generateCacheKey(key: string, dependencies: any[]): string {
    return `${key}_${JSON.stringify(dependencies)}`
  }

  static clearCache(): void {
    this.cache.clear()
  }
}

// Optimized list component
@Component
struct OptimizedList {
  @Prop items: any[]
  @Prop renderItem: (item: any, index: number) => void
  @State private visibleRange: [number, number] = [0, 0]
  @State private itemHeight: number = 80
  private scrollController = new Scroller()

  build() {
    List({ scroller: this.scrollController }) {
      LazyForEach(
        new VirtualDataSource(this.items, this.visibleRange),
        (item: any, index: number) => `${index}_${item.id}`,
        (item: any, index: number) => {
          ListItem() {
            this.buildOptimizedItem(item, index)
          }
          .height(this.itemHeight)
        }
      )
    }
    .onScrollIndex((start: number, end: number) => {
      this.updateVisibleRange(start, end)
    })
    .cachedCount(5) // Optimize rendering
  }

  @Builder
  buildOptimizedItem(item: any, index: number) {
    // Use memoization for expensive components
    MemoizedComponent.create(
      `list_item_${index}`,
      () => this.renderItem(item, index),
      [item.id, item.updatedAt]
    )
  }

  private updateVisibleRange(start: number, end: number): void {
    this.visibleRange = [start, end]
  }
}

// Virtual data source implementation
class VirtualDataSource<T> implements IDataSource {
  private data: T[]
  private visibleRange: [number, number]
  private listeners: DataChangeListener[] = []

  constructor(data: T[], visibleRange: [number, number]) {
    this.data = data
    this.visibleRange = visibleRange
  }

  totalCount(): number {
    return this.data.length
  }

  getData(index: number): T {
    return this.data[index]
  }

  registerDataChangeListener(listener: DataChangeListener): void {
    this.listeners.push(listener)
  }

  unregisterDataChangeListener(listener: DataChangeListener): void {
    const index = this.listeners.indexOf(listener)
    if (index >= 0) {
      this.listeners.splice(index, 1)
    }
  }
}
```

### Image Optimization

```typescript
class ImageOptimizer {
  private imageCache = new Map<string, ImageBitmap>()
  private loadingPromises = new Map<string, Promise<ImageBitmap>>()

  async loadOptimizedImage(url: string, options: ImageOptions): Promise<ImageBitmap> {
    const cacheKey = this.generateCacheKey(url, options)

    // Return cached image if available
    if (this.imageCache.has(cacheKey)) {
      return this.imageCache.get(cacheKey)!
    }

    // Return ongoing promise if already loading
    if (this.loadingPromises.has(cacheKey)) {
      return this.loadingPromises.get(cacheKey)!
    }

    // Start loading the image
    const loadingPromise = this.loadAndProcessImage(url, options)
    this.loadingPromises.set(cacheKey, loadingPromise)

    try {
      const bitmap = await loadingPromise
      this.imageCache.set(cacheKey, bitmap)
      return bitmap
    } finally {
      this.loadingPromises.delete(cacheKey)
    }
  }

  private async loadAndProcessImage(url: string, options: ImageOptions): Promise<ImageBitmap> {
    const response = await fetch(url)
    const blob = await response.blob()

    // Resize image if needed
    if (options.maxWidth || options.maxHeight) {
      return this.resizeImage(blob, options)
    }

    return createImageBitmap(blob)
  }

  private async resizeImage(blob: Blob, options: ImageOptions): Promise<ImageBitmap> {
    const canvas = new OffscreenCanvas(
      options.maxWidth || 800,
      options.maxHeight || 600
    )
    const ctx = canvas.getContext('2d')!

    const originalBitmap = await createImageBitmap(blob)

    // Calculate new dimensions maintaining aspect ratio
    const { width, height } = this.calculateDimensions(
      originalBitmap.width,
      originalBitmap.height,
      options
    )

    canvas.width = width
    canvas.height = height

    ctx.drawImage(originalBitmap, 0, 0, width, height)

    return canvas.transferToImageBitmap()
  }

  private calculateDimensions(
    originalWidth: number,
    originalHeight: number,
    options: ImageOptions
  ): { width: number; height: number } {
    const aspectRatio = originalWidth / originalHeight

    let width = options.maxWidth || originalWidth
    let height = options.maxHeight || originalHeight

    if (options.maxWidth && options.maxHeight) {
      if (width / height > aspectRatio) {
        width = height * aspectRatio
      } else {
        height = width / aspectRatio
      }
    } else if (options.maxWidth) {
      height = width / aspectRatio
    } else if (options.maxHeight) {
      width = height * aspectRatio
    }

    return { width, height }
  }

  private generateCacheKey(url: string, options: ImageOptions): string {
    return `${url}_${options.maxWidth || 0}_${options.maxHeight || 0}_${options.quality || 1}`
  }

  clearCache(): void {
    this.imageCache.clear()
  }
}

interface ImageOptions {
  maxWidth?: number
  maxHeight?: number
  quality?: number
}

// Optimized image component
@Component
struct OptimizedImage {
  @Prop src: string
  @Prop width?: number
  @Prop height?: number
  @State private imageBitmap: ImageBitmap | null = null
  @State private loading: boolean = true
  private imageOptimizer = new ImageOptimizer()

  aboutToAppear() {
    this.loadImage()
  }

  build() {
    if (this.loading) {
      this.buildLoadingPlaceholder()
    } else if (this.imageBitmap) {
      this.buildImage()
    } else {
      this.buildErrorPlaceholder()
    }
  }

  @Builder
  buildImage() {
    Canvas(this.imageBitmap!)
      .width(this.width || '100%')
      .height(this.height || '100%')
      .objectFit(ImageFit.Cover)
  }

  @Builder
  buildLoadingPlaceholder() {
    Column() {
      LoadingProgress()
        .width(32)
        .height(32)
    }
    .width(this.width || '100%')
    .height(this.height || '100%')
    .justifyContent(FlexAlign.Center)
    .backgroundColor('#f0f0f0')
  }

  @Builder
  buildErrorPlaceholder() {
    Column() {
      Image('/resources/image_error.png')
        .width(32)
        .height(32)
    }
    .width(this.width || '100%')
    .height(this.height || '100%')
    .justifyContent(FlexAlign.Center)
    .backgroundColor('#f0f0f0')
  }

  private async loadImage(): Promise<void> {
    try {
      this.imageBitmap = await this.imageOptimizer.loadOptimizedImage(this.src, {
        maxWidth: this.width,
        maxHeight: this.height,
        quality: 0.8
      })
    } catch (error) {
      console.error('Failed to load image:', error)
    } finally {
      this.loading = false
    }
  }
}
```

## Performance Debugging

### Debug Tools Integration

```typescript
class PerformanceDebugger {
  private isEnabled = false
  private debugOverlay: DebugOverlay | null = null

  enable(): void {
    if (this.isEnabled) return

    this.isEnabled = true
    this.createDebugOverlay()
    this.startProfiling()
  }

  disable(): void {
    if (!this.isEnabled) return

    this.isEnabled = false
    this.removeDebugOverlay()
    this.stopProfiling()
  }

  private createDebugOverlay(): void {
    this.debugOverlay = new DebugOverlay()
    this.debugOverlay.show()
  }

  private startProfiling(): void {
    const monitor = new PerformanceMonitor()
    monitor.subscribe((metrics) => {
      if (this.debugOverlay) {
        this.debugOverlay.updateMetrics(metrics)
      }
    })
    monitor.startMonitoring()
  }
}

// Debug overlay component
@Component
struct DebugOverlay {
  @State private metrics: PerformanceMetrics | null = null
  @State private visible: boolean = false

  build() {
    if (this.visible && this.metrics) {
      Column() {
        Text('Performance Metrics')
          .fontSize(16)
          .fontWeight(FontWeight.Bold)
          .margin({ bottom: 8 })

        Text(`FPS: ${this.metrics.frameRate.toFixed(1)}`)
          .fontSize(12)
          .fontColor(this.getFPSColor())

        Text(`Render: ${this.metrics.renderTime.toFixed(2)}ms`)
          .fontSize(12)

        Text(`Memory: ${this.metrics.memoryUsage.toFixed(1)}MB`)
          .fontSize(12)
          .fontColor(this.getMemoryColor())
      }
      .padding(8)
      .backgroundColor('rgba(0, 0, 0, 0.8)')
      .borderRadius(4)
      .position({ x: 10, y: 10 })
    }
  }

  updateMetrics(metrics: PerformanceMetrics): void {
    this.metrics = metrics
  }

  show(): void {
    this.visible = true
  }

  hide(): void {
    this.visible = false
  }

  private getFPSColor(): Color {
    if (!this.metrics) return Color.White

    if (this.metrics.frameRate >= 55) return Color.Green
    if (this.metrics.frameRate >= 30) return Color.Yellow
    return Color.Red
  }

  private getMemoryColor(): Color {
    if (!this.metrics) return Color.White

    if (this.metrics.memoryUsage < 50) return Color.Green
    if (this.metrics.memoryUsage < 100) return Color.Yellow
    return Color.Red
  }
}
```

## Conclusion

Performance monitoring and optimization in ArkUI applications require:

- Real-time performance metrics collection
- Bottleneck detection and analysis
- Component optimization strategies
- Image and resource optimization
- Virtual scrolling for large datasets
- Memory management best practices
- Debug tools integration

Implementing these strategies ensures smooth, responsive user experiences across all HarmonyOS devices while maintaining optimal resource utilization.
