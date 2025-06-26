# 39. ArkUI Animation and Transition Systems

## Introduction

Animation and transitions are essential for creating engaging and intuitive user interfaces in HarmonyOS applications. This article explores ArkUI's comprehensive animation system, from basic transitions to complex choreographed animations and performance optimization techniques.

## Core Animation Concepts

### Animation Fundamentals

```typescript
// Custom Animation Controller
export class AnimationController {
  private animations: Map<string, any> = new Map();
  private timeScale: number = 1.0;
  private globalState: "running" | "paused" | "stopped" = "stopped";

  createAnimation(
    id: string,
    config: {
      duration: number;
      curve?: Curve;
      delay?: number;
      iterations?: number;
      playMode?: PlayMode;
    }
  ): AnimationController {
    this.animations.set(id, {
      ...config,
      state: "ready",
    });
    return this;
  }

  play(id?: string): AnimationController {
    if (id) {
      const animation = this.animations.get(id);
      if (animation) {
        animation.state = "playing";
      }
    } else {
      this.globalState = "running";
      this.animations.forEach((animation) => {
        animation.state = "playing";
      });
    }
    return this;
  }

  pause(id?: string): AnimationController {
    if (id) {
      const animation = this.animations.get(id);
      if (animation) {
        animation.state = "paused";
      }
    } else {
      this.globalState = "paused";
      this.animations.forEach((animation) => {
        animation.state = "paused";
      });
    }
    return this;
  }

  setTimeScale(scale: number): AnimationController {
    this.timeScale = scale;
    return this;
  }

  getAnimationState(id: string): string {
    return this.animations.get(id)?.state || "not found";
  }
}

// Advanced Tween System
export class TweenManager {
  private static instance: TweenManager;
  private tweens: Map<string, Tween> = new Map();
  private requestId: number = 0;

  static getInstance(): TweenManager {
    if (!this.instance) {
      this.instance = new TweenManager();
    }
    return this.instance;
  }

  createTween<T extends Record<string, number>>(
    id: string,
    target: T,
    to: Partial<T>,
    duration: number,
    easing: (t: number) => number = this.easeInOut
  ): Tween {
    const tween = new Tween(target, to, duration, easing);
    this.tweens.set(id, tween);
    return tween;
  }

  update(deltaTime: number): void {
    this.tweens.forEach((tween, id) => {
      if (tween.isActive()) {
        tween.update(deltaTime);
        if (tween.isComplete()) {
          this.tweens.delete(id);
        }
      }
    });
  }

  startUpdateLoop(): void {
    let lastTime = Date.now();

    const update = () => {
      const currentTime = Date.now();
      const deltaTime = currentTime - lastTime;
      lastTime = currentTime;

      this.update(deltaTime);

      if (this.tweens.size > 0) {
        this.requestId = requestAnimationFrame(update);
      }
    };

    update();
  }

  // Easing functions
  private easeInOut = (t: number): number => {
    return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
  };

  easeInQuad = (t: number): number => t * t;
  easeOutQuad = (t: number): number => t * (2 - t);
  easeInCubic = (t: number): number => t * t * t;
  easeOutCubic = (t: number): number => --t * t * t + 1;
}

class Tween {
  private startTime: number = 0;
  private isPlaying: boolean = false;
  private startValues: Record<string, number> = {};

  constructor(
    private target: Record<string, number>,
    private to: Record<string, number>,
    private duration: number,
    private easing: (t: number) => number
  ) {
    // Store initial values
    Object.keys(to).forEach((key) => {
      this.startValues[key] = target[key];
    });
  }

  start(): Tween {
    this.startTime = Date.now();
    this.isPlaying = true;
    return this;
  }

  update(deltaTime: number): void {
    if (!this.isPlaying) return;

    const elapsed = Date.now() - this.startTime;
    const progress = Math.min(elapsed / this.duration, 1);
    const easedProgress = this.easing(progress);

    Object.keys(this.to).forEach((key) => {
      const start = this.startValues[key];
      const end = this.to[key];
      this.target[key] = start + (end - start) * easedProgress;
    });

    if (progress >= 1) {
      this.isPlaying = false;
    }
  }

  isActive(): boolean {
    return this.isPlaying;
  }

  isComplete(): boolean {
    return !this.isPlaying && this.startTime > 0;
  }

  stop(): Tween {
    this.isPlaying = false;
    return this;
  }
}
```

