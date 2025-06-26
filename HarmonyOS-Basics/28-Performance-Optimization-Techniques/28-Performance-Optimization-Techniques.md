# 28-Performance Optimization Techniques

## Introduction

Performance optimization is crucial for creating smooth, responsive HarmonyOS applications. This article covers various optimization techniques including UI rendering optimization, memory management, CPU usage optimization, and profiling tools.

## UI Rendering Optimization

### Efficient Component Design

```typescript
// Use @Builder for reusable UI patterns
@Builder
function ListItemBuilder(title: string, subtitle: string, icon: Resource) {
  Row() {
    Image(icon)
      .width(40)
      .height(40)
      .margin({ right: 12 })

    Column() {
      Text(title)
        .fontSize(16)
        .fontWeight(FontWeight.Medium)
      Text(subtitle)
        .fontSize(14)
        .fontColor(Color.Gray)
    }
    .alignItems(HorizontalAlign.Start)
  }
  .width('100%')
  .padding(16)
}

// Optimized list component with lazy loading
@Component
struct OptimizedList {
  @State private items: ListItem[] = [];
  private readonly BATCH_SIZE = 20;
  private currentIndex = 0;

  aboutToAppear(): void {
    this.loadNextBatch();
  }

  private loadNextBatch(): void {
    const nextBatch = this.generateItems(this.currentIndex, this.BATCH_SIZE);
    this.items = [...this.items, ...nextBatch];
    this.currentIndex += this.BATCH_SIZE;
  }

  build() {
    List() {
      LazyForEach(new ListDataSource(this.items), (item: ListItem) => {
        ListItem() {
          ListItemBuilder(item.title, item.subtitle, item.icon)
        }
      }, (item: ListItem) => item.id)
    }
    .onReachEnd(() => {
      this.loadNextBatch();
    })
    .cachedCount(5) // Cache 5 items for smooth scrolling
  }
}

// Efficient data source for LazyForEach
class ListDataSource implements IDataSource {
  private dataArray: ListItem[] = [];

  constructor(dataArray: ListItem[]) {
    this.dataArray = dataArray;
  }

  totalCount(): number {
    return this.dataArray.length;
  }

  getData(index: number): ListItem {
    return this.dataArray[index];
  }

  registerDataChangeListener(): void {}
  unregisterDataChangeListener(): void {}
}
```

### State Management Optimization

```typescript
// Use @ObjectLink for complex object updates
@Observed
class ListState {
  items: ObservableItem[] = [];
  selectedId: string = '';

  updateItem(id: string, updates: Partial<ObservableItem>): void {
    const item = this.items.find(item => item.id === id);
    if (item) {
      Object.assign(item, updates);
    }
  }

  toggleSelection(id: string): void {
    this.selectedId = this.selectedId === id ? '' : id;
  }
}

@Observed
class ObservableItem {
  id: string = '';
  title: string = '';
  completed: boolean = false;

  constructor(id: string, title: string) {
    this.id = id;
    this.title = title;
  }
}

// Efficient item component
@Component
struct OptimizedItemComponent {
  @ObjectLink item: ObservableItem;
  @Link selectedId: string;
  private onToggle?: (id: string) => void;

  build() {
    Row() {
      Checkbox({ name: this.item.id })
        .select(this.item.completed)
        .onChange((checked: boolean) => {
          this.item.completed = checked; // Direct update, no re-render of parent
        })

      Text(this.item.title)
        .layoutWeight(1)
        .fontSize(16)
        .margin({ left: 12 })
        .decoration({
          type: this.item.completed ? TextDecorationType.LineThrough : TextDecorationType.None
        })
    }
    .width('100%')
    .padding(12)
    .backgroundColor(this.selectedId === this.item.id ? '#E3F2FD' : Color.Transparent)
    .onClick(() => {
      if (this.onToggle) {
        this.onToggle(this.item.id);
      }
    })
  }

  setToggleHandler(handler: (id: string) => void): OptimizedItemComponent {
    this.onToggle = handler;
    return this;
  }
}
```

## Memory Management

