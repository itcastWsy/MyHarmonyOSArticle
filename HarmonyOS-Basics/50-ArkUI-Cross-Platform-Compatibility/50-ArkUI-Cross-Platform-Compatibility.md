# ArkUI Cross-Platform Compatibility

## Introduction

ArkUI enables developers to build applications that run across multiple HarmonyOS devices with different form factors and capabilities. This guide explores cross-platform compatibility strategies, adaptive UI design patterns, and device-specific optimization techniques.

## Understanding Device Capabilities

### Device Detection and Adaptation

```typescript
// Device capability detection
interface DeviceCapabilities {
  screenSize: ScreenSize;
  inputMethods: InputMethod[];
  connectivity: ConnectivityInfo;
  sensors: SensorCapabilities;
  hardware: HardwareInfo;
}

enum ScreenSize {
  Phone = "phone",
  Tablet = "tablet",
  Desktop = "desktop",
  Watch = "watch",
  TV = "tv",
}

enum InputMethod {
  Touch = "touch",
  Mouse = "mouse",
  Keyboard = "keyboard",
  Voice = "voice",
  Gesture = "gesture",
}

class DeviceManager {
  private static instance: DeviceManager;
  private capabilities: DeviceCapabilities;

  static getInstance(): DeviceManager {
    if (!DeviceManager.instance) {
      DeviceManager.instance = new DeviceManager();
    }
    return DeviceManager.instance;
  }

  private constructor() {
    this.capabilities = this.detectCapabilities();
  }

  getCapabilities(): DeviceCapabilities {
    return this.capabilities;
  }

  getScreenSize(): ScreenSize {
    const windowSize = window.getLastWindow();
    const width = windowSize.getWindowProperties().windowRect.width;

    if (width < 600) return ScreenSize.Phone;
    if (width < 840) return ScreenSize.Tablet;
    if (width < 1200) return ScreenSize.Desktop;
    return ScreenSize.Desktop;
  }

  supportsInputMethod(method: InputMethod): boolean {
    return this.capabilities.inputMethods.includes(method);
  }

  private detectCapabilities(): DeviceCapabilities {
    return {
      screenSize: this.getScreenSize(),
      inputMethods: this.detectInputMethods(),
      connectivity: this.detectConnectivity(),
      sensors: this.detectSensors(),
      hardware: this.detectHardware(),
    };
  }

  private detectInputMethods(): InputMethod[] {
    const methods: InputMethod[] = [InputMethod.Touch];

    // Detect additional input capabilities
    if (this.hasKeyboard()) methods.push(InputMethod.Keyboard);
    if (this.hasMouse()) methods.push(InputMethod.Mouse);
    if (this.hasVoiceCapability()) methods.push(InputMethod.Voice);

    return methods;
  }
}
```

### Responsive Layout System

