# 45. ArkUI Gesture Recognition and Touch Events

## Introduction

Gesture recognition and touch event handling are fundamental to creating intuitive and responsive user interfaces in HarmonyOS applications. This article explores advanced gesture recognition techniques, custom gesture implementation, and optimized touch event handling strategies.

## Core Gesture System

### Advanced Gesture Recognition Framework

```typescript
// Comprehensive gesture recognition system
export enum GestureType {
  Tap = "tap",
  DoubleTap = "doubleTap",
  LongPress = "longPress",
  Pan = "pan",
  Pinch = "pinch",
  Rotation = "rotation",
  Swipe = "swipe",
  Custom = "custom",
}

export interface GestureEvent {
  type: GestureType;
  timestamp: number;
  position: { x: number; y: number };
  velocity?: { x: number; y: number };
  scale?: number;
  rotation?: number;
  distance?: number;
  direction?: SwipeDirection;
  target: any;
}

export interface GestureConfig {
  type: GestureType;
  threshold?: number;
  sensitivity?: number;
  direction?: PanDirection | SwipeDirection;
  fingers?: number;
  distance?: number;
  duration?: number;
}

export class GestureRecognizer {
  private gestures: Map<string, GestureConfig> = new Map();
  private activeGestures: Map<string, GestureState> = new Map();
  private handlers: Map<string, Array<(event: GestureEvent) => void>> =
    new Map();
  private touchHistory: TouchPoint[] = [];
  private maxHistorySize = 50;

  registerGesture(id: string, config: GestureConfig): void {
    this.gestures.set(id, config);
  }

  unregisterGesture(id: string): void {
    this.gestures.delete(id);
    this.activeGestures.delete(id);
    this.handlers.delete(id);
  }

  onGesture(
    gestureId: string,
    handler: (event: GestureEvent) => void
  ): () => void {
    if (!this.handlers.has(gestureId)) {
      this.handlers.set(gestureId, []);
    }
    this.handlers.get(gestureId)!.push(handler);

    return () => {
      const handlers = this.handlers.get(gestureId);
      if (handlers) {
        const index = handlers.indexOf(handler);
        if (index > -1) {
          handlers.splice(index, 1);
        }
      }
    };
  }

  processTouchEvent(touchEvent: TouchEvent): void {
    this.updateTouchHistory(touchEvent);

    // Check all registered gestures
    this.gestures.forEach((config, id) => {
      this.recognizeGesture(id, config, touchEvent);
    });
  }

  private updateTouchHistory(touchEvent: TouchEvent): void {
    const touchPoint: TouchPoint = {
      x: touchEvent.touches[0]?.screenX || 0,
      y: touchEvent.touches[0]?.screenY || 0,
      timestamp: Date.now(),
      type: touchEvent.type as TouchEventType,
    };

    this.touchHistory.push(touchPoint);

    if (this.touchHistory.length > this.maxHistorySize) {
      this.touchHistory.shift();
    }
  }

  private recognizeGesture(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    switch (config.type) {
      case GestureType.Tap:
        this.recognizeTap(id, config, touchEvent);
        break;
      case GestureType.DoubleTap:
        this.recognizeDoubleTap(id, config, touchEvent);
        break;
      case GestureType.LongPress:
        this.recognizeLongPress(id, config, touchEvent);
        break;
      case GestureType.Pan:
        this.recognizePan(id, config, touchEvent);
        break;
      case GestureType.Pinch:
        this.recognizePinch(id, config, touchEvent);
        break;
      case GestureType.Rotation:
        this.recognizeRotation(id, config, touchEvent);
        break;
      case GestureType.Swipe:
        this.recognizeSwipe(id, config, touchEvent);
        break;
    }
  }

  private recognizeTap(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    if (touchEvent.type === "touchend") {
      const tapDuration = this.getTouchDuration();
      const maxTapDuration = config.duration || 200;

      if (tapDuration < maxTapDuration) {
        this.emitGesture(id, {
          type: GestureType.Tap,
          timestamp: Date.now(),
          position: this.getLastTouchPosition(),
          target: touchEvent.target,
        });
      }
    }
  }

  private recognizeDoubleTap(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    if (touchEvent.type === "touchend") {
      const state = this.getOrCreateGestureState(id);
      const now = Date.now();
      const maxInterval = config.duration || 300;

      if (state.lastTapTime && now - state.lastTapTime < maxInterval) {
        this.emitGesture(id, {
          type: GestureType.DoubleTap,
          timestamp: now,
          position: this.getLastTouchPosition(),
          target: touchEvent.target,
        });
        state.lastTapTime = 0;
      } else {
        state.lastTapTime = now;
      }
    }
  }

  private recognizeLongPress(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    const state = this.getOrCreateGestureState(id);
    const pressDuration = config.duration || 500;

    if (touchEvent.type === "touchstart") {
      state.startTime = Date.now();
      state.startPosition = this.getLastTouchPosition();

      state.timer = setTimeout(() => {
        const distance = this.getDistanceFromStart(state.startPosition!);
        const maxMovement = config.threshold || 10;

        if (distance < maxMovement) {
          this.emitGesture(id, {
            type: GestureType.LongPress,
            timestamp: Date.now(),
            position: state.startPosition!,
            target: touchEvent.target,
          });
        }
      }, pressDuration);
    } else if (
      touchEvent.type === "touchend" ||
      touchEvent.type === "touchmove"
    ) {
      if (state.timer) {
        clearTimeout(state.timer);
        state.timer = undefined;
      }
    }
  }

  private recognizePan(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    const state = this.getOrCreateGestureState(id);

    if (touchEvent.type === "touchstart") {
      state.startPosition = this.getLastTouchPosition();
      state.lastPosition = state.startPosition;
    } else if (touchEvent.type === "touchmove") {
      const currentPosition = this.getLastTouchPosition();
      const distance = this.getDistance(state.startPosition!, currentPosition);
      const threshold = config.threshold || 10;

      if (distance > threshold) {
        const velocity = this.calculateVelocity();

        this.emitGesture(id, {
          type: GestureType.Pan,
          timestamp: Date.now(),
          position: currentPosition,
          velocity,
          distance,
          target: touchEvent.target,
        });

        state.lastPosition = currentPosition;
      }
    }
  }

  private recognizePinch(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    if (touchEvent.touches.length !== 2) return;

    const state = this.getOrCreateGestureState(id);
    const touch1 = touchEvent.touches[0];
    const touch2 = touchEvent.touches[1];
    const currentDistance = this.getDistance(
      { x: touch1.screenX, y: touch1.screenY },
      { x: touch2.screenX, y: touch2.screenY }
    );

    if (touchEvent.type === "touchstart") {
      state.initialDistance = currentDistance;
    } else if (touchEvent.type === "touchmove" && state.initialDistance) {
      const scale = currentDistance / state.initialDistance;
      const center = {
        x: (touch1.screenX + touch2.screenX) / 2,
        y: (touch1.screenY + touch2.screenY) / 2,
      };

      this.emitGesture(id, {
        type: GestureType.Pinch,
        timestamp: Date.now(),
        position: center,
        scale,
        distance: currentDistance,
        target: touchEvent.target,
      });
    }
  }

  private recognizeRotation(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    if (touchEvent.touches.length !== 2) return;

    const state = this.getOrCreateGestureState(id);
    const touch1 = touchEvent.touches[0];
    const touch2 = touchEvent.touches[1];
    const currentAngle = Math.atan2(
      touch2.screenY - touch1.screenY,
      touch2.screenX - touch1.screenX
    );

    if (touchEvent.type === "touchstart") {
      state.initialAngle = currentAngle;
    } else if (
      touchEvent.type === "touchmove" &&
      state.initialAngle !== undefined
    ) {
      const rotation = currentAngle - state.initialAngle;
      const center = {
        x: (touch1.screenX + touch2.screenX) / 2,
        y: (touch1.screenY + touch2.screenY) / 2,
      };

      this.emitGesture(id, {
        type: GestureType.Rotation,
        timestamp: Date.now(),
        position: center,
        rotation,
        target: touchEvent.target,
      });
    }
  }

  private recognizeSwipe(
    id: string,
    config: GestureConfig,
    touchEvent: TouchEvent
  ): void {
    const state = this.getOrCreateGestureState(id);

    if (touchEvent.type === "touchstart") {
      state.startPosition = this.getLastTouchPosition();
      state.startTime = Date.now();
    } else if (touchEvent.type === "touchend") {
      const currentPosition = this.getLastTouchPosition();
      const distance = this.getDistance(state.startPosition!, currentPosition);
      const duration = Date.now() - state.startTime!;
      const velocity = distance / duration;

      const minDistance = config.distance || 50;
      const minVelocity = config.threshold || 0.3;

      if (distance > minDistance && velocity > minVelocity) {
        const direction = this.getSwipeDirection(
          state.startPosition!,
          currentPosition
        );

        this.emitGesture(id, {
          type: GestureType.Swipe,
          timestamp: Date.now(),
          position: currentPosition,
          direction,
          velocity: { x: velocity, y: velocity },
          distance,
          target: touchEvent.target,
        });
      }
    }
  }

  private getOrCreateGestureState(id: string): GestureState {
    if (!this.activeGestures.has(id)) {
      this.activeGestures.set(id, {});
    }
    return this.activeGestures.get(id)!;
  }

  private emitGesture(gestureId: string, event: GestureEvent): void {
    const handlers = this.handlers.get(gestureId) || [];
    handlers.forEach((handler) => {
      try {
        handler(event);
      } catch (error) {
        console.error("Gesture handler error:", error);
      }
    });
  }

  private getLastTouchPosition(): { x: number; y: number } {
    const lastTouch = this.touchHistory[this.touchHistory.length - 1];
    return { x: lastTouch?.x || 0, y: lastTouch?.y || 0 };
  }

  private getTouchDuration(): number {
    if (this.touchHistory.length < 2) return 0;
    const start = this.touchHistory.find((t) => t.type === "touchstart");
    const end = this.touchHistory[this.touchHistory.length - 1];
    return start && end ? end.timestamp - start.timestamp : 0;
  }

  private getDistance(
    point1: { x: number; y: number },
    point2: { x: number; y: number }
  ): number {
    const dx = point2.x - point1.x;
    const dy = point2.y - point1.y;
    return Math.sqrt(dx * dx + dy * dy);
  }

  private getDistanceFromStart(startPosition: {
    x: number;
    y: number;
  }): number {
    const currentPosition = this.getLastTouchPosition();
    return this.getDistance(startPosition, currentPosition);
  }

  private calculateVelocity(): { x: number; y: number } {
    if (this.touchHistory.length < 2) return { x: 0, y: 0 };

    const recent = this.touchHistory.slice(-5);
    const first = recent[0];
    const last = recent[recent.length - 1];
    const timeDiff = last.timestamp - first.timestamp;

    if (timeDiff === 0) return { x: 0, y: 0 };

    return {
      x: (last.x - first.x) / timeDiff,
      y: (last.y - first.y) / timeDiff,
    };
  }

  private getSwipeDirection(
    start: { x: number; y: number },
    end: { x: number; y: number }
  ): SwipeDirection {
    const dx = end.x - start.x;
    const dy = end.y - start.y;
    const absDx = Math.abs(dx);
    const absDy = Math.abs(dy);

    if (absDx > absDy) {
      return dx > 0 ? SwipeDirection.Right : SwipeDirection.Left;
    } else {
      return dy > 0 ? SwipeDirection.Down : SwipeDirection.Up;
    }
  }
}

// Supporting interfaces and types
interface TouchPoint {
  x: number;
  y: number;
  timestamp: number;
  type: TouchEventType;
}

interface GestureState {
  startTime?: number;
  startPosition?: { x: number; y: number };
  lastPosition?: { x: number; y: number };
  lastTapTime?: number;
  timer?: number;
  initialDistance?: number;
  initialAngle?: number;
}

enum TouchEventType {
  TouchStart = "touchstart",
  TouchMove = "touchmove",
  TouchEnd = "touchend",
  TouchCancel = "touchcancel",
}

enum SwipeDirection {
  Up = "up",
  Down = "down",
  Left = "left",
  Right = "right",
}
```

