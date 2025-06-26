# Grid Element Drag and Drop Exchange

## Understanding Grid Drag and Drop

### 🎯 What is Grid Drag and Drop?

Grid drag and drop allows users to:

- **Drag elements** from one position to another within a grid
- **Reorder items** by dragging and dropping
- **Exchange positions** between different grid items
- **Provide visual feedback** during the drag operation

### 🎮 Common Use Cases

- **Dashboard customization**: Rearranging widgets and cards
- **Photo gallery management**: Organizing images and albums
- **Task management**: Reordering priority items
- **Game development**: Puzzle games and inventory systems
- **File management**: Organizing files and folders

---

## Basic Grid Drag and Drop Implementation

### 🎯 Step 1: Data Model Setup

```typescript
// Data model for grid items
@Observed
class GridItem {
  id: string;
  title: string;
  content: string;
  backgroundColor: Color;
  isDragging: boolean = false;
  dragPreview: boolean = false;

  constructor(id: string, title: string, content: string, color: Color) {
    this.id = id;
    this.title = title;
    this.content = content;
    this.backgroundColor = color;
  }
}

// Data source for grid
@Observed
class GridDataSource {
  items: GridItem[] = [];

  constructor() {
    this.initializeData();
  }

  private initializeData(): void {
    const colors = [
      Color.Red,
      Color.Blue,
      Color.Green,
      Color.Orange,
      Color.Purple,
    ];
    this.items = Array.from(
      { length: 20 },
      (_, index) =>
        new GridItem(
          `item_${index}`,
          `Item ${index + 1}`,
          `Content ${index + 1}`,
          colors[index % colors.length]
        )
    );
  }

  // Swap two items by their indices
  swapItems(fromIndex: number, toIndex: number): void {
    if (
      fromIndex >= 0 &&
      fromIndex < this.items.length &&
      toIndex >= 0 &&
      toIndex < this.items.length &&
      fromIndex !== toIndex
    ) {
      const temp = this.items[fromIndex];
      this.items[fromIndex] = this.items[toIndex];
      this.items[toIndex] = temp;
    }
  }

  // Move item from one position to another
  moveItem(fromIndex: number, toIndex: number): void {
    if (
      fromIndex >= 0 &&
      fromIndex < this.items.length &&
      toIndex >= 0 &&
      toIndex < this.items.length &&
      fromIndex !== toIndex
    ) {
      const item = this.items.splice(fromIndex, 1)[0];
      this.items.splice(toIndex, 0, item);
    }
  }
}
```

### 🎯 Step 2: Draggable Grid Item Component

```typescript
@Component
struct DraggableGridItem {
  @ObjectLink item: GridItem;
  @State dragOpacity: number = 1.0;
  @State dragScale: number = 1.0;
  @State dragRotation: number = 0;
  onDragStart?: (item: GridItem) => void;
  onDragEnd?: (item: GridItem) => void;
  onDrop?: (draggedItem: GridItem, targetItem: GridItem) => void;

  build() {
    Column({ space: 8 }) {
      Text(this.item.title)
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .fontColor(Color.White)
        .maxLines(1);

      Text(this.item.content)
        .fontSize(12)
        .fontColor(Color.White)
        .opacity(0.8)
        .maxLines(2);
    }
    .width("100%")
    .height(120)
    .padding(12)
    .backgroundColor(this.item.backgroundColor)
    .borderRadius(8)
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
    .opacity(this.dragOpacity)
    .scale({ x: this.dragScale, y: this.dragScale })
    .rotate({ angle: this.dragRotation })
    .shadow({
      radius: this.item.isDragging ? 12 : 4,
      color: Color.Black,
      offsetX: 0,
      offsetY: this.item.isDragging ? 8 : 2
    })
    .gesture(
      // Long press to start drag
      LongPressGesture({ repeat: false, duration: 500 })
        .onAction(() => {
          this.startDrag();
        })
    )
    .gesture(
      // Pan gesture for dragging
      PanGesture()
        .onActionStart(() => {
          if (this.item.isDragging) {
            this.animateDragStart();
          }
        })
        .onActionUpdate((event: GestureEvent) => {
          if (this.item.isDragging) {
            // Handle drag movement
            console.log(`Dragging: x=${event.offsetX}, y=${event.offsetY}`);
          }
        })
        .onActionEnd(() => {
          if (this.item.isDragging) {
            this.endDrag();
          }
        })
    )
    .onTouch((event: TouchEvent) => {
      if (event.type === TouchType.Down && this.item.isDragging) {
        // Handle drop on this item
        this.handleDrop();
      }
    });
  }

  private startDrag(): void {
    this.item.isDragging = true;
    this.onDragStart?.(this.item);
    this.animateDragStart();
  }

  private endDrag(): void {
    this.item.isDragging = false;
    this.onDragEnd?.(this.item);
    this.animateDragEnd();
  }

  private handleDrop(): void {
    // This will be handled by the parent grid component
    this.onDrop?.(this.item, this.item);
  }

  private animateDragStart(): void {
    animateTo({
      duration: 200,
      curve: Curve.EaseOut
    }, () => {
      this.dragOpacity = 0.8;
      this.dragScale = 1.1;
      this.dragRotation = 5;
    });
  }

  private animateDragEnd(): void {
    animateTo({
      duration: 300,
      curve: Curve.EaseInOut
    }, () => {
      this.dragOpacity = 1.0;
      this.dragScale = 1.0;
      this.dragRotation = 0;
    });
  }
}
```