### Memory Pool Pattern

```typescript
class ObjectPool<T> {
  private pool: T[] = [];
  private createFn: () => T;
  private resetFn: (obj: T) => void;
  private maxSize: number;

  constructor(
    createFn: () => T,
    resetFn: (obj: T) => void,
    maxSize: number = 50
  ) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.maxSize = maxSize;
  }

  acquire(): T {
    if (this.pool.length > 0) {
      return this.pool.pop()!;
    }
    return this.createFn();
  }

  release(obj: T): void {
    if (this.pool.length < this.maxSize) {
      this.resetFn(obj);
      this.pool.push(obj);
    }
  }

  clear(): void {
    this.pool = [];
  }
}

// Example usage with image cache
class ImageCache {
  private static readonly CACHE_SIZE = 100;
  private cache: Map<string, PixelMap> = new Map();
  private accessOrder: string[] = [];

  set(key: string, image: PixelMap): void {
    if (this.cache.has(key)) {
      // Move to end of access order
      const index = this.accessOrder.indexOf(key);
      this.accessOrder.splice(index, 1);
      this.accessOrder.push(key);
      return;
    }

    if (this.cache.size >= ImageCache.CACHE_SIZE) {
      // Remove least recently used
      const lruKey = this.accessOrder.shift()!;
      this.cache.delete(lruKey);
    }

    this.cache.set(key, image);
    this.accessOrder.push(key);
  }

  get(key: string): PixelMap | null {
    const image = this.cache.get(key);
    if (image) {
      // Update access order
      const index = this.accessOrder.indexOf(key);
      this.accessOrder.splice(index, 1);
      this.accessOrder.push(key);
    }
    return image || null;
  }

  clear(): void {
    this.cache.clear();
    this.accessOrder = [];
  }
}
```

### Resource Management

```typescript
export class ResourceManager {
  private static timers: Set<number> = new Set();
  private static intervals: Set<number> = new Set();
  private static subscriptions: Set<() => void> = new Set();

  static setTimeout(callback: () => void, delay: number): number {
    const timerId = setTimeout(() => {
      this.timers.delete(timerId);
      callback();
    }, delay);

    this.timers.add(timerId);
    return timerId;
  }

  static setInterval(callback: () => void, interval: number): number {
    const intervalId = setInterval(callback, interval);
    this.intervals.add(intervalId);
    return intervalId;
  }

  static clearTimeout(timerId: number): void {
    clearTimeout(timerId);
    this.timers.delete(timerId);
  }

  static clearInterval(intervalId: number): void {
    clearInterval(intervalId);
    this.intervals.delete(intervalId);
  }

  static addSubscription(unsubscribe: () => void): void {
    this.subscriptions.add(unsubscribe);
  }

  static cleanupAll(): void {
    // Clear all timers
    this.timers.forEach(timerId => clearTimeout(timerId));
    this.timers.clear();

    // Clear all intervals
    this.intervals.forEach(intervalId => clearInterval(intervalId));
    this.intervals.clear();

    // Unsubscribe from all subscriptions
    this.subscriptions.forEach(unsubscribe => unsubscribe());
    this.subscriptions.clear();
  }
}

// Component with proper cleanup
@Component
struct ManagedComponent {
  @State private data: string[] = [];
  private updateTimer: number = -1;

  aboutToAppear(): void {
    this.startDataUpdates();
  }

  aboutToDisappear(): void {
    this.cleanup();
  }

  private startDataUpdates(): void {
    this.updateTimer = ResourceManager.setInterval(() => {
      this.fetchLatestData();
    }, 5000);
  }

  private cleanup(): void {
    if (this.updateTimer !== -1) {
      ResourceManager.clearInterval(this.updateTimer);
      this.updateTimer = -1;
    }
  }

  private async fetchLatestData(): Promise<void> {
    // Fetch data implementation
  }

  build() {
    // Component UI
  }
}
```

## CPU Optimization

### Asynchronous Operations

