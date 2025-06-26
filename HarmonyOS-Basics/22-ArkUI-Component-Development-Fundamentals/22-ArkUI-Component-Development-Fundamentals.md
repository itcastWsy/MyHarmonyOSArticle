# 22-ArkUI Component Development Fundamentals

## Introduction

ArkUI is HarmonyOS's declarative UI framework that enables developers to build rich, interactive user interfaces using ArkTS. This article covers the fundamental concepts of ArkUI component development, including built-in components, custom components, layout systems, and component lifecycle management.

## Understanding ArkUI Architecture

### Declarative UI Paradigm

ArkUI follows a declarative programming model where you describe what the UI should look like rather than how to build it step by step:

```typescript
@Entry
@Component
struct MainPage {
  @State message: string = 'Hello World';

  build() {
    Column() {
      Text(this.message)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .margin({ top: 20 })

      Button('Click Me')
        .onClick(() => {
          this.message = 'Button Clicked!';
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### Component Hierarchy and Composition

ArkUI components form a tree structure where parent components contain child components:

```typescript
@Component
struct UserCard {
  @Prop user: UserInfo;

  build() {
    Card() {
      Column() {
        // Header section
        Row() {
          Image(this.user.avatar)
            .width(50)
            .height(50)
            .borderRadius(25)

          Column() {
            Text(this.user.name)
              .fontSize(16)
              .fontWeight(FontWeight.Medium)
            Text(this.user.email)
              .fontSize(12)
              .fontColor(Color.Gray)
          }
          .alignItems(HorizontalAlign.Start)
          .margin({ left: 12 })
        }
        .width('100%')
        .alignItems(VerticalAlign.Center)

        // Content section
        Text(this.user.bio)
          .fontSize(14)
          .margin({ top: 16 })
          .maxLines(3)
          .textOverflow({ overflow: TextOverflow.Ellipsis })
      }
      .padding(16)
    }
    .width('100%')
    .margin({ bottom: 12 })
  }
}
```

## Built-in Components Overview

### Basic Components

#### Text Component

```typescript
@Component
struct TextExamples {
  build() {
    Column({ space: 16 }) {
      // Basic text
      Text('Simple Text')

      // Styled text
      Text('Styled Text')
        .fontSize(20)
        .fontColor(Color.Blue)
        .fontWeight(FontWeight.Bold)
        .textAlign(TextAlign.Center)

      // Rich text with spans
      Text() {
        Span('Hello ')
          .fontColor(Color.Red)
        Span('World')
          .fontColor(Color.Green)
          .fontSize(24)
      }

      // Text with overflow handling
      Text('This is a very long text that might overflow the container width')
        .width(200)
        .maxLines(2)
        .textOverflow({ overflow: TextOverflow.Ellipsis })
    }
    .padding(20)
  }
}
```

#### Button Component

```typescript
@Component
struct ButtonExamples {
  @State counter: number = 0;

  build() {
    Column({ space: 16 }) {
      // Basic button
      Button('Basic Button')
        .onClick(() => {
          this.counter++;
        })

      // Custom styled button
      Button('Custom Button')
        .backgroundColor(Color.Orange)
        .borderRadius(8)
        .width(200)
        .height(40)

      // Button with icon
      Button({ type: ButtonType.Circle }) {
        Image($r('app.media.icon_add'))
          .width(24)
          .height(24)
      }
      .width(50)
      .height(50)

      // Button states
      Button(`Clicked: ${this.counter}`)
        .enabled(this.counter < 10)
        .backgroundColor(this.counter >= 10 ? Color.Gray : Color.Blue)
    }
    .padding(20)
  }
}
```

#### Image Component

```typescript
@Component
struct ImageExamples {
  build() {
    Column({ space: 16 }) {
      // Local resource image
      Image($r('app.media.logo'))
        .width(100)
        .height(100)

      // Network image with placeholder
      Image('https://example.com/image.jpg')
        .width(150)
        .height(150)
        .alt($r('app.media.placeholder'))
        .objectFit(ImageFit.Cover)

      // Circular image
      Image($r('app.media.avatar'))
        .width(80)
        .height(80)
        .borderRadius(40)
        .border({ width: 2, color: Color.White })

      // Image with loading states
      Image('https://example.com/large-image.jpg')
        .width(200)
        .height(150)
        .onComplete(() => {
          console.log('Image loaded successfully');
        })
        .onError(() => {
          console.error('Failed to load image');
        })
    }
    .padding(20)
  }
}
```

### Container Components

#### Column and Row Layout

```typescript
@Component
struct LayoutExamples {
  build() {
    Column() {
      // Horizontal layout
      Row({ space: 16 }) {
        Text('Item 1')
          .backgroundColor(Color.Red)
          .padding(8)
        Text('Item 2')
          .backgroundColor(Color.Green)
          .padding(8)
        Text('Item 3')
          .backgroundColor(Color.Blue)
          .padding(8)
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceEvenly)

      // Vertical layout with alignment
      Column({ space: 12 }) {
        Text('Top')
          .backgroundColor(Color.Yellow)
          .padding(8)
        Text('Center')
          .backgroundColor(Color.Orange)
          .padding(8)
        Text('Bottom')
          .backgroundColor(Color.Purple)
          .padding(8)
      }
      .width('100%')
      .height(200)
      .justifyContent(FlexAlign.SpaceBetween)
      .alignItems(HorizontalAlign.Center)
    }
    .padding(20)
  }
}
```

#### Stack Layout

```typescript
@Component
struct StackExample {
  build() {
    Stack() {
      // Background
      Rectangle()
        .width(200)
        .height(200)
        .backgroundColor(Color.Gray)
        .borderRadius(8)

      // Overlaid content
      Column() {
        Text('Overlay Content')
          .fontColor(Color.White)
          .fontSize(18)
        Button('Action')
          .margin({ top: 16 })
      }

      // Badge in corner
      Text('99+')
        .fontSize(12)
        .fontColor(Color.White)
        .backgroundColor(Color.Red)
        .padding({ left: 6, right: 6, top: 2, bottom: 2 })
        .borderRadius(10)
        .position({ x: 160, y: 10 })
    }
    .alignContent(Alignment.Center)
  }
}
```

## Custom Component Development

### Creating Reusable Components

```typescript
// Define component interface
interface LoadingButtonProps {
  text: string;
  loading?: boolean;
  disabled?: boolean;
  onTap?: () => void;
}

