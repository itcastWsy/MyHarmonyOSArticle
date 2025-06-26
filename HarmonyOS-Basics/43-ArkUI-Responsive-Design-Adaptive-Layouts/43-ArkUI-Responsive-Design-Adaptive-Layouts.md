# 43. ArkUI Responsive Design and Adaptive Layouts

## Introduction

Creating responsive and adaptive user interfaces is essential for HarmonyOS applications that run across diverse device types and screen sizes. This article explores advanced responsive design patterns, adaptive layout techniques, and device-specific optimization strategies.

## Responsive Design Foundation

### Breakpoint System

```typescript
// Comprehensive breakpoint management system
export enum DeviceType {
  Phone = "phone",
  Tablet = "tablet",
  Desktop = "desktop",
  Watch = "watch",
  TV = "tv",
  Car = "car",
}

export interface Breakpoint {
  name: string;
  minWidth: number;
  maxWidth?: number;
  deviceType: DeviceType;
  columns: number;
  margins: number;
  gutters: number;
}

export class ResponsiveManager {
  private static instance: ResponsiveManager;
  private breakpoints: Breakpoint[] = [
    {
      name: "xs",
      minWidth: 0,
      maxWidth: 320,
      deviceType: DeviceType.Watch,
      columns: 4,
      margins: 8,
      gutters: 4,
    },
    {
      name: "sm",
      minWidth: 321,
      maxWidth: 600,
      deviceType: DeviceType.Phone,
      columns: 4,
      margins: 16,
      gutters: 8,
    },
    {
      name: "md",
      minWidth: 601,
      maxWidth: 840,
      deviceType: DeviceType.Tablet,
      columns: 8,
      margins: 24,
      gutters: 16,
    },
    {
      name: "lg",
      minWidth: 841,
      maxWidth: 1200,
      deviceType: DeviceType.Desktop,
      columns: 12,
      margins: 32,
      gutters: 24,
    },
    {
      name: "xl",
      minWidth: 1201,
      deviceType: DeviceType.TV,
      columns: 12,
      margins: 48,
      gutters: 32,
    },
  ];

  private currentBreakpoint: Breakpoint | null = null;
  private listeners: Array<(breakpoint: Breakpoint) => void> = [];

  static getInstance(): ResponsiveManager {
    if (!this.instance) {
      this.instance = new ResponsiveManager();
    }
    return this.instance;
  }

  constructor() {
    this.updateBreakpoint();
    this.setupResizeListener();
  }

  private setupResizeListener(): void {
    // In actual implementation, this would listen to window resize events
    // For HarmonyOS, this would integrate with system display APIs
  }

  getCurrentBreakpoint(): Breakpoint | null {
    return this.currentBreakpoint;
  }

  getBreakpointFor(width: number): Breakpoint | null {
    return (
      this.breakpoints.find(
        (bp) => width >= bp.minWidth && (!bp.maxWidth || width <= bp.maxWidth)
      ) || null
    );
  }

  addBreakpointListener(
    callback: (breakpoint: Breakpoint) => void
  ): () => void {
    this.listeners.push(callback);
    return () => {
      const index = this.listeners.indexOf(callback);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }

  private updateBreakpoint(): void {
    // Get current screen width (would use actual system API)
    const screenWidth = this.getScreenWidth();
    const newBreakpoint = this.getBreakpointFor(screenWidth);

    if (newBreakpoint && newBreakpoint !== this.currentBreakpoint) {
      this.currentBreakpoint = newBreakpoint;
      this.notifyListeners();
    }
  }

  private getScreenWidth(): number {
    // Placeholder - would use actual system API
    return 800;
  }

  private notifyListeners(): void {
    if (this.currentBreakpoint) {
      this.listeners.forEach((listener) => listener(this.currentBreakpoint!));
    }
  }

  isDevice(deviceType: DeviceType): boolean {
    return this.currentBreakpoint?.deviceType === deviceType;
  }

  getColumns(): number {
    return this.currentBreakpoint?.columns || 4;
  }

  getMargins(): number {
    return this.currentBreakpoint?.margins || 16;
  }

  getGutters(): number {
    return this.currentBreakpoint?.gutters || 8;
  }
}
```

### Adaptive Grid System

