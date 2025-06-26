# ArkUI Advanced Gestures and Touch Handling

## Introduction

Advanced gesture recognition and touch handling are essential for creating intuitive user interfaces. This guide explores sophisticated gesture patterns, multi-touch support, and custom gesture recognizers in ArkUI applications.

## Custom Gesture Recognizers

### Base Gesture System

```typescript
interface GestureConfig {
  minDistance?: number;
  maxDuration?: number;
  threshold?: number;
  fingers?: number;
}

interface TouchPoint {
  id: number;
  x: number;
  y: number;
  timestamp: number;
}

interface GestureEvent {
  type: string;
  touches: TouchPoint[];
  center: { x: number; y: number };
  distance?: number;
  angle?: number;
  velocity?: { x: number; y: number };
}

abstract class BaseGestureRecognizer {
  protected config: GestureConfig;
  protected isActive = false;
  protected startTime = 0;
  protected initialTouches: TouchPoint[] = [];

  constructor(config: GestureConfig = {}) {
    this.config = {
      minDistance: 10,
      maxDuration: 2000,
      threshold: 0.1,
      fingers: 1,
      ...config,
    };
  }

  abstract recognize(touches: TouchPoint[]): GestureEvent | null;

  start(touches: TouchPoint[]): void {
    if (touches.length === this.config.fingers) {
      this.isActive = true;
      this.startTime = Date.now();
      this.initialTouches = [...touches];
    }
  }

  end(): void {
    this.isActive = false;
    this.initialTouches = [];
  }

  protected calculateCenter(touches: TouchPoint[]): { x: number; y: number } {
    const sumX = touches.reduce((sum, touch) => sum + touch.x, 0);
    const sumY = touches.reduce((sum, touch) => sum + touch.y, 0);
    return {
      x: sumX / touches.length,
      y: sumY / touches.length,
    };
  }

  protected calculateDistance(touch1: TouchPoint, touch2: TouchPoint): number {
    const dx = touch2.x - touch1.x;
    const dy = touch2.y - touch1.y;
    return Math.sqrt(dx * dx + dy * dy);
  }

  protected calculateVelocity(
    start: TouchPoint,
    end: TouchPoint,
    duration: number
  ): { x: number; y: number } {
    const dx = end.x - start.x;
    const dy = end.y - start.y;
    return {
      x: dx / duration,
      y: dy / duration,
    };
  }
}

class SwipeGestureRecognizer extends BaseGestureRecognizer {
  recognize(touches: TouchPoint[]): GestureEvent | null {
    if (!this.isActive || touches.length !== 1) return null;

    const currentTouch = touches[0];
    const initialTouch = this.initialTouches[0];
    const distance = this.calculateDistance(initialTouch, currentTouch);
    const duration = Date.now() - this.startTime;

    if (distance < this.config.minDistance!) return null;

    const velocity = this.calculateVelocity(
      initialTouch,
      currentTouch,
      duration
    );
    const angle =
      (Math.atan2(
        currentTouch.y - initialTouch.y,
        currentTouch.x - initialTouch.x
      ) *
        180) /
      Math.PI;

    return {
      type: "swipe",
      touches,
      center: this.calculateCenter(touches),
      distance,
      angle,
      velocity,
    };
  }
}

class PinchGestureRecognizer extends BaseGestureRecognizer {
  private initialDistance = 0;

  constructor(config: GestureConfig = {}) {
    super({ ...config, fingers: 2 });
  }

  start(touches: TouchPoint[]): void {
    super.start(touches);
    if (this.initialTouches.length === 2) {
      this.initialDistance = this.calculateDistance(
        this.initialTouches[0],
        this.initialTouches[1]
      );
    }
  }

  recognize(touches: TouchPoint[]): GestureEvent | null {
    if (!this.isActive || touches.length !== 2) return null;

    const currentDistance = this.calculateDistance(touches[0], touches[1]);
    const scale = currentDistance / this.initialDistance;

    return {
      type: "pinch",
      touches,
      center: this.calculateCenter(touches),
      distance: currentDistance,
      velocity: { x: scale, y: scale },
    };
  }
}

class RotationGestureRecognizer extends BaseGestureRecognizer {
  private initialAngle = 0;

  constructor(config: GestureConfig = {}) {
    super({ ...config, fingers: 2 });
  }

  start(touches: TouchPoint[]): void {
    super.start(touches);
    if (this.initialTouches.length === 2) {
      this.initialAngle = this.calculateAngle(
        this.initialTouches[0],
        this.initialTouches[1]
      );
    }
  }

  recognize(touches: TouchPoint[]): GestureEvent | null {
    if (!this.isActive || touches.length !== 2) return null;

    const currentAngle = this.calculateAngle(touches[0], touches[1]);
    const rotation = currentAngle - this.initialAngle;

    return {
      type: "rotation",
      touches,
      center: this.calculateCenter(touches),
      angle: rotation,
      velocity: { x: 0, y: 0 },
    };
  }

  private calculateAngle(touch1: TouchPoint, touch2: TouchPoint): number {
    return Math.atan2(touch2.y - touch1.y, touch2.x - touch1.x);
  }
}
```