```typescript
// Breakpoint-based responsive design
interface Breakpoints {
  xs: number; // Extra small devices
  sm: number; // Small devices
  md: number; // Medium devices
  lg: number; // Large devices
  xl: number; // Extra large devices
}

const defaultBreakpoints: Breakpoints = {
  xs: 0,
  sm: 600,
  md: 840,
  lg: 1200,
  xl: 1600,
};

class ResponsiveManager {
  private breakpoints: Breakpoints;
  private currentBreakpoint: keyof Breakpoints;
  private listeners: Set<(breakpoint: keyof Breakpoints) => void> = new Set();

  constructor(breakpoints: Breakpoints = defaultBreakpoints) {
    this.breakpoints = breakpoints;
    this.currentBreakpoint = this.calculateBreakpoint();
    this.setupResizeListener();
  }

  getCurrentBreakpoint(): keyof Breakpoints {
    return this.currentBreakpoint;
  }

  isBreakpoint(breakpoint: keyof Breakpoints): boolean {
    return this.currentBreakpoint === breakpoint;
  }

  isBreakpointUp(breakpoint: keyof Breakpoints): boolean {
    const current = this.breakpoints[this.currentBreakpoint];
    const target = this.breakpoints[breakpoint];
    return current >= target;
  }

  isBreakpointDown(breakpoint: keyof Breakpoints): boolean {
    const current = this.breakpoints[this.currentBreakpoint];
    const target = this.breakpoints[breakpoint];
    return current <= target;
  }

  subscribe(listener: (breakpoint: keyof Breakpoints) => void): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private calculateBreakpoint(): keyof Breakpoints {
    const width = window.innerWidth;

    if (width >= this.breakpoints.xl) return "xl";
    if (width >= this.breakpoints.lg) return "lg";
    if (width >= this.breakpoints.md) return "md";
    if (width >= this.breakpoints.sm) return "sm";
    return "xs";
  }

  private setupResizeListener(): void {
    window.addEventListener("resize", () => {
      const newBreakpoint = this.calculateBreakpoint();
      if (newBreakpoint !== this.currentBreakpoint) {
        this.currentBreakpoint = newBreakpoint;
        this.listeners.forEach((listener) => listener(newBreakpoint));
      }
    });
  }
}

// Responsive component decorator
function Responsive(config: ResponsiveConfig) {
  return function (target: any) {
    const originalBuild = target.prototype.build;

    target.prototype.build = function () {
      const responsiveManager = new ResponsiveManager();
      const breakpoint = responsiveManager.getCurrentBreakpoint();

      // Apply responsive configuration
      this.applyResponsiveStyles(config, breakpoint);

      return originalBuild.call(this);
    };
  };
}

interface ResponsiveConfig {
  [key: string]: {
    [breakpoint in keyof Breakpoints]?: any;
  };
}
```

## Adaptive UI Components

### Universal Navigation Component

```typescript
@Component
struct UniversalNavigation {
  @Prop title: string = 'App'
  @Prop showBackButton: boolean = false
  @Prop actions: NavigationAction[] = []
  @State private currentDevice: ScreenSize = ScreenSize.Phone

  aboutToAppear() {
    const deviceManager = DeviceManager.getInstance()
    this.currentDevice = deviceManager.getScreenSize()
  }

  build() {
    if (this.currentDevice === ScreenSize.Phone) {
      this.buildMobileNavigation()
    } else if (this.currentDevice === ScreenSize.Tablet) {
      this.buildTabletNavigation()
    } else {
      this.buildDesktopNavigation()
    }
  }

  @Builder
  buildMobileNavigation() {
    Navigation() {
      Column() {
        // Mobile-specific navigation content
      }
    }
    .title(this.title)
    .titleMode(NavigationTitleMode.Mini)
    .toolBar({
      items: this.getMobileToolbarItems()
    })
  }

  @Builder
  buildTabletNavigation() {
    Row() {
      // Sidebar navigation for tablet
      Column() {
        Text(this.title)
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
          .padding(16)

        List() {
          ForEach(this.getNavigationItems(), (item: NavigationItem) => {
            ListItem() {
              Row() {
                Image(item.icon)
                  .width(24)
                  .height(24)
                Text(item.label)
                  .fontSize(16)
                  .margin({ left: 12 })
              }
              .width('100%')
              .padding(12)
              .onClick(() => item.onTap?.())
            }
          })
        }
        .layoutWeight(1)
      }
      .width(280)
      .height('100%')
      .backgroundColor('#f5f5f5')

      // Main content area
      Column() {
        // Content
      }
      .layoutWeight(1)
    }
  }

  @Builder
  buildDesktopNavigation() {
    Column() {
      // Top navigation bar for desktop
      Row() {
        Text(this.title)
          .fontSize(18)
          .fontWeight(FontWeight.Bold)

        Blank()

        Row({ space: 16 }) {
          ForEach(this.actions, (action: NavigationAction) => {
            Button(action.label)
              .onClick(() => action.onTap?.())
          })
        }
      }
      .width('100%')
      .height(56)
      .padding({ horizontal: 16 })
      .backgroundColor('#ffffff')
      .border({ width: { bottom: 1 }, color: '#e0e0e0' })

      // Content area
      Row() {
        // Desktop content layout
      }
      .layoutWeight(1)
    }
  }

  private getMobileToolbarItems(): ToolbarItem[] {
    return this.actions.map(action => ({
      value: action.label,
      icon: action.icon,
      action: action.onTap
    }))
  }

  private getNavigationItems(): NavigationItem[] {
    return [
      { icon: '/resources/home.png', label: 'Home', onTap: () => {} },
      { icon: '/resources/products.png', label: 'Products', onTap: () => {} },
      { icon: '/resources/profile.png', label: 'Profile', onTap: () => {} }
    ]
  }
}

interface NavigationAction {
  label: string
  icon?: string
  onTap?: () => void
}

interface NavigationItem {
  icon: string
  label: string
  onTap?: () => void
}
```