### Custom Gesture Components

```typescript
// Advanced gesture-enabled components
@Component
export struct GestureCanvas {
  @State private paths: DrawPath[] = [];
  @State private currentPath: DrawPath | null = null;
  @State private isDrawing: boolean = false;
  @State private lastPinchScale: number = 1;
  @State private canvasScale: number = 1;
  @State private canvasOffset: { x: number; y: number } = { x: 0, y: 0 };

  private gestureRecognizer = new GestureRecognizer();
  private canvas: CanvasRenderingContext2D | null = null;

  aboutToAppear() {
    this.setupGestures();
  }

  private setupGestures(): void {
    // Drawing gesture (single finger)
    this.gestureRecognizer.registerGesture('draw', {
      type: GestureType.Pan,
      threshold: 5,
      fingers: 1
    });

    // Pinch to zoom
    this.gestureRecognizer.registerGesture('zoom', {
      type: GestureType.Pinch,
      threshold: 0.1,
      fingers: 2
    });

    // Pan to move canvas
    this.gestureRecognizer.registerGesture('pan', {
      type: GestureType.Pan,
      threshold: 10,
      fingers: 2
    });

    // Double tap to reset
    this.gestureRecognizer.registerGesture('reset', {
      type: GestureType.DoubleTap,
      duration: 300
    });

    // Long press for menu
    this.gestureRecognizer.registerGesture('menu', {
      type: GestureType.LongPress,
      duration: 500,
      threshold: 10
    });

    this.setupGestureHandlers();
  }

  private setupGestureHandlers(): void {
    this.gestureRecognizer.onGesture('draw', (event) => {
      this.handleDrawGesture(event);
    });

    this.gestureRecognizer.onGesture('zoom', (event) => {
      this.handleZoomGesture(event);
    });

    this.gestureRecognizer.onGesture('pan', (event) => {
      this.handlePanGesture(event);
    });

    this.gestureRecognizer.onGesture('reset', (event) => {
      this.resetCanvas();
    });

    this.gestureRecognizer.onGesture('menu', (event) => {
      this.showContextMenu(event.position);
    });
  }

  private handleDrawGesture(event: GestureEvent): void {
    const adjustedPosition = this.adjustPositionForTransform(event.position);

    if (!this.isDrawing) {
      this.currentPath = {
        points: [adjustedPosition],
        color: '#000000',
        width: 2,
        timestamp: Date.now()
      };
      this.isDrawing = true;
    } else if (this.currentPath) {
      this.currentPath.points.push(adjustedPosition);
      this.redrawCanvas();
    }
  }

  private handleZoomGesture(event: GestureEvent): void {
    if (event.scale) {
      const deltaScale = event.scale / this.lastPinchScale;
      this.canvasScale *= deltaScale;
      this.canvasScale = Math.max(0.5, Math.min(3.0, this.canvasScale));
      this.lastPinchScale = event.scale;
      this.redrawCanvas();
    }
  }

  private handlePanGesture(event: GestureEvent): void {
    if (event.velocity) {
      this.canvasOffset.x += event.velocity.x * 10;
      this.canvasOffset.y += event.velocity.y * 10;
      this.redrawCanvas();
    }
  }

  private adjustPositionForTransform(position: { x: number; y: number }): { x: number; y: number } {
    return {
      x: (position.x - this.canvasOffset.x) / this.canvasScale,
      y: (position.y - this.canvasOffset.y) / this.canvasScale
    };
  }

  private resetCanvas(): void {
    this.canvasScale = 1;
    this.canvasOffset = { x: 0, y: 0 };
    this.paths = [];
    this.redrawCanvas();
  }

  private showContextMenu(position: { x: number; y: number }): void {
    // Show context menu at position
    console.log('Show context menu at:', position);
  }

  private redrawCanvas(): void {
    if (!this.canvas) return;

    this.canvas.clearRect(0, 0, 400, 400);
    this.canvas.save();
    this.canvas.scale(this.canvasScale, this.canvasScale);
    this.canvas.translate(this.canvasOffset.x, this.canvasOffset.y);

    // Draw all paths
    [...this.paths, this.currentPath].forEach(path => {
      if (path && path.points.length > 1) {
        this.drawPath(path);
      }
    });

    this.canvas.restore();
  }

  private drawPath(path: DrawPath): void {
    if (!this.canvas || path.points.length < 2) return;

    this.canvas.beginPath();
    this.canvas.strokeStyle = path.color;
    this.canvas.lineWidth = path.width;
    this.canvas.lineCap = 'round';
    this.canvas.lineJoin = 'round';

    this.canvas.moveTo(path.points[0].x, path.points[0].y);
    for (let i = 1; i < path.points.length; i++) {
      this.canvas.lineTo(path.points[i].x, path.points[i].y);
    }
    this.canvas.stroke();
  }

  private finishDrawing(): void {
    if (this.currentPath && this.currentPath.points.length > 1) {
      this.paths.push(this.currentPath);
    }
    this.currentPath = null;
    this.isDrawing = false;
  }

  build() {
    Column({ space: 16 }) {
      Text('Gesture Canvas')
        .fontSize(20)
        .fontWeight(FontWeight.Bold);

      Text('• Single finger to draw')
        .fontSize(12)
        .fontColor(Color.Gray);
      Text('• Two fingers to zoom/pan')
        .fontSize(12)
        .fontColor(Color.Gray);
      Text('• Double tap to reset')
        .fontSize(12)
        .fontColor(Color.Gray);
      Text('• Long press for menu')
        .fontSize(12)
        .fontColor(Color.Gray);

      Stack() {
        Canvas(this.canvas)
          .width(400)
          .height(400)
          .backgroundColor(Color.White)
          .border({ width: 2, color: Color.Gray })
          .onReady(() => {
            // Canvas ready
          })
          .gesture(
            // Use ArkUI gesture system integration
            PanGesture()
              .onActionStart((event) => {
                const touchEvent = this.createTouchEvent('touchstart', event);
                this.gestureRecognizer.processTouchEvent(touchEvent);
              })
              .onActionUpdate((event) => {
                const touchEvent = this.createTouchEvent('touchmove', event);
                this.gestureRecognizer.processTouchEvent(touchEvent);
              })
              .onActionEnd((event) => {
                const touchEvent = this.createTouchEvent('touchend', event);
                this.gestureRecognizer.processTouchEvent(touchEvent);
                this.finishDrawing();
              })
          );

        // Scale indicator
        Text(`${Math.round(this.canvasScale * 100)}%`)
          .fontSize(14)
          .fontColor(Color.White)
          .backgroundColor(Color.Black)
          .padding(8)
          .borderRadius(4)
          .position({ x: 10, y: 10 });
      }

      Row({ space: 12 }) {
        Button('Clear')
          .onClick(() => {
            this.paths = [];
            this.redrawCanvas();
          });

        Button('Undo')
          .onClick(() => {
            this.paths.pop();
            this.redrawCanvas();
          })
          .enabled(this.paths.length > 0);
      }
    }
    .width('100%')
    .height('100%')
    .padding(16);
  }

  private createTouchEvent(type: string, gestureEvent: any): TouchEvent {
    // Convert ArkUI gesture event to standard touch event
    return {
      type,
      touches: [{
        screenX: gestureEvent.fingerList?.[0]?.globalX || 0,
        screenY: gestureEvent.fingerList?.[0]?.globalY || 0
      }],
      target: null
    } as TouchEvent;
  }
}

interface DrawPath {
  points: { x: number; y: number }[];
  color: string;
  width: number;
  timestamp: number;
}
```

