# Common List Operations

## Understanding List Performance Fundamentals

### 🎯 List Component Selection Guide

| Component              | Use Case                      | Performance | Advantages                     | Limitations                        |
| ---------------------- | ----------------------------- | ----------- | ------------------------------ | ---------------------------------- |
| `List` + `ForEach`     | Small datasets (<100 items)   | Good        | Simple to implement            | Poor performance with large data   |
| `List` + `LazyForEach` | Large datasets (>100 items)   | Excellent   | Lazy loading, memory efficient | Complex data source implementation |
| `Grid` + `LazyForEach` | Grid layouts, image galleries | Excellent   | Grid-specific optimizations    | Limited to grid layouts            |
| `WaterFlow`            | Pinterest-style layouts       | Good        | Dynamic height support         | More complex implementation        |

### 🚀 Performance Impact Comparison

```typescript
// ❌ Poor: ForEach with large dataset (1000+ items)
List() {
  ForEach(this.largeDataArray, (item) => {
    ListItem() {
      ItemComponent({ data: item });
    }
  });
}
// Result: All 1000+ components created at once, high memory usage

// ✅ Good: LazyForEach with large dataset
List() {
  LazyForEach(this.dataSource, (item) => {
    ListItem() {
      ItemComponent({ data: item });
    }
  });
}
// Result: Only visible items created, efficient memory usage
```

---

## Implementing Efficient LazyForEach

### 🎯 Basic LazyForEach Implementation

#### Step 1: Create Data Source Class

```typescript
class MyDataSource implements IDataSource {
  private dataArray: Array<any> = [];
  private listeners: DataChangeListener[] = [];

  constructor(dataArray: Array<any>) {
    this.dataArray = dataArray;
  }

  // Required: Return total count
  totalCount(): number {
    return this.dataArray.length;
  }

  // Required: Return data at specific index
  getData(index: number): any {
    return this.dataArray[index];
  }

  // Required: Register data change listener
  registerDataChangeListener(listener: DataChangeListener): void {
    if (this.listeners.indexOf(listener) < 0) {
      this.listeners.push(listener);
    }
  }

  // Required: Unregister data change listener
  unregisterDataChangeListener(listener: DataChangeListener): void {
    const pos = this.listeners.indexOf(listener);
    if (pos >= 0) {
      this.listeners.splice(pos, 1);
    }
  }

  // Notify listeners when data changes
  public notifyDataReload(): void {
    this.listeners.forEach((listener) => {
      listener.onDataReloaded();
    });
  }

  public notifyDataAdd(index: number): void {
    this.listeners.forEach((listener) => {
      listener.onDataAdd(index);
    });
  }

  public notifyDataChange(index: number): void {
    this.listeners.forEach((listener) => {
      listener.onDataChange(index);
    });
  }

  public notifyDataDelete(index: number): void {
    this.listeners.forEach((listener) => {
      listener.onDataDelete(index);
    });
  }

  // Data manipulation methods
  public pushData(data: any): void {
    this.dataArray.push(data);
    this.notifyDataAdd(this.dataArray.length - 1);
  }

  public unshiftData(data: any): void {
    this.dataArray.unshift(data);
    this.notifyDataAdd(0);
  }

  public deleteData(index: number): void {
    this.dataArray.splice(index, 1);
    this.notifyDataDelete(index);
  }

  public updateData(index: number, data: any): void {
    this.dataArray[index] = data;
    this.notifyDataChange(index);
  }
}
```

#### Step 2: Use LazyForEach in Component

```typescript
@Component
struct EfficientList {
  private dataSource: MyDataSource = new MyDataSource([]);

  aboutToAppear() {
    // Initialize with sample data
    const sampleData = Array.from({ length: 1000 }, (_, index) => ({
      id: index,
      title: `Item ${index}`,
      content: `Content for item ${index}`,
      timestamp: Date.now() + index
    }));

    this.dataSource = new MyDataSource(sampleData);
  }

  build() {
    Column() {
      List({ space: 8 }) {
        LazyForEach(this.dataSource, (item: any, index: number) => {
          ListItem() {
            ItemComponent({ data: item, index: index });
          }
        }, (item: any) => item.id.toString()); // 👈 Important: Unique key function
      }
      .cachedCount(3) // Cache 3 items above and below visible area
      .width("100%")
      .height("100%");
    }
    .padding(16);
  }
}
```

### 🎯 Reusable List Item Component