### Gesture Manager

```typescript
type GestureHandler = (event: GestureEvent) => void;

interface GestureBinding {
  recognizer: BaseGestureRecognizer;
  handler: GestureHandler;
  priority: number;
}

class GestureManager {
  private bindings: GestureBinding[] = [];
  private activeTouches = new Map<number, TouchPoint>();
  private activeGestures = new Set<BaseGestureRecognizer>();

  addGesture(
    recognizer: BaseGestureRecognizer,
    handler: GestureHandler,
    priority = 0
  ): () => void {
    const binding: GestureBinding = { recognizer, handler, priority };
    this.bindings.push(binding);
    this.bindings.sort((a, b) => b.priority - a.priority);

    return () => {
      const index = this.bindings.indexOf(binding);
      if (index >= 0) {
        this.bindings.splice(index, 1);
      }
    };
  }

  handleTouchStart(touches: TouchEvent): void {
    touches.changedTouches.forEach((touch) => {
      const touchPoint: TouchPoint = {
        id: touch.identifier,
        x: touch.clientX,
        y: touch.clientY,
        timestamp: Date.now(),
      };
      this.activeTouches.set(touch.identifier, touchPoint);
    });

    const allTouches = Array.from(this.activeTouches.values());
    this.bindings.forEach((binding) => {
      binding.recognizer.start(allTouches);
    });
  }

  handleTouchMove(touches: TouchEvent): void {
    touches.changedTouches.forEach((touch) => {
      const touchPoint: TouchPoint = {
        id: touch.identifier,
        x: touch.clientX,
        y: touch.clientY,
        timestamp: Date.now(),
      };
      this.activeTouches.set(touch.identifier, touchPoint);
    });

    const allTouches = Array.from(this.activeTouches.values());
    this.recognizeGestures(allTouches);
  }

  handleTouchEnd(touches: TouchEvent): void {
    touches.changedTouches.forEach((touch) => {
      this.activeTouches.delete(touch.identifier);
    });

    const allTouches = Array.from(this.activeTouches.values());

    if (allTouches.length === 0) {
      this.bindings.forEach((binding) => {
        binding.recognizer.end();
      });
      this.activeGestures.clear();
    } else {
      this.recognizeGestures(allTouches);
    }
  }

  private recognizeGestures(touches: TouchPoint[]): void {
    for (const binding of this.bindings) {
      const gesture = binding.recognizer.recognize(touches);
      if (gesture) {
        this.activeGestures.add(binding.recognizer);
        binding.handler(gesture);
        break; // Stop at first recognized gesture (priority order)
      }
    }
  }
}
```

## Advanced Touch Interactions

### Multi-Touch Canvas