```typescript
// Flexible grid system with responsive behavior
@Component
export struct ResponsiveGrid {
  @Prop children: any[] = [];
  @Prop spacing: number = 16;
  @Prop autoRows: boolean = true;
  @Prop minItemWidth: number = 200;
  @State private columns: number = 1;
  @State private margins: number = 16;

  private responsiveManager = ResponsiveManager.getInstance();
  private unsubscribe?: () => void;

  aboutToAppear() {
    this.updateLayout();
    this.unsubscribe = this.responsiveManager.addBreakpointListener(() => {
      this.updateLayout();
    });
  }

  aboutToDisappear() {
    this.unsubscribe?.();
  }

  private updateLayout(): void {
    this.columns = this.responsiveManager.getColumns();
    this.margins = this.responsiveManager.getMargins();
  }

  build() {
    Column() {
      Grid() {
        ForEach(this.children, (child: any, index: number) => {
          GridItem() {
            child
          }
          .padding(this.spacing / 2);
        }, (child: any, index: number) => `grid-item-${index}`)
      }
      .columnsTemplate(this.getColumnsTemplate())
      .rowsTemplate(this.autoRows ? 'auto' : undefined)
      .columnsGap(this.spacing)
      .rowsGap(this.spacing)
      .padding(this.margins)
    }
    .width('100%')
    .height('100%');
  }

  private getColumnsTemplate(): string {
    const availableColumns = Math.min(
      this.columns,
      Math.floor((this.getScreenWidth() - this.margins * 2) / this.minItemWidth)
    );
    return '1fr '.repeat(Math.max(1, availableColumns)).trim();
  }

  private getScreenWidth(): number {
    // Would use actual screen width API
    return 800;
  }
}

// Responsive container with breakpoint-specific layouts
@Component
export struct ResponsiveContainer {
  @BuilderParam phoneLayout?: () => void;
  @BuilderParam tabletLayout?: () => void;
  @BuilderParam desktopLayout?: () => void;
  @BuilderParam defaultLayout?: () => void;

  @State private currentDevice: DeviceType = DeviceType.Phone;
  private responsiveManager = ResponsiveManager.getInstance();
  private unsubscribe?: () => void;

  aboutToAppear() {
    this.updateDevice();
    this.unsubscribe = this.responsiveManager.addBreakpointListener((breakpoint) => {
      this.currentDevice = breakpoint.deviceType;
    });
  }

  aboutToDisappear() {
    this.unsubscribe?.();
  }

  private updateDevice(): void {
    const breakpoint = this.responsiveManager.getCurrentBreakpoint();
    if (breakpoint) {
      this.currentDevice = breakpoint.deviceType;
    }
  }

  build() {
    if (this.currentDevice === DeviceType.Phone && this.phoneLayout) {
      this.phoneLayout();
    } else if (this.currentDevice === DeviceType.Tablet && this.tabletLayout) {
      this.tabletLayout();
    } else if (this.currentDevice === DeviceType.Desktop && this.desktopLayout) {
      this.desktopLayout();
    } else if (this.defaultLayout) {
      this.defaultLayout();
    } else {
      Text('No layout defined for current device')
        .fontSize(16)
        .textAlign(TextAlign.Center)
        .width('100%')
        .height('100%');
    }
  }
}
```

## Advanced Layout Patterns

### Fluid Typography System