```typescript
@Reusable
@Component
struct ItemComponent {
  @State data: any = {};
  @State index: number = 0;

  // Handle component reuse
  aboutToReuse(params: Record<string, Object>): void {
    this.data = params.data;
    this.index = params.index as number;
  }

  build() {
    Row({ space: 12 }) {
      // Avatar or image
      Circle({ width: 40, height: 40 })
        .fill(Color.Blue)
        .overlay(
          Text(this.data.title?.[0] || "?")
            .fontColor(Color.White)
            .fontSize(16)
        );

      // Content area
      Column({ space: 4 }) {
        Text(this.data.title || "")
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
          .maxLines(1)
          .textOverflow({ overflow: TextOverflow.Ellipsis });

        Text(this.data.content || "")
          .fontSize(14)
          .fontColor(Color.Gray)
          .maxLines(2)
          .textOverflow({ overflow: TextOverflow.Ellipsis });

        Text(`Index: ${this.index}`)
          .fontSize(12)
          .fontColor(Color.Gray);
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1);

      // Action button
      Button("Action")
        .type(ButtonType.Capsule)
        .fontSize(12)
        .padding({ left: 8, right: 8, top: 4, bottom: 4 });
    }
    .width("100%")
    .padding(16)
    .backgroundColor(Color.White)
    .borderRadius(8)
    .shadow({ radius: 2, color: Color.Gray, offsetY: 1 });
  }
}
```

---

## Advanced List Operations

### 🎯 Pull-to-Refresh Implementation

```typescript
@Component
struct PullToRefreshList {
  @State isRefreshing: boolean = false;
  @State refreshText: string = "Pull to refresh";
  private dataSource: MyDataSource = new MyDataSource([]);
  private scroller: Scroller = new Scroller();

  build() {
    Column() {
      // Refresh indicator
      if (this.isRefreshing) {
        Row({ space: 8 }) {
          LoadingProgress()
            .width(20)
            .height(20);
          Text("Refreshing...")
            .fontSize(14)
            .fontColor(Color.Gray);
        }
        .width("100%")
        .justifyContent(FlexAlign.Center)
        .padding(16);
      }

      // Main list
      List({ space: 8, scroller: this.scroller }) {
        LazyForEach(this.dataSource, (item: any, index: number) => {
          ListItem() {
            ItemComponent({ data: item, index: index });
          }
        }, (item: any) => item.id.toString());
      }
      .width("100%")
      .layoutWeight(1)
      .onReachStart(() => {
        if (!this.isRefreshing) {
          this.handleRefresh();
        }
      })
      .onScrollFrameBegin((offset: number) => {
        // Handle pull-to-refresh gesture
        if (offset > 0 && this.scroller.currentOffset().yOffset <= 0) {
          this.refreshText = offset > 100 ? "Release to refresh" : "Pull to refresh";
        }
        return { offsetRemain: offset };
      });
    }
  }

  private async handleRefresh(): Promise<void> {
    this.isRefreshing = true;

    try {
      // Simulate network request
      await new Promise(resolve => setTimeout(resolve, 2000));

      // Add new data to the beginning
      const newItems = Array.from({ length: 5 }, (_, index) => ({
        id: Date.now() + index,
        title: `New Item ${index}`,
        content: `Fresh content ${index}`,
        timestamp: Date.now()
      }));

      newItems.reverse().forEach(item => {
        this.dataSource.unshiftData(item);
      });

    } catch (error) {
      console.error("Refresh failed:", error);
    } finally {
      this.isRefreshing = false;
      this.refreshText = "Pull to refresh";
    }
  }
}
```

### 🎯 Load More Implementation

