# 33. Animations and Gestures

## Introduction

Creating fluid animations and responsive gesture interactions is essential for modern mobile applications. HarmonyOS provides powerful animation APIs and gesture recognition capabilities that enable developers to create engaging user experiences. This article covers the comprehensive implementation of animations and gesture handling in HarmonyOS applications.

## Animation Fundamentals

### Basic Property Animations

```typescript
@Component
export struct BasicAnimationExample {
  @State private translateX: number = 0;
  @State private opacity: number = 1;
  @State private scale: number = 1;
  @State private rotation: number = 0;

  private animateTranslation(): void {
    animateTo({
      duration: 1000,
      curve: Curve.EaseInOut,
      iterations: 1,
      playMode: PlayMode.Normal
    }, () => {
      this.translateX = this.translateX === 0 ? 200 : 0;
    });
  }

  private animateOpacity(): void {
    animateTo({
      duration: 800,
      curve: Curve.EaseIn,
    }, () => {
      this.opacity = this.opacity === 1 ? 0.3 : 1;
    });
  }

  private animateScale(): void {
    animateTo({
      duration: 600,
      curve: Curve.EaseOut,
    }, () => {
      this.scale = this.scale === 1 ? 1.5 : 1;
    });
  }

  private animateRotation(): void {
    animateTo({
      duration: 1200,
      curve: Curve.Linear,
    }, () => {
      this.rotation += 360;
    });
  }

  build() {
    Column({ space: 20 }) {
      Text('Animation Examples')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // Animated element
      Column() {
        Text('Animated Box')
          .fontSize(16)
          .fontColor(Color.White)
          .textAlign(TextAlign.Center)
      }
      .width(100)
      .height(100)
      .backgroundColor(Color.Blue)
      .borderRadius(10)
      .translate({ x: this.translateX })
      .opacity(this.opacity)
      .scale({ x: this.scale, y: this.scale })
      .rotate({ angle: this.rotation })

      // Control buttons
      Column({ space: 10 }) {
        Button('Translate')
          .onClick(() => this.animateTranslation())

        Button('Opacity')
          .onClick(() => this.animateOpacity())

        Button('Scale')
          .onClick(() => this.animateScale())

        Button('Rotate')
          .onClick(() => this.animateRotation())
      }
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

### Advanced Animation Controller

```typescript
export class AnimationController {
  private animationStates: Map<string, AnimationState> = new Map();
  private onAnimationComplete?: (animationId: string) => void;

  interface AnimationState {
    id: string;
    isRunning: boolean;
    startTime: number;
    duration: number;
    progress: number;
  }

  interface AnimationConfig {
    id: string;
    duration: number;
    curve: Curve;
    iterations?: number;
    playMode?: PlayMode;
    delay?: number;
    onComplete?: () => void;
  }

  startAnimation(config: AnimationConfig, animationCallback: () => void): void {
    const animationState: AnimationState = {
      id: config.id,
      isRunning: true,
      startTime: Date.now(),
      duration: config.duration,
      progress: 0
    };

    this.animationStates.set(config.id, animationState);

    // Add delay if specified
    const delay = config.delay || 0;

    setTimeout(() => {
      animateTo({
        duration: config.duration,
        curve: config.curve,
        iterations: config.iterations || 1,
        playMode: config.playMode || PlayMode.Normal,
        onFinish: () => {
          this.onAnimationFinish(config.id);
          config.onComplete?.();
        }
      }, animationCallback);
    }, delay);
  }

  private onAnimationFinish(animationId: string): void {
    const state = this.animationStates.get(animationId);
    if (state) {
      state.isRunning = false;
      state.progress = 1;
      this.onAnimationComplete?.(animationId);
    }
  }

  stopAnimation(animationId: string): void {
    const state = this.animationStates.get(animationId);
    if (state && state.isRunning) {
      state.isRunning = false;
      // Note: HarmonyOS doesn't have a direct stop method,
      // so we mark as finished
      this.onAnimationFinish(animationId);
    }
  }

  isAnimationRunning(animationId: string): boolean {
    const state = this.animationStates.get(animationId);
    return state ? state.isRunning : false;
  }