### 🎯 Step 3: Main Grid Component with Drag and Drop

```typescript
@Component
struct DragDropGrid {
  @State dataSource: GridDataSource = new GridDataSource();
  @State draggedItem: GridItem | null = null;
  @State dropTarget: GridItem | null = null;
  private gridScroller: Scroller = new Scroller();

  build() {
    Column({ space: 16 }) {
      // Header
      Row() {
        Text("Grid Drag & Drop Demo")
          .fontSize(24)
          .fontWeight(FontWeight.Bold);

        Blank();

        Button("Reset")
          .type(ButtonType.Capsule)
          .onClick(() => {
            this.resetGrid();
          });
      }
      .width("100%")
      .padding({ left: 16, right: 16 });

      // Grid with drag and drop
      Grid(this.gridScroller) {
        ForEach(this.dataSource.items, (item: GridItem, index: number) => {
          GridItem() {
            DraggableGridItem({
              item: item,
              onDragStart: (draggedItem: GridItem) => {
                this.handleDragStart(draggedItem);
              },
              onDragEnd: (draggedItem: GridItem) => {
                this.handleDragEnd(draggedItem);
              },
              onDrop: (draggedItem: GridItem, targetItem: GridItem) => {
                this.handleDrop(draggedItem, targetItem);
              }
            });
          }
          .onHover((isHover: boolean) => {
            if (isHover && this.draggedItem && this.draggedItem.id !== item.id) {
              this.dropTarget = item;
              this.highlightDropTarget(item, true);
            } else {
              this.highlightDropTarget(item, false);
            }
          });
        }, (item: GridItem) => item.id);
      }
      .columnsTemplate("1fr 1fr 1fr 1fr")
      .rowsGap(12)
      .columnsGap(12)
      .padding(16)
      .width("100%")
      .layoutWeight(1);

      // Status info
      if (this.draggedItem) {
        Text(`Dragging: ${this.draggedItem.title}`)
          .fontSize(14)
          .fontColor(Color.Gray)
          .padding(16);
      }
    }
    .width("100%")
    .height("100%")
    .backgroundColor(Color.Grey);
  }

  private handleDragStart(item: GridItem): void {
    this.draggedItem = item;
    this.provideDragFeedback();
  }

  private handleDragEnd(item: GridItem): void {
    if (this.dropTarget && this.draggedItem) {
      this.performDrop();
    }

    this.draggedItem = null;
    this.dropTarget = null;
    this.clearDragFeedback();
  }

  private handleDrop(draggedItem: GridItem, targetItem: GridItem): void {
    if (draggedItem.id !== targetItem.id) {
      this.swapItems(draggedItem, targetItem);
    }
  }

  private swapItems(item1: GridItem, item2: GridItem): void {
    const index1 = this.dataSource.items.findIndex(item => item.id === item1.id);
    const index2 = this.dataSource.items.findIndex(item => item.id === item2.id);

    if (index1 !== -1 && index2 !== -1) {
      this.dataSource.swapItems(index1, index2);
      this.animateSwap();
    }
  }

  private performDrop(): void {
    if (this.draggedItem && this.dropTarget) {
      this.swapItems(this.draggedItem, this.dropTarget);
    }
  }

  private highlightDropTarget(item: GridItem, highlight: boolean): void {
    // Visual feedback for drop target
    if (highlight) {
      item.backgroundColor = Color.Yellow;
    } else {
      // Restore original color
      this.restoreOriginalColor(item);
    }
  }

  private restoreOriginalColor(item: GridItem): void {
    const colors = [Color.Red, Color.Blue, Color.Green, Color.Orange, Color.Purple];
    const index = this.dataSource.items.findIndex(i => i.id === item.id);
    if (index !== -1) {
      item.backgroundColor = colors[index % colors.length];
    }
  }

  private provideDragFeedback(): void {
    // Haptic feedback
    try {
      vibrator.vibrate({ type: "time", duration: 50 });
    } catch (error) {
      console.log("Vibration not available");
    }
  }

  private clearDragFeedback(): void {
    // Clear any remaining visual feedback
    this.dataSource.items.forEach(item => {
      this.restoreOriginalColor(item);
    });
  }

  private animateSwap(): void {
    animateTo({
      duration: 300,
      curve: Curve.EaseInOut
    }, () => {
      // Animation handled by the framework
    });
  }

  private resetGrid(): void {
    this.dataSource = new GridDataSource();
    this.draggedItem = null;
    this.dropTarget = null;
  }
}
```

