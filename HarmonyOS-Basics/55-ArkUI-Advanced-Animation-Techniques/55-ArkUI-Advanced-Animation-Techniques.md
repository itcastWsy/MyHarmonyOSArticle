# ArkUI Advanced Animation Techniques

## Introduction

ArkUI provides powerful animation capabilities for creating engaging user experiences. This guide explores advanced animation techniques, performance optimization, and complex interactive animations.

## Core Animation System

### Custom Animation Framework

```typescript
enum AnimationType {
  Transform = "transform",
  Opacity = "opacity",
  Color = "color",
  Size = "size",
  Position = "position",
}

enum EasingType {
  Linear = "linear",
  EaseIn = "ease-in",
  EaseOut = "ease-out",
  EaseInOut = "ease-in-out",
  Bounce = "bounce",
  Elastic = "elastic",
}

interface AnimationConfig {
  duration: number;
  delay?: number;
  easing: EasingType;
  repeat?: number;
  alternate?: boolean;
  fillMode?: "forwards" | "backwards" | "both";
}

interface KeyFrame {
  time: number; // 0-1
  value: any;
  easing?: EasingType;
}

class AnimationController {
  private animationId: number = 0;
  private runningAnimations = new Map<number, Animation>();
  private timeline: AnimationTimeline;

  constructor() {
    this.timeline = new AnimationTimeline();
  }

  animate(
    target: any,
    property: string,
    keyframes: KeyFrame[],
    config: AnimationConfig
  ): number {
    const id = ++this.animationId;

    const animation = new CustomAnimation(
      target,
      property,
      keyframes,
      config,
      () => this.onAnimationComplete(id)
    );

    this.runningAnimations.set(id, animation);
    animation.start();

    return id;
  }

  animateSequence(animations: AnimationSequenceItem[]): Promise<void> {
    return this.timeline.playSequence(animations);
  }

  animateParallel(animations: AnimationSequenceItem[]): Promise<void> {
    return this.timeline.playParallel(animations);
  }

  pauseAnimation(id: number): void {
    const animation = this.runningAnimations.get(id);
    animation?.pause();
  }

  resumeAnimation(id: number): void {
    const animation = this.runningAnimations.get(id);
    animation?.resume();
  }

  stopAnimation(id: number): void {
    const animation = this.runningAnimations.get(id);
    if (animation) {
      animation.stop();
      this.runningAnimations.delete(id);
    }
  }

  private onAnimationComplete(id: number): void {
    this.runningAnimations.delete(id);
  }
}

class CustomAnimation {
  private startTime: number = 0;
  private pausedTime: number = 0;
  private isPlaying: boolean = false;
  private isPaused: boolean = false;
  private rafId: number = 0;

  constructor(
    private target: any,
    private property: string,
    private keyframes: KeyFrame[],
    private config: AnimationConfig,
    private onComplete: () => void
  ) {}

  start(): void {
    this.startTime = performance.now();
    this.isPlaying = true;
    this.tick();
  }

  pause(): void {
    if (this.isPlaying && !this.isPaused) {
      this.isPaused = true;
      this.pausedTime = performance.now();
      cancelAnimationFrame(this.rafId);
    }
  }

  resume(): void {
    if (this.isPaused) {
      this.isPaused = false;
      this.startTime += performance.now() - this.pausedTime;
      this.tick();
    }
  }

  stop(): void {
    this.isPlaying = false;
    cancelAnimationFrame(this.rafId);
  }

  private tick(): void {
    if (!this.isPlaying || this.isPaused) return;

    const currentTime = performance.now();
    const elapsed = currentTime - this.startTime;
    const progress = Math.min(elapsed / this.config.duration, 1);

    const easedProgress = this.applyEasing(progress);
    const value = this.interpolateKeyframes(easedProgress);

    this.updateTarget(value);

    if (progress >= 1) {
      this.handleAnimationEnd();
    } else {
      this.rafId = requestAnimationFrame(() => this.tick());
    }
  }

  private interpolateKeyframes(progress: number): any {
    if (this.keyframes.length === 0) return null;
    if (this.keyframes.length === 1) return this.keyframes[0].value;

    // Find surrounding keyframes
    let prevFrame = this.keyframes[0];
    let nextFrame = this.keyframes[this.keyframes.length - 1];

    for (let i = 0; i < this.keyframes.length - 1; i++) {
      if (
        progress >= this.keyframes[i].time &&
        progress <= this.keyframes[i + 1].time
      ) {
        prevFrame = this.keyframes[i];
        nextFrame = this.keyframes[i + 1];
        break;
      }
    }

    // Calculate local progress between keyframes
    const localProgress =
      (progress - prevFrame.time) / (nextFrame.time - prevFrame.time);

    return this.interpolateValues(
      prevFrame.value,
      nextFrame.value,
      localProgress
    );
  }

  private interpolateValues(start: any, end: any, progress: number): any {
    if (typeof start === "number" && typeof end === "number") {
      return start + (end - start) * progress;
    }

    if (typeof start === "string" && typeof end === "string") {
      // Handle color interpolation
      if (this.isColor(start) && this.isColor(end)) {
        return this.interpolateColor(start, end, progress);
      }
    }

    return progress >= 0.5 ? end : start;
  }

  private interpolateColor(
    start: string,
    end: string,
    progress: number
  ): string {
    const startRgb = this.parseColor(start);
    const endRgb = this.parseColor(end);

    const r = Math.round(startRgb.r + (endRgb.r - startRgb.r) * progress);
    const g = Math.round(startRgb.g + (endRgb.g - startRgb.g) * progress);
    const b = Math.round(startRgb.b + (endRgb.b - startRgb.b) * progress);

    return `rgb(${r}, ${g}, ${b})`;
  }

  private applyEasing(progress: number): number {
    switch (this.config.easing) {
      case EasingType.Linear:
        return progress;
      case EasingType.EaseIn:
        return progress * progress;
      case EasingType.EaseOut:
        return 1 - Math.pow(1 - progress, 2);
      case EasingType.EaseInOut:
        return progress < 0.5
          ? 2 * progress * progress
          : 1 - Math.pow(-2 * progress + 2, 2) / 2;
      case EasingType.Bounce:
        return this.bounceEasing(progress);
      case EasingType.Elastic:
        return this.elasticEasing(progress);
      default:
        return progress;
    }
  }

  private bounceEasing(t: number): number {
    if (t < 1 / 2.75) {
      return 7.5625 * t * t;
    } else if (t < 2 / 2.75) {
      return 7.5625 * (t -= 1.5 / 2.75) * t + 0.75;
    } else if (t < 2.5 / 2.75) {
      return 7.5625 * (t -= 2.25 / 2.75) * t + 0.9375;
    } else {
      return 7.5625 * (t -= 2.625 / 2.75) * t + 0.984375;
    }
  }

  private elasticEasing(t: number): number {
    return t === 0
      ? 0
      : t === 1
      ? 1
      : -Math.pow(2, 10 * (t - 1)) * Math.sin((t - 1.1) * 5 * Math.PI);
  }
}
```