### Advanced Animation Components

```typescript
@Component
export struct AnimatedContainer {
  @State private animationProgress: number = 0;
  @State private isAnimating: boolean = false;
  @State private containerWidth: number = 200;
  @State private containerHeight: number = 200;
  @State private rotation: number = 0;
  @State private opacity: number = 1;
  @State private scaleX: number = 1;
  @State private scaleY: number = 1;

  private animationController: AnimationController = new AnimationController();
  private tweenManager: TweenManager = TweenManager.getInstance();

  // Spring Animation Configuration
  private springConfig = {
    stiffness: 100,
    damping: 10,
    mass: 1
  };

  aboutToAppear() {
    this.setupAnimations();
  }

  private setupAnimations(): void {
    // Setup entrance animation
    this.animationController
      .createAnimation('entrance', {
        duration: 800,
        curve: Curve.EaseInOut,
        delay: 200
      })
      .createAnimation('pulse', {
        duration: 1200,
        curve: Curve.Linear,
        iterations: -1,
        playMode: PlayMode.Alternate
      });
  }

  private startComplexAnimation(): void {
    this.isAnimating = true;

    // Create coordinated animation sequence
    const animationSequence = [
      () => this.animateTransform(),
      () => this.animateColorGradient(),
      () => this.animateSpringBounce(),
      () => this.animateParticleEffect()
    ];

    this.executeSequence(animationSequence);
  }

  private async executeSequence(sequence: Array<() => void>): Promise<void> {
    for (const animation of sequence) {
      animation();
      await this.delay(300); // Stagger animations
    }
    this.isAnimating = false;
  }

  private animateTransform(): void {
    animateTo({
      duration: 1000,
      curve: curves.springMotion(0.6, 0.8),
      onFinish: () => {
        console.log('Transform animation completed');
      }
    }, () => {
      this.rotation = this.rotation === 0 ? 360 : 0;
      this.scaleX = this.scaleX === 1 ? 1.2 : 1;
      this.scaleY = this.scaleY === 1 ? 1.2 : 1;
    });
  }

  private animateColorGradient(): void {
    // Custom gradient animation using tween
    const gradientTween = this.tweenManager.createTween(
      'gradient',
      { hue: 0, saturation: 50, lightness: 50 },
      { hue: 360, saturation: 80, lightness: 70 },
      2000,
      this.tweenManager.easeInOutQuad
    );

    gradientTween.start();
    this.tweenManager.startUpdateLoop();
  }

  private animateSpringBounce(): void {
    animateTo({
      duration: 1500,
      curve: curves.springCurve(
        this.springConfig.stiffness,
        this.springConfig.damping,
        this.springConfig.mass
      )
    }, () => {
      this.containerHeight = this.containerHeight === 200 ? 150 : 200;
      this.containerWidth = this.containerWidth === 200 ? 250 : 200;
    });
  }

  private animateParticleEffect(): void {
    // Simulate particle animation with multiple child elements
    for (let i = 0; i < 5; i++) {
      setTimeout(() => {
        animateTo({
          duration: 800,
          delay: i * 100,
          curve: Curve.EaseOut
        }, () => {
          // Trigger particle animations
          this.opacity = this.opacity > 0.5 ? 0.3 : 1;
        });
      }, i * 150);
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  build() {
    Column({ space: 20 }) {
      Text('Advanced Animation Demo')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // Animated Container
      Stack() {
        // Background gradient
        Column()
          .width(this.containerWidth)
          .height(this.containerHeight)
          .linearGradient({
            angle: 45,
            colors: [
              [Color.Blue, 0],
              [Color.Purple, 0.5],
              [Color.Pink, 1]
            ]
          })
          .borderRadius(20)
          .animation({
            duration: 1000,
            curve: Curve.EaseInOut,
            iterations: 1,
            playMode: PlayMode.Normal
          })

        // Animated content
        Column({ space: 10 }) {
          Text('🚀')
            .fontSize(32)
            .rotate({ angle: this.rotation })
            .scale({ x: this.scaleX, y: this.scaleY })
            .animation({
              duration: 800,
              curve: curves.springMotion(0.5, 1.0)
            })

          Text('Animated')
            .fontSize(18)
            .fontColor(Color.White)
            .opacity(this.opacity)
            .animation({
              duration: 600,
              curve: Curve.EaseInOut
            })
        }
      }
      .width(this.containerWidth)
      .height(this.containerHeight)

      // Control buttons
      Row({ space: 15 }) {
        Button('Start Animation')
          .onClick(() => this.startComplexAnimation())
          .enabled(!this.isAnimating)
          .backgroundColor(Color.Blue)

        Button('Reset')
          .onClick(() => this.resetAnimation())
          .backgroundColor(Color.Gray)

        Button('Spring Test')
          .onClick(() => this.testSpringAnimation())
          .backgroundColor(Color.Green)
      }

      // Animation progress indicator
      if (this.isAnimating) {
        Progress({
          value: this.animationProgress * 100,
          total: 100,
          type: ProgressType.Linear
        })
          .width('80%')
          .color(Color.Blue)
      }
    }
    .width('100%')
    .height('100%')
    .padding(20)
    .justifyContent(FlexAlign.Center)
  }

  private resetAnimation(): void {
    this.rotation = 0;
    this.scaleX = 1;
    this.scaleY = 1;
    this.opacity = 1;
    this.containerWidth = 200;
    this.containerHeight = 200;
    this.animationProgress = 0;
  }

  private testSpringAnimation(): void {
    animateTo({
      duration: 2000,
      curve: curves.springCurve(150, 15, 1),
      onFinish: () => {
        // Bounce back
        animateTo({
          duration: 1000,
          curve: curves.springCurve(100, 8, 1)
        }, () => {
          this.scaleX = 1;
          this.scaleY = 1;
        });
      }
    }, () => {
      this.scaleX = 1.5;
      this.scaleY = 0.8;
    });
  }
}
```