### Multi-Touch Interaction Component

```typescript
// Advanced multi-touch handling component
@Component
export struct MultiTouchDemo {
  @State private touchPoints: TouchPointVisual[] = [];
  @State private gestureInfo: string = 'No gesture';
  @State private currentScale: number = 1;
  @State private currentRotation: number = 0;

  private gestureRecognizer = new GestureRecognizer();
  private maxTouchPoints = 10;

  aboutToAppear() {
    this.setupMultiTouchGestures();
  }

  private setupMultiTouchGestures(): void {
    // Register various multi-touch gestures
    this.gestureRecognizer.registerGesture('multiTap', {
      type: GestureType.Tap,
      fingers: 1
    });

    this.gestureRecognizer.registerGesture('twoFingerTap', {
      type: GestureType.Tap,
      fingers: 2
    });

    this.gestureRecognizer.registerGesture('threeFingerTap', {
      type: GestureType.Tap,
      fingers: 3
    });

    this.gestureRecognizer.registerGesture('pinchZoom', {
      type: GestureType.Pinch,
      threshold: 0.05
    });

    this.gestureRecognizer.registerGesture('rotation', {
      type: GestureType.Rotation,
      threshold: 0.1
    });

    this.setupMultiTouchHandlers();
  }

  private setupMultiTouchHandlers(): void {
    this.gestureRecognizer.onGesture('multiTap', (event) => {
      this.addTouchPoint(event.position, '#FF6B6B');
      this.gestureInfo = 'Single Tap';
    });

    this.gestureRecognizer.onGesture('twoFingerTap', (event) => {
      this.gestureInfo = 'Two Finger Tap';
    });

    this.gestureRecognizer.onGesture('threeFingerTap', (event) => {
      this.gestureInfo = 'Three Finger Tap - Clear';
      this.clearTouchPoints();
    });

    this.gestureRecognizer.onGesture('pinchZoom', (event) => {
      if (event.scale) {
        this.currentScale = event.scale;
        this.gestureInfo = `Pinch Scale: ${event.scale.toFixed(2)}`;
      }
    });

    this.gestureRecognizer.onGesture('rotation', (event) => {
      if (event.rotation) {
        this.currentRotation = event.rotation * (180 / Math.PI);
        this.gestureInfo = `Rotation: ${this.currentRotation.toFixed(1)}°`;
      }
    });
  }

  private addTouchPoint(position: { x: number; y: number }, color: string): void {
    const touchPoint: TouchPointVisual = {
      id: Date.now().toString(),
      x: position.x,
      y: position.y,
      color,
      timestamp: Date.now(),
      size: 20
    };

    this.touchPoints.push(touchPoint);

    // Limit number of visible touch points
    if (this.touchPoints.length > this.maxTouchPoints) {
      this.touchPoints.shift();
    }

    // Auto-remove after delay
    setTimeout(() => {
      this.removeTouchPoint(touchPoint.id);
    }, 2000);
  }

  private removeTouchPoint(id: string): void {
    const index = this.touchPoints.findIndex(point => point.id === id);
    if (index > -1) {
      this.touchPoints.splice(index, 1);
    }
  }

  private clearTouchPoints(): void {
    this.touchPoints = [];
  }

  build() {
    Column({ space: 16 }) {
      Text('Multi-Touch Gesture Demo')
        .fontSize(24)
        .fontWeight(FontWeight.Bold);

      Text(this.gestureInfo)
        .fontSize(16)
        .fontColor(Color.Blue)
        .height(40);

      Row({ space: 20 }) {
        Text(`Scale: ${this.currentScale.toFixed(2)}`)
          .fontSize(14);
        Text(`Rotation: ${this.currentRotation.toFixed(1)}°`)
          .fontSize(14);
      }

      Stack() {
        // Touch area
        Column()
          .width('100%')
          .height(400)
          .backgroundColor('#f0f0f0')
          .border({ width: 2, color: Color.Gray, style: BorderStyle.Dashed })
          .gesture(
            // Multi-touch gesture handling
            GestureGroup(GestureMode.Parallel,
              TapGesture({ count: 1, fingers: 1 })
                .onAction((event) => {
                  const touchEvent = this.createTouchEventFromTap(event);
                  this.gestureRecognizer.processTouchEvent(touchEvent);
                }),

              TapGesture({ count: 1, fingers: 2 })
                .onAction((event) => {
                  this.gestureInfo = 'Two Finger Tap';
                }),

              TapGesture({ count: 1, fingers: 3 })
                .onAction((event) => {
                  this.gestureInfo = 'Three Finger Tap';
                  this.clearTouchPoints();
                }),

              PinchGesture()
                .onActionStart((event) => {
                  this.gestureInfo = 'Pinch Started';
                })
                .onActionUpdate((event) => {
                  this.currentScale = event.scale;
                  this.gestureInfo = `Pinching: ${event.scale.toFixed(2)}`;
                })
                .onActionEnd((event) => {
                  this.gestureInfo = `Pinch Ended: ${event.scale.toFixed(2)}`;
                }),

              RotationGesture()
                .onActionStart((event) => {
                  this.gestureInfo = 'Rotation Started';
                })
                .onActionUpdate((event) => {
                  this.currentRotation = event.angle;
                  this.gestureInfo = `Rotating: ${event.angle.toFixed(1)}°`;
                })
                .onActionEnd((event) => {
                  this.gestureInfo = `Rotation Ended: ${event.angle.toFixed(1)}°`;
                })
            )
          );

        // Visual touch points
        ForEach(this.touchPoints, (point: TouchPointVisual) => {
          Circle({ width: point.size, height: point.size })
            .fill(point.color)
            .position({ x: point.x - point.size / 2, y: point.y - point.size / 2 })
            .opacity(0.7)
            .animation({
              duration: 300,
              curve: Curve.EaseOut
            });
        }, (point: TouchPointVisual) => point.id);

        // Instructions overlay
        Column({ space: 8 }) {
          Text('Touch Gestures:')
            .fontSize(14)
            .fontWeight(FontWeight.Bold);
          Text('• Single tap: Add touch point')
            .fontSize(12);
          Text('• Two finger tap: Special action')
            .fontSize(12);
          Text('• Three finger tap: Clear all')
            .fontSize(12);
          Text('• Pinch: Scale gesture')
            .fontSize(12);
          Text('• Rotate: Rotation gesture')
            .fontSize(12);
        }
        .alignItems(HorizontalAlign.Start)
        .padding(12)
        .backgroundColor(Color.White)
        .borderRadius(8)
        .opacity(0.9)
        .position({ x: 10, y: 10 });
      }

      // Gesture statistics
      Row({ space: 20 }) {
        Text(`Touch Points: ${this.touchPoints.length}`)
          .fontSize(14);

        Button('Reset')
          .onClick(() => {
            this.clearTouchPoints();
            this.currentScale = 1;
            this.currentRotation = 0;
            this.gestureInfo = 'Reset';
          });
      }
    }
    .width('100%')
    .height('100%')
    .padding(16);
  }

  private createTouchEventFromTap(event: any): TouchEvent {
    const position = {
      x: event.fingerList[0]?.globalX || 0,
      y: event.fingerList[0]?.globalY || 0
    };

    return {
      type: 'touchend',
      touches: [{
        screenX: position.x,
        screenY: position.y
      }],
      target: null
    } as TouchEvent;
  }
}

interface TouchPointVisual {
  id: string;
  x: number;
  y: number;
  color: string;
  timestamp: number;
  size: number;
}
```