  getAnimationProgress(animationId: string): number {
    const state = this.animationStates.get(animationId);
    if (!state) return 0;

    if (!state.isRunning) return state.progress;

    const elapsed = Date.now() - state.startTime;
    return Math.min(elapsed / state.duration, 1);
  }

  setOnAnimationComplete(callback: (animationId: string) => void): void {
    this.onAnimationComplete = callback;
  }

  clearAllAnimations(): void {
    this.animationStates.clear();
  }
}
```

## Transition Animations

### Page Transition Manager

```typescript
export enum TransitionType {
  SLIDE_LEFT = 'slideLeft',
  SLIDE_RIGHT = 'slideRight',
  SLIDE_UP = 'slideUp',
  SLIDE_DOWN = 'slideDown',
  FADE = 'fade',
  SCALE = 'scale',
  ROTATE = 'rotate'
}

export class TransitionManager {
  private static instance: TransitionManager;
  private transitions: Map<string, TransitionConfig> = new Map();

  interface TransitionConfig {
    type: TransitionType;
    duration: number;
    curve: Curve;
    delay?: number;
  }

  static getInstance(): TransitionManager {
    if (!this.instance) {
      this.instance = new TransitionManager();
    }
    return this.instance;
  }

  registerTransition(name: string, config: TransitionConfig): void {
    this.transitions.set(name, config);
  }

  getTransition(name: string): TransitionConfig | undefined {
    return this.transitions.get(name);
  }

  createSlideTransition(direction: 'left' | 'right' | 'up' | 'down'): TransitionConfig {
    return {
      type: this.getSlideType(direction),
      duration: 300,
      curve: Curve.EaseInOut
    };
  }

  private getSlideType(direction: string): TransitionType {
    switch (direction) {
      case 'left': return TransitionType.SLIDE_LEFT;
      case 'right': return TransitionType.SLIDE_RIGHT;
      case 'up': return TransitionType.SLIDE_UP;
      case 'down': return TransitionType.SLIDE_DOWN;
      default: return TransitionType.SLIDE_LEFT;
    }
  }

  createFadeTransition(duration: number = 400): TransitionConfig {
    return {
      type: TransitionType.FADE,
      duration: duration,
      curve: Curve.EaseInOut
    };
  }

  createScaleTransition(duration: number = 350): TransitionConfig {
    return {
      type: TransitionType.SCALE,
      duration: duration,
      curve: Curve.EaseOut
    };
  }
}

@Component
export struct TransitionContainer {
  @State private isVisible: boolean = false;
  @State private translateX: number = 0;
  @State private translateY: number = 0;
  @State private opacity: number = 0;
  @State private scale: number = 0;
  @State private rotation: number = 0;

  private transitionType: TransitionType = TransitionType.FADE;
  private duration: number = 300;
  private curve: Curve = Curve.EaseInOut;

  @BuilderParam content: () => void;

  aboutToAppear() {
    this.initializeTransition();
  }

  private initializeTransition(): void {
    switch (this.transitionType) {
      case TransitionType.SLIDE_LEFT:
        this.translateX = -300;
        this.opacity = 1;
        break;
      case TransitionType.SLIDE_RIGHT:
        this.translateX = 300;
        this.opacity = 1;
        break;
      case TransitionType.SLIDE_UP:
        this.translateY = -300;
        this.opacity = 1;
        break;
      case TransitionType.SLIDE_DOWN:
        this.translateY = 300;
        this.opacity = 1;
        break;
      case TransitionType.FADE:
        this.opacity = 0;
        break;
      case TransitionType.SCALE:
        this.scale = 0;
        this.opacity = 1;
        break;
      case TransitionType.ROTATE:
        this.rotation = -180;
        this.opacity = 0;
        break;
    }
  }

  show(): void {
    this.isVisible = true;
    animateTo({
      duration: this.duration,
      curve: this.curve
    }, () => {
      this.translateX = 0;
      this.translateY = 0;
      this.opacity = 1;
      this.scale = 1;
      this.rotation = 0;
    });
  }

  hide(): void {
    animateTo({
      duration: this.duration,
      curve: this.curve,
      onFinish: () => {
        this.isVisible = false;
      }
    }, () => {
      this.initializeTransition();
    });
  }