### Physics-based Animations

```typescript
interface PhysicsConfig {
  mass: number;
  stiffness: number;
  damping: number;
  precision: number;
}

class SpringAnimation {
  private position: number = 0;
  private velocity: number = 0;
  private isRunning: boolean = false;

  constructor(
    private target: number,
    private config: PhysicsConfig,
    private onUpdate: (value: number) => void,
    private onComplete: () => void
  ) {}

  start(initialPosition: number = 0): void {
    this.position = initialPosition;
    this.velocity = 0;
    this.isRunning = true;
    this.tick();
  }

  stop(): void {
    this.isRunning = false;
  }

  private tick(): void {
    if (!this.isRunning) return;

    const dt = 16 / 1000; // 60fps

    // Spring physics calculation
    const force = -this.config.stiffness * (this.position - this.target);
    const damping = -this.config.damping * this.velocity;
    const acceleration = (force + damping) / this.config.mass;

    this.velocity += acceleration * dt;
    this.position += this.velocity * dt;

    this.onUpdate(this.position);

    // Check if animation should stop
    const distance = Math.abs(this.position - this.target);
    const speed = Math.abs(this.velocity);

    if (distance < this.config.precision && speed < this.config.precision) {
      this.position = this.target;
      this.onUpdate(this.position);
      this.onComplete();
      return;
    }

    requestAnimationFrame(() => this.tick());
  }
}

// Gesture-driven animations
class GestureAnimationController {
  private dragAnimation: SpringAnimation | null = null;
  private panGesture: PanGesture | null = null;

  createDraggableElement(
    element: any,
    bounds: { x: [number, number]; y: [number, number] }
  ): void {
    let startPosition = { x: 0, y: 0 };
    let currentPosition = { x: 0, y: 0 };

    this.panGesture = PanGesture({ fingers: 1 })
      .onActionStart((event: GestureEvent) => {
        startPosition = { x: currentPosition.x, y: currentPosition.y };
      })
      .onActionUpdate((event: GestureEvent) => {
        const newX = this.clamp(
          startPosition.x + event.offsetX,
          bounds.x[0],
          bounds.x[1]
        );
        const newY = this.clamp(
          startPosition.y + event.offsetY,
          bounds.y[0],
          bounds.y[1]
        );

        currentPosition = { x: newX, y: newY };
        this.updateElementPosition(element, currentPosition);
      })
      .onActionEnd((event: GestureEvent) => {
        this.handleDragEnd(
          element,
          currentPosition,
          event.velocityX,
          event.velocityY,
          bounds
        );
      });
  }

  private handleDragEnd(
    element: any,
    position: { x: number; y: number },
    velocityX: number,
    velocityY: number,
    bounds: { x: [number, number]; y: [number, number] }
  ): void {
    // Calculate momentum and snap to bounds
    const momentumX = position.x + velocityX * 0.3;
    const momentumY = position.y + velocityY * 0.3;

    const targetX = this.clamp(momentumX, bounds.x[0], bounds.x[1]);
    const targetY = this.clamp(momentumY, bounds.y[0], bounds.y[1]);

    // Animate to final position
    this.animateToPosition(element, { x: targetX, y: targetY });
  }

  private animateToPosition(
    element: any,
    target: { x: number; y: number }
  ): void {
    const springConfig: PhysicsConfig = {
      mass: 1,
      stiffness: 100,
      damping: 10,
      precision: 0.01,
    };

    // Animate X
    new SpringAnimation(
      target.x,
      springConfig,
      (value) => this.updateElementPosition(element, { x: value, y: target.y }),
      () => {}
    ).start(element.position.x);

    // Animate Y
    new SpringAnimation(
      target.y,
      springConfig,
      (value) => this.updateElementPosition(element, { x: target.x, y: value }),
      () => {}
    ).start(element.position.y);
  }

  private clamp(value: number, min: number, max: number): number {
    return Math.min(Math.max(value, min), max);
  }

  private updateElementPosition(
    element: any,
    position: { x: number; y: number }
  ): void {
    element.position = position;
    // Update UI element position
  }
}
```