## Complex Animation Patterns

### Page Transition Animations

```typescript
@Component
export struct PageTransitionManager {
  @State currentPage: number = 0;
  @State pageOffset: number = 0;
  @State isTransitioning: boolean = false;

  private pageWidth: number = 360;
  private pages: string[] = ['Home', 'Profile', 'Settings', 'About'];

  // Gesture handling for swipe transitions
  private panGesture = PanGesture({
    fingers: 1,
    direction: PanDirection.Horizontal,
    distance: 10
  })
    .onActionStart((event) => {
      this.isTransitioning = true;
    })
    .onActionUpdate((event) => {
      this.pageOffset = event.offsetX;
    })
    .onActionEnd((event) => {
      const threshold = this.pageWidth * 0.3;

      if (Math.abs(event.offsetX) > threshold) {
        if (event.offsetX > 0 && this.currentPage > 0) {
          this.navigateToPage(this.currentPage - 1);
        } else if (event.offsetX < 0 && this.currentPage < this.pages.length - 1) {
          this.navigateToPage(this.currentPage + 1);
        } else {
          this.snapToCurrentPage();
        }
      } else {
        this.snapToCurrentPage();
      }
    });

  private navigateToPage(pageIndex: number): void {
    const direction = pageIndex > this.currentPage ? -1 : 1;

    animateTo({
      duration: 400,
      curve: curves.cubicBezierCurve(0.25, 0.1, 0.25, 1),
      onFinish: () => {
        this.currentPage = pageIndex;
        this.pageOffset = 0;
        this.isTransitioning = false;
      }
    }, () => {
      this.pageOffset = direction * this.pageWidth;
    });
  }

  private snapToCurrentPage(): void {
    animateTo({
      duration: 300,
      curve: curves.springMotion(0.6, 0.8),
      onFinish: () => {
        this.isTransitioning = false;
      }
    }, () => {
      this.pageOffset = 0;
    });
  }

  // Custom page transition effects
  private getPageTransform(pageIndex: number): {
    translateX: number;
    scale: number;
    opacity: number;
    rotateY: number;
  } {
    const distance = pageIndex - this.currentPage;
    const offset = this.pageOffset / this.pageWidth;
    const totalDistance = distance - offset;

    // Card stack effect
    const translateX = totalDistance * this.pageWidth;
    const scale = 1 - Math.abs(totalDistance) * 0.1;
    const opacity = 1 - Math.abs(totalDistance) * 0.3;
    const rotateY = totalDistance * 15; // 3D rotation effect

    return { translateX, scale, opacity, rotateY };
  }

  build() {
    Column() {
      // Page indicator
      Row({ space: 8 }) {
        ForEach(this.pages, (page: string, index: number) => {
          Circle({ width: 8, height: 8 })
            .fill(index === this.currentPage ? Color.Blue : Color.Gray)
            .animation({
              duration: 300,
              curve: Curve.EaseInOut
            })
        }, (page: string, index: number) => `${page}-${index}`)
      }
      .margin({ bottom: 20 })

      // Pages container
      Stack() {
        ForEach(this.pages, (page: string, index: number) => {
          const transform = this.getPageTransform(index);

          PageContent({ title: page, index: index })
            .width(this.pageWidth)
            .height(500)
            .translate({
              x: transform.translateX,
              y: 0
            })
            .scale({
              x: transform.scale,
              y: transform.scale
            })
            .opacity(transform.opacity)
            .rotate({
              x: 0,
              y: 1,
              z: 0,
              angle: transform.rotateY
            })
            .animation({
              duration: this.isTransitioning ? 0 : 400,
              curve: curves.cubicBezierCurve(0.25, 0.1, 0.25, 1)
            })
        }, (page: string, index: number) => `${page}-${index}`)
      }
      .width('100%')
      .height(500)
      .gesture(this.panGesture)

      // Navigation buttons
      Row({ space: 20 }) {
        Button('Previous')
          .onClick(() => {
            if (this.currentPage > 0) {
              this.navigateToPage(this.currentPage - 1);
            }
          })
          .enabled(this.currentPage > 0 && !this.isTransitioning)

        Button('Next')
          .onClick(() => {
            if (this.currentPage < this.pages.length - 1) {
              this.navigateToPage(this.currentPage + 1);
            }
          })
          .enabled(this.currentPage < this.pages.length - 1 && !this.isTransitioning)
      }
      .margin({ top: 20 })
    }
    .width('100%')
    .height('100%')
    .padding(20)
  }
}

@Component
struct PageContent {
  @Prop title: string;
  @Prop index: number;

  build() {
    Column({ space: 20 }) {
      Text(this.title)
        .fontSize(32)
        .fontWeight(FontWeight.Bold)
        .fontColor(Color.White)

      Text(`Page ${this.index + 1}`)
        .fontSize(16)
        .fontColor(Color.White)
        .opacity(0.8)

      // Sample content
      Grid() {
        ForEach(Array.from({ length: 6 }, (_, i) => i), (item: number) => {
          GridItem() {
            Circle({ width: 40, height: 40 })
              .fill(Color.White)
              .opacity(0.3)
          }
        }, (item: number) => item.toString())
      }
      .columnsTemplate('1fr 1fr 1fr')
      .rowsTemplate('1fr 1fr')
      .columnsGap(10)
      .rowsGap(10)
      .width(200)
      .height(100)
    }
    .width('100%')
    .height('100%')
    .backgroundColor([
      Color.Red, Color.Green, Color.Blue, Color.Orange
    ][this.index])
    .borderRadius(20)
    .justifyContent(FlexAlign.Center)
  }
}
```

