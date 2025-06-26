# HarmonyOS Component Reuse Guide

## What is Component Reuse?

### 🎯 Simple Understanding

Imagine you're playing with building blocks. When you no longer need a certain block, you don't throw it away, but put it in a box. Next time you need the same type of block, you just take it out of the box and use it.

Component reuse follows the same principle:

- **Component** = A part of the interface (like an item in a list)
- **Reuse** = Use repeatedly instead of recreating
- **Cache Pool** = The box for storing "building blocks"

### 🔍 Technical Definition

Component reuse refers to custom components being placed into a cache pool after being removed from the component tree, and later when creating the same type of component nodes, directly reusing component objects from the cache pool.

---

## Why Do We Need Component Reuse?

### 🚀 Performance Advantages

**Without component reuse:**

```
User scrolls list → Create new component → Display content → Scroll off screen → Destroy component
                      ↑                                       ↓
                  Time & memory consuming                   Waste resources
```

**With component reuse:**

```
User scrolls list → Get component from cache → Update content → Display → Scroll off screen → Put in cache
                      ↑                                             ↓
                  Fast & efficient                              Circular utilization
```

### 📊 Actual Effects

- ✅ Reduce memory garbage collection frequency
- ✅ Lower CPU computation overhead
- ✅ Improve scrolling smoothness
- ✅ Enhance user experience

### 🎮 Typical Application Scenarios

- 📱 Long list scrolling (like social feeds, product lists)
- 🔄 Frequent interface switching scenarios
- 📊 Data display applications
- 🎯 Any scenario requiring frequent component creation/destruction

---

## Basic Principles of Component Reuse

### 🔄 Reuse Flow Diagram

![Component Reuse Flow](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250613165819.17333207595350562350793044372277:50001231000000:2800:7D95D5342B0C55355D8C6559221E78CD3C7614A7A65E1845D2C6138C4CE9CCBD.png)

### 📝 Three Key Steps

1. **Marking Phase**: Add `@Reusable` tag to components
2. **Recycling Phase**: When components scroll off screen, put them in cache pool
3. **Reuse Phase**: When new components are needed, get them from cache pool and update data

### 🎯 Key Concept Explanations

| Concept        | Simple Understanding                              | Technical Meaning                            |
| -------------- | ------------------------------------------------- | -------------------------------------------- |
| @Reusable      | Add "reusable" label to components                | Decorator marking components as reusable     |
| reuseId        | Categorize different types of components          | Reuse identifier, distinguishing cache pools |
| aboutToReuse() | Preparation work when component "returns to work" | Lifecycle callback for handling data updates |
| Cache Pool     | "Warehouse" storing components to be reused       | CachedRecycleNodes collection                |

---

## Getting Started: Basic List Reuse

### 🎯 Scenario 1: Same Structure List Items

This is the simplest reuse scenario where each item in the list looks the same, only content differs.

![Same Structure List](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250613165819.17368666382558048165253880931869:50001231000000:2800:8DCACF64166D761F3F28C2FFCA95DE20AEB3585905406F1DA4ED1BA95C16E4E8.png)

#### 🛠️ Implementation Steps

**Step 1: Create Reusable Component**

```typescript
@Reusable  // 👈 Key: Mark component as reusable
@Component
struct ItemView {
  @State title: string = "";
  @State content: string = "";

  // 👈 Key: Implement reuse callback
  aboutToReuse(params: Record<string, Object>): void {
    // When component is taken from cache pool, update data
    this.title = params.title as string;
    this.content = params.content as string;
  }

  build() {
    Column() {
      Text(this.title)
        .fontSize(16)
        .fontWeight(FontWeight.Bold);
      Text(this.content)
        .fontSize(14)
        .fontColor(Color.Gray);
    }
    .padding(10);
  }
}
```

**Step 2: Use in List**

```typescript
@Component
struct ContentPage {
  @State dataList: Array<any> = [
    { title: "Title 1", content: "Content 1" },
    { title: "Title 2", content: "Content 2" },
    // ... more data
  ];

  build() {
    List() {
      LazyForEach(this.dataSource, (item: any) => {
        ListItem() {
          ItemView({ title: item.title, content: item.content });
        }
        .reuseId("item_view");  // 👈 Key: Set reuse ID
      });
    }
  }
}
```

#### 💡 Beginner Tips