## Complex Animation Components

### Morphing Animations

```typescript
@Component
struct MorphingCard {
  @State private isExpanded: boolean = false
  @State private animationProgress: number = 0
  @Prop content: CardContent

  build() {
    Column() {
      this.buildCard()
    }
    .animation({
      duration: 600,
      curve: Curve.FastOutSlowIn
    })
  }

  @Builder
  buildCard() {
    Column() {
      // Header section
      Row() {
        this.buildCardHeader()
      }
      .height(this.getHeaderHeight())
      .padding(16)

      // Expandable content
      if (this.isExpanded) {
        Column() {
          this.buildExpandedContent()
        }
        .height(this.getContentHeight())
        .padding({ horizontal: 16, bottom: 16 })
        .transition({
          type: TransitionType.Insert,
          opacity: 0,
          translate: { y: -20 }
        })
      }
    }
    .width(this.getCardWidth())
    .backgroundColor(this.getCardColor())
    .borderRadius(this.getBorderRadius())
    .shadow({
      radius: this.getShadowRadius(),
      color: this.getShadowColor(),
      offsetX: 0,
      offsetY: this.getShadowOffset()
    })
    .onClick(() => this.toggleExpansion())
  }

  @Builder
  buildCardHeader() {
    Image(this.content.imageUrl)
      .width(this.getImageSize())
      .height(this.getImageSize())
      .borderRadius(this.getImageBorderRadius())
      .objectFit(ImageFit.Cover)

    Column() {
      Text(this.content.title)
        .fontSize(this.getTitleFontSize())
        .fontWeight(FontWeight.Bold)
        .maxLines(this.getTitleMaxLines())

      Text(this.content.subtitle)
        .fontSize(this.getSubtitleFontSize())
        .fontColor('#666')
        .maxLines(2)
        .opacity(this.getSubtitleOpacity())
    }
    .alignItems(HorizontalAlign.Start)
    .layoutWeight(1)
    .margin({ left: 12 })

    Image('/resources/chevron_down.png')
      .width(24)
      .height(24)
      .rotate({ angle: this.isExpanded ? 180 : 0 })
  }

  @Builder
  buildExpandedContent() {
    Text(this.content.description)
      .fontSize(14)
      .lineHeight(20)
      .margin({ bottom: 16 })

    Row() {
      ForEach(this.content.tags, (tag: string) => {
        Text(tag)
          .fontSize(12)
          .padding({ horizontal: 8, vertical: 4 })
          .backgroundColor('#f0f0f0')
          .borderRadius(12)
          .margin({ right: 8 })
      })
    }
    .margin({ bottom: 16 })

    Button('Learn More')
      .width('100%')
      .backgroundColor('#007AFF')
  }

  private toggleExpansion(): void {
    animateTo(
      {
        duration: 600,
        curve: Curve.FastOutSlowIn
      },
      () => {
        this.isExpanded = !this.isExpanded
      }
    )
  }

  // Interpolated values based on expansion state
  private getCardWidth(): string {
    return this.isExpanded ? '100%' : '90%'
  }

  private getHeaderHeight(): number {
    return this.isExpanded ? 80 : 60
  }

  private getContentHeight(): number {
    return this.isExpanded ? 200 : 0
  }

  private getImageSize(): number {
    return this.isExpanded ? 60 : 40
  }

  private getTitleFontSize(): number {
    return this.isExpanded ? 18 : 16
  }

  private getSubtitleFontSize(): number {
    return this.isExpanded ? 14 : 12
  }

  private getSubtitleOpacity(): number {
    return this.isExpanded ? 1 : 0.7
  }

  private getBorderRadius(): number {
    return this.isExpanded ? 16 : 12
  }

  private getShadowRadius(): number {
    return this.isExpanded ? 20 : 10
  }

  private getShadowOffset(): number {
    return this.isExpanded ? 8 : 4
  }
}

interface CardContent {
  imageUrl: string
  title: string
  subtitle: string
  description: string
  tags: string[]
}
```

