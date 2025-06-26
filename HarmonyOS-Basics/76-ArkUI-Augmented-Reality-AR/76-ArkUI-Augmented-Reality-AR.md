# ArkUI Augmented Reality (AR) Integration

## Introduction

Augmented Reality integration enables ArkUI applications to overlay digital content onto the real world. This guide covers AR scene management, 3D object placement, gesture recognition, and spatial tracking for immersive AR experiences.

## AR Scene Management

### AR Session Manager

```typescript
interface ARConfiguration {
  trackingMode: "world" | "face" | "image" | "object";
  lightEstimation: boolean;
  planeDetection: "horizontal" | "vertical" | "both" | "none";
  environmentTexturing: boolean;
  worldAlignment: "gravity" | "gravityAndHeading" | "camera";
}

interface ARFrame {
  timestamp: number;
  camera: ARCamera;
  anchors: ARAnchor[];
  lightEstimate?: ARLightEstimate;
  detectedPlanes: ARPlane[];
}

interface ARCamera {
  position: Vector3;
  rotation: Quaternion;
  projectionMatrix: Matrix4;
  viewMatrix: Matrix4;
  intrinsics: CameraIntrinsics;
}

interface ARAnchor {
  id: string;
  position: Vector3;
  rotation: Quaternion;
  confidence: number;
  type: "plane" | "image" | "object" | "face";
}

class ARSessionManager {
  private isRunning = false;
  private configuration: ARConfiguration;
  private frameListeners = new Set<(frame: ARFrame) => void>();
  private anchors = new Map<string, ARAnchor>();
  private lastFrame?: ARFrame;

  constructor(config: ARConfiguration) {
    this.configuration = config;
  }

  async startSession(): Promise<boolean> {
    try {
      // Initialize AR session
      await this.initializeARSession();
      this.isRunning = true;
      this.startFrameLoop();
      return true;
    } catch (error) {
      console.error("Failed to start AR session:", error);
      return false;
    }
  }

  stopSession(): void {
    this.isRunning = false;
  }

  addAnchor(position: Vector3, rotation: Quaternion = [0, 0, 0, 1]): string {
    const anchor: ARAnchor = {
      id: `anchor_${Date.now()}`,
      position,
      rotation,
      confidence: 0.8,
      type: "object",
    };

    this.anchors.set(anchor.id, anchor);
    return anchor.id;
  }

  removeAnchor(anchorId: string): boolean {
    return this.anchors.delete(anchorId);
  }

  getAnchor(anchorId: string): ARAnchor | undefined {
    return this.anchors.get(anchorId);
  }

  onFrameUpdate(listener: (frame: ARFrame) => void): void {
    this.frameListeners.add(listener);
  }

  getLastFrame(): ARFrame | undefined {
    return this.lastFrame;
  }

  hitTest(screenPoint: Vector2): ARHitTestResult[] {
    if (!this.lastFrame) return [];

    // Simulate hit testing
    const results: ARHitTestResult[] = [];

    this.lastFrame.detectedPlanes.forEach((plane) => {
      if (this.isPointOnPlane(screenPoint, plane)) {
        results.push({
          type: "existingPlane",
          distance: Math.random() * 5,
          worldPosition: this.screenToWorld(screenPoint),
          anchor: plane.anchor,
        });
      }
    });

    return results.sort((a, b) => a.distance - b.distance);
  }

  private async initializeARSession(): Promise<void> {
    // Initialize AR framework
    await new Promise((resolve) => setTimeout(resolve, 1000));
  }

  private startFrameLoop(): void {
    const updateFrame = () => {
      if (!this.isRunning) return;

      this.lastFrame = this.generateARFrame();
      this.notifyFrameListeners(this.lastFrame);

      requestAnimationFrame(updateFrame);
    };

    updateFrame();
  }

  private generateARFrame(): ARFrame {
    return {
      timestamp: Date.now(),
      camera: this.generateCameraData(),
      anchors: Array.from(this.anchors.values()),
      lightEstimate: this.generateLightEstimate(),
      detectedPlanes: this.generateDetectedPlanes(),
    };
  }

  private generateCameraData(): ARCamera {
    return {
      position: [0, 1.6, 0], // Typical phone height
      rotation: [0, 0, 0, 1],
      projectionMatrix: this.createProjectionMatrix(),
      viewMatrix: this.createViewMatrix(),
      intrinsics: {
        focalLength: [640, 640],
        principalPoint: [320, 240],
        imageResolution: [640, 480],
      },
    };
  }

  private generateLightEstimate(): ARLightEstimate {
    return {
      ambientIntensity: 0.5 + Math.random() * 0.5,
      ambientColorTemperature: 5000 + Math.random() * 2000,
      primaryLightDirection: [0.3, -0.7, 0.6],
      primaryLightIntensity: 0.8,
    };
  }

  private generateDetectedPlanes(): ARPlane[] {
    return [
      {
        id: "plane_floor",
        anchor: {
          id: "anchor_floor",
          position: [0, 0, 0],
          rotation: [0, 0, 0, 1],
          confidence: 0.9,
          type: "plane",
        },
        center: [0, 0, 0],
        extent: [5, 0, 5],
        normal: [0, 1, 0],
        classification: "floor",
      },
    ];
  }

  private createProjectionMatrix(): Matrix4 {
    // Simplified projection matrix
    return new Array(16)
      .fill(0)
      .map((_, i) => (i === 0 || i === 5 || i === 10 || i === 15 ? 1 : 0));
  }

  private createViewMatrix(): Matrix4 {
    // Simplified view matrix
    return new Array(16)
      .fill(0)
      .map((_, i) => (i === 0 || i === 5 || i === 10 || i === 15 ? 1 : 0));
  }

  private isPointOnPlane(screenPoint: Vector2, plane: ARPlane): boolean {
    // Simplified plane intersection test
    return Math.random() > 0.7;
  }

  private screenToWorld(screenPoint: Vector2): Vector3 {
    // Convert screen coordinates to world coordinates
    return [(screenPoint[0] - 0.5) * 2, 0, (screenPoint[1] - 0.5) * 2];
  }

  private notifyFrameListeners(frame: ARFrame): void {
    this.frameListeners.forEach((listener) => listener(frame));
  }
}

// Type definitions
type Vector2 = [number, number];
type Vector3 = [number, number, number];
type Quaternion = [number, number, number, number];
type Matrix4 = number[];

interface CameraIntrinsics {
  focalLength: Vector2;
  principalPoint: Vector2;
  imageResolution: Vector2;
}

interface ARLightEstimate {
  ambientIntensity: number;
  ambientColorTemperature: number;
  primaryLightDirection: Vector3;
  primaryLightIntensity: number;
}

interface ARPlane {
  id: string;
  anchor: ARAnchor;
  center: Vector3;
  extent: Vector3;
  normal: Vector3;
  classification: "floor" | "wall" | "ceiling" | "table" | "seat" | "unknown";
}

interface ARHitTestResult {
  type: "existingPlane" | "estimatedPlane" | "feature";
  distance: number;
  worldPosition: Vector3;
  anchor?: ARAnchor;
}
```