```typescript
@Component
struct LoadMoreList {
  @State isLoadingMore: boolean = false;
  @State hasMoreData: boolean = true;
  private dataSource: MyDataSource = new MyDataSource([]);
  private currentPage: number = 1;
  private pageSize: number = 20;

  aboutToAppear() {
    this.loadInitialData();
  }

  build() {
    Column() {
      List({ space: 8 }) {
        LazyForEach(this.dataSource, (item: any, index: number) => {
          ListItem() {
            ItemComponent({ data: item, index: index });
          }
        }, (item: any) => item.id.toString());

        // Load more footer
        if (this.hasMoreData) {
          ListItem() {
            this.buildLoadMoreFooter();
          }
        } else {
          ListItem() {
            Text("No more data")
              .width("100%")
              .textAlign(TextAlign.Center)
              .padding(16)
              .fontColor(Color.Gray);
          }
        }
      }
      .width("100%")
      .height("100%")
      .onReachEnd(() => {
        if (this.hasMoreData && !this.isLoadingMore) {
          this.loadMoreData();
        }
      });
    }
  }

  @Builder
  buildLoadMoreFooter() {
    Row({ space: 8 }) {
      if (this.isLoadingMore) {
        LoadingProgress()
          .width(20)
          .height(20);
        Text("Loading more...")
          .fontSize(14)
          .fontColor(Color.Gray);
      } else {
        Text("Pull up to load more")
          .fontSize(14)
          .fontColor(Color.Gray);
      }
    }
    .width("100%")
    .justifyContent(FlexAlign.Center)
    .padding(16);
  }

  private async loadInitialData(): Promise<void> {
    const data = await this.fetchData(1);
    this.dataSource = new MyDataSource(data);
  }

  private async loadMoreData(): Promise<void> {
    if (this.isLoadingMore) return;

    this.isLoadingMore = true;

    try {
      const nextPage = this.currentPage + 1;
      const newData = await this.fetchData(nextPage);

      if (newData.length > 0) {
        newData.forEach(item => {
          this.dataSource.pushData(item);
        });
        this.currentPage = nextPage;
      } else {
        this.hasMoreData = false;
      }

    } catch (error) {
      console.error("Load more failed:", error);
    } finally {
      this.isLoadingMore = false;
    }
  }

  private async fetchData(page: number): Promise<any[]> {
    // Simulate API call
    return new Promise((resolve) => {
      setTimeout(() => {
        const data = Array.from({ length: this.pageSize }, (_, index) => ({
          id: (page - 1) * this.pageSize + index,
          title: `Page ${page} Item ${index}`,
          content: `Content for page ${page}, item ${index}`,
          timestamp: Date.now() + index
        }));
        resolve(page > 10 ? [] : data); // Simulate no more data after page 10
      }, 1000);
    });
  }
}
```

---

## Performance Optimization Tips

### 🎯 Optimal Configuration Settings

```typescript
@Component
struct OptimizedList {
  build() {
    List({ space: 8 }) {
      LazyForEach(this.dataSource, (item: any) => {
        ListItem() {
          ItemComponent({ data: item });
        }
      }, (item: any) => item.id.toString());
    }
    .cachedCount(5)        // Cache 5 items above/below visible area
    .lanes(1)              // Single lane for vertical list
    .friction(0.9)         // Smooth scrolling friction
    .nestedScroll({         // Handle nested scrolling
      scrollForward: NestedScrollMode.PARENT_FIRST,
      scrollBackward: NestedScrollMode.SELF_FIRST
    })
    .flingSpeedLimit(5000); // Limit fling speed for better control
  }
}
```

### 🎯 Memory Management Best Practices

```typescript
// ✅ Good: Implement proper cleanup
@Component
struct MemoryEfficientList {
  private dataSource: MyDataSource = new MyDataSource([]);
  private imageCache: Map<string, PixelMap> = new Map();

  aboutToDisappear() {
    // Clear image cache to free memory
    this.imageCache.clear();

    // Clear data source
    this.dataSource = new MyDataSource([]);
  }

  build() {
    List() {
      LazyForEach(this.dataSource, (item: any) => {
        ListItem() {
          EfficientItemComponent({
            data: item,
            imageCache: this.imageCache
          });
        }
      }, (item: any) => item.id);
    }
    .cachedCount(3); // Conservative cache count
  }
}
```

## Best Practices Summary

### 🎯 Performance Checklist

- [ ] Use `LazyForEach` for lists with >50 items
- [ ] Implement proper `@Reusable` components
- [ ] Provide unique key functions
- [ ] Set appropriate `cachedCount` (3-5 recommended)
- [ ] Implement data source change notifications
- [ ] Use debouncing for search functionality
- [ ] Clean up resources in `aboutToDisappear`

### 🎯 User Experience Enhancements

- [ ] Add pull-to-refresh functionality
- [ ] Implement infinite scrolling/load more
- [ ] Provide search and filter capabilities
- [ ] Show loading states and empty states
- [ ] Handle error scenarios gracefully
- [ ] Add smooth animations and transitions

### 🎯 Code Quality Guidelines

- [ ] Extract reusable components
- [ ] Use TypeScript for better type safety
- [ ] Implement proper error handling
- [ ] Add performance monitoring
- [ ] Write unit tests for data source operations
- [ ] Document component APIs

With these patterns and optimizations, your HarmonyOS lists will provide excellent performance and user experience!