### Adaptive Layout Components

```typescript
@Component
struct AdaptiveGrid {
  @Prop items: any[]
  @Prop renderItem: (item: any) => void
  @State private columns: number = 1
  @State private itemSpacing: number = 8

  aboutToAppear() {
    this.calculateLayout()

    // Listen for screen size changes
    const responsiveManager = new ResponsiveManager()
    responsiveManager.subscribe((breakpoint) => {
      this.calculateLayout()
    })
  }

  build() {
    Grid() {
      ForEach(this.items, (item: any, index: number) => {
        GridItem() {
          this.renderItem(item)
        }
      })
    }
    .columnsTemplate(this.getColumnsTemplate())
    .rowsGap(this.itemSpacing)
    .columnsGap(this.itemSpacing)
    .padding(16)
  }

  private calculateLayout(): void {
    const deviceManager = DeviceManager.getInstance()
    const screenSize = deviceManager.getScreenSize()

    switch (screenSize) {
      case ScreenSize.Phone:
        this.columns = 1
        this.itemSpacing = 8
        break
      case ScreenSize.Tablet:
        this.columns = 2
        this.itemSpacing = 12
        break
      case ScreenSize.Desktop:
        this.columns = 3
        this.itemSpacing = 16
        break
      default:
        this.columns = 1
        this.itemSpacing = 8
    }
  }

  private getColumnsTemplate(): string {
    return Array(this.columns).fill('1fr').join(' ')
  }
}

// Usage example
@Entry
@Component
struct ProductCatalogPage {
  @State products: Product[] = []

  build() {
    Column() {
      UniversalNavigation({
        title: 'Product Catalog',
        actions: [
          { label: 'Search', onTap: () => this.openSearch() },
          { label: 'Filter', onTap: () => this.openFilter() }
        ]
      })

      AdaptiveGrid({
        items: this.products,
        renderItem: (product: Product) => this.buildProductCard(product)
      })
        .layoutWeight(1)
    }
  }

  @Builder
  buildProductCard(product: Product) {
    Card() {
      Column() {
        Image(product.imageUrl)
          .width('100%')
          .height(200)
          .objectFit(ImageFit.Cover)

        Text(product.name)
          .fontSize(16)
          .fontWeight(FontWeight.Bold)
          .maxLines(2)
          .textOverflow({ overflow: TextOverflow.Ellipsis })

        Text(`$${product.price}`)
          .fontSize(14)
          .fontColor('#007AFF')
      }
      .alignItems(HorizontalAlign.Start)
      .padding(12)
    }
    .width('100%')
    .onClick(() => this.openProductDetails(product))
  }
}
```

## Input Method Adaptation

### Multi-Input Support