```typescript
@Component
struct MultiTouchCanvas {
  @State private paths: DrawPath[] = []
  @State private activePaths = new Map<number, DrawPath>()
  private gestureManager = new GestureManager()

  build() {
    Canvas(this.canvasContext)
      .width('100%')
      .height('100%')
      .backgroundColor('#f5f5f5')
      .onTouch((event) => this.handleTouch(event))
  }

  aboutToAppear() {
    this.setupGestures()
  }

  private setupGestures(): void {
    // Clear gesture
    const clearGesture = new class extends BaseGestureRecognizer {
      constructor() {
        super({ fingers: 3 })
      }

      recognize(touches: TouchPoint[]): GestureEvent | null {
        if (touches.length === 3) {
          return {
            type: 'clear',
            touches,
            center: this.calculateCenter(touches)
          }
        }
        return null
      }
    }()

    this.gestureManager.addGesture(clearGesture, () => {
      this.clearCanvas()
    }, 10)

    // Zoom gesture
    const pinchGesture = new PinchGestureRecognizer()
    this.gestureManager.addGesture(pinchGesture, (event) => {
      this.handleZoom(event)
    }, 5)
  }

  private handleTouch(event: TouchEvent): void {
    switch (event.type) {
      case TouchType.Down:
        this.handleTouchStart(event)
        this.gestureManager.handleTouchStart(event)
        break
      case TouchType.Move:
        this.handleTouchMove(event)
        this.gestureManager.handleTouchMove(event)
        break
      case TouchType.Up:
        this.handleTouchEnd(event)
        this.gestureManager.handleTouchEnd(event)
        break
    }
  }

  private handleTouchStart(event: TouchEvent): void {
    event.touches.forEach(touch => {
      const path: DrawPath = {
        id: touch.id,
        points: [{ x: touch.x, y: touch.y }],
        color: this.getRandomColor(),
        width: 3
      }
      this.activePaths.set(touch.id, path)
    })
  }

  private handleTouchMove(event: TouchEvent): void {
    event.touches.forEach(touch => {
      const path = this.activePaths.get(touch.id)
      if (path) {
        path.points.push({ x: touch.x, y: touch.y })
        this.redraw()
      }
    })
  }

  private handleTouchEnd(event: TouchEvent): void {
    event.changedTouches.forEach(touch => {
      const path = this.activePaths.get(touch.id)
      if (path) {
        this.paths.push(path)
        this.activePaths.delete(touch.id)
      }
    })
    this.redraw()
  }

  private clearCanvas(): void {
    this.paths = []
    this.activePaths.clear()
    this.redraw()
  }

  private handleZoom(event: GestureEvent): void {
    // Implement zoom functionality
    console.log('Zoom gesture:', event.velocity?.x)
  }

  private redraw(): void {
    // Canvas redraw logic
    this.canvasContext.clearRect(0, 0, this.canvasContext.width, this.canvasContext.height)

    this.paths.forEach(path => this.drawPath(path))
    this.activePaths.forEach(path => this.drawPath(path))
  }

  private drawPath(path: DrawPath): void {
    if (path.points.length < 2) return

    this.canvasContext.strokeStyle = path.color
    this.canvasContext.lineWidth = path.width
    this.canvasContext.beginPath()

    this.canvasContext.moveTo(path.points[0].x, path.points[0].y)
    for (let i = 1; i < path.points.length; i++) {
      this.canvasContext.lineTo(path.points[i].x, path.points[i].y)
    }

    this.canvasContext.stroke()
  }

  private getRandomColor(): string {
    const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FECA57']
    return colors[Math.floor(Math.random() * colors.length)]
  }
}

interface DrawPath {
  id: number
  points: { x: number; y: number }[]
  color: string
  width: number
}
```

## Gesture-Based Navigation

### Swipe Navigation Component

```typescript
@Component
struct SwipeNavigationView {
  @State private currentIndex: number = 0
  @State private translateX: number = 0
  @State private isDragging: boolean = false
  private pages: string[] = ['Page 1', 'Page 2', 'Page 3']
  private gestureManager = new GestureManager()
  private swipeThreshold = 100

  build() {
    Stack() {
      Row() {
        ForEach(this.pages, (page: string, index: number) => {
          this.buildPage(page, index)
        })
      }
      .width(`${this.pages.length * 100}%`)
      .translate({ x: this.calculateTranslateX() })
      .animation({
        duration: this.isDragging ? 0 : 300,
        curve: Curve.EaseOut
      })

      // Page indicators
      Row() {
        ForEach(this.pages, (_, index: number) => {
          Circle()
            .width(8)
            .height(8)
            .fill(index === this.currentIndex ? '#007AFF' : '#C7C7CC')
            .margin({ horizontal: 4 })
        })
      }
      .position({ x: '50%', y: '90%' })
      .translate({ x: '-50%' })
    }
    .width('100%')
    .height('100%')
    .onTouch((event) => this.handleTouch(event))
  }

  aboutToAppear() {
    this.setupSwipeGestures()
  }

  private setupSwipeGestures(): void {
    const swipeGesture = new SwipeGestureRecognizer({
      minDistance: 50
    })

    this.gestureManager.addGesture(swipeGesture, (event) => {
      this.handleSwipe(event)
    })

    // Pan gesture for real-time dragging
    const panGesture = new class extends BaseGestureRecognizer {
      recognize(touches: TouchPoint[]): GestureEvent | null {
        if (!this.isActive || touches.length !== 1) return null

        const currentTouch = touches[0]
        const initialTouch = this.initialTouches[0]
        const deltaX = currentTouch.x - initialTouch.x

        return {
          type: 'pan',
          touches,
          center: this.calculateCenter(touches),
          distance: Math.abs(deltaX),
          velocity: { x: deltaX, y: 0 }
        }
      }
    }()

    this.gestureManager.addGesture(panGesture, (event) => {
      this.handlePan(event)
    })
  }

  private handleTouch(event: TouchEvent): void {
    switch (event.type) {
      case TouchType.Down:
        this.isDragging = true
        this.gestureManager.handleTouchStart(event)
        break
      case TouchType.Move:
        this.gestureManager.handleTouchMove(event)
        break
      case TouchType.Up:
        this.isDragging = false
        this.gestureManager.handleTouchEnd(event)
        this.snapToNearestPage()
        break
    }
  }

  private handleSwipe(event: GestureEvent): void {
    const isLeftSwipe = event.velocity!.x < 0

    if (isLeftSwipe && this.currentIndex < this.pages.length - 1) {
      this.navigateToPage(this.currentIndex + 1)
    } else if (!isLeftSwipe && this.currentIndex > 0) {
      this.navigateToPage(this.currentIndex - 1)
    }
  }

  private handlePan(event: GestureEvent): void {
    const deltaX = event.velocity!.x
    const maxTranslate = this.swipeThreshold

    // Limit the drag range
    if (Math.abs(deltaX) <= maxTranslate) {
      this.translateX = deltaX
    }
  }

  private snapToNearestPage(): void {
    const threshold = this.swipeThreshold / 2

    if (Math.abs(this.translateX) > threshold) {
      const direction = this.translateX > 0 ? -1 : 1
      const newIndex = this.currentIndex + direction

      if (newIndex >= 0 && newIndex < this.pages.length) {
        this.navigateToPage(newIndex)
      }
    }

    this.translateX = 0
  }

  private navigateToPage(index: number): void {
    this.currentIndex = Math.max(0, Math.min(index, this.pages.length - 1))
    this.translateX = 0
  }

  private calculateTranslateX(): number {
    const baseTranslate = -this.currentIndex * 100
    const dragOffset = (this.translateX / this.swipeThreshold) * 100
    return `${baseTranslate + dragOffset}%`
  }

  @Builder
  private buildPage(content: string, index: number) {
    Column() {
      Text(content)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`Swipe left or right to navigate`)
        .fontSize(16)
        .fontColor('#666')
        .margin({ top: 16 })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .backgroundColor(this.getPageColor(index))
  }

  private getPageColor(index: number): string {
    const colors = ['#FFE5E5', '#E5F3FF', '#E5FFE5']
    return colors[index % colors.length]
  }
}
```