### 3D Object Manager

```typescript
interface AR3DObject {
  id: string;
  name: string;
  modelPath: string;
  position: Vector3;
  rotation: Quaternion;
  scale: Vector3;
  anchorId?: string;
  visible: boolean;
  interactive: boolean;
  animations?: ARAnimation[];
}

interface ARAnimation {
  name: string;
  type: "rotation" | "translation" | "scale" | "opacity";
  duration: number;
  loop: boolean;
  keyframes: AnimationKeyframe[];
}

interface AnimationKeyframe {
  time: number;
  value: any;
  easing?: "linear" | "ease-in" | "ease-out" | "ease-in-out";
}

class AR3DObjectManager {
  private objects = new Map<string, AR3DObject>();
  private animationTimers = new Map<string, number>();
  private renderListeners = new Set<(objects: AR3DObject[]) => void>();

  addObject(object: AR3DObject): void {
    this.objects.set(object.id, object);
    this.notifyRenderListeners();
  }

  removeObject(objectId: string): void {
    const removed = this.objects.delete(objectId);
    if (removed) {
      this.stopObjectAnimations(objectId);
      this.notifyRenderListeners();
    }
  }

  getObject(objectId: string): AR3DObject | undefined {
    return this.objects.get(objectId);
  }

  getObjects(): AR3DObject[] {
    return Array.from(this.objects.values());
  }

  updateObjectPosition(objectId: string, position: Vector3): void {
    const object = this.objects.get(objectId);
    if (object) {
      object.position = position;
      this.notifyRenderListeners();
    }
  }

  updateObjectRotation(objectId: string, rotation: Quaternion): void {
    const object = this.objects.get(objectId);
    if (object) {
      object.rotation = rotation;
      this.notifyRenderListeners();
    }
  }

  updateObjectScale(objectId: string, scale: Vector3): void {
    const object = this.objects.get(objectId);
    if (object) {
      object.scale = scale;
      this.notifyRenderListeners();
    }
  }

  playAnimation(objectId: string, animationName: string): void {
    const object = this.objects.get(objectId);
    if (!object || !object.animations) return;

    const animation = object.animations.find(
      (anim) => anim.name === animationName
    );
    if (!animation) return;

    this.stopObjectAnimations(objectId);
    this.startAnimation(object, animation);
  }

  stopAnimation(objectId: string, animationName?: string): void {
    if (animationName) {
      // Stop specific animation
      const timerId = this.animationTimers.get(`${objectId}_${animationName}`);
      if (timerId) {
        clearInterval(timerId);
        this.animationTimers.delete(`${objectId}_${animationName}`);
      }
    } else {
      // Stop all animations for object
      this.stopObjectAnimations(objectId);
    }
  }

  onRenderUpdate(listener: (objects: AR3DObject[]) => void): void {
    this.renderListeners.add(listener);
  }

  intersectRay(rayOrigin: Vector3, rayDirection: Vector3): AR3DObject | null {
    // Simplified ray-object intersection
    for (const object of this.objects.values()) {
      if (!object.visible || !object.interactive) continue;

      const distance = this.calculateRayObjectDistance(
        rayOrigin,
        rayDirection,
        object
      );
      if (distance < 1.0) {
        // Within interaction range
        return object;
      }
    }
    return null;
  }

  private startAnimation(object: AR3DObject, animation: ARAnimation): void {
    const animationKey = `${object.id}_${animation.name}`;
    const startTime = Date.now();

    const animate = () => {
      const elapsed = Date.now() - startTime;
      const progress = Math.min(elapsed / animation.duration, 1);

      this.applyAnimationFrame(object, animation, progress);
      this.notifyRenderListeners();

      if (progress < 1) {
        const timerId = setTimeout(animate, 16); // ~60fps
        this.animationTimers.set(animationKey, timerId);
      } else if (animation.loop) {
        // Restart animation
        this.startAnimation(object, animation);
      }
    };

    animate();
  }

  private applyAnimationFrame(
    object: AR3DObject,
    animation: ARAnimation,
    progress: number
  ): void {
    const currentKeyframe = this.getCurrentKeyframe(
      animation.keyframes,
      progress
    );

    switch (animation.type) {
      case "rotation":
        object.rotation = this.interpolateQuaternion(
          object.rotation,
          currentKeyframe.value,
          progress
        );
        break;
      case "translation":
        object.position = this.interpolateVector3(
          object.position,
          currentKeyframe.value,
          progress
        );
        break;
      case "scale":
        object.scale = this.interpolateVector3(
          object.scale,
          currentKeyframe.value,
          progress
        );
        break;
    }
  }

  private getCurrentKeyframe(
    keyframes: AnimationKeyframe[],
    progress: number
  ): AnimationKeyframe {
    for (let i = 0; i < keyframes.length - 1; i++) {
      if (progress >= keyframes[i].time && progress <= keyframes[i + 1].time) {
        return keyframes[i + 1];
      }
    }
    return keyframes[keyframes.length - 1];
  }

  private interpolateVector3(from: Vector3, to: Vector3, t: number): Vector3 {
    return [
      from[0] + (to[0] - from[0]) * t,
      from[1] + (to[1] - from[1]) * t,
      from[2] + (to[2] - from[2]) * t,
    ];
  }

  private interpolateQuaternion(
    from: Quaternion,
    to: Quaternion,
    t: number
  ): Quaternion {
    // Simplified quaternion interpolation (slerp)
    return [
      from[0] + (to[0] - from[0]) * t,
      from[1] + (to[1] - from[1]) * t,
      from[2] + (to[2] - from[2]) * t,
      from[3] + (to[3] - from[3]) * t,
    ];
  }

  private calculateRayObjectDistance(
    rayOrigin: Vector3,
    rayDirection: Vector3,
    object: AR3DObject
  ): number {
    // Simplified distance calculation
    const dx = object.position[0] - rayOrigin[0];
    const dy = object.position[1] - rayOrigin[1];
    const dz = object.position[2] - rayOrigin[2];
    return Math.sqrt(dx * dx + dy * dy + dz * dz);
  }

  private stopObjectAnimations(objectId: string): void {
    Array.from(this.animationTimers.keys())
      .filter((key) => key.startsWith(objectId))
      .forEach((key) => {
        const timerId = this.animationTimers.get(key);
        if (timerId) {
          clearInterval(timerId);
          this.animationTimers.delete(key);
        }
      });
  }

  private notifyRenderListeners(): void {
    const objects = this.getObjects();
    this.renderListeners.forEach((listener) => listener(objects));
  }
}
```