1. **@Reusable** is essential, without it there's no reuse effect
2. **aboutToReuse()** is where data updates happen, don't forget to implement it
3. **reuseId** is used to distinguish different component types, same types use same ID

---

### 🎯 Scenario 2: Different Structure List Items

When lists have multiple different types of items, like some being pure text, some with images, some being videos.

![Different Structure List](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250613165819.23133547733233562016983210909722:50001231000000:2800:7F82145E8364F300518E9B00AD8550EC3AF0DFEA9DC0B4AEA70FF1E466146EB6.png)

#### 🛠️ Implementation Steps

**Step 1: Create Different Type Components**

```typescript
// Text type component
@Reusable
@Component
struct TextItemView {
  @State title: string = "";
  @State content: string = "";

  aboutToReuse(params: Record<string, Object>): void {
    this.title = params.title as string;
    this.content = params.content as string;
  }

  build() {
    Column() {
      Text(this.title).fontSize(16);
      Text(this.content).fontSize(14);
    }
  }
}

// Image type component
@Reusable
@Component
struct ImageItemView {
  @State title: string = "";
  @State imageUrl: string = "";

  aboutToReuse(params: Record<string, Object>): void {
    this.title = params.title as string;
    this.imageUrl = params.imageUrl as string;
  }

  build() {
    Column() {
      Text(this.title).fontSize(16);
      Image(this.imageUrl)
        .width(100)
        .height(100);
    }
  }
}
```

**Step 2: Use in List with Conditional Logic**

```typescript
List() {
  LazyForEach(this.dataSource, (item: any) => {
    ListItem() {
      if (item.type === "text") {
        TextItemView({ title: item.title, content: item.content });
      } else if (item.type === "image") {
        ImageItemView({ title: item.title, imageUrl: item.imageUrl });
      }
    }
    .reuseId(item.type); // 👈 Use data type as reuse ID
  });
}
```

---

## Advanced Scenarios: Cross-Page Component Reuse

### 🎯 Scenario Description

Applications often have scenarios where different tab pages display data, each page implementing a list. When pages switch, if lists contain structurally identical list items, there are component reuse optimization opportunities.

![Cross-Page Reuse](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250613165820.51566056190300889586988628742526:50001231000000:2800:4E982A2A907F6A1EA33AB2682A1C1540BCE68032F8190BE89A4F8FA9B8BCBA13.png)

### 🛠️ Implementation with NodePool

```typescript
// Custom global reuse cache pool
class NodePool {
  private static instance: NodePool;
  private nodeMap: Map<string, NodeItem[]> = new Map();

  static getInstance(): NodePool {
    if (!NodePool.instance) {
      NodePool.instance = new NodePool();
    }
    return NodePool.instance;
  }

  // Get node from cache
  getNode(type: string, data: any): NodeItem {
    const nodes = this.nodeMap.get(type) || [];
    let nodeItem = nodes.find((item) => item.getParent() === null);

    if (!nodeItem) {
      nodeItem = new NodeItem(type);
    } else {
      // Remove from cache
      const index = nodes.indexOf(nodeItem);
      nodes.splice(index, 1);
    }

    nodeItem.updateData(data);
    return nodeItem;
  }

  // Recycle node to cache
  recycleNode(nodeItem: NodeItem): void {
    const type = nodeItem.getType();
    const nodes = this.nodeMap.get(type) || [];

    // Reset node properties
    nodeItem.reset();
    nodes.push(nodeItem);
    this.nodeMap.set(type, nodes);
  }
}

// Node item class
class NodeItem extends NodeController {
  private type: string;
  private data: any;
  private builderNode: BuilderNode<[any]> | null = null;

  constructor(type: string) {
    super();
    this.type = type;
  }

  makeNode(uiContext: UIContext): FrameNode | null {
    this.builderNode = new BuilderNode(uiContext);
    this.builderNode.build(wrapBuilder(this.getBuilder()), this.data);
    return this.builderNode.getFrameNode();
  }

  updateData(data: any): void {
    this.data = data;
    if (this.builderNode) {
      this.builderNode.update(data);
    }
  }

  private getBuilder(): WrappedBuilder<[any]> {
    // Return corresponding builder based on type
    switch (this.type) {
      case "text":
        return wrapBuilder(buildTextItem);
      case "image":
        return wrapBuilder(buildImageItem);
      default:
        return wrapBuilder(buildDefaultItem);
    }
  }
}
```