@Component
export struct LoadingButton {
  @Prop text: string = '';
  @Prop loading: boolean = false;
  @Prop disabled: boolean = false;
  private onTap?: () => void;

  build() {
    Button() {
      Row({ space: 8 }) {
        if (this.loading) {
          LoadingProgress()
            .width(16)
            .height(16)
            .color(Color.White)
        }
        Text(this.loading ? 'Loading...' : this.text)
          .fontColor(Color.White)
      }
    }
    .enabled(!this.disabled && !this.loading)
    .backgroundColor(this.disabled ? Color.Gray : Color.Blue)
    .onClick(() => {
      if (this.onTap && !this.loading && !this.disabled) {
        this.onTap();
      }
    })
  }

  setClickHandler(handler: () => void): LoadingButton {
    this.onTap = handler;
    return this;
  }
}
```

### Component with State Management

```typescript
@Component
export struct Counter {
  @State private count: number = 0;
  @Prop initialValue: number = 0;
  @Prop step: number = 1;
  @Prop min?: number;
  @Prop max?: number;
  private onValueChange?: (value: number) => void;

  aboutToAppear(): void {
    this.count = this.initialValue;
  }

  private increment(): void {
    const newValue = this.count + this.step;
    if (this.max === undefined || newValue <= this.max) {
      this.count = newValue;
      this.notifyValueChange();
    }
  }

  private decrement(): void {
    const newValue = this.count - this.step;
    if (this.min === undefined || newValue >= this.min) {
      this.count = newValue;
      this.notifyValueChange();
    }
  }

  private notifyValueChange(): void {
    if (this.onValueChange) {
      this.onValueChange(this.count);
    }
  }

  build() {
    Row({ space: 16 }) {
      Button('-')
        .width(40)
        .height(40)
        .enabled(this.min === undefined || this.count > this.min)
        .onClick(() => this.decrement())

      Text(this.count.toString())
        .fontSize(18)
        .fontWeight(FontWeight.Medium)
        .width(60)
        .textAlign(TextAlign.Center)

      Button('+')
        .width(40)
        .height(40)
        .enabled(this.max === undefined || this.count < this.max)
        .onClick(() => this.increment())
    }
    .justifyContent(FlexAlign.Center)
  }

  setValueChangeHandler(handler: (value: number) => void): Counter {
    this.onValueChange = handler;
    return this;
  }
}
```

## Component Lifecycle

### Lifecycle Methods

```typescript
@Component
struct LifecycleDemo {
  @State private data: string[] = [];
  private timer: number = -1;

  // Called before component is created
  aboutToAppear(): void {
    console.log('Component about to appear');
    this.loadData();
    this.startTimer();
  }

  // Called before component is destroyed
  aboutToDisappear(): void {
    console.log('Component about to disappear');
    this.cleanup();
  }

  // Called when component size changes
  onAreaChange(oldArea: Area, newArea: Area): void {
    console.log(`Area changed from ${oldArea.width}x${oldArea.height} to ${newArea.width}x${newArea.height}`);
  }