```typescript
export class AsyncProcessor {
  private static readonly BATCH_SIZE = 1000;

  // Process large datasets in chunks to avoid blocking UI
  static async processLargeDataset<T, R>(
    data: T[],
    processor: (item: T) => R,
    onProgress?: (progress: number) => void
  ): Promise<R[]> {
    const results: R[] = [];
    const total = data.length;

    for (let i = 0; i < total; i += this.BATCH_SIZE) {
      const batch = data.slice(i, i + this.BATCH_SIZE);

      // Process batch
      const batchResults = batch.map(processor);
      results.push(...batchResults);

      // Update progress
      if (onProgress) {
        const progress = Math.min((i + this.BATCH_SIZE) / total, 1);
        onProgress(progress);
      }

      // Yield control to UI thread
      await new Promise(resolve => setTimeout(resolve, 0));
    }

    return results;
  }

  // Debounce function calls to reduce CPU usage
  static debounce<T extends any[]>(
    func: (...args: T) => void,
    delay: number
  ): (...args: T) => void {
    let timeoutId: number;

    return (...args: T) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func(...args), delay);
    };
  }

  // Throttle function calls to limit execution frequency
  static throttle<T extends any[]>(
    func: (...args: T) => void,
    limit: number
  ): (...args: T) => void {
    let inThrottle: boolean = false;

    return (...args: T) => {
      if (!inThrottle) {
        func(...args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  }
}

// Usage example
@Component
struct SearchComponent {
  @State private searchResults: SearchResult[] = [];
  @State private isSearching: boolean = false;
  private searchInput: string = '';

  // Debounced search to avoid excessive API calls
  private debouncedSearch = AsyncProcessor.debounce(async (query: string) => {
    await this.performSearch(query);
  }, 300);

  private async performSearch(query: string): Promise<void> {
    if (!query.trim()) {
      this.searchResults = [];
      return;
    }

    this.isSearching = true;

    try {
      const results = await SearchService.search(query);
      this.searchResults = results;
    } catch (error) {
      console.error('Search failed:', error);
    } finally {
      this.isSearching = false;
    }
  }

  build() {
    Column() {
      TextInput({ placeholder: 'Search...' })
        .onChange((value: string) => {
          this.searchInput = value;
          this.debouncedSearch(value);
        })

      if (this.isSearching) {
        LoadingProgress()
          .width(40)
          .height(40)
      } else {
        List() {
          ForEach(this.searchResults, (result: SearchResult) => {
            ListItem() {
              SearchResultItem({ result: result })
            }
          })
        }
      }
    }
  }
}
```

## Image and Asset Optimization

### Image Loading and Caching

```typescript
export class OptimizedImageLoader {
  private static imageCache = new ImageCache();
  private static loadingPromises: Map<string, Promise<PixelMap>> = new Map();

  static async loadImage(src: string): Promise<PixelMap | null> {
    // Check cache first
    const cached = this.imageCache.get(src);
    if (cached) {
      return cached;
    }

    // Check if already loading
    if (this.loadingPromises.has(src)) {
      return await this.loadingPromises.get(src)!;
    }

    // Start loading
    const loadingPromise = this.doLoadImage(src);
    this.loadingPromises.set(src, loadingPromise);

    try {
      const image = await loadingPromise;
      this.imageCache.set(src, image);
      return image;
    } catch (error) {
      console.error(`Failed to load image ${src}:`, error);
      return null;
    } finally {
      this.loadingPromises.delete(src);
    }
  }

  private static async doLoadImage(src: string): Promise<PixelMap> {
    // Image loading implementation
    return new Promise((resolve, reject) => {
      // Actual image loading logic would go here
      setTimeout(() => {
        // Simulate successful load
        resolve({} as PixelMap);
      }, 100);
    });
  }

  static preloadImages(srcs: string[]): void {
    srcs.forEach(src => {
      this.loadImage(src).catch(error => {
        console.warn(`Failed to preload image ${src}:`, error);
      });
    });
  }
}

// Optimized image component
@Component
struct OptimizedImage {
  @Prop src: string = '';
  @Prop placeholder?: Resource;
  @State private imageData: PixelMap | null = null;
  @State private isLoading: boolean = false;
  @State private hasError: boolean = false;

  async aboutToAppear(): Promise<void> {
    await this.loadImage();
  }

  private async loadImage(): Promise<void> {
    if (!this.src) return;

    this.isLoading = true;
    this.hasError = false;

    try {
      const image = await OptimizedImageLoader.loadImage(this.src);
      this.imageData = image;
    } catch (error) {
      this.hasError = true;
      console.error(`Failed to load image: ${this.src}`, error);
    } finally {
      this.isLoading = false;
    }
  }

  build() {
    Stack() {
      if (this.imageData) {
        Image(this.imageData)
          .width('100%')
          .height('100%')
          .objectFit(ImageFit.Cover)
      } else if (this.isLoading) {
        LoadingProgress()
          .width(30)
          .height(30)
      } else if (this.hasError && this.placeholder) {
        Image(this.placeholder)
          .width('100%')
          .height('100%')
          .objectFit(ImageFit.Cover)
      }
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

## Performance Monitoring

### Performance Metrics Collection

```typescript
export class PerformanceMonitor {
  private static metrics: Map<string, PerformanceMetric> = new Map();