```typescript
class InputMethodManager {
  private currentInputMethods: Set<InputMethod> = new Set()
  private inputListeners: Map<InputMethod, Set<Function>> = new Map()

  constructor() {
    this.detectAvailableInputMethods()
    this.setupInputListeners()
  }

  isInputMethodAvailable(method: InputMethod): boolean {
    return this.currentInputMethods.has(method)
  }

  registerInputListener(method: InputMethod, listener: Function): void {
    if (!this.inputListeners.has(method)) {
      this.inputListeners.set(method, new Set())
    }
    this.inputListeners.get(method)!.add(listener)
  }

  private detectAvailableInputMethods(): void {
    // Always available on HarmonyOS
    this.currentInputMethods.add(InputMethod.Touch)

    // Detect keyboard
    if (this.hasPhysicalKeyboard()) {
      this.currentInputMethods.add(InputMethod.Keyboard)
    }

    // Detect mouse/trackpad
    if (this.hasMouseInput()) {
      this.currentInputMethods.add(InputMethod.Mouse)
    }

    // Detect voice capability
    if (this.hasVoiceInput()) {
      this.currentInputMethods.add(InputMethod.Voice)
    }
  }

  private setupInputListeners(): void {
    // Touch events
    this.setupTouchListeners()

    // Keyboard events
    if (this.isInputMethodAvailable(InputMethod.Keyboard)) {
      this.setupKeyboardListeners()
    }

    // Mouse events
    if (this.isInputMethodAvailable(InputMethod.Mouse)) {
      this.setupMouseListeners()
    }
  }
}

@Component
struct MultiInputComponent {
  private inputManager = new InputMethodManager()
  @State private selectedItem: string = ''
  @State private searchText: string = ''

  build() {
    Column() {
      if (this.inputManager.isInputMethodAvailable(InputMethod.Voice)) {
        this.buildVoiceInput()
      }

      this.buildSearchInput()
      this.buildItemList()
    }
  }

  @Builder
  buildVoiceInput() {
    Button('Voice Search')
      .onClick(() => this.startVoiceInput())
  }

  @Builder
  buildSearchInput() {
    TextInput({
      placeholder: 'Search...',
      text: this.searchText
    })
      .onChange((value: string) => {
        this.searchText = value
      })
      .onSubmit(() => {
        this.performSearch()
      })
      // Add keyboard shortcuts if available
      .gesture(
        this.inputManager.isInputMethodAvailable(InputMethod.Keyboard)
          ? this.createKeyboardGestures()
          : undefined
      )
  }

  private createKeyboardGestures(): GestureGroup {
    return GestureGroup(GroupGestureMode.Exclusive,
      // Ctrl+F for focus search
      TapGesture({ count: 1 })
        .onAction(() => {
          if (this.isCtrlPressed()) {
            // Focus search input
          }
        })
    )
  }
}
```

## Performance Optimization

### Device-Specific Optimization

```typescript
class PerformanceManager {
  private deviceCapabilities: DeviceCapabilities;
  private optimizationSettings: OptimizationSettings;

  constructor() {
    this.deviceCapabilities = DeviceManager.getInstance().getCapabilities();
    this.optimizationSettings = this.calculateOptimizations();
  }

  getOptimizationSettings(): OptimizationSettings {
    return this.optimizationSettings;
  }

  private calculateOptimizations(): OptimizationSettings {
    const screenSize = this.deviceCapabilities.screenSize;

    return {
      virtualScrollThreshold: this.getVirtualScrollThreshold(screenSize),
      imageQuality: this.getImageQuality(screenSize),
      animationSettings: this.getAnimationSettings(screenSize),
      cacheSettings: this.getCacheSettings(screenSize),
    };
  }

  private getVirtualScrollThreshold(screenSize: ScreenSize): number {
    switch (screenSize) {
      case ScreenSize.Phone:
        return 50;
      case ScreenSize.Tablet:
        return 100;
      case ScreenSize.Desktop:
        return 200;
      default:
        return 50;
    }
  }

  private getImageQuality(screenSize: ScreenSize): ImageQuality {
    switch (screenSize) {
      case ScreenSize.Phone:
        return { width: 400, height: 400, quality: 0.8 };
      case ScreenSize.Tablet:
        return { width: 800, height: 800, quality: 0.9 };
      case ScreenSize.Desktop:
        return { width: 1200, height: 1200, quality: 1.0 };
      default:
        return { width: 400, height: 400, quality: 0.8 };
    }
  }
}

interface OptimizationSettings {
  virtualScrollThreshold: number;
  imageQuality: ImageQuality;
  animationSettings: AnimationSettings;
  cacheSettings: CacheSettings;
}

interface ImageQuality {
  width: number;
  height: number;
  quality: number;
}
```

## Conclusion

ArkUI cross-platform compatibility enables developers to create applications that provide optimal user experiences across different device types and form factors. Key strategies include:

- Device capability detection and adaptation
- Responsive layout systems with breakpoints
- Universal component patterns
- Multi-input method support
- Performance optimization per device type
- Adaptive navigation patterns

By implementing these patterns, developers can ensure their applications work seamlessly across phones, tablets, desktops, and other HarmonyOS devices while maintaining native performance and user experience quality.