  private async loadData(): Promise<void> {
    try {
      // Simulate data loading
      const response = await fetch('/api/data');
      this.data = await response.json();
    } catch (error) {
      console.error('Failed to load data:', error);
    }
  }

  private startTimer(): void {
    this.timer = setInterval(() => {
      console.log('Timer tick');
    }, 1000);
  }

  private cleanup(): void {
    if (this.timer !== -1) {
      clearInterval(this.timer);
      this.timer = -1;
    }
  }

  build() {
    Column() {
      Text('Lifecycle Demo')
        .fontSize(20)
        .margin({ bottom: 16 })

      List() {
        ForEach(this.data, (item: string, index: number) => {
          ListItem() {
            Text(item)
              .padding(12)
          }
        })
      }
    }
    .padding(20)
    .onAreaChange((oldArea: Area, newArea: Area) => {
      this.onAreaChange(oldArea, newArea);
    })
  }
}
```

## Layout and Styling

### Flex Layout System

```typescript
@Component
struct FlexLayoutExamples {
  build() {
    Column({ space: 20 }) {
      // Basic flex row
      Flex({ direction: FlexDirection.Row, justifyContent: FlexAlign.SpaceBetween }) {
        Text('Start')
          .backgroundColor(Color.Red)
          .padding(8)
        Text('Center')
          .backgroundColor(Color.Green)
          .padding(8)
        Text('End')
          .backgroundColor(Color.Blue)
          .padding(8)
      }
      .width('100%')
      .height(60)

      // Flex with wrap
      Flex({ wrap: FlexWrap.Wrap, justifyContent: FlexAlign.SpaceEvenly }) {
        ForEach([1, 2, 3, 4, 5, 6], (item: number) => {
          Text(`Item ${item}`)
            .width(80)
            .height(40)
            .backgroundColor(Color.Orange)
            .textAlign(TextAlign.Center)
            .margin(4)
        })
      }
      .width('100%')

      // Flex with grow
      Flex({ direction: FlexDirection.Row }) {
        Text('Fixed')
          .width(80)
          .backgroundColor(Color.Gray)
          .padding(8)

        Text('Flexible')
          .flexGrow(1)
          .backgroundColor(Color.Yellow)
          .padding(8)

        Text('Fixed')
          .width(80)
          .backgroundColor(Color.Gray)
          .padding(8)
      }
      .width('100%')
      .height(50)
    }
    .padding(20)
  }
}
```

### Responsive Design

```typescript
@Component
struct ResponsiveLayout {
  @State private screenWidth: number = 0;

  build() {
    Column() {
      Text(`Screen Width: ${this.screenWidth}px`)
        .fontSize(16)
        .margin({ bottom: 20 })

      // Adaptive grid
      GridRow({ columns: this.getColumnCount() }) {
        ForEach([1, 2, 3, 4, 5, 6], (item: number) => {
          GridCol({ span: 1 }) {
            Card() {
              Text(`Card ${item}`)
                .textAlign(TextAlign.Center)
                .height(100)
            }
            .width('100%')
            .height(100)
          }
          .margin(8)
        })
      }
      .width('100%')
    }
    .padding(16)
    .onAreaChange((oldArea: Area, newArea: Area) => {
      this.screenWidth = Number(newArea.width);
    })
  }

  private getColumnCount(): number {
    if (this.screenWidth < 600) {
      return 2; // Mobile: 2 columns
    } else if (this.screenWidth < 900) {
      return 3; // Tablet: 3 columns
    } else {
      return 4; // Desktop: 4 columns
    }
  }
}
```

## Component Communication

### Parent-Child Communication

```typescript
// Child component
@Component
struct ChildComponent {
  @Prop message: string = '';
  @Link counter: number;
  private onButtonClick?: (data: string) => void;

  build() {
    Column({ space: 12 }) {
      Text(`Message: ${this.message}`)
      Text(`Counter: ${this.counter}`)

      Button('Increment')
        .onClick(() => {
          this.counter++;
        })

      Button('Send Data to Parent')
        .onClick(() => {
          if (this.onButtonClick) {
            this.onButtonClick(`Data from child at ${new Date().toISOString()}`);
          }
        })
    }
    .padding(16)
    .border({ width: 1, color: Color.Gray })
    .borderRadius(8)
  }

  setButtonClickHandler(handler: (data: string) => void): ChildComponent {
    this.onButtonClick = handler;
    return this;
  }
}

// Parent component
@Component
struct ParentComponent {
  @State private parentMessage: string = 'Hello from parent';
  @State private sharedCounter: number = 0;
  @State private childData: string = '';