## Performance Optimization

### Animation Performance Monitor

```typescript
export class AnimationPerformanceMonitor {
  private static instance: AnimationPerformanceMonitor;
  private frameCount: number = 0;
  private lastTime: number = 0;
  private fps: number = 0;
  private frameDrops: number = 0;
  private isMonitoring: boolean = false;

  static getInstance(): AnimationPerformanceMonitor {
    if (!this.instance) {
      this.instance = new AnimationPerformanceMonitor();
    }
    return this.instance;
  }

  startMonitoring(): void {
    this.isMonitoring = true;
    this.lastTime = performance.now();
    this.monitorFrames();
  }

  stopMonitoring(): void {
    this.isMonitoring = false;
  }

  private monitorFrames(): void {
    if (!this.isMonitoring) return;

    const currentTime = performance.now();
    const deltaTime = currentTime - this.lastTime;

    this.frameCount++;

    // Calculate FPS every second
    if (deltaTime >= 1000) {
      this.fps = this.frameCount;
      this.frameCount = 0;
      this.lastTime = currentTime;

      // Detect frame drops (FPS < 50)
      if (this.fps < 50) {
        this.frameDrops++;
        console.warn(`Frame drop detected: ${this.fps} FPS`);
      }
    }

    requestAnimationFrame(() => this.monitorFrames());
  }

  getPerformanceMetrics(): {
    fps: number;
    frameDrops: number;
    recommendations: string[];
  } {
    const recommendations: string[] = [];

    if (this.fps < 30) {
      recommendations.push("Consider reducing animation complexity");
      recommendations.push(
        "Use transform animations instead of layout animations"
      );
    }

    if (this.frameDrops > 10) {
      recommendations.push("Optimize animation easing curves");
      recommendations.push("Reduce simultaneous animations");
    }

    return {
      fps: this.fps,
      frameDrops: this.frameDrops,
      recommendations,
    };
  }
}

// Optimized animation utilities
export class OptimizedAnimationUtils {
  // Use GPU-accelerated transforms
  static createGPUOptimizedAnimation(
    element: any,
    transformations: {
      translateX?: number;
      translateY?: number;
      scaleX?: number;
      scaleY?: number;
      rotate?: number;
    },
    duration: number = 300
  ): void {
    animateTo(
      {
        duration,
        curve: Curve.EaseInOut,
      },
      () => {
        // GPU-optimized properties
        Object.keys(transformations).forEach((key) => {
          element[key] = transformations[key];
        });
      }
    );
  }

  // Batch animation updates
  static batchAnimations(animations: Array<() => void>): void {
    requestAnimationFrame(() => {
      animations.forEach((animation) => animation());
    });
  }

  // Lazy animation loading
  static createLazyAnimation(
    trigger: () => boolean,
    animation: () => void,
    cleanup?: () => void
  ): () => void {
    let isSetup = false;

    const checkAndAnimate = () => {
      if (trigger() && !isSetup) {
        isSetup = true;
        animation();
      } else if (!trigger() && isSetup && cleanup) {
        isSetup = false;
        cleanup();
      }
    };

    return checkAndAnimate;
  }
}
```

## Best Practices

1. **Performance**: Use transform animations over layout changes for better performance
2. **GPU Acceleration**: Leverage GPU-accelerated properties (transform, opacity)
3. **Animation Curves**: Choose appropriate easing curves for natural motion
4. **Memory Management**: Clean up animation controllers and listeners
5. **User Experience**: Respect system accessibility settings for reduced motion
6. **Testing**: Monitor animation performance across different devices

## Conclusion

ArkUI's animation system provides powerful tools for creating engaging user interfaces. By understanding core animation concepts, implementing complex transitions, and optimizing for performance, developers can create smooth, responsive animations that enhance the user experience while maintaining excellent system performance.

The key to successful animation implementation lies in balancing visual appeal with performance considerations, ensuring animations feel natural and responsive across all device capabilities.