  build() {
    if (this.isVisible) {
      Column() {
        this.content()
      }
      .translate({ x: this.translateX, y: this.translateY })
      .opacity(this.opacity)
      .scale({ x: this.scale, y: this.scale })
      .rotate({ angle: this.rotation })
    }
  }
}
```

## Gesture Recognition

### Basic Gesture Handlers

```typescript
@Component
export struct GestureExamples {
  @State private tapCount: number = 0;
  @State private panOffset: { x: number, y: number } = { x: 0, y: 0 };
  @State private scale: number = 1;
  @State private rotation: number = 0;
  @State private longPressTriggered: boolean = false;

  private handleTap(): void {
    this.tapCount++;
    console.log(`Tap detected: ${this.tapCount}`);
  }

  private handleDoubleTap(): void {
    console.log('Double tap detected');
    this.scale = this.scale === 1 ? 1.5 : 1;
  }

  private handlePanStart(event: GestureEvent): void {
    console.log('Pan start:', event.fingerList[0]);
  }

  private handlePanUpdate(event: GestureEvent): void {
    this.panOffset = {
      x: event.offsetX,
      y: event.offsetY
    };
  }

  private handlePanEnd(event: GestureEvent): void {
    console.log('Pan end');
    // Optionally animate back to original position
    animateTo({ duration: 300 }, () => {
      this.panOffset = { x: 0, y: 0 };
    });
  }

  private handlePinchUpdate(event: GestureEvent): void {
    this.scale = event.scale;
  }

  private handleRotationUpdate(event: GestureEvent): void {
    this.rotation = event.angle;
  }

  private handleLongPress(): void {
    this.longPressTriggered = true;
    console.log('Long press detected');

    // Reset after animation
    setTimeout(() => {
      this.longPressTriggered = false;
    }, 500);
  }