```typescript
// Responsive typography that scales with viewport
export class TypographyScale {
  private static baseSize = 16;
  private static scaleRatio = 1.25;
  private static breakpoints = {
    sm: { multiplier: 0.875, lineHeight: 1.4 },
    md: { multiplier: 1, lineHeight: 1.5 },
    lg: { multiplier: 1.125, lineHeight: 1.6 },
    xl: { multiplier: 1.25, lineHeight: 1.7 }
  };

  static getSize(level: number, breakpoint: string = 'md'): number {
    const config = this.breakpoints[breakpoint] || this.breakpoints.md;
    const size = this.baseSize * Math.pow(this.scaleRatio, level);
    return size * config.multiplier;
  }

  static getLineHeight(breakpoint: string = 'md'): number {
    const config = this.breakpoints[breakpoint] || this.breakpoints.md;
    return config.lineHeight;
  }

  static clamp(min: number, preferred: number, max: number): string {
    return `clamp(${min}px, ${preferred}vw, ${max}px)`;
  }
}

@Component
export struct ResponsiveText {
  @Prop text: string = '';
  @Prop level: number = 0; // Typography scale level
  @Prop weight: FontWeight = FontWeight.Normal;
  @Prop color: Color = Color.Black;
  @Prop maxLines?: number;

  @State private fontSize: number = 16;
  @State private lineHeight: number = 1.5;

  private responsiveManager = ResponsiveManager.getInstance();
  private unsubscribe?: () => void;

  aboutToAppear() {
    this.updateTypography();
    this.unsubscribe = this.responsiveManager.addBreakpointListener((breakpoint) => {
      this.updateTypography();
    });
  }

  aboutToDisappear() {
    this.unsubscribe?.();
  }

  private updateTypography(): void {
    const breakpoint = this.responsiveManager.getCurrentBreakpoint();
    const breakpointName = breakpoint?.name || 'md';

    this.fontSize = TypographyScale.getSize(this.level, breakpointName);
    this.lineHeight = TypographyScale.getLineHeight(breakpointName);
  }

  build() {
    Text(this.text)
      .fontSize(this.fontSize)
      .fontWeight(this.weight)
      .fontColor(this.color)
      .lineHeight(this.lineHeight)
      .maxLines(this.maxLines)
      .textOverflow({ overflow: TextOverflow.Ellipsis });
  }
}
```

### Adaptive Navigation System

```typescript
// Navigation component that adapts to different devices
@Component
export struct AdaptiveNavigation {
  @Prop menuItems: NavItem[] = [];
  @Prop brand?: string;
  @Prop onItemSelect?: (item: NavItem) => void;

  @State private showSidebar: boolean = false;
  @State private navigationStyle: 'horizontal' | 'sidebar' | 'bottom' = 'horizontal';

  private responsiveManager = ResponsiveManager.getInstance();
  private unsubscribe?: () => void;

  aboutToAppear() {
    this.updateNavigationStyle();
    this.unsubscribe = this.responsiveManager.addBreakpointListener(() => {
      this.updateNavigationStyle();
    });
  }

  aboutToDisappear() {
    this.unsubscribe?.();
  }

  private updateNavigationStyle(): void {
    const deviceType = this.responsiveManager.getCurrentBreakpoint()?.deviceType;

    switch (deviceType) {
      case DeviceType.Phone:
        this.navigationStyle = 'bottom';
        break;
      case DeviceType.Tablet:
        this.navigationStyle = 'sidebar';
        break;
      case DeviceType.Desktop:
      case DeviceType.TV:
        this.navigationStyle = 'horizontal';
        break;
      default:
        this.navigationStyle = 'horizontal';
    }
  }

  build() {
    if (this.navigationStyle === 'horizontal') {
      this.buildHorizontalNav();
    } else if (this.navigationStyle === 'sidebar') {
      this.buildSidebarNav();
    } else {
      this.buildBottomNav();
    }
  }

  @Builder
  private buildHorizontalNav() {
    Row({ space: 24 }) {
      if (this.brand) {
        Text(this.brand)
          .fontSize(20)
          .fontWeight(FontWeight.Bold);
      }

      Flex({ wrap: FlexWrap.Wrap, space: { main: LengthMetrics.vp(16) } }) {
        ForEach(this.menuItems, (item: NavItem) => {
          Button(item.label)
            .type(ButtonType.Normal)
            .backgroundColor(Color.Transparent)
            .fontColor(item.active ? Color.Blue : Color.Black)
            .onClick(() => {
              if (this.onItemSelect) {
                this.onItemSelect(item);
              }
            });
        }, (item: NavItem) => item.id)
      }
      .layoutWeight(1);
    }
    .width('100%')
    .height(60)
    .padding({ horizontal: 24 })
    .backgroundColor(Color.White)
    .border({ width: { bottom: 1 }, color: Color.Gray });
  }

  @Builder
  private buildSidebarNav() {
    Row() {
      // Sidebar
      Column({ space: 8 }) {
        if (this.brand) {
          Text(this.brand)
            .fontSize(18)
            .fontWeight(FontWeight.Bold)
            .padding(16);
        }

        ForEach(this.menuItems, (item: NavItem) => {
          Row({ space: 12 }) {
            if (item.icon) {
              Text(item.icon)
                .fontSize(20);
            }
            Text(item.label)
              .fontSize(16)
              .layoutWeight(1);
          }
          .width('100%')
          .height(48)
          .padding({ horizontal: 16 })
          .backgroundColor(item.active ? '#e3f2fd' : Color.Transparent)
          .borderRadius(8)
          .margin({ horizontal: 8 })
          .onClick(() => {
            if (this.onItemSelect) {
              this.onItemSelect(item);
            }
          });
        }, (item: NavItem) => item.id)
      }
      .width(240)
      .height('100%')
      .backgroundColor('#f5f5f5')
      .alignItems(HorizontalAlign.Start);

      // Main content area
      Column()
        .layoutWeight(1)
        .height('100%')
        .backgroundColor(Color.White);
    }
    .width('100%')
    .height('100%');
  }

  @Builder
  private buildBottomNav() {
    Column() {
      // Main content area
      Column()
        .layoutWeight(1)
        .width('100%')
        .backgroundColor(Color.White);

      // Bottom navigation
      Row() {
        ForEach(this.menuItems.slice(0, 5), (item: NavItem) => { // Limit to 5 items
          Column({ space: 4 }) {
            if (item.icon) {
              Text(item.icon)
                .fontSize(24)
                .fontColor(item.active ? Color.Blue : Color.Gray);
            }
            Text(item.label)
              .fontSize(12)
              .fontColor(item.active ? Color.Blue : Color.Gray)
              .maxLines(1)
              .textOverflow({ overflow: TextOverflow.Ellipsis });
          }
          .layoutWeight(1)
          .height(64)
          .justifyContent(FlexAlign.Center)
          .onClick(() => {
            if (this.onItemSelect) {
              this.onItemSelect(item);
            }
          });
        }, (item: NavItem) => item.id)
      }
      .width('100%')
      .height(80)
      .backgroundColor(Color.White)
      .border({ width: { top: 1 }, color: Color.Gray });
    }
    .width('100%')
    .height('100%');
  }
}

interface NavItem {
  id: string;
  label: string;
  icon?: string;
  active?: boolean;
}
```