## Custom Gesture Components

### Draggable Component

```typescript
@Component
struct DraggableItem {
  @State private position: { x: number; y: number } = { x: 0, y: 0 }
  @State private isDragging: boolean = false
  @State private scale: number = 1
  private startPosition: { x: number; y: number } = { x: 0, y: 0 }
  private gestureManager = new GestureManager()

  build() {
    Text('Drag Me!')
      .fontSize(16)
      .fontColor(Color.White)
      .backgroundColor('#007AFF')
      .padding(16)
      .borderRadius(8)
      .position({ x: this.position.x, y: this.position.y })
      .scale({ x: this.scale, y: this.scale })
      .shadow({
        radius: this.isDragging ? 20 : 10,
        color: '#000000',
        offsetX: 0,
        offsetY: this.isDragging ? 10 : 5
      })
      .animation({
        duration: 200,
        curve: Curve.EaseOut
      })
      .onTouch((event) => this.handleTouch(event))
  }

  aboutToAppear() {
    this.setupDragGesture()
  }

  private setupDragGesture(): void {
    const dragGesture = new class extends BaseGestureRecognizer {
      recognize(touches: TouchPoint[]): GestureEvent | null {
        if (!this.isActive || touches.length !== 1) return null

        const currentTouch = touches[0]
        const initialTouch = this.initialTouches[0]

        return {
          type: 'drag',
          touches,
          center: this.calculateCenter(touches),
          velocity: {
            x: currentTouch.x - initialTouch.x,
            y: currentTouch.y - initialTouch.y
          }
        }
      }
    }()

    this.gestureManager.addGesture(dragGesture, (event) => {
      this.handleDrag(event)
    })
  }

  private handleTouch(event: TouchEvent): void {
    switch (event.type) {
      case TouchType.Down:
        this.startDrag(event)
        this.gestureManager.handleTouchStart(event)
        break
      case TouchType.Move:
        this.gestureManager.handleTouchMove(event)
        break
      case TouchType.Up:
        this.endDrag()
        this.gestureManager.handleTouchEnd(event)
        break
    }
  }

  private startDrag(event: TouchEvent): void {
    this.isDragging = true
    this.scale = 1.1
    this.startPosition = { ...this.position }
  }

  private handleDrag(event: GestureEvent): void {
    const deltaX = event.velocity!.x
    const deltaY = event.velocity!.y

    this.position = {
      x: this.startPosition.x + deltaX,
      y: this.startPosition.y + deltaY
    }
  }

  private endDrag(): void {
    this.isDragging = false
    this.scale = 1

    // Optional: Snap to grid or bounds
    this.snapToGrid()
  }

  private snapToGrid(): void {
    const gridSize = 50
    this.position = {
      x: Math.round(this.position.x / gridSize) * gridSize,
      y: Math.round(this.position.y / gridSize) * gridSize
    }
  }
}
```

## Conclusion

Advanced gesture and touch handling in ArkUI enables:

- Custom gesture recognition systems
- Multi-touch interaction support
- Gesture-based navigation patterns
- Real-time touch feedback
- Complex interactive components
- Natural user interface experiences

These techniques provide the foundation for creating engaging and intuitive user interfaces that respond naturally to user input.