## AR User Interface

### AR View Component

```typescript
@Component
struct ARView {
  @State private isSessionRunning: boolean = false
  @State private detectedPlanes: ARPlane[] = []
  @State private arObjects: AR3DObject[] = []
  @State private selectedObject: AR3DObject | null = null
  @State private showPlaneVisualization: boolean = true

  private arSession = new ARSessionManager({
    trackingMode: 'world',
    lightEstimation: true,
    planeDetection: 'horizontal',
    environmentTexturing: true,
    worldAlignment: 'gravity'
  })

  private objectManager = new AR3DObjectManager()

  build() {
    Stack() {
      // Camera view background
      this.buildCameraView()

      // AR content overlay
      this.buildAROverlay()

      // UI controls
      this.buildUIControls()
    }
    .width('100%')
    .height('100%')
  }

  aboutToAppear() {
    this.initializeAR()
  }

  aboutToDisappear() {
    this.arSession.stopSession()
  }

  @Builder
  private buildCameraView() {
    // Camera preview component
    Image($r('app.media.camera_preview'))
      .width('100%')
      .height('100%')
      .objectFit(ImageFit.Cover)
  }

  @Builder
  private buildAROverlay() {
    Canvas(this.getCanvasContext())
      .width('100%')
      .height('100%')
      .onReady(() => this.setupARRendering())
      .onTouch((event) => this.handleTouch(event))
  }

  @Builder
  private buildUIControls() {
    Column() {
      // Top controls
      Row() {
        Button(this.isSessionRunning ? 'Stop AR' : 'Start AR')
          .onClick(() => this.toggleARSession())
          .backgroundColor('#007AFF')
          .fontColor('#FFFFFF')

        Button('Reset')
          .onClick(() => this.resetARSession())
          .backgroundColor('#FF3B30')
          .fontColor('#FFFFFF')
          .margin({ left: 8 })
      }
      .margin({ top: 50, horizontal: 16 })

      Spacer()

      // Bottom controls
      Column() {
        if (this.selectedObject) {
          this.buildObjectControls()
        }

        this.buildPlacementControls()
      }
      .margin({ bottom: 50, horizontal: 16 })
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  private buildObjectControls() {
    Column() {
      Text(`Selected: ${this.selectedObject!.name}`)
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .fontColor('#FFFFFF')
        .backgroundColor('#00000080')
        .padding(8)
        .borderRadius(4)

      Row() {
        Button('Move')
          .onClick(() => this.enterMoveMode())
          .backgroundColor('#34C759')
          .fontColor('#FFFFFF')
          .margin({ right: 8 })

        Button('Rotate')
          .onClick(() => this.enterRotateMode())
          .backgroundColor('#FF9500')
          .fontColor('#FFFFFF')
          .margin({ right: 8 })

        Button('Delete')
          .onClick(() => this.deleteSelectedObject())
          .backgroundColor('#FF3B30')
          .fontColor('#FFFFFF')
      }
      .margin({ top: 8 })
    }
    .margin({ bottom: 16 })
  }

  @Builder
  private buildPlacementControls() {
    Row() {
      Button('📦 Cube')
        .onClick(() => this.placeObject('cube'))
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .margin({ right: 8 })

      Button('⚽ Sphere')
        .onClick(() => this.placeObject('sphere'))
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .margin({ right: 8 })

      Button('✈️ Model')
        .onClick(() => this.placeObject('airplane'))
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
    }
  }

  private async initializeAR(): Promise<void> {
    this.arSession.onFrameUpdate((frame) => {
      this.detectedPlanes = frame.detectedPlanes
      this.updateARRendering(frame)
    })

    this.objectManager.onRenderUpdate((objects) => {
      this.arObjects = objects
    })
  }

  private async toggleARSession(): Promise<void> {
    if (this.isSessionRunning) {
      this.arSession.stopSession()
      this.isSessionRunning = false
    } else {
      const success = await this.arSession.startSession()
      this.isSessionRunning = success
    }
  }

  private resetARSession(): void {
    this.objectManager.getObjects().forEach(obj => {
      this.objectManager.removeObject(obj.id)
    })
    this.selectedObject = null
  }

  private handleTouch(event: TouchEvent): void {
    if (event.type === TouchType.Down) {
      const touchPoint: Vector2 = [
        event.touches[0].x / this.getScreenWidth(),
        event.touches[0].y / this.getScreenHeight()
      ]

      // Check for object selection
      const frame = this.arSession.getLastFrame()
      if (frame) {
        const hitResults = this.arSession.hitTest(touchPoint)
        if (hitResults.length > 0) {
          this.handleARHit(hitResults[0])
        }
      }
    }
  }

  private handleARHit(hitResult: ARHitTestResult): void {
    // Check if we hit an existing object
    const rayOrigin = hitResult.worldPosition
    const rayDirection: Vector3 = [0, -1, 0] // Downward ray

    const hitObject = this.objectManager.intersectRay(rayOrigin, rayDirection)
    if (hitObject) {
      this.selectedObject = hitObject
    } else {
      this.selectedObject = null
    }
  }

  private placeObject(type: string): void {
    const frame = this.arSession.getLastFrame()
    if (!frame || this.detectedPlanes.length === 0) return

    const plane = this.detectedPlanes[0]
    const objectId = `${type}_${Date.now()}`

    const arObject: AR3DObject = {
      id: objectId,
      name: `${type.charAt(0).toUpperCase() + type.slice(1)} Object`,
      modelPath: `/models/${type}.obj`,
      position: plane.center,
      rotation: [0, 0, 0, 1],
      scale: [0.1, 0.1, 0.1],
      anchorId: plane.anchor.id,
      visible: true,
      interactive: true,
      animations: [
        {
          name: 'spin',
          type: 'rotation',
          duration: 2000,
          loop: true,
          keyframes: [
            { time: 0, value: [0, 0, 0, 1] },
            { time: 1, value: [0, 1, 0, 0] }
          ]
        }
      ]
    }

    this.objectManager.addObject(arObject)
    this.selectedObject = arObject
  }

  private enterMoveMode(): void {
    // Enable object movement mode
    console.log('Entering move mode for object:', this.selectedObject?.name)
  }

  private enterRotateMode(): void {
    // Enable object rotation mode
    console.log('Entering rotate mode for object:', this.selectedObject?.name)
  }

  private deleteSelectedObject(): void {
    if (this.selectedObject) {
      this.objectManager.removeObject(this.selectedObject.id)
      this.selectedObject = null
    }
  }

  private setupARRendering(): void {
    // Initialize AR rendering context
  }

  private updateARRendering(frame: ARFrame): void {
    const ctx = this.getCanvasContext()
    if (!ctx) return

    // Clear canvas
    ctx.clearRect(0, 0, this.getScreenWidth(), this.getScreenHeight())

    // Render detected planes
    if (this.showPlaneVisualization) {
      this.renderPlanes(ctx, frame.detectedPlanes)
    }

    // Render 3D objects
    this.renderObjects(ctx, this.arObjects, frame.camera)
  }

  private renderPlanes(ctx: CanvasRenderingContext2D, planes: ARPlane[]): void {
    ctx.strokeStyle = '#00FF0080'
    ctx.lineWidth = 2

    planes.forEach(plane => {
      const screenPos = this.worldToScreen(plane.center)
      const extent = plane.extent

      ctx.strokeRect(
        screenPos[0] - extent[0] * 50,
        screenPos[1] - extent[2] * 50,
        extent[0] * 100,
        extent[2] * 100
      )
    })
  }

  private renderObjects(ctx: CanvasRenderingContext2D, objects: AR3DObject[], camera: ARCamera): void {
    objects.forEach(object => {
      if (!object.visible) return

      const screenPos = this.worldToScreen(object.position)
      const size = 40 * object.scale[0]

      // Simple object representation
      ctx.fillStyle = object === this.selectedObject ? '#FF0000' : '#0000FF'
      ctx.fillRect(
        screenPos[0] - size / 2,
        screenPos[1] - size / 2,
        size,
        size
      )

      // Object label
      ctx.fillStyle = '#FFFFFF'
      ctx.font = '12px Arial'
      ctx.fillText(object.name, screenPos[0] - 30, screenPos[1] - size / 2 - 10)
    })
  }

  private worldToScreen(worldPos: Vector3): Vector2 {
    // Simplified world to screen conversion
    const screenX = (worldPos[0] + 2) / 4 * this.getScreenWidth()
    const screenY = (worldPos[2] + 2) / 4 * this.getScreenHeight()
    return [screenX, screenY]
  }

  private getCanvasContext(): CanvasRenderingContext2D | null {
    // Return canvas rendering context
    return null
  }

  private getScreenWidth(): number {
    return 375 // iPhone width
  }

  private getScreenHeight(): number {
    return 667 // iPhone height
  }
}
```

## Conclusion

Augmented Reality integration in ArkUI applications provides:

- Real-time camera feed with digital overlay capabilities
- 3D object placement and manipulation in AR space
- Plane detection and spatial understanding
- Interactive AR object management with animations
- Touch-based object selection and manipulation
- Cross-platform AR experiences within HarmonyOS

These AR capabilities enable developers to create immersive applications that blend digital content with the real world, suitable for gaming, education, shopping, and industrial applications.