### Responsive Image Component

```typescript
// Adaptive image component with multiple source support
@Component
export struct ResponsiveImage {
  @Prop src: string = '';
  @Prop srcSet?: ImageSource[];
  @Prop alt: string = '';
  @Prop aspectRatio?: number;
  @Prop fit: ImageFit = ImageFit.Cover;
  @Prop lazy: boolean = true;
  @Prop placeholder?: string;

  @State private currentSrc: string = '';
  @State private isLoaded: boolean = false;
  @State private isVisible: boolean = false;

  private responsiveManager = ResponsiveManager.getInstance();
  private unsubscribe?: () => void;

  aboutToAppear() {
    this.selectAppropriateSource();
    this.unsubscribe = this.responsiveManager.addBreakpointListener(() => {
      this.selectAppropriateSource();
    });

    if (!this.lazy) {
      this.loadImage();
    }
  }

  aboutToDisappear() {
    this.unsubscribe?.();
  }

  private selectAppropriateSource(): void {
    if (!this.srcSet || this.srcSet.length === 0) {
      this.currentSrc = this.src;
      return;
    }

    const breakpoint = this.responsiveManager.getCurrentBreakpoint();
    const devicePixelRatio = this.getDevicePixelRatio();

    // Sort sources by width descending
    const sortedSources = [...this.srcSet].sort((a, b) => b.width - a.width);

    // Find the best match based on current viewport and pixel ratio
    const targetWidth = (breakpoint?.minWidth || 320) * devicePixelRatio;

    const selectedSource = sortedSources.find(source =>
      source.width >= targetWidth
    ) || sortedSources[sortedSources.length - 1];

    this.currentSrc = selectedSource.url;
  }

  private getDevicePixelRatio(): number {
    // Would use actual device pixel ratio API
    return 2;
  }

  private async loadImage(): Promise<void> {
    if (this.isLoaded || !this.currentSrc) return;

    try {
      // In actual implementation, this would preload the image
      await this.preloadImage(this.currentSrc);
      this.isLoaded = true;
    } catch (error) {
      console.error('Failed to load image:', error);
    }
  }

  private preloadImage(src: string): Promise<void> {
    return new Promise((resolve, reject) => {
      // Simulate image loading
      setTimeout(() => {
        Math.random() > 0.1 ? resolve() : reject(new Error('Failed to load'));
      }, 500);
    });
  }

  build() {
    Stack() {
      // Placeholder or loading state
      if (!this.isLoaded) {
        if (this.placeholder) {
          Image(this.placeholder)
            .width('100%')
            .height('100%')
            .objectFit(this.fit)
            .opacity(0.3);
        } else {
          Column() {
            Text('Loading...')
              .fontSize(14)
              .fontColor(Color.Gray);
          }
          .width('100%')
          .height('100%')
          .backgroundColor('#f0f0f0')
          .justifyContent(FlexAlign.Center);
        }
      }

      // Actual image
      if (this.isLoaded && this.currentSrc) {
        Image(this.currentSrc)
          .width('100%')
          .height('100%')
          .objectFit(this.fit)
          .alt(this.alt)
          .onComplete(() => {
            this.isLoaded = true;
          })
          .onError(() => {
            console.error('Image failed to load:', this.currentSrc);
          });
      }
    }
    .width('100%')
    .aspectRatio(this.aspectRatio || 1)
    .onAppear(() => {
      this.isVisible = true;
      if (this.lazy && this.isVisible) {
        this.loadImage();
      }
    })
    .onDisAppear(() => {
      this.isVisible = false;
    });
  }
}

interface ImageSource {
  url: string;
  width: number;
  format?: 'webp' | 'jpeg' | 'png';
}
```