---

## Advanced Features

### 🎯 Smooth Visual Feedback

```typescript
@Component
struct EnhancedDragDropGrid {
  @State dataSource: GridDataSource = new GridDataSource();
  @State draggedItem: GridItem | null = null;
  @State dragOffset: { x: number, y: number } = { x: 0, y: 0 };
  @State dragPosition: { x: number, y: number } = { x: 0, y: 0 };

  build() {
    Stack() {
      // Main grid
      Grid() {
        ForEach(this.dataSource.items, (item: GridItem, index: number) => {
          GridItem() {
            if (item.isDragging) {
              // Placeholder for dragged item
              this.buildPlaceholder();
            } else {
              DraggableGridItem({ item: item });
            }
          }
        }, (item: GridItem) => item.id);
      }
      .columnsTemplate("1fr 1fr 1fr 1fr")
      .rowsGap(12)
      .columnsGap(12)
      .padding(16);

      // Floating drag preview
      if (this.draggedItem) {
        DraggableGridItem({ item: this.draggedItem })
          .position({
            x: this.dragPosition.x,
            y: this.dragPosition.y
          })
          .zIndex(999)
          .shadow({
            radius: 20,
            color: Color.Black,
            offsetX: 0,
            offsetY: 10
          });
      }
    }
    .width("100%")
    .height("100%");
  }

  @Builder
  buildPlaceholder() {
    Column()
      .width("100%")
      .height(120)
      .backgroundColor(Color.Transparent)
      .borderRadius(8)
      .border({
        width: 2,
        color: Color.Gray,
        style: BorderStyle.Dashed
      });
  }
}
```

### 🎯 Auto-Scroll During Drag

```typescript
@Component
struct AutoScrollDragGrid {
  private scrollTimer: number = -1;
  private scrollSpeed: number = 0;
  private gridScroller: Scroller = new Scroller();

  private handleDragMove(event: GestureEvent): void {
    const containerHeight = 800; // Grid container height
    const scrollThreshold = 100;
    const currentY = event.offsetY;

    // Check if near top or bottom edges
    if (currentY < scrollThreshold) {
      // Near top - scroll up
      this.scrollSpeed = -2;
      this.startAutoScroll();
    } else if (currentY > containerHeight - scrollThreshold) {
      // Near bottom - scroll down
      this.scrollSpeed = 2;
      this.startAutoScroll();
    } else {
      // In middle - stop auto scroll
      this.stopAutoScroll();
    }
  }

  private startAutoScroll(): void {
    if (this.scrollTimer !== -1) return;

    this.scrollTimer = setInterval(() => {
      const currentOffset = this.gridScroller.currentOffset();
      this.gridScroller.scrollTo({
        xOffset: currentOffset.xOffset,
        yOffset: currentOffset.yOffset + this.scrollSpeed,
        animation: false
      });
    }, 16); // 60fps
  }

  private stopAutoScroll(): void {
    if (this.scrollTimer !== -1) {
      clearInterval(this.scrollTimer);
      this.scrollTimer = -1;
    }
  }
}
```

### 🎯 Multi-Select Drag and Drop