  build() {
    Column({ space: 20 }) {
      Text('Parent Component')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)

      Text(`Child sent: ${this.childData}`)
        .fontSize(14)
        .fontColor(Color.Gray)

      ChildComponent({
        message: this.parentMessage,
        counter: $sharedCounter
      })
      .setButtonClickHandler((data: string) => {
        this.childData = data;
      })

      Button('Update Message')
        .onClick(() => {
          this.parentMessage = `Updated at ${new Date().toLocaleTimeString()}`;
        })
    }
    .padding(20)
  }
}
```

## Performance Optimization

### Efficient Rendering

```typescript
@Component
struct OptimizedList {
  @State private items: ListItem[] = [];
  @State private visibleItems: ListItem[] = [];
  private readonly ITEMS_PER_PAGE = 20;
  private currentPage = 0;

  aboutToAppear(): void {
    this.loadInitialData();
  }

  private async loadInitialData(): Promise<void> {
    this.items = await this.fetchItems(0, this.ITEMS_PER_PAGE);
    this.visibleItems = [...this.items];
  }

  private async loadMoreItems(): Promise<void> {
    this.currentPage++;
    const newItems = await this.fetchItems(
      this.currentPage * this.ITEMS_PER_PAGE,
      this.ITEMS_PER_PAGE
    );

    this.items = [...this.items, ...newItems];
    this.visibleItems = [...this.visibleItems, ...newItems];
  }

  build() {
    List() {
      LazyForEach(new ItemDataSource(this.visibleItems), (item: ListItem) => {
        ListItem() {
          OptimizedListItem({ item: item })
        }
      }, (item: ListItem) => item.id.toString())
    }
    .onReachEnd(() => {
      this.loadMoreItems();
    })
    .cachedCount(10) // Cache 10 items for better performance
  }

  private async fetchItems(offset: number, limit: number): Promise<ListItem[]> {
    // Simulate API call
    return new Promise((resolve) => {
      setTimeout(() => {
        const items: ListItem[] = [];
        for (let i = 0; i < limit; i++) {
          items.push({
            id: offset + i,
            title: `Item ${offset + i}`,
            subtitle: `Subtitle for item ${offset + i}`
          });
        }
        resolve(items);
      }, 500);
    });
  }
}

// Optimized list item component
@Component
struct OptimizedListItem {
  @ObjectLink item: ListItem;

  build() {
    Row() {
      Column() {
        Text(this.item.title)
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
        Text(this.item.subtitle)
          .fontSize(14)
          .fontColor(Color.Gray)
      }
      .alignItems(HorizontalAlign.Start)
      .layoutWeight(1)

      Image($r('app.media.arrow_right'))
        .width(16)
        .height(16)
    }
    .width('100%')
    .padding(16)
    .justifyContent(FlexAlign.SpaceBetween)
  }
}

// Data source for lazy loading
class ItemDataSource implements IDataSource {
  private items: ListItem[];

  constructor(items: ListItem[]) {
    this.items = items;
  }

  totalCount(): number {
    return this.items.length;
  }

  getData(index: number): ListItem {
    return this.items[index];
  }

  registerDataChangeListener(listener: DataChangeListener): void {
    // Implementation for data change notifications
  }

  unregisterDataChangeListener(listener: DataChangeListener): void {
    // Implementation for data change notifications
  }
}
```

## Best Practices

### 1. Component Design Principles

- **Single Responsibility**: Each component should have one clear purpose
- **Reusability**: Design components to be reusable across different contexts
- **Composability**: Build complex UIs by combining simple components
- **Prop Interface**: Define clear and type-safe interfaces for component props

### 2. Performance Guidelines

- Use `LazyForEach` for large lists
- Implement proper key functions for list items
- Minimize state updates and component re-renders
- Use `@ObjectLink` for complex object state management

### 3. Code Organization

- Group related components in modules
- Use consistent naming conventions
- Implement proper error boundaries
- Document component APIs and usage examples

## Conclusion

ArkUI component development forms the foundation of HarmonyOS application UI development. By mastering built-in components, creating reusable custom components, and following best practices for layout and performance, you can build efficient and maintainable user interfaces.

Key takeaways:

1. **Declarative Approach**: Embrace the declarative paradigm for cleaner, more predictable UI code
2. **Component Composition**: Build complex interfaces by combining simple, focused components
3. **State Management**: Use appropriate state management patterns for different scenarios
4. **Lifecycle Awareness**: Properly handle component lifecycle events for resource management
5. **Performance Optimization**: Implement efficient rendering strategies for smooth user experiences
6. **Responsive Design**: Create adaptive layouts that work across different screen sizes

Understanding these fundamentals will enable you to create sophisticated and performant user interfaces for HarmonyOS applications.