### Particle System

```typescript
interface Particle {
  x: number
  y: number
  velocityX: number
  velocityY: number
  life: number
  maxLife: number
  size: number
  color: string
  opacity: number
}

class ParticleSystem {
  private particles: Particle[] = []
  private canvas: Canvas
  private isRunning: boolean = false
  private lastTime: number = 0

  constructor(canvas: Canvas) {
    this.canvas = canvas
  }

  start(): void {
    this.isRunning = true
    this.lastTime = performance.now()
    this.tick()
  }

  stop(): void {
    this.isRunning = false
  }

  emit(config: EmissionConfig): void {
    for (let i = 0; i < config.count; i++) {
      const particle: Particle = {
        x: config.x + (Math.random() - 0.5) * config.spread,
        y: config.y + (Math.random() - 0.5) * config.spread,
        velocityX: (Math.random() - 0.5) * config.velocity,
        velocityY: (Math.random() - 0.5) * config.velocity,
        life: 0,
        maxLife: config.life + Math.random() * config.lifeVariation,
        size: config.size + Math.random() * config.sizeVariation,
        color: config.color,
        opacity: 1
      }

      this.particles.push(particle)
    }
  }

  private tick(): void {
    if (!this.isRunning) return

    const currentTime = performance.now()
    const deltaTime = (currentTime - this.lastTime) / 1000
    this.lastTime = currentTime

    this.updateParticles(deltaTime)
    this.renderParticles()

    requestAnimationFrame(() => this.tick())
  }

  private updateParticles(deltaTime: number): void {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      const particle = this.particles[i]

      // Update position
      particle.x += particle.velocityX * deltaTime * 60
      particle.y += particle.velocityY * deltaTime * 60

      // Update life
      particle.life += deltaTime

      // Update opacity based on life
      const lifeRatio = particle.life / particle.maxLife
      particle.opacity = Math.max(0, 1 - lifeRatio)

      // Remove dead particles
      if (particle.life >= particle.maxLife) {
        this.particles.splice(i, 1)
      }
    }
  }

  private renderParticles(): void {
    const context = this.canvas.getContext('2d')
    context.clearRect(0, 0, this.canvas.width, this.canvas.height)

    for (const particle of this.particles) {
      context.save()

      context.globalAlpha = particle.opacity
      context.fillStyle = particle.color

      context.beginPath()
      context.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2)
      context.fill()

      context.restore()
    }
  }
}

interface EmissionConfig {
  x: number
  y: number
  count: number
  spread: number
  velocity: number
  life: number
  lifeVariation: number
  size: number
  sizeVariation: number
  color: string
}

@Component
struct ParticleEffect {
  @State private canvasRef: Canvas | null = null
  private particleSystem: ParticleSystem | null = null

  build() {
    Canvas(this.canvasRef)
      .width('100%')
      .height('100%')
      .onReady(() => {
        this.initializeParticleSystem()
      })
      .onClick((event) => {
        this.createExplosion(event.x, event.y)
      })
  }

  private initializeParticleSystem(): void {
    if (this.canvasRef) {
      this.particleSystem = new ParticleSystem(this.canvasRef)
      this.particleSystem.start()
    }
  }

  private createExplosion(x: number, y: number): void {
    if (this.particleSystem) {
      this.particleSystem.emit({
        x: x,
        y: y,
        count: 30,
        spread: 50,
        velocity: 200,
        life: 2,
        lifeVariation: 1,
        size: 3,
        sizeVariation: 2,
        color: '#FF6B6B'
      })
    }
  }
}
```