### 🔄 Using onIdle() for Pre-creation

```typescript
// Frame callback class for pre-creation
class IdleCallback extends FrameCallback {
  private preCreateData: any[];
  private currentIndex: number = 0;

  constructor(data: any[]) {
    super();
    this.preCreateData = data;
  }

  onIdle(idleTimeInNano: number): void {
    const SINGLE_CREATE_TIME = 1000000; // 1ms
    let remainingTime = idleTimeInNano;

    while (
      remainingTime > SINGLE_CREATE_TIME &&
      this.currentIndex < this.preCreateData.length
    ) {
      // Pre-create component
      const data = this.preCreateData[this.currentIndex];
      NodePool.getInstance().preBuild(data.type, data);

      this.currentIndex++;
      remainingTime -= SINGLE_CREATE_TIME;
    }

    if (this.currentIndex < this.preCreateData.length) {
      // Continue with next frame
      this.getUIContext()?.postFrameCallback(this);
    }
  }
}

// Start pre-creation
function startPreCreation(uiContext: UIContext, data: any[]): void {
  const callback = new IdleCallback(data);
  uiContext.postFrameCallback(callback);
}
```

## Performance Optimization Tips

### 1. Use attributeUpdater for Partial Refresh

```typescript
@Reusable
@Component
struct OptimizedItemView {
  @State title: string = "";
  private titleUpdater: AttributeUpdater<TextAttribute> = new AttributeUpdater();

  aboutToReuse(params: Record<string, Object>): void {
    // Only update specific attributes instead of full refresh
    this.titleUpdater.applyNormalAttribute((attr: TextAttribute) => {
      attr.fontColor(params.isHighlighted ? Color.Red : Color.Black);
    });
  }

  build() {
    Text(this.title)
      .attributeUpdater(this.titleUpdater);
  }
}
```

### 2. Use @Link/@ObjectLink Instead of @Prop

```typescript
// ❌ Avoid: @Prop causes deep copy
@Reusable
@Component
struct BadItemView {
  @Prop data: ItemData; // Deep copy overhead
}

// ✅ Recommended: @ObjectLink shares same address
@Reusable
@Component
struct GoodItemView {
  @ObjectLink data: ItemData; // Shared reference, better performance
}
```

### 3. Avoid Redundant Assignment in aboutToReuse()

```typescript
// ❌ Avoid: Redundant assignment
@Reusable
@Component
struct BadReuse {
  @ObjectLink data: ItemData;

  aboutToReuse(params: Record<string, Object>): void {
    // Don't reassign @ObjectLink/@Link/@Prop variables
    // They auto-sync with parent component
    this.data = params.data as ItemData; // ❌ Unnecessary
  }
}

// ✅ Recommended: Let auto-sync handle it
@Reusable
@Component
struct GoodReuse {
  @ObjectLink data: ItemData;

  aboutToReuse(params: Record<string, Object>): void {
    // Only handle non-auto-sync properties
    // @ObjectLink will auto-update
  }
}
```

## Best Practices Summary

### 🎯 When to Use Component Reuse

- **Long list scrolling**: Lists with many items
- **Frequent switching**: Tabs or page switching scenarios
- **Complex components**: Components with heavy creation overhead
- **Performance critical**: Scenarios requiring smooth 60fps/120fps

### 🔧 Implementation Checklist

- [ ] Add `@Reusable` decorator
- [ ] Implement `aboutToReuse()` method
- [ ] Set appropriate `reuseId`
- [ ] Consider using `@ObjectLink` instead of `@Prop`
- [ ] Avoid redundant assignments in reuse callback
- [ ] Test with performance tools

### 🚀 Performance Monitoring

```typescript
// Monitor reuse effectiveness
class ReusableMonitor {
  private static createCount: number = 0;
  private static reuseCount: number = 0;

  static recordCreate(): void {
    this.createCount++;
    console.log(`Component created: ${this.createCount}`);
  }

  static recordReuse(): void {
    this.reuseCount++;
    console.log(`Component reused: ${this.reuseCount}`);
    console.log(
      `Reuse rate: ${(
        (this.reuseCount / (this.createCount + this.reuseCount)) *
        100
      ).toFixed(2)}%`
    );
  }
}
```

Through component reuse, you can significantly improve your HarmonyOS application's performance, especially in scenarios with frequent component creation and destruction. Start with simple list reuse and gradually explore more advanced techniques!