  build() {
    Column({ space: 20 }) {
      Text('Gesture Examples')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`Tap Count: ${this.tapCount}`)
        .fontSize(16)

      // Gesture target
      Column() {
        Text('Touch Me!')
          .fontSize(16)
          .fontColor(Color.White)
          .textAlign(TextAlign.Center)
      }
      .width(200)
      .height(200)
      .backgroundColor(this.longPressTriggered ? Color.Red : Color.Blue)
      .borderRadius(10)
      .translate({ x: this.panOffset.x, y: this.panOffset.y })
      .scale({ x: this.scale, y: this.scale })
      .rotate({ angle: this.rotation })
      .gesture(
        // Tap gesture
        TapGesture({ count: 1 })
          .onAction(() => this.handleTap())
      )
      .gesture(
        // Double tap gesture
        TapGesture({ count: 2 })
          .onAction(() => this.handleDoubleTap())
      )
      .gesture(
        // Pan gesture
        PanGesture()
          .onActionStart((event: GestureEvent) => this.handlePanStart(event))
          .onActionUpdate((event: GestureEvent) => this.handlePanUpdate(event))
          .onActionEnd((event: GestureEvent) => this.handlePanEnd(event))
      )
      .gesture(
        // Pinch gesture
        PinchGesture()
          .onActionUpdate((event: GestureEvent) => this.handlePinchUpdate(event))
      )
      .gesture(
        // Rotation gesture
        RotationGesture()
          .onActionUpdate((event: GestureEvent) => this.handleRotationUpdate(event))
      )
      .gesture(
        // Long press gesture
        LongPressGesture({ repeat: false })
          .onAction(() => this.handleLongPress())
      )

      // Reset button
      Button('Reset')
        .onClick(() => {
          this.tapCount = 0;
          this.panOffset = { x: 0, y: 0 };
          this.scale = 1;
          this.rotation = 0;
          this.longPressTriggered = false;
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

### Advanced Gesture Manager

```typescript
export enum GestureType {
  TAP = "tap",
  DOUBLE_TAP = "doubleTap",
  LONG_PRESS = "longPress",
  PAN = "pan",
  PINCH = "pinch",
  ROTATION = "rotation",
  SWIPE = "swipe",
}

export interface GestureConfig {
  type: GestureType;
  enabled: boolean;
  threshold?: number;
  duration?: number;
  direction?: PanDirection;
}

export interface GestureEventData {
  type: GestureType;
  position: { x: number; y: number };
  velocity?: { x: number; y: number };
  scale?: number;
  angle?: number;
  distance?: number;
  timestamp: number;
}

export class GestureManager {
  private gestureConfigs: Map<GestureType, GestureConfig> = new Map();
  private gestureCallbacks: Map<GestureType, (data: GestureEventData) => void> =
    new Map();
  private gestureHistory: GestureEventData[] = [];
  private maxHistorySize: number = 50;

  constructor() {
    this.initializeDefaultConfigs();
  }

  private initializeDefaultConfigs(): void {
    this.gestureConfigs.set(GestureType.TAP, {
      type: GestureType.TAP,
      enabled: true,
    });

    this.gestureConfigs.set(GestureType.DOUBLE_TAP, {
      type: GestureType.DOUBLE_TAP,
      enabled: true,
      duration: 300,
    });

    this.gestureConfigs.set(GestureType.LONG_PRESS, {
      type: GestureType.LONG_PRESS,
      enabled: true,
      duration: 500,
    });

    this.gestureConfigs.set(GestureType.PAN, {
      type: GestureType.PAN,
      enabled: true,
      threshold: 10,
    });

    this.gestureConfigs.set(GestureType.PINCH, {
      type: GestureType.PINCH,
      enabled: true,
      threshold: 0.1,
    });

    this.gestureConfigs.set(GestureType.ROTATION, {
      type: GestureType.ROTATION,
      enabled: true,
      threshold: 5,
    });

    this.gestureConfigs.set(GestureType.SWIPE, {
      type: GestureType.SWIPE,
      enabled: true,
      threshold: 100,
    });
  }

  configureGesture(type: GestureType, config: Partial<GestureConfig>): void {
    const existingConfig = this.gestureConfigs.get(type);
    if (existingConfig) {
      this.gestureConfigs.set(type, { ...existingConfig, ...config });
    }
  }

  setGestureCallback(
    type: GestureType,
    callback: (data: GestureEventData) => void
  ): void {
    this.gestureCallbacks.set(type, callback);
  }

  removeGestureCallback(type: GestureType): void {
    this.gestureCallbacks.delete(type);
  }

  private recordGesture(data: GestureEventData): void {
    this.gestureHistory.push(data);

    if (this.gestureHistory.length > this.maxHistorySize) {
      this.gestureHistory.shift();
    }
  }

  private triggerGestureCallback(data: GestureEventData): void {
    this.recordGesture(data);

    const callback = this.gestureCallbacks.get(data.type);
    if (callback) {
      callback(data);
    }
  }

  handleTap(event: GestureEvent): void {
    const config = this.gestureConfigs.get(GestureType.TAP);
    if (!config || !config.enabled) return;

    const data: GestureEventData = {
      type: GestureType.TAP,
      position: {
        x: event.fingerList[0].localX,
        y: event.fingerList[0].localY,
      },
      timestamp: Date.now(),
    };

    this.triggerGestureCallback(data);
  }

  handlePan(event: GestureEvent, phase: "start" | "update" | "end"): void {
    const config = this.gestureConfigs.get(GestureType.PAN);
    if (!config || !config.enabled) return;

    const data: GestureEventData = {
      type: GestureType.PAN,
      position: { x: event.offsetX, y: event.offsetY },
      velocity: { x: event.velocityX, y: event.velocityY },
      timestamp: Date.now(),
    };

    this.triggerGestureCallback(data);
  }

  handlePinch(event: GestureEvent): void {
    const config = this.gestureConfigs.get(GestureType.PINCH);
    if (!config || !config.enabled) return;

    if (Math.abs(event.scale - 1) < (config.threshold || 0.1)) return;

    const data: GestureEventData = {
      type: GestureType.PINCH,
      position: { x: event.pinchCenterX, y: event.pinchCenterY },
      scale: event.scale,
      timestamp: Date.now(),
    };

    this.triggerGestureCallback(data);
  }

  handleRotation(event: GestureEvent): void {
    const config = this.gestureConfigs.get(GestureType.ROTATION);
    if (!config || !config.enabled) return;

    if (Math.abs(event.angle) < (config.threshold || 5)) return;

    const data: GestureEventData = {
      type: GestureType.ROTATION,
      position: {
        x: event.fingerList[0].localX,
        y: event.fingerList[0].localY,
      },
      angle: event.angle,
      timestamp: Date.now(),
    };

    this.triggerGestureCallback(data);
  }

  detectSwipe(panData: GestureEventData[]): GestureEventData | null {
    if (panData.length < 2) return null;

    const config = this.gestureConfigs.get(GestureType.SWIPE);
    if (!config || !config.enabled) return null;

    const start = panData[0];
    const end = panData[panData.length - 1];

    const deltaX = end.position.x - start.position.x;
    const deltaY = end.position.y - start.position.y;
    const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);

    if (distance < (config.threshold || 100)) return null;

    const timeDelta = end.timestamp - start.timestamp;
    const velocity = distance / timeDelta;

    return {
      type: GestureType.SWIPE,
      position: start.position,
      velocity: { x: deltaX / timeDelta, y: deltaY / timeDelta },
      distance: distance,
      timestamp: Date.now(),
    };
  }

  getGestureHistory(type?: GestureType, limit?: number): GestureEventData[] {
    let history = this.gestureHistory;

    if (type) {
      history = history.filter((gesture) => gesture.type === type);
    }

    if (limit) {
      history = history.slice(-limit);
    }

    return [...history];
  }

  clearGestureHistory(): void {
    this.gestureHistory = [];
  }

  isGestureEnabled(type: GestureType): boolean {
    const config = this.gestureConfigs.get(type);
    return config ? config.enabled : false;
  }

  enableGesture(type: GestureType): void {
    this.configureGesture(type, { enabled: true });
  }

  disableGesture(type: GestureType): void {
    this.configureGesture(type, { enabled: false });
  }
}
```

## Physics-Based Animations

### Spring Animation System

```typescript
export interface SpringConfig {
  tension: number;
  friction: number;
  velocity: number;
  tolerance: number;
}

export class SpringAnimationSystem {
  private animations: Map<string, SpringAnimation> = new Map();

  interface SpringAnimation {
    id: string;
    from: number;
    to: number;
    current: number;
    velocity: number;
    config: SpringConfig;
    isRunning: boolean;
    onUpdate: (value: number) => void;
    onComplete?: () => void;
  }

  createSpringAnimation(
    id: string,
    from: number,
    to: number,
    config: SpringConfig,
    onUpdate: (value: number) => void,
    onComplete?: () => void
  ): void {
    const animation: SpringAnimation = {
      id,
      from,
      to,
      current: from,
      velocity: config.velocity,
      config,
      isRunning: false,
      onUpdate,
      onComplete
    };

    this.animations.set(id, animation);
  }

  startAnimation(id: string): void {
    const animation = this.animations.get(id);
    if (!animation) return;

    animation.isRunning = true;
    this.runSpringLoop(animation);
  }

  private runSpringLoop(animation: SpringAnimation): void {
    if (!animation.isRunning) return;

    const dt = 16; // ~60fps
    const { tension, friction, tolerance } = animation.config;

    // Spring physics calculations
    const displacement = animation.current - animation.to;
    const springForce = -tension * displacement;
    const dampingForce = -friction * animation.velocity;

    const acceleration = springForce + dampingForce;
    animation.velocity += acceleration * dt / 1000;
    animation.current += animation.velocity * dt / 1000;

    animation.onUpdate(animation.current);

    // Check if animation should continue
    if (Math.abs(displacement) < tolerance && Math.abs(animation.velocity) < tolerance) {
      animation.current = animation.to;
      animation.velocity = 0;
      animation.isRunning = false;
      animation.onUpdate(animation.current);
      animation.onComplete?.();
      return;
    }

    // Schedule next frame
    setTimeout(() => this.runSpringLoop(animation), dt);
  }

  stopAnimation(id: string): void {
    const animation = this.animations.get(id);
    if (animation) {
      animation.isRunning = false;
    }
  }

  removeAnimation(id: string): void {
    this.stopAnimation(id);
    this.animations.delete(id);
  }

  isAnimationRunning(id: string): boolean {
    const animation = this.animations.get(id);
    return animation ? animation.isRunning : false;
  }
}

@Component
export struct SpringAnimationExample {
  @State private translateY: number = 0;
  @State private scale: number = 1;

  private springSystem: SpringAnimationSystem = new SpringAnimationSystem();

  aboutToAppear() {
    // Setup spring animations
    this.springSystem.createSpringAnimation(
      'bounce',
      0,
      -100,
      { tension: 200, friction: 20, velocity: 0, tolerance: 0.1 },
      (value) => { this.translateY = value; }
    );

    this.springSystem.createSpringAnimation(
      'scale',
      1,
      1.2,
      { tension: 300, friction: 25, velocity: 0, tolerance: 0.01 },
      (value) => { this.scale = value; }
    );
  }

  private triggerBounce(): void {
    // Animate up first
    this.springSystem.createSpringAnimation(
      'bounceUp',
      this.translateY,
      -100,
      { tension: 200, friction: 15, velocity: 0, tolerance: 0.1 },
      (value) => { this.translateY = value; },
      () => {
        // Then animate back down
        this.springSystem.createSpringAnimation(
          'bounceDown',
          -100,
          0,
          { tension: 150, friction: 20, velocity: 0, tolerance: 0.1 },
          (value) => { this.translateY = value; }
        );
        this.springSystem.startAnimation('bounceDown');
      }
    );
    this.springSystem.startAnimation('bounceUp');
  }

  private triggerPulse(): void {
    this.springSystem.createSpringAnimation(
      'pulseUp',
      this.scale,
      1.3,
      { tension: 400, friction: 15, velocity: 0, tolerance: 0.01 },
      (value) => { this.scale = value; },
      () => {
        this.springSystem.createSpringAnimation(
          'pulseDown',
          1.3,
          1,
          { tension: 300, friction: 20, velocity: 0, tolerance: 0.01 },
          (value) => { this.scale = value; }
        );
        this.springSystem.startAnimation('pulseDown');
      }
    );
    this.springSystem.startAnimation('pulseUp');
  }

  build() {
    Column({ space: 30 }) {
      Text('Spring Animations')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // Animated element
      Column() {
        Text('Spring Box')
          .fontSize(16)
          .fontColor(Color.White)
      }
      .width(100)
      .height(100)
      .backgroundColor(Color.Green)
      .borderRadius(10)
      .translate({ y: this.translateY })
      .scale({ x: this.scale, y: this.scale })

      // Control buttons
      Row({ space: 20 }) {
        Button('Bounce')
          .onClick(() => this.triggerBounce())

        Button('Pulse')
          .onClick(() => this.triggerPulse())
      }
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

## Advanced Interaction Patterns

### Swipe-to-Dismiss Component

```typescript
@Component
export struct SwipeToDismissCard {
  @State private translateX: number = 0;
  @State private opacity: number = 1;
  @State private isDismissed: boolean = false;

  @Prop cardData: any;
  @Prop onDismiss: (data: any) => void;

  private dismissThreshold: number = 120;
  private animationDuration: number = 300;

  private handlePanStart(event: GestureEvent): void {
    // Optional: Add visual feedback when pan starts
  }

  private handlePanUpdate(event: GestureEvent): void {
    if (this.isDismissed) return;

    this.translateX = event.offsetX;

    // Calculate opacity based on swipe distance
    const progress = Math.abs(this.translateX) / this.dismissThreshold;
    this.opacity = Math.max(0.3, 1 - progress * 0.7);
  }

  private handlePanEnd(event: GestureEvent): void {
    if (this.isDismissed) return;

    const shouldDismiss = Math.abs(this.translateX) > this.dismissThreshold;

    if (shouldDismiss) {
      this.dismiss();
    } else {
      this.snapBack();
    }
  }

  private dismiss(): void {
    const direction = this.translateX > 0 ? 1 : -1;
    const targetX = direction * 400; // Move off screen

    animateTo({
      duration: this.animationDuration,
      curve: Curve.EaseOut,
      onFinish: () => {
        this.isDismissed = true;
        this.onDismiss(this.cardData);
      }
    }, () => {
      this.translateX = targetX;
      this.opacity = 0;
    });
  }

  private snapBack(): void {
    animateTo({
      duration: this.animationDuration,
      curve: Curve.EaseOut
    }, () => {
      this.translateX = 0;
      this.opacity = 1;
    });
  }

  build() {
    if (!this.isDismissed) {
      Column() {
        Text(this.cardData.title)
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .margin({ bottom: 10 })

        Text(this.cardData.content)
          .fontSize(14)
          .fontColor(Color.Grey)
      }
      .width('90%')
      .padding(20)
      .backgroundColor(Color.White)
      .borderRadius(10)
      .shadow({ radius: 5, color: '#00000020' })
      .translate({ x: this.translateX })
      .opacity(this.opacity)
      .gesture(
        PanGesture({ direction: PanDirection.Horizontal })
          .onActionStart((event: GestureEvent) => this.handlePanStart(event))
          .onActionUpdate((event: GestureEvent) => this.handlePanUpdate(event))
          .onActionEnd((event: GestureEvent) => this.handlePanEnd(event))
      )
    }
  }
}
```

### Pull-to-Refresh Component

```typescript
@Component
export struct PullToRefreshContainer {
  @State private refreshOffset: number = 0;
  @State private isRefreshing: boolean = false;
  @State private refreshProgress: number = 0;

  @Prop onRefresh: () => Promise<void>;
  @BuilderParam content: () => void;

  private refreshThreshold: number = 80;
  private maxPullDistance: number = 120;

  private handlePanUpdate(event: GestureEvent): void {
    if (this.isRefreshing) return;

    // Only allow downward pull at the top
    if (event.offsetY > 0) {
      this.refreshOffset = Math.min(event.offsetY, this.maxPullDistance);
      this.refreshProgress = this.refreshOffset / this.refreshThreshold;
    }
  }

  private async handlePanEnd(event: GestureEvent): Promise<void> {
    if (this.isRefreshing) return;

    if (this.refreshOffset >= this.refreshThreshold) {
      await this.triggerRefresh();
    } else {
      this.snapBack();
    }
  }

  private async triggerRefresh(): Promise<void> {
    this.isRefreshing = true;
    this.refreshProgress = 1;

    // Animate to refresh position
    animateTo({ duration: 200 }, () => {
      this.refreshOffset = this.refreshThreshold;
    });

    try {
      await this.onRefresh();
    } finally {
      this.completeRefresh();
    }
  }

  private completeRefresh(): void {
    animateTo({
      duration: 300,
      onFinish: () => {
        this.isRefreshing = false;
        this.refreshProgress = 0;
      }
    }, () => {
      this.refreshOffset = 0;
    });
  }

  private snapBack(): void {
    animateTo({ duration: 200 }, () => {
      this.refreshOffset = 0;
      this.refreshProgress = 0;
    });
  }

  @Builder
  refreshIndicator() {
    Column() {
      if (this.isRefreshing) {
        LoadingProgress()
          .width(30)
          .height(30)
          .color(Color.Blue)
      } else {
        Image($r('app.media.arrow_down'))
          .width(24)
          .height(24)
          .rotate({ angle: this.refreshProgress >= 1 ? 180 : 0 })
          .opacity(this.refreshProgress)
      }

      Text(this.isRefreshing ? 'Refreshing...' :
           this.refreshProgress >= 1 ? 'Release to refresh' : 'Pull to refresh')
        .fontSize(12)
        .fontColor(Color.Grey)
        .margin({ top: 5 })
    }
    .height(this.refreshOffset)
    .justifyContent(FlexAlign.Center)
    .opacity(this.refreshOffset > 0 ? 1 : 0)
  }

  build() {
    Column() {
      this.refreshIndicator()

      Column() {
        this.content()
      }
      .translate({ y: this.refreshOffset })
      .gesture(
        PanGesture({ direction: PanDirection.Vertical })
          .onActionUpdate((event: GestureEvent) => this.handlePanUpdate(event))
          .onActionEnd((event: GestureEvent) => this.handlePanEnd(event))
      )
    }
    .width('100%')
    .height('100%')
  }
}
```

## Motion Design Patterns

### Staggered Animation Manager

```typescript
export class StaggeredAnimationManager {
  private animations: StaggeredAnimation[] = [];
  private isRunning: boolean = false;

  interface StaggeredAnimation {
    id: string;
    delay: number;
    duration: number;
    curve: Curve;
    animationCallback: () => void;
    onComplete?: () => void;
  }

  addAnimation(
    id: string,
    delay: number,
    duration: number,
    curve: Curve,
    animationCallback: () => void,
    onComplete?: () => void
  ): void {
    this.animations.push({
      id,
      delay,
      duration,
      curve,
      animationCallback,
      onComplete
    });
  }

  startStaggeredAnimations(baseDelay: number = 100): void {
    if (this.isRunning) return;

    this.isRunning = true;

    this.animations.forEach((animation, index) => {
      const totalDelay = animation.delay + (index * baseDelay);

      setTimeout(() => {
        animateTo({
          duration: animation.duration,
          curve: animation.curve,
          onFinish: () => {
            animation.onComplete?.();

            // Check if this is the last animation
            if (index === this.animations.length - 1) {
              this.isRunning = false;
            }
          }
        }, animation.animationCallback);
      }, totalDelay);
    });
  }

  clearAnimations(): void {
    this.animations = [];
    this.isRunning = false;
  }

  isAnimating(): boolean {
    return this.isRunning;
  }
}

@Component
export struct StaggeredListExample {
  @State private itemStates: boolean[] = [];
  private staggerManager: StaggeredAnimationManager = new StaggeredAnimationManager();
  private itemCount: number = 6;

  aboutToAppear() {
    this.itemStates = new Array(this.itemCount).fill(false);
    this.setupStaggeredAnimations();
  }

  private setupStaggeredAnimations(): void {
    for (let i = 0; i < this.itemCount; i++) {
      this.staggerManager.addAnimation(
        `item_${i}`,
        0,
        400,
        Curve.EaseOut,
        () => {
          this.itemStates[i] = true;
        }
      );
    }
  }

  private startAnimations(): void {
    // Reset states
    this.itemStates = new Array(this.itemCount).fill(false);

    // Start staggered animations
    this.staggerManager.startStaggeredAnimations(150);
  }

  build() {
    Column({ space: 20 }) {
      Text('Staggered Animations')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Button('Start Animation')
        .onClick(() => this.startAnimations())
        .enabled(!this.staggerManager.isAnimating())

      Column({ space: 10 }) {
        ForEach(Array.from({ length: this.itemCount }, (_, i) => i), (index: number) => {
          Row() {
            Text(`Item ${index + 1}`)
              .fontSize(16)
              .fontColor(Color.White)
          }
          .width('80%')
          .height(50)
          .backgroundColor(Color.Blue)
          .borderRadius(8)
          .justifyContent(FlexAlign.Center)
          .opacity(this.itemStates[index] ? 1 : 0)
          .translate({ x: this.itemStates[index] ? 0 : -100 })
        })
      }
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
  }
}
```

## Best Practices

### 1. Performance Optimization

- Use `animateTo` for state-based animations
- Avoid animating expensive properties (avoid layout changes when possible)
- Batch multiple property changes in a single animation call
- Use appropriate animation curves for natural motion

### 2. Gesture Handling

- Implement proper gesture conflict resolution
- Provide visual feedback for gesture interactions
- Consider accessibility when implementing custom gestures
- Handle edge cases and gesture cancellation

### 3. Animation Timing

- Follow platform design guidelines for animation durations
- Use consistent easing curves throughout your app
- Implement proper cleanup for animation resources
- Consider reduced motion accessibility settings

### 4. User Experience

- Provide clear visual feedback for interactive elements
- Ensure animations enhance rather than hinder usability
- Implement proper loading states and progress indicators
- Test animations on different device performance levels

## Conclusion

Animations and gestures are fundamental to creating engaging HarmonyOS applications. By implementing proper animation systems, gesture recognition, and interaction patterns, you can create fluid and responsive user experiences. Remember to consider performance implications, accessibility requirements, and platform design guidelines when implementing animations and gestures in your applications.

The examples provided demonstrate various approaches to animation and gesture handling, from basic property animations to complex physics-based systems. Choose the appropriate techniques based on your specific use cases and performance requirements.
