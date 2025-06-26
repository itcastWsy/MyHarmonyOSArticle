# ArkUI Time-Space Interfaces

## Introduction

Time-Space Interfaces in ArkUI enable temporal and spatial manipulation of user interfaces, multi-dimensional navigation, and time-based interactions. This guide covers temporal UI patterns, spatial dimensions, and chronological interface design.

## Time-Space Framework

```typescript
interface TemporalDimension {
  id: string;
  timelineId: string;
  currentTime: number;
  timeScale: number;
  direction: TimeDirection;
  bounds: TimeBounds;
}

interface SpatialDimension {
  id: string;
  spaceId: string;
  coordinates: SpaceCoordinates;
  scale: number;
  orientation: SpaceOrientation;
  bounds: SpaceBounds;
}

type TimeDirection = "forward" | "backward" | "paused" | "loop";

class TimeSpaceInterfaceManager {
  private temporalLayers = new Map<string, TemporalLayer>();
  private spatialLayers = new Map<string, SpatialLayer>();
  private chronoNavigator = new ChronoNavigator();
  private spatialNavigator = new SpatialNavigator();
  private dimensionSynchronizer = new DimensionSynchronizer();

  createTemporalInterface(config: TemporalConfig): string {
    const interfaceId = `temporal_${Date.now()}`;

    const temporalLayer: TemporalLayer = {
      id: interfaceId,
      timeline: this.createTimeline(config.timeline),
      elements: new Map(),
      currentTime: config.startTime || 0,
      playbackRate: config.playbackRate || 1.0,
      isPlaying: false,
    };

    this.temporalLayers.set(interfaceId, temporalLayer);
    return interfaceId;
  }

  createSpatialInterface(config: SpatialConfig): string {
    const interfaceId = `spatial_${Date.now()}`;

    const spatialLayer: SpatialLayer = {
      id: interfaceId,
      space: this.createSpace(config.space),
      elements: new Map(),
      viewpoint: config.viewpoint || { x: 0, y: 0, z: 0 },
      scale: config.scale || 1.0,
    };

    this.spatialLayers.set(interfaceId, spatialLayer);
    return interfaceId;
  }

  addTemporalElement(layerId: string, element: TemporalElement): string {
    const layer = this.temporalLayers.get(layerId);
    if (!layer) throw new Error("Temporal layer not found");

    const elementId = `element_${Date.now()}`;
    layer.elements.set(elementId, element);

    return elementId;
  }

  addSpatialElement(layerId: string, element: SpatialElement): string {
    const layer = this.spatialLayers.get(layerId);
    if (!layer) throw new Error("Spatial layer not found");

    const elementId = `element_${Date.now()}`;
    layer.elements.set(elementId, element);

    return elementId;
  }

  navigateTime(
    layerId: string,
    targetTime: number,
    transition?: TimeTransition
  ): void {
    const layer = this.temporalLayers.get(layerId);
    if (!layer) return;

    if (transition) {
      this.chronoNavigator.animateToTime(layer, targetTime, transition);
    } else {
      layer.currentTime = targetTime;
      this.updateTemporalElements(layer);
    }
  }

  navigateSpace(
    layerId: string,
    targetPosition: SpaceCoordinates,
    transition?: SpaceTransition
  ): void {
    const layer = this.spatialLayers.get(layerId);
    if (!layer) return;

    if (transition) {
      this.spatialNavigator.animateToPosition(
        layer,
        targetPosition,
        transition
      );
    } else {
      layer.viewpoint = targetPosition;
      this.updateSpatialElements(layer);
    }
  }

  playTimeline(layerId: string, rate: number = 1.0): void {
    const layer = this.temporalLayers.get(layerId);
    if (!layer) return;

    layer.isPlaying = true;
    layer.playbackRate = rate;
    this.startTimelinePlayback(layer);
  }

  pauseTimeline(layerId: string): void {
    const layer = this.temporalLayers.get(layerId);
    if (!layer) return;

    layer.isPlaying = false;
  }

  createTimeWarp(layerId: string, config: TimeWarpConfig): string {
    const warpId = `warp_${Date.now()}`;

    const layer = this.temporalLayers.get(layerId);
    if (!layer) throw new Error("Temporal layer not found");

    this.chronoNavigator.createTimeWarp(layer, config);
    return warpId;
  }

  createSpatialPortal(
    fromLayerId: string,
    toLayerId: string,
    config: PortalConfig
  ): string {
    const portalId = `portal_${Date.now()}`;

    const fromLayer = this.spatialLayers.get(fromLayerId);
    const toLayer = this.spatialLayers.get(toLayerId);

    if (!fromLayer || !toLayer) throw new Error("Spatial layers not found");

    this.spatialNavigator.createPortal(fromLayer, toLayer, config);
    return portalId;
  }

  synchronizeDimensions(
    temporalLayerId: string,
    spatialLayerId: string,
    config: SynchronizationConfig
  ): void {
    const temporalLayer = this.temporalLayers.get(temporalLayerId);
    const spatialLayer = this.spatialLayers.get(spatialLayerId);

    if (!temporalLayer || !spatialLayer) return;

    this.dimensionSynchronizer.link(temporalLayer, spatialLayer, config);
  }

  createTemporalBranch(
    layerId: string,
    branchPoint: number,
    config: BranchConfig
  ): string {
    const branchId = `branch_${Date.now()}`;

    const layer = this.temporalLayers.get(layerId);
    if (!layer) throw new Error("Temporal layer not found");

    const branch = this.chronoNavigator.createBranch(
      layer,
      branchPoint,
      config
    );
    this.temporalLayers.set(branchId, branch);

    return branchId;
  }

  queryTemporalState(layerId: string, timeRange: TimeRange): TemporalState[] {
    const layer = this.temporalLayers.get(layerId);
    if (!layer) return [];

    return this.chronoNavigator.queryStateInRange(layer, timeRange);
  }

  querySpatialRegion(layerId: string, region: SpaceRegion): SpatialElement[] {
    const layer = this.spatialLayers.get(layerId);
    if (!layer) return [];

    return this.spatialNavigator.queryElementsInRegion(layer, region);
  }

  private createTimeline(config: TimelineConfig): Timeline {
    return {
      id: `timeline_${Date.now()}`,
      duration: config.duration,
      resolution: config.resolution || 1000, // ms
      keyframes: config.keyframes || [],
      looping: config.looping || false,
    };
  }

  private createSpace(config: SpaceConfig): Space {
    return {
      id: `space_${Date.now()}`,
      dimensions: config.dimensions,
      bounds: config.bounds,
      coordinate_system: config.coordinateSystem || "cartesian",
      units: config.units || "pixels",
    };
  }

  private updateTemporalElements(layer: TemporalLayer): void {
    for (const [elementId, element] of layer.elements) {
      this.updateElementAtTime(element, layer.currentTime);
    }
  }

  private updateSpatialElements(layer: SpatialLayer): void {
    for (const [elementId, element] of layer.elements) {
      this.updateElementInSpace(element, layer.viewpoint, layer.scale);
    }
  }

  private startTimelinePlayback(layer: TemporalLayer): void {
    const updateInterval = setInterval(() => {
      if (!layer.isPlaying) {
        clearInterval(updateInterval);
        return;
      }

      layer.currentTime += 16 * layer.playbackRate; // 60fps

      if (layer.currentTime >= layer.timeline.duration) {
        if (layer.timeline.looping) {
          layer.currentTime = 0;
        } else {
          layer.isPlaying = false;
          clearInterval(updateInterval);
        }
      }

      this.updateTemporalElements(layer);
    }, 16);
  }

  private updateElementAtTime(element: TemporalElement, time: number): void {
    // Update element properties based on current time
    const keyframe = this.findKeyframe(element, time);
    if (keyframe) {
      this.applyKeyframe(element, keyframe, time);
    }
  }

  private updateElementInSpace(
    element: SpatialElement,
    viewpoint: SpaceCoordinates,
    scale: number
  ): void {
    // Update element position relative to viewpoint
    element.transformedPosition = this.transformPosition(
      element.position,
      viewpoint,
      scale
    );
  }

  private findKeyframe(
    element: TemporalElement,
    time: number
  ): Keyframe | null {
    for (let i = 0; i < element.keyframes.length - 1; i++) {
      const current = element.keyframes[i];
      const next = element.keyframes[i + 1];

      if (time >= current.time && time <= next.time) {
        return this.interpolateKeyframes(current, next, time);
      }
    }

    return null;
  }

  private interpolateKeyframes(
    keyframe1: Keyframe,
    keyframe2: Keyframe,
    time: number
  ): Keyframe {
    const progress =
      (time - keyframe1.time) / (keyframe2.time - keyframe1.time);

    return {
      time,
      properties: this.interpolateProperties(
        keyframe1.properties,
        keyframe2.properties,
        progress
      ),
    };
  }

  private interpolateProperties(
    props1: ElementProperties,
    props2: ElementProperties,
    progress: number
  ): ElementProperties {
    const interpolated: ElementProperties = {};

    for (const key in props1) {
      if (typeof props1[key] === "number" && typeof props2[key] === "number") {
        interpolated[key] =
          props1[key] + (props2[key] - props1[key]) * progress;
      } else {
        interpolated[key] = progress < 0.5 ? props1[key] : props2[key];
      }
    }

    return interpolated;
  }

  private applyKeyframe(
    element: TemporalElement,
    keyframe: Keyframe,
    time: number
  ): void {
    Object.assign(element.currentProperties, keyframe.properties);
  }

  private transformPosition(
    position: SpaceCoordinates,
    viewpoint: SpaceCoordinates,
    scale: number
  ): SpaceCoordinates {
    return {
      x: (position.x - viewpoint.x) * scale,
      y: (position.y - viewpoint.y) * scale,
      z: (position.z - viewpoint.z) * scale,
    };
  }
}

// Supporting classes
class ChronoNavigator {
  animateToTime(
    layer: TemporalLayer,
    targetTime: number,
    transition: TimeTransition
  ): void {
    // Animate timeline to target time
  }

  createTimeWarp(layer: TemporalLayer, config: TimeWarpConfig): void {
    // Create time distortion effects
  }

  createBranch(
    layer: TemporalLayer,
    branchPoint: number,
    config: BranchConfig
  ): TemporalLayer {
    // Create temporal branch
    return { ...layer, id: `branch_${Date.now()}` };
  }

  queryStateInRange(
    layer: TemporalLayer,
    timeRange: TimeRange
  ): TemporalState[] {
    return [];
  }
}

class SpatialNavigator {
  animateToPosition(
    layer: SpatialLayer,
    targetPosition: SpaceCoordinates,
    transition: SpaceTransition
  ): void {
    // Animate spatial viewpoint
  }

  createPortal(
    fromLayer: SpatialLayer,
    toLayer: SpatialLayer,
    config: PortalConfig
  ): void {
    // Create spatial portal
  }

  queryElementsInRegion(
    layer: SpatialLayer,
    region: SpaceRegion
  ): SpatialElement[] {
    return [];
  }
}

class DimensionSynchronizer {
  link(
    temporalLayer: TemporalLayer,
    spatialLayer: SpatialLayer,
    config: SynchronizationConfig
  ): void {
    // Synchronize temporal and spatial dimensions
  }
}

// Interfaces and types
interface TemporalLayer {
  id: string;
  timeline: Timeline;
  elements: Map<string, TemporalElement>;
  currentTime: number;
  playbackRate: number;
  isPlaying: boolean;
}

interface SpatialLayer {
  id: string;
  space: Space;
  elements: Map<string, SpatialElement>;
  viewpoint: SpaceCoordinates;
  scale: number;
}

interface Timeline {
  id: string;
  duration: number;
  resolution: number;
  keyframes: Keyframe[];
  looping: boolean;
}

interface Space {
  id: string;
  dimensions: number;
  bounds: SpaceBounds;
  coordinate_system: string;
  units: string;
}

interface TemporalElement {
  id: string;
  type: string;
  keyframes: Keyframe[];
  currentProperties: ElementProperties;
}

interface SpatialElement {
  id: string;
  type: string;
  position: SpaceCoordinates;
  transformedPosition?: SpaceCoordinates;
  properties: ElementProperties;
}

interface Keyframe {
  time: number;
  properties: ElementProperties;
}

interface ElementProperties {
  [key: string]: any;
}

interface SpaceCoordinates {
  x: number;
  y: number;
  z: number;
}

interface SpaceBounds {
  min: SpaceCoordinates;
  max: SpaceCoordinates;
}

interface TimeBounds {
  start: number;
  end: number;
}

interface SpaceOrientation {
  rotation: SpaceCoordinates;
  scale: SpaceCoordinates;
}

interface TemporalConfig {
  timeline: TimelineConfig;
  startTime?: number;
  playbackRate?: number;
}

interface SpatialConfig {
  space: SpaceConfig;
  viewpoint?: SpaceCoordinates;
  scale?: number;
}

interface TimelineConfig {
  duration: number;
  resolution?: number;
  keyframes?: Keyframe[];
  looping?: boolean;
}

interface SpaceConfig {
  dimensions: number;
  bounds: SpaceBounds;
  coordinateSystem?: string;
  units?: string;
}

interface TimeTransition {
  duration: number;
  easing: string;
}

interface SpaceTransition {
  duration: number;
  easing: string;
}

interface TimeWarpConfig {
  factor: number;
  region: TimeRange;
}

interface PortalConfig {
  position: SpaceCoordinates;
  size: SpaceCoordinates;
}

interface SynchronizationConfig {
  timeToSpace: (time: number) => SpaceCoordinates;
  spaceToTime: (space: SpaceCoordinates) => number;
}

interface BranchConfig {
  divergence: number;
  duration: number;
}

interface TimeRange {
  start: number;
  end: number;
}

interface SpaceRegion {
  center: SpaceCoordinates;
  radius: number;
}

interface TemporalState {
  time: number;
  elements: Map<string, ElementProperties>;
}
```