```typescript
@Component
struct MultiSelectDragGrid {
  @State selectedItems: Set<string> = new Set();
  @State isMultiSelectMode: boolean = false;

  build() {
    Column() {
      // Multi-select controls
      Row({ space: 16 }) {
        Button(this.isMultiSelectMode ? "Exit Multi-Select" : "Multi-Select")
          .onClick(() => {
            this.isMultiSelectMode = !this.isMultiSelectMode;
            if (!this.isMultiSelectMode) {
              this.selectedItems.clear();
            }
          });

        if (this.isMultiSelectMode && this.selectedItems.size > 0) {
          Text(`${this.selectedItems.size} selected`)
            .fontColor(Color.Blue);

          Button("Delete Selected")
            .backgroundColor(Color.Red)
            .fontColor(Color.White)
            .onClick(() => {
              this.deleteSelectedItems();
            });
        }
      }
      .width("100%")
      .padding(16);

      // Grid with multi-select support
      Grid() {
        ForEach(this.dataSource.items, (item: GridItem) => {
          GridItem() {
            Stack() {
              DraggableGridItem({ item: item });

              // Selection overlay
              if (this.isMultiSelectMode) {
                Column()
                  .width("100%")
                  .height("100%")
                  .backgroundColor(
                    this.selectedItems.has(item.id)
                      ? Color.Blue.opacity(0.3)
                      : Color.Transparent
                  )
                  .borderRadius(8)
                  .onClick(() => {
                    this.toggleSelection(item.id);
                  });

                // Checkmark
                if (this.selectedItems.has(item.id)) {
                  Image($r("app.media.checkmark"))
                    .width(24)
                    .height(24)
                    .position({ x: "90%", y: "5%" });
                }
              }
            }
          }
        }, (item: GridItem) => item.id);
      }
      .columnsTemplate("1fr 1fr 1fr 1fr")
      .rowsGap(12)
      .columnsGap(12)
      .padding(16)
      .layoutWeight(1);
    }
  }

  private toggleSelection(itemId: string): void {
    if (this.selectedItems.has(itemId)) {
      this.selectedItems.delete(itemId);
    } else {
      this.selectedItems.add(itemId);
    }
  }

  private deleteSelectedItems(): void {
    this.selectedItems.forEach(itemId => {
      const index = this.dataSource.items.findIndex(item => item.id === itemId);
      if (index !== -1) {
        this.dataSource.items.splice(index, 1);
      }
    });
    this.selectedItems.clear();
  }
}
```

---

## Performance Optimization

### 🎯 Optimized Rendering

```typescript
// Use @Reusable for better performance
@Reusable
@Component
struct OptimizedGridItem {
  @State item: GridItem = new GridItem("", "", "", Color.White);

  aboutToReuse(params: Record<string, Object>): void {
    this.item = params.item as GridItem;
  }

  build() {
    // Optimized component implementation
    Column({ space: 8 }) {
      Text(this.item.title)
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .fontColor(Color.White)
        .maxLines(1)
        .textOverflow({ overflow: TextOverflow.Ellipsis });

      Text(this.item.content)
        .fontSize(12)
        .fontColor(Color.White)
        .opacity(0.8)
        .maxLines(2)
        .textOverflow({ overflow: TextOverflow.Ellipsis });
    }
    .width("100%")
    .height(120)
    .padding(12)
    .backgroundColor(this.item.backgroundColor)
    .borderRadius(8)
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
    .renderGroup(true); // Enable render group optimization
  }
}
```

## Best Practices Summary

### 🎯 Implementation Guidelines

- [ ] Use proper data models with `@Observed` and `@ObjectLink`
- [ ] Implement smooth animations for drag feedback
- [ ] Provide visual cues for drop targets
- [ ] Handle edge cases (boundary checking, invalid drops)
- [ ] Add haptic feedback for better user experience
- [ ] Implement auto-scroll for long grids
- [ ] Use `@Reusable` components for performance

### 🎯 User Experience Tips

- [ ] Start drag with long press (500ms recommended)
- [ ] Provide immediate visual feedback on drag start
- [ ] Show clear drop zones and invalid areas
- [ ] Animate position changes smoothly
- [ ] Support both swap and insert operations
- [ ] Add undo/redo functionality for complex operations
- [ ] Test on different screen sizes and orientations

With these patterns, you can create intuitive and performant grid drag-and-drop experiences in HarmonyOS!