## Device-Specific Optimizations

### Multi-Device Layout Manager

```typescript
@Component
export struct MultiDeviceApp {
  @State private currentLayout: 'phone' | 'tablet' | 'desktop' | 'tv' = 'phone';

  private responsiveManager = ResponsiveManager.getInstance();

  aboutToAppear() {
    this.updateLayout();
    this.responsiveManager.addBreakpointListener(() => {
      this.updateLayout();
    });
  }

  private updateLayout(): void {
    const deviceType = this.responsiveManager.getCurrentBreakpoint()?.deviceType;

    switch (deviceType) {
      case DeviceType.Phone:
        this.currentLayout = 'phone';
        break;
      case DeviceType.Tablet:
        this.currentLayout = 'tablet';
        break;
      case DeviceType.Desktop:
        this.currentLayout = 'desktop';
        break;
      case DeviceType.TV:
        this.currentLayout = 'tv';
        break;
      default:
        this.currentLayout = 'phone';
    }
  }

  build() {
    Stack() {
      if (this.currentLayout === 'phone') {
        this.buildPhoneLayout();
      } else if (this.currentLayout === 'tablet') {
        this.buildTabletLayout();
      } else if (this.currentLayout === 'desktop') {
        this.buildDesktopLayout();
      } else if (this.currentLayout === 'tv') {
        this.buildTVLayout();
      }
    }
    .width('100%')
    .height('100%');
  }

  @Builder
  private buildPhoneLayout() {
    Column({ space: 16 }) {
      // Single column layout optimized for touch
      Text('Phone Layout')
        .fontSize(24)
        .fontWeight(FontWeight.Bold);

      // Content stacked vertically
      ScrollView() {
        Column({ space: 16 }) {
          ForEach([1, 2, 3, 4, 5], (item: number) => {
            Card() {
              Text(`Card ${item}`)
                .fontSize(16)
                .padding(16);
            }
            .width('100%')
            .height(120);
          }, (item: number) => `phone-card-${item}`)
        }
        .width('100%')
        .padding(16);
      }
      .layoutWeight(1);
    }
    .width('100%')
    .height('100%');
  }

  @Builder
  private buildTabletLayout() {
    Row() {
      // Sidebar navigation
      Column() {
        Text('Navigation')
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .padding(16);
      }
      .width(200)
      .height('100%')
      .backgroundColor('#f5f5f5');

      // Main content area
      Column({ space: 16 }) {
        Text('Tablet Layout')
          .fontSize(24)
          .fontWeight(FontWeight.Bold)
          .padding(16);

        // Grid layout
        Grid() {
          ForEach([1, 2, 3, 4, 5, 6], (item: number) => {
            GridItem() {
              Card() {
                Text(`Card ${item}`)
                  .fontSize(16)
                  .textAlign(TextAlign.Center);
              }
              .width('100%')
              .height(120);
            }
          }, (item: number) => `tablet-card-${item}`)
        }
        .columnsTemplate('1fr 1fr')
        .rowsGap(16)
        .columnsGap(16)
        .padding(16);
      }
      .layoutWeight(1)
      .height('100%');
    }
    .width('100%')
    .height('100%');
  }

  @Builder
  private buildDesktopLayout() {
    Column() {
      // Top navigation bar
      Row() {
        Text('Desktop App')
          .fontSize(20)
          .fontWeight(FontWeight.Bold);
      }
      .width('100%')
      .height(60)
      .backgroundColor('#333')
      .fontColor(Color.White)
      .padding({ horizontal: 24 });

      Row() {
        // Left sidebar
        Column() {
          Text('Sidebar')
            .fontSize(16)
            .padding(16);
        }
        .width(240)
        .height('100%')
        .backgroundColor('#f0f0f0');

        // Main content
        Column({ space: 24 }) {
          Text('Desktop Layout')
            .fontSize(28)
            .fontWeight(FontWeight.Bold)
            .padding(24);

          // Multi-column grid
          Grid() {
            ForEach([1, 2, 3, 4, 5, 6, 7, 8], (item: number) => {
              GridItem() {
                Card() {
                  Text(`Card ${item}`)
                    .fontSize(16)
                    .textAlign(TextAlign.Center);
                }
                .width('100%')
                .height(150);
              }
            }, (item: number) => `desktop-card-${item}`)
          }
          .columnsTemplate('1fr 1fr 1fr 1fr')
          .rowsGap(20)
          .columnsGap(20)
          .padding(24);
        }
        .layoutWeight(1)
        .height('100%');

        // Right panel
        Column() {
          Text('Details')
            .fontSize(16)
            .padding(16);
        }
        .width(280)
        .height('100%')
        .backgroundColor('#f9f9f9');
      }
      .layoutWeight(1);
    }
    .width('100%')
    .height('100%');
  }

  @Builder
  private buildTVLayout() {
    // TV layout optimized for remote control navigation
    Column({ space: 32 }) {
      Text('TV Interface')
        .fontSize(36)
        .fontWeight(FontWeight.Bold)
        .textAlign(TextAlign.Center)
        .padding(48);

      // Large, focusable elements
      Grid() {
        ForEach([1, 2, 3, 4], (item: number) => {
          GridItem() {
            Card() {
              Column({ space: 16 }) {
                Text(`🎬`)
                  .fontSize(48);
                Text(`Content ${item}`)
                  .fontSize(24)
                  .fontWeight(FontWeight.Medium);
              }
              .width('100%')
              .height(200)
              .justifyContent(FlexAlign.Center);
            }
            .focusable(true)
            .borderWidth(2)
            .borderColor(Color.Transparent)
            .onFocus(() => {
              // Handle focus for TV remote navigation
            });
          }
        }, (item: number) => `tv-card-${item}`)
      }
      .columnsTemplate('1fr 1fr')
      .rowsGap(32)
      .columnsGap(32)
      .padding(48);
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#1a1a1a')
    .fontColor(Color.White);
  }
}
```

## Best Practices

1. **Mobile-First Design**: Start with mobile layouts and progressively enhance
2. **Flexible Units**: Use relative units (vw, vh, %) instead of fixed pixels
3. **Content Strategy**: Prioritize content based on device capabilities
4. **Touch Targets**: Ensure interactive elements are appropriately sized for touch
5. **Performance**: Optimize images and resources for different screen densities
6. **Accessibility**: Maintain accessibility across all device types
7. **Testing**: Test layouts on actual devices whenever possible

## Conclusion

Responsive design and adaptive layouts are essential for creating successful HarmonyOS applications that work seamlessly across the diverse ecosystem of devices. By implementing flexible breakpoint systems, adaptive components, and device-specific optimizations, developers can ensure their applications provide optimal user experiences regardless of screen size or device type.

The key to successful responsive design lies in understanding user context, leveraging appropriate design patterns for each device type, and maintaining consistency while adapting to different interaction paradigms.