## ArkUI Time-Space Interface Component

```typescript
@Component
export struct TimeSpaceInterfaceDashboard {
  @State private timeSpaceManager: TimeSpaceInterfaceManager = new TimeSpaceInterfaceManager()
  @State private temporalLayerId: string = ''
  @State private spatialLayerId: string = ''
  @State private currentTime: number = 0
  @State private isPlaying: boolean = false
  @State private currentPosition: SpaceCoordinates = { x: 0, y: 0, z: 0 }

  aboutToAppear() {
    this.initializeInterfaces()
  }

  build() {
    Column({ space: 16 }) {
      this.buildHeader()
      this.buildTemporalControls()
      this.buildSpatialControls()
      this.buildSynchronizationControls()
      this.buildDimensionStatus()
    }
    .width('100%')
    .padding(16)
  }

  @Builder
  private buildHeader() {
    Text('Time-Space Interfaces')
      .fontSize(24)
      .fontWeight(FontWeight.Bold)
  }

  @Builder
  private buildTemporalControls() {
    Column() {
      Text('Temporal Navigation')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Button(this.isPlaying ? 'Pause' : 'Play')
          .flexGrow(1)
          .margin({ right: 8 })
          .backgroundColor(this.isPlaying ? '#FF9500' : '#34C759')
          .onClick(() => {
            this.toggleTimePlayback()
          })

        Text(`${this.currentTime.toFixed(1)}s`)
          .fontSize(14)
          .fontColor('#666666')
          .margin({ left: 8 })
      }
      .width('100%')

      Slider({
        value: this.currentTime,
        min: 0,
        max: 10000,
        step: 100
      })
      .width('100%')
      .margin({ top: 8 })
      .onChange((value) => {
        this.navigateToTime(value)
      })

      Row() {
        Button('Create Branch')
          .flexGrow(1)
          .margin({ right: 8 })
          .onClick(() => {
            this.createTemporalBranch()
          })

        Button('Time Warp')
          .flexGrow(1)
          .margin({ left: 8 })
          .onClick(() => {
            this.createTimeWarp()
          })
      }
      .width('100%')
      .margin({ top: 8 })
    }
    .padding(16)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
  }

  @Builder
  private buildSpatialControls() {
    Column() {
      Text('Spatial Navigation')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Column() {
          Text('X')
            .fontSize(12)
            .fontColor('#666666')
          Text(this.currentPosition.x.toFixed(1))
            .fontSize(14)
            .fontWeight(FontWeight.Bold)
        }
        .flexGrow(1)

        Column() {
          Text('Y')
            .fontSize(12)
            .fontColor('#666666')
          Text(this.currentPosition.y.toFixed(1))
            .fontSize(14)
            .fontWeight(FontWeight.Bold)
        }
        .flexGrow(1)

        Column() {
          Text('Z')
            .fontSize(12)
            .fontColor('#666666')
          Text(this.currentPosition.z.toFixed(1))
            .fontSize(14)
            .fontWeight(FontWeight.Bold)
        }
        .flexGrow(1)
      }
      .width('100%')
      .padding(12)
      .backgroundColor('#FFFFFF')
      .borderRadius(8)

      Row() {
        Button('Navigate')
          .flexGrow(1)
          .margin({ right: 8 })
          .onClick(() => {
            this.navigateSpace()
          })

        Button('Create Portal')
          .flexGrow(1)
          .margin({ left: 8 })
          .onClick(() => {
            this.createSpatialPortal()
          })
      }
      .width('100%')
      .margin({ top: 8 })
    }
    .padding(16)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
  }

  @Builder
  private buildSynchronizationControls() {
    Column() {
      Text('Dimension Synchronization')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Button('Sync Time & Space')
        .width('100%')
        .onClick(() => {
          this.synchronizeDimensions()
        })
    }
    .padding(16)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
  }

  @Builder
  private buildDimensionStatus() {
    Column() {
      Text('Dimension Status')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        this.buildStatusCard('Time', `${this.currentTime.toFixed(1)}s`, '#007AFF')
        this.buildStatusCard('Space', `(${this.currentPosition.x.toFixed(0)}, ${this.currentPosition.y.toFixed(0)})`, '#34C759')
        this.buildStatusCard('Status', this.isPlaying ? 'Active' : 'Paused', '#FF9500')
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)
    }
  }

  @Builder
  private buildStatusCard(title: string, value: string, color: string) {
    Column() {
      Text(value)
        .fontSize(14)
        .fontWeight(FontWeight.Bold)
        .fontColor(color)
      Text(title)
        .fontSize(12)
        .fontColor('#666666')
    }
    .width('30%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .alignItems(HorizontalAlign.Center)
  }

  private initializeInterfaces(): void {
    this.temporalLayerId = this.timeSpaceManager.createTemporalInterface({
      timeline: {
        duration: 10000,
        resolution: 100,
        looping: true
      }
    })

    this.spatialLayerId = this.timeSpaceManager.createSpatialInterface({
      space: {
        dimensions: 3,
        bounds: {
          min: { x: -100, y: -100, z: -100 },
          max: { x: 100, y: 100, z: 100 }
        }
      }
    })
  }

  private toggleTimePlayback(): void {
    if (this.isPlaying) {
      this.timeSpaceManager.pauseTimeline(this.temporalLayerId)
    } else {
      this.timeSpaceManager.playTimeline(this.temporalLayerId, 1.0)
    }
    this.isPlaying = !this.isPlaying
  }

  private navigateToTime(time: number): void {
    this.currentTime = time
    this.timeSpaceManager.navigateTime(this.temporalLayerId, time)
  }

  private navigateSpace(): void {
    const newPosition: SpaceCoordinates = {
      x: Math.random() * 200 - 100,
      y: Math.random() * 200 - 100,
      z: Math.random() * 200 - 100
    }

    this.currentPosition = newPosition
    this.timeSpaceManager.navigateSpace(this.spatialLayerId, newPosition, {
      duration: 1000,
      easing: 'ease-in-out'
    })
  }

  private createTemporalBranch(): void {
    const branchId = this.timeSpaceManager.createTemporalBranch(
      this.temporalLayerId,
      this.currentTime,
      {
        divergence: 0.5,
        duration: 5000
      }
    )

    console.log('Created temporal branch:', branchId)
  }

  private createTimeWarp(): void {
    const warpId = this.timeSpaceManager.createTimeWarp(
      this.temporalLayerId,
      {
        factor: 2.0,
        region: {
          start: this.currentTime,
          end: this.currentTime + 2000
        }
      }
    )

    console.log('Created time warp:', warpId)
  }

  private createSpatialPortal(): void {
    const portalId = this.timeSpaceManager.createSpatialPortal(
      this.spatialLayerId,
      this.spatialLayerId, // Self-referential portal
      {
        position: this.currentPosition,
        size: { x: 10, y: 10, z: 10 }
      }
    )

    console.log('Created spatial portal:', portalId)
  }

  private synchronizeDimensions(): void {
    this.timeSpaceManager.synchronizeDimensions(
      this.temporalLayerId,
      this.spatialLayerId,
      {
        timeToSpace: (time: number) => ({
          x: Math.sin(time / 1000) * 50,
          y: Math.cos(time / 1000) * 50,
          z: time / 100
        }),
        spaceToTime: (space: SpaceCoordinates) =>
          Math.sqrt(space.x * space.x + space.y * space.y) * 20
      }
    )

    console.log('Synchronized dimensions')
  }
}
```

## Conclusion

Time-Space Interfaces in ArkUI provide:

- Temporal navigation and timeline manipulation
- Multi-dimensional spatial interfaces
- Time-space synchronization and correlation
- Temporal branching and alternate timelines
- Spatial portals and dimensional transitions

These capabilities enable sophisticated time-aware and spatially-conscious user interfaces for advanced HarmonyOS applications.