  static startTimer(name: string): void {
    this.metrics.set(name, {
      name,
      startTime: Date.now(),
      endTime: 0,
      duration: 0
    });
  }

  static endTimer(name: string): number {
    const metric = this.metrics.get(name);
    if (!metric) {
      console.warn(`Timer ${name} not found`);
      return 0;
    }

    metric.endTime = Date.now();
    metric.duration = metric.endTime - metric.startTime;

    console.log(`Performance: ${name} took ${metric.duration}ms`);
    return metric.duration;
  }

  static measureAsync<T>(name: string, asyncFn: () => Promise<T>): Promise<T> {
    this.startTimer(name);

    return asyncFn().finally(() => {
      this.endTimer(name);
    });
  }

  static getMetrics(): PerformanceMetric[] {
    return Array.from(this.metrics.values());
  }

  static clearMetrics(): void {
    this.metrics.clear();
  }
}

interface PerformanceMetric {
  name: string;
  startTime: number;
  endTime: number;
  duration: number;
}

// Usage example
@Component
struct PerformanceAwareComponent {
  @State private data: any[] = [];

  async aboutToAppear(): Promise<void> {
    await PerformanceMonitor.measureAsync('component-initialization', async () => {
      await this.loadInitialData();
    });
  }

  private async loadInitialData(): Promise<void> {
    PerformanceMonitor.startTimer('data-loading');

    try {
      const data = await DataService.loadData();
      this.data = data;
    } finally {
      PerformanceMonitor.endTimer('data-loading');
    }
  }

  build() {
    // Component UI
  }
}
```

## Best Practices

### 1. UI Performance

- Use LazyForEach for large lists
- Implement proper caching strategies
- Minimize component re-renders
- Use @Builder for reusable UI patterns

### 2. Memory Management

- Implement object pooling for frequently created objects
- Use weak references where appropriate
- Clean up resources in component lifecycle methods
- Monitor memory usage in development

### 3. CPU Optimization

- Use asynchronous processing for heavy computations
- Implement debouncing and throttling for user interactions
- Break up long-running tasks into smaller chunks
- Use Web Workers for CPU-intensive operations

### 4. Network and I/O

- Implement proper caching strategies
- Use compression for data transfer
- Batch multiple requests when possible
- Implement request deduplication

## Conclusion

Performance optimization in HarmonyOS applications requires attention to multiple aspects including UI rendering, memory management, CPU usage, and I/O operations. By implementing these techniques and following best practices, you can create smooth, responsive applications that provide excellent user experiences.

Key takeaways:

1. **Measure First**: Use profiling tools to identify performance bottlenecks
2. **Optimize Rendering**: Implement efficient UI patterns and state management
3. **Manage Resources**: Properly handle memory, timers, and other resources
4. **Asynchronous Processing**: Use async patterns to keep UI responsive
5. **Monitor Continuously**: Implement performance monitoring in production