## Performance Optimization

### Gesture Performance Monitoring

```typescript
export class GesturePerformanceMonitor {
  private static instance: GesturePerformanceMonitor;
  private metrics: GestureMetrics[] = [];
  private isMonitoring: boolean = false;

  static getInstance(): GesturePerformanceMonitor {
    if (!this.instance) {
      this.instance = new GesturePerformanceMonitor();
    }
    return this.instance;
  }

  startMonitoring(): void {
    this.isMonitoring = true;
    this.metrics = [];
  }

  stopMonitoring(): void {
    this.isMonitoring = false;
  }

  recordGesture(
    gestureType: GestureType,
    processingTime: number,
    complexity: number
  ): void {
    if (!this.isMonitoring) return;

    this.metrics.push({
      gestureType,
      processingTime,
      complexity,
      timestamp: Date.now(),
    });
  }

  getAverageProcessingTime(gestureType?: GestureType): number {
    let filteredMetrics = this.metrics;

    if (gestureType) {
      filteredMetrics = this.metrics.filter(
        (m) => m.gestureType === gestureType
      );
    }

    if (filteredMetrics.length === 0) return 0;

    const totalTime = filteredMetrics.reduce(
      (sum, m) => sum + m.processingTime,
      0
    );
    return totalTime / filteredMetrics.length;
  }

  getPerformanceReport(): GesturePerformanceReport {
    const byType = new Map<GestureType, GestureMetrics[]>();

    this.metrics.forEach((metric) => {
      if (!byType.has(metric.gestureType)) {
        byType.set(metric.gestureType, []);
      }
      byType.get(metric.gestureType)!.push(metric);
    });

    const report: GesturePerformanceReport = {
      totalGestures: this.metrics.length,
      averageProcessingTime: this.getAverageProcessingTime(),
      gestureTypes: {},
      recommendations: [],
    };

    byType.forEach((metrics, gestureType) => {
      const avgTime =
        metrics.reduce((sum, m) => sum + m.processingTime, 0) / metrics.length;
      const maxTime = Math.max(...metrics.map((m) => m.processingTime));

      report.gestureTypes[gestureType] = {
        count: metrics.length,
        averageTime: avgTime,
        maxTime,
        complexity:
          metrics.reduce((sum, m) => sum + m.complexity, 0) / metrics.length,
      };

      if (avgTime > 16) {
        // > 16ms might impact 60fps
        report.recommendations.push(
          `${gestureType} gestures are taking too long (${avgTime.toFixed(
            1
          )}ms avg)`
        );
      }
    });

    return report;
  }
}

interface GestureMetrics {
  gestureType: GestureType;
  processingTime: number;
  complexity: number;
  timestamp: number;
}

interface GesturePerformanceReport {
  totalGestures: number;
  averageProcessingTime: number;
  gestureTypes: Record<
    string,
    {
      count: number;
      averageTime: number;
      maxTime: number;
      complexity: number;
    }
  >;
  recommendations: string[];
}
```

## Best Practices

1. **Touch Target Size**: Ensure interactive elements meet minimum touch target requirements
2. **Gesture Feedback**: Provide immediate visual feedback for gesture recognition
3. **Performance**: Optimize gesture processing to maintain smooth 60fps interactions
4. **Accessibility**: Support alternative input methods and assistive technologies
5. **Conflict Resolution**: Handle gesture conflicts gracefully with priority systems
6. **Cross-Platform**: Ensure gestures work consistently across different device types
7. **User Education**: Provide discoverable gesture hints and tutorials

## Conclusion

Advanced gesture recognition and touch event handling are essential for creating intuitive and responsive HarmonyOS applications. By implementing comprehensive gesture systems, optimizing performance, and following best practices, developers can create applications that feel natural and responsive to user interactions.

The key to successful gesture implementation lies in understanding user expectations, providing appropriate feedback, and maintaining consistent behavior across different interaction patterns and device capabilities.