## Performance Optimization

### Animation Performance Monitor

```typescript
class AnimationPerformanceMonitor {
  private frameCount: number = 0;
  private droppedFrames: number = 0;
  private lastFrameTime: number = 0;
  private averageFPS: number = 60;
  private isMonitoring: boolean = false;

  startMonitoring(): void {
    this.isMonitoring = true;
    this.frameCount = 0;
    this.droppedFrames = 0;
    this.lastFrameTime = performance.now();
    this.monitorFrame();
  }

  stopMonitoring(): AnimationPerformanceReport {
    this.isMonitoring = false;

    return {
      totalFrames: this.frameCount,
      droppedFrames: this.droppedFrames,
      averageFPS: this.averageFPS,
      efficiency:
        ((this.frameCount - this.droppedFrames) / this.frameCount) * 100,
    };
  }

  private monitorFrame(): void {
    if (!this.isMonitoring) return;

    const currentTime = performance.now();
    const frameDuration = currentTime - this.lastFrameTime;

    this.frameCount++;

    // Check for dropped frames (> 16.67ms for 60fps)
    if (frameDuration > 16.67) {
      this.droppedFrames++;
    }

    // Calculate rolling average FPS
    this.averageFPS = this.calculateAverageFPS(frameDuration);

    this.lastFrameTime = currentTime;
    requestAnimationFrame(() => this.monitorFrame());
  }

  private calculateAverageFPS(frameDuration: number): number {
    const currentFPS = 1000 / frameDuration;
    return this.averageFPS * 0.9 + currentFPS * 0.1; // Rolling average
  }
}

interface AnimationPerformanceReport {
  totalFrames: number;
  droppedFrames: number;
  averageFPS: number;
  efficiency: number;
}
```

## Conclusion

Advanced ArkUI animation techniques enable developers to create engaging, performant user experiences. Key aspects include:

- Custom animation frameworks with keyframe interpolation
- Physics-based spring animations for natural motion
- Gesture-driven interactive animations
- Complex morphing and transition effects
- Particle systems for visual effects
- Performance monitoring and optimization
- Hardware acceleration utilization

These techniques provide the foundation for creating sophisticated animations while maintaining smooth 60fps performance across all HarmonyOS devices.
