# 34. Custom Component Library Development

## Introduction

Building reusable component libraries is essential for maintaining consistency and efficiency across HarmonyOS applications. This article covers the comprehensive approach to developing, packaging, and distributing custom component libraries that can be shared across multiple projects and teams.

## Component Architecture Fundamentals

### Base Component Framework

```typescript
// BaseComponent.ets
export interface ComponentConfig {
  id?: string;
  className?: string;
  disabled?: boolean;
  visible?: boolean;
  theme?: ThemeConfig;
}

export interface ThemeConfig {
  primaryColor?: string;
  secondaryColor?: string;
  backgroundColor?: string;
  textColor?: string;
  borderColor?: string;
  borderRadius?: number;
  padding?: Padding;
  margin?: Margin;
}

export interface ComponentEventHandler<T = any> {
  onClick?: (event: T) => void;
  onLongPress?: (event: T) => void;
  onHover?: (event: T) => void;
  onFocus?: (event: T) => void;
  onBlur?: (event: T) => void;
}

export abstract class BaseComponent {
  protected config: ComponentConfig;
  protected theme: ThemeConfig;
  protected eventHandlers: ComponentEventHandler;

  constructor(config?: ComponentConfig) {
    this.config = config || {};
    this.theme = this.mergeTheme(config?.theme);
    this.eventHandlers = {};
  }

  private mergeTheme(customTheme?: ThemeConfig): ThemeConfig {
    const defaultTheme: ThemeConfig = {
      primaryColor: "#007DFF",
      secondaryColor: "#19000000",
      backgroundColor: "#FFFFFF",
      textColor: "#182431",
      borderColor: "#33182431",
      borderRadius: 8,
      padding: { top: 8, right: 12, bottom: 8, left: 12 },
      margin: { top: 0, right: 0, bottom: 0, left: 0 },
    };

    return { ...defaultTheme, ...customTheme };
  }

  setTheme(theme: Partial<ThemeConfig>): void {
    this.theme = { ...this.theme, ...theme };
  }

  setEventHandler(handlers: ComponentEventHandler): void {
    this.eventHandlers = { ...this.eventHandlers, ...handlers };
  }

  abstract render(): void;
}
```

### Component Lifecycle Manager

```typescript
export enum ComponentLifecycleState {
  CREATED = "created",
  MOUNTED = "mounted",
  UPDATED = "updated",
  DESTROYED = "destroyed",
}

export interface LifecycleHooks {
  onCreated?: () => void;
  onMounted?: () => void;
  onUpdated?: (prevProps?: any) => void;
  onDestroyed?: () => void;
}

export class ComponentLifecycleManager {
  private hooks: LifecycleHooks = {};
  private currentState: ComponentLifecycleState =
    ComponentLifecycleState.CREATED;
  private subscribers: Map<ComponentLifecycleState, (() => void)[]> = new Map();

  constructor(hooks?: LifecycleHooks) {
    this.hooks = hooks || {};
    this.initializeSubscribers();
  }

  private initializeSubscribers(): void {
    Object.values(ComponentLifecycleState).forEach((state) => {
      this.subscribers.set(state, []);
    });
  }

  subscribe(state: ComponentLifecycleState, callback: () => void): void {
    const callbacks = this.subscribers.get(state) || [];
    callbacks.push(callback);
    this.subscribers.set(state, callbacks);
  }

  unsubscribe(state: ComponentLifecycleState, callback: () => void): void {
    const callbacks = this.subscribers.get(state) || [];
    const index = callbacks.indexOf(callback);
    if (index > -1) {
      callbacks.splice(index, 1);
    }
  }

  setState(newState: ComponentLifecycleState, ...args: any[]): void {
    if (this.currentState === newState) return;

    this.currentState = newState;
    this.executeHook(newState, ...args);
    this.notifySubscribers(newState);
  }

  private executeHook(state: ComponentLifecycleState, ...args: any[]): void {
    switch (state) {
      case ComponentLifecycleState.CREATED:
        this.hooks.onCreated?.();
        break;
      case ComponentLifecycleState.MOUNTED:
        this.hooks.onMounted?.();
        break;
      case ComponentLifecycleState.UPDATED:
        this.hooks.onUpdated?.(args[0]);
        break;
      case ComponentLifecycleState.DESTROYED:
        this.hooks.onDestroyed?.();
        break;
    }
  }

  private notifySubscribers(state: ComponentLifecycleState): void {
    const callbacks = this.subscribers.get(state) || [];
    callbacks.forEach((callback) => callback());
  }

  getCurrentState(): ComponentLifecycleState {
    return this.currentState;
  }

  isDestroyed(): boolean {
    return this.currentState === ComponentLifecycleState.DESTROYED;
  }
}
```

## Core Component Examples

### Advanced Button Component

```typescript
export interface ButtonConfig extends ComponentConfig {
  text?: string;
  type?: ButtonType;
  size?: ButtonSize;
  loading?: boolean;
  icon?: Resource;
  iconPosition?: 'left' | 'right';
  block?: boolean;
  ghost?: boolean;
  danger?: boolean;
}

export enum ButtonType {
  PRIMARY = 'primary',
  DEFAULT = 'default',
  DASHED = 'dashed',
  TEXT = 'text',
  LINK = 'link'
}

export enum ButtonSize {
  LARGE = 'large',
  MEDIUM = 'medium',
  SMALL = 'small'
}

@Component
export struct CustomButton {
  @Prop config: ButtonConfig = {};
  @State private isPressed: boolean = false;
  @State private isHovered: boolean = false;
  @State private isFocused: boolean = false;

  private lifecycleManager: ComponentLifecycleManager = new ComponentLifecycleManager();

  aboutToAppear() {
    this.lifecycleManager.setState(ComponentLifecycleState.CREATED);
    this.lifecycleManager.setState(ComponentLifecycleState.MOUNTED);
  }

  aboutToDisappear() {
    this.lifecycleManager.setState(ComponentLifecycleState.DESTROYED);
  }

  private getButtonStyles(): ButtonStyles {
    const baseStyles = this.getBaseStyles();
    const typeStyles = this.getTypeStyles();
    const sizeStyles = this.getSizeStyles();
    const stateStyles = this.getStateStyles();

    return {
      ...baseStyles,
      ...typeStyles,
      ...sizeStyles,
      ...stateStyles
    };
  }

  private getBaseStyles(): Partial<ButtonStyles> {
    return {
      borderRadius: this.config.theme?.borderRadius || 8,
      borderWidth: 1,
      justifyContent: FlexAlign.Center,
      alignItems: HorizontalAlign.Center,
      flexDirection: FlexDirection.Row,
      cursor: this.config.disabled ? 'not-allowed' : 'pointer',
      transition: 'all 0.2s ease'
    };
  }

  private getTypeStyles(): Partial<ButtonStyles> {
    const type = this.config.type || ButtonType.DEFAULT;
    const primaryColor = this.config.theme?.primaryColor || '#007DFF';
    const textColor = this.config.theme?.textColor || '#182431';

    switch (type) {
      case ButtonType.PRIMARY:
        return {
          backgroundColor: this.config.ghost ? 'transparent' : primaryColor,
          borderColor: primaryColor,
          fontColor: this.config.ghost ? primaryColor : '#FFFFFF'
        };
      case ButtonType.DEFAULT:
        return {
          backgroundColor: this.config.ghost ? 'transparent' : '#FFFFFF',
          borderColor: '#D9D9D9',
          fontColor: textColor
        };
      case ButtonType.DASHED:
        return {
          backgroundColor: 'transparent',
          borderColor: '#D9D9D9',
          borderStyle: BorderStyle.Dashed,
          fontColor: textColor
        };
      case ButtonType.TEXT:
        return {
          backgroundColor: 'transparent',
          borderColor: 'transparent',
          fontColor: primaryColor
        };
      case ButtonType.LINK:
        return {
          backgroundColor: 'transparent',
          borderColor: 'transparent',
          fontColor: primaryColor,
          textDecoration: { type: TextDecorationType.Underline }
        };
      default:
        return {};
    }
  }

  private getSizeStyles(): Partial<ButtonStyles> {
    const size = this.config.size || ButtonSize.MEDIUM;

    switch (size) {
      case ButtonSize.LARGE:
        return {
          height: 48,
          fontSize: 16,
          padding: { top: 12, right: 20, bottom: 12, left: 20 }
        };
      case ButtonSize.MEDIUM:
        return {
          height: 40,
          fontSize: 14,
          padding: { top: 8, right: 16, bottom: 8, left: 16 }
        };
      case ButtonSize.SMALL:
        return {
          height: 32,
          fontSize: 12,
          padding: { top: 4, right: 12, bottom: 4, left: 12 }
        };
      default:
        return {};
    }
  }

  private getStateStyles(): Partial<ButtonStyles> {
    if (this.config.disabled) {
      return {
        opacity: 0.6,
        backgroundColor: '#F5F5F5',
        borderColor: '#D9D9D9',
        fontColor: '#BFBFBF'
      };
    }

    if (this.isPressed) {
      return {
        opacity: 0.8,
        transform: { scale: 0.98 }
      };
    }

    if (this.isHovered) {
      return {
        opacity: 0.9,
        elevation: 2
      };
    }

    return {};
  }

  private handleClick(event: ClickEvent): void {
    if (this.config.disabled || this.config.loading) return;

    // Trigger ripple effect
    this.isPressed = true;
    setTimeout(() => {
      this.isPressed = false;
    }, 150);

    // Execute onClick handler
    this.config.onClick?.(event);
  }

  private handleTouch(event: TouchEvent): void {
    if (event.type === TouchType.Down) {
      this.isPressed = true;
    } else if (event.type === TouchType.Up || event.type === TouchType.Cancel) {
      this.isPressed = false;
    }
  }

  @Builder
  buttonContent() {
    Row({ space: 8 }) {
      if (this.config.loading) {
        LoadingProgress()
          .width(16)
          .height(16)
          .color(this.getButtonStyles().fontColor)
      } else if (this.config.icon && this.config.iconPosition !== 'right') {
        Image(this.config.icon)
          .width(16)
          .height(16)
      }

      if (this.config.text) {
        Text(this.config.text)
          .fontSize(this.getButtonStyles().fontSize)
          .fontColor(this.getButtonStyles().fontColor)
          .fontWeight(FontWeight.Medium)
      }

      if (this.config.icon && this.config.iconPosition === 'right') {
        Image(this.config.icon)
          .width(16)
          .height(16)
      }
    }
  }

  build() {
    Button() {
      this.buttonContent()
    }
    .type(ButtonType.Normal)
    .width(this.config.block ? '100%' : undefined)
    .height(this.getButtonStyles().height)
    .backgroundColor(this.getButtonStyles().backgroundColor)
    .borderRadius(this.getButtonStyles().borderRadius)
    .borderWidth(this.getButtonStyles().borderWidth)
    .borderColor(this.getButtonStyles().borderColor)
    .borderStyle(this.getButtonStyles().borderStyle)
    .padding(this.getButtonStyles().padding)
    .opacity(this.getButtonStyles().opacity)
    .enabled(!this.config.disabled && !this.config.loading)
    .onClick((event: ClickEvent) => this.handleClick(event))
    .onTouch((event: TouchEvent) => this.handleTouch(event))
    .onHover((isHover: boolean) => {
      this.isHovered = isHover;
    })
    .onFocus(() => {
      this.isFocused = true;
    })
    .onBlur(() => {
      this.isFocused = false;
    })
  }
}

interface ButtonStyles {
  height: number;
  fontSize: number;
  fontColor: ResourceColor;
  backgroundColor: ResourceColor;
  borderRadius: number;
  borderWidth: number;
  borderColor: ResourceColor;
  borderStyle?: BorderStyle;
  padding: Padding;
  opacity: number;
  elevation?: number;
  cursor?: string;
  transition?: string;
  transform?: any;
  textDecoration?: any;
  justifyContent: FlexAlign;
  alignItems: HorizontalAlign;
  flexDirection: FlexDirection;
}
```

### Advanced Input Component

```typescript
export interface InputConfig extends ComponentConfig {
  value?: string;
  placeholder?: string;
  type?: InputType;
  maxLength?: number;
  multiline?: boolean;
  rows?: number;
  showCounter?: boolean;
  showClearButton?: boolean;
  prefix?: Resource;
  suffix?: Resource;
  validation?: ValidationConfig;
  autoFocus?: boolean;
  readOnly?: boolean;
}

export interface ValidationConfig {
  required?: boolean;
  pattern?: RegExp;
  minLength?: number;
  maxLength?: number;
  customValidator?: (value: string) => string | null;
}

export enum InputType {
  TEXT = 'text',
  PASSWORD = 'password',
  EMAIL = 'email',
  NUMBER = 'number',
  PHONE = 'phone',
  URL = 'url'
}

@Component
export struct CustomInput {
  @Prop config: InputConfig = {};
  @State private inputValue: string = '';
  @State private isFocused: boolean = false;
  @State private errorMessage: string = '';
  @State private showPassword: boolean = false;

  private lifecycleManager: ComponentLifecycleManager = new ComponentLifecycleManager();

  aboutToAppear() {
    this.inputValue = this.config.value || '';
    this.lifecycleManager.setState(ComponentLifecycleState.CREATED);
    this.lifecycleManager.setState(ComponentLifecycleState.MOUNTED);
  }

  aboutToDisappear() {
    this.lifecycleManager.setState(ComponentLifecycleState.DESTROYED);
  }

  private validateInput(value: string): string {
    const validation = this.config.validation;
    if (!validation) return '';

    // Required validation
    if (validation.required && !value.trim()) {
      return 'This field is required';
    }

    // Min length validation
    if (validation.minLength && value.length < validation.minLength) {
      return `Minimum length is ${validation.minLength} characters`;
    }

    // Max length validation
    if (validation.maxLength && value.length > validation.maxLength) {
      return `Maximum length is ${validation.maxLength} characters`;
    }

    // Pattern validation
    if (validation.pattern && !validation.pattern.test(value)) {
      return 'Invalid format';
    }

    // Custom validation
    if (validation.customValidator) {
      const customError = validation.customValidator(value);
      if (customError) return customError;
    }

    return '';
  }

  private getInputType(): InputType {
    if (this.config.type === InputType.PASSWORD && this.showPassword) {
      return InputType.TEXT;
    }
    return this.config.type || InputType.TEXT;
  }

  private handleInputChange(value: string): void {
    this.inputValue = value;
    this.errorMessage = this.validateInput(value);
    this.config.onTextChange?.(value);
  }

  private handleFocus(): void {
    this.isFocused = true;
    this.config.onFocus?.();
  }

  private handleBlur(): void {
    this.isFocused = false;
    this.errorMessage = this.validateInput(this.inputValue);
    this.config.onBlur?.();
  }

  private handleClear(): void {
    this.inputValue = '';
    this.errorMessage = '';
    this.config.onTextChange?.('');
  }

  private togglePasswordVisibility(): void {
    this.showPassword = !this.showPassword;
  }

  private getInputStyles(): InputStyles {
    const theme = this.config.theme;
    const hasError = !!this.errorMessage;

    return {
      borderColor: hasError ? '#FF4D4F' :
                   this.isFocused ? (theme?.primaryColor || '#007DFF') :
                   (theme?.borderColor || '#D9D9D9'),
      borderWidth: this.isFocused ? 2 : 1,
      borderRadius: theme?.borderRadius || 8,
      backgroundColor: this.config.readOnly ? '#F5F5F5' :
                      (theme?.backgroundColor || '#FFFFFF'),
      padding: theme?.padding || { top: 12, right: 16, bottom: 12, left: 16 },
      fontSize: 14,
      fontColor: this.config.readOnly ? '#BFBFBF' :
                 (theme?.textColor || '#182431')
    };
  }

  @Builder
  prefixContent() {
    if (this.config.prefix) {
      Image(this.config.prefix)
        .width(16)
        .height(16)
        .margin({ right: 8 })
    }
  }

  @Builder
  suffixContent() {
    Row({ space: 8 }) {
      if (this.config.showClearButton && this.inputValue.length > 0 && !this.config.readOnly) {
        Image($r('app.media.ic_clear'))
          .width(16)
          .height(16)
          .onClick(() => this.handleClear())
      }

      if (this.config.type === InputType.PASSWORD) {
        Image(this.showPassword ? $r('app.media.ic_eye_off') : $r('app.media.ic_eye'))
          .width(16)
          .height(16)
          .onClick(() => this.togglePasswordVisibility())
      }

      if (this.config.suffix) {
        Image(this.config.suffix)
          .width(16)
          .height(16)
      }
    }
  }

  @Builder
  counterContent() {
    if (this.config.showCounter && this.config.maxLength) {
      Text(`${this.inputValue.length}/${this.config.maxLength}`)
        .fontSize(12)
        .fontColor('#8C8C8C')
        .margin({ top: 4 })
        .alignSelf(ItemAlign.End)
    }
  }

  @Builder
  errorContent() {
    if (this.errorMessage) {
      Text(this.errorMessage)
        .fontSize(12)
        .fontColor('#FF4D4F')
        .margin({ top: 4 })
    }
  }

  build() {
    Column({ space: 4 }) {
      Row() {
        this.prefixContent()

        if (this.config.multiline) {
          TextArea({
            text: this.inputValue,
            placeholder: this.config.placeholder
          })
            .layoutWeight(1)
            .maxLines(this.config.rows || 3)
            .borderWidth(0)
            .backgroundColor('transparent')
            .onChange((value: string) => this.handleInputChange(value))
            .onFocus(() => this.handleFocus())
            .onBlur(() => this.handleBlur())
        } else {
          TextInput({
            text: this.inputValue,
            placeholder: this.config.placeholder
          })
            .layoutWeight(1)
            .type(this.getInputType())
            .maxLength(this.config.maxLength)
            .borderWidth(0)
            .backgroundColor('transparent')
            .caretColor(this.config.theme?.primaryColor || '#007DFF')
            .onChange((value: string) => this.handleInputChange(value))
            .onFocus(() => this.handleFocus())
            .onBlur(() => this.handleBlur())
        }

        this.suffixContent()
      }
      .width('100%')
      .borderWidth(this.getInputStyles().borderWidth)
      .borderColor(this.getInputStyles().borderColor)
      .borderRadius(this.getInputStyles().borderRadius)
      .backgroundColor(this.getInputStyles().backgroundColor)
      .padding(this.getInputStyles().padding)

      this.counterContent()
      this.errorContent()
    }
    .width('100%')
  }
}

interface InputStyles {
  borderColor: ResourceColor;
  borderWidth: number;
  borderRadius: number;
  backgroundColor: ResourceColor;
  padding: Padding;
  fontSize: number;
  fontColor: ResourceColor;
}
```

## Component Composition Patterns

### Higher-Order Components

```typescript
export interface WithLoadingProps {
  loading?: boolean;
  loadingText?: string;
  loadingIndicator?: () => void;
}

export function withLoading<T>(WrappedComponent: T): T {
  @Component
  struct WithLoadingComponent {
    @Prop loading: boolean = false;
    @Prop loadingText: string = 'Loading...';
    @BuilderParam loadingIndicator?: () => void;
    @BuilderParam content: () => void;

    @Builder
    defaultLoadingIndicator() {
      Column({ space: 12 }) {
        LoadingProgress()
          .width(32)
          .height(32)
          .color('#007DFF')

        Text(this.loadingText)
          .fontSize(14)
          .fontColor('#8C8C8C')
      }
      .justifyContent(FlexAlign.Center)
      .alignItems(HorizontalAlign.Center)
    }

    build() {
      if (this.loading) {
        Column() {
          if (this.loadingIndicator) {
            this.loadingIndicator()
          } else {
            this.defaultLoadingIndicator()
          }
        }
        .width('100%')
        .height('100%')
        .justifyContent(FlexAlign.Center)
        .alignItems(HorizontalAlign.Center)
        .backgroundColor('rgba(255, 255, 255, 0.8)')
      } else {
        this.content()
      }
    }
  }

  return WithLoadingComponent as T;
}

// Usage example
const ButtonWithLoading = withLoading(CustomButton);
```

### Render Props Pattern

```typescript
export interface RenderProps<T> {
  children: (data: T) => void;
}

@Component
export struct DataProvider<T> {
  @State private data: T | null = null;
  @State private loading: boolean = false;
  @State private error: string = '';

  @Prop dataSource: () => Promise<T>;
  @BuilderParam children: (data: T | null, loading: boolean, error: string) => void;

  async aboutToAppear() {
    await this.fetchData();
  }

  private async fetchData(): Promise<void> {
    this.loading = true;
    this.error = '';

    try {
      this.data = await this.dataSource();
    } catch (error) {
      this.error = error.message || 'An error occurred';
    } finally {
      this.loading = false;
    }
  }

  async refresh(): Promise<void> {
    await this.fetchData();
  }

  build() {
    this.children(this.data, this.loading, this.error)
  }
}

// Usage example
@Component
struct UserProfile {
  build() {
    DataProvider({
      dataSource: () => fetchUserData(),
      children: (data, loading, error) => {
        if (loading) {
          return LoadingIndicator()
        }
        if (error) {
          return ErrorMessage({ message: error })
        }
        if (data) {
          return UserCard({ user: data })
        }
        return EmptyState()
      }
    })
  }
}
```

## Styling System

### Theme Provider

```typescript
export interface ThemeProviderConfig {
  primary: string;
  secondary: string;
  success: string;
  warning: string;
  error: string;
  info: string;
  background: string;
  surface: string;
  text: {
    primary: string;
    secondary: string;
    disabled: string;
  };
  spacing: {
    xs: number;
    sm: number;
    md: number;
    lg: number;
    xl: number;
  };
  borderRadius: {
    sm: number;
    md: number;
    lg: number;
  };
  shadows: {
    sm: ShadowOptions;
    md: ShadowOptions;
    lg: ShadowOptions;
  };
}

export class ThemeProvider {
  private static instance: ThemeProvider;
  private theme: ThemeProviderConfig;
  private subscribers: ((theme: ThemeProviderConfig) => void)[] = [];

  private constructor() {
    this.theme = this.getDefaultTheme();
  }

  static getInstance(): ThemeProvider {
    if (!this.instance) {
      this.instance = new ThemeProvider();
    }
    return this.instance;
  }

  private getDefaultTheme(): ThemeProviderConfig {
    return {
      primary: "#007DFF",
      secondary: "#19000000",
      success: "#52C41A",
      warning: "#FAAD14",
      error: "#FF4D4F",
      info: "#1890FF",
      background: "#F5F5F5",
      surface: "#FFFFFF",
      text: {
        primary: "#182431",
        secondary: "#8C8C8C",
        disabled: "#BFBFBF",
      },
      spacing: {
        xs: 4,
        sm: 8,
        md: 16,
        lg: 24,
        xl: 32,
      },
      borderRadius: {
        sm: 4,
        md: 8,
        lg: 16,
      },
      shadows: {
        sm: { radius: 2, color: "#00000010", offsetX: 0, offsetY: 1 },
        md: { radius: 8, color: "#00000020", offsetX: 0, offsetY: 2 },
        lg: { radius: 16, color: "#00000030", offsetX: 0, offsetY: 4 },
      },
    };
  }

  getTheme(): ThemeProviderConfig {
    return { ...this.theme };
  }

  updateTheme(newTheme: Partial<ThemeProviderConfig>): void {
    this.theme = { ...this.theme, ...newTheme };
    this.notifySubscribers();
  }

  subscribe(callback: (theme: ThemeProviderConfig) => void): () => void {
    this.subscribers.push(callback);

    // Return unsubscribe function
    return () => {
      const index = this.subscribers.indexOf(callback);
      if (index > -1) {
        this.subscribers.splice(index, 1);
      }
    };
  }

  private notifySubscribers(): void {
    this.subscribers.forEach((callback) => callback(this.theme));
  }

  // Predefined color palettes
  static readonly LIGHT_THEME: Partial<ThemeProviderConfig> = {
    background: "#FFFFFF",
    surface: "#F8F9FA",
    text: {
      primary: "#212529",
      secondary: "#6C757D",
      disabled: "#ADB5BD",
    },
  };

  static readonly DARK_THEME: Partial<ThemeProviderConfig> = {
    background: "#121212",
    surface: "#1E1E1E",
    text: {
      primary: "#FFFFFF",
      secondary: "#B0B0B0",
      disabled: "#666666",
    },
  };
}

// Hook for components to use theme
export function useTheme(): ThemeProviderConfig {
  const themeProvider = ThemeProvider.getInstance();
  return themeProvider.getTheme();
}
```

### Style Utilities

```typescript
export class StyleUtils {
  private static theme = ThemeProvider.getInstance().getTheme();

  static spacing(size: keyof ThemeProviderConfig["spacing"]): number {
    return this.theme.spacing[size];
  }

  static color(colorKey: string): string {
    const keys = colorKey.split(".");
    let value: any = this.theme;

    for (const key of keys) {
      value = value[key];
      if (value === undefined) break;
    }

    return value || "#000000";
  }

  static borderRadius(size: keyof ThemeProviderConfig["borderRadius"]): number {
    return this.theme.borderRadius[size];
  }

  static shadow(size: keyof ThemeProviderConfig["shadows"]): ShadowOptions {
    return this.theme.shadows[size];
  }

  static responsive(mobile: any, tablet?: any, desktop?: any): any {
    // Simple responsive helper - in real implementation,
    // this would detect screen size and return appropriate value
    const screenWidth = 360; // This would be actual screen width

    if (screenWidth < 768) {
      return mobile;
    } else if (screenWidth < 1024) {
      return tablet || mobile;
    } else {
      return desktop || tablet || mobile;
    }
  }

  static createButtonStyle(variant: "primary" | "secondary" | "outline"): any {
    const baseStyle = {
      borderRadius: this.borderRadius("md"),
      padding: {
        top: this.spacing("sm"),
        right: this.spacing("md"),
        bottom: this.spacing("sm"),
        left: this.spacing("md"),
      },
    };

    switch (variant) {
      case "primary":
        return {
          ...baseStyle,
          backgroundColor: this.color("primary"),
          borderColor: this.color("primary"),
          fontColor: "#FFFFFF",
        };
      case "secondary":
        return {
          ...baseStyle,
          backgroundColor: this.color("secondary"),
          borderColor: this.color("secondary"),
          fontColor: this.color("text.primary"),
        };
      case "outline":
        return {
          ...baseStyle,
          backgroundColor: "transparent",
          borderColor: this.color("primary"),
          borderWidth: 1,
          fontColor: this.color("primary"),
        };
      default:
        return baseStyle;
    }
  }
}
```

## Component Documentation

### Component Story System

````typescript
export interface ComponentStory {
  title: string;
  component: any;
  parameters?: {
    docs?: {
      description?: string;
      source?: string;
    };
    controls?: {
      [key: string]: ControlConfig;
    };
  };
}

export interface ControlConfig {
  type: "text" | "number" | "boolean" | "select" | "color";
  defaultValue?: any;
  options?: any[];
  description?: string;
}

export class ComponentStorybook {
  private stories: Map<string, ComponentStory> = new Map();

  addStory(id: string, story: ComponentStory): void {
    this.stories.set(id, story);
  }

  getStory(id: string): ComponentStory | undefined {
    return this.stories.get(id);
  }

  getAllStories(): ComponentStory[] {
    return Array.from(this.stories.values());
  }

  generateDocumentation(): string {
    let documentation = "# Component Library Documentation\n\n";

    this.stories.forEach((story, id) => {
      documentation += `## ${story.title}\n\n`;

      if (story.parameters?.docs?.description) {
        documentation += `${story.parameters.docs.description}\n\n`;
      }

      documentation += "### Props\n\n";
      if (story.parameters?.controls) {
        documentation += "| Prop | Type | Default | Description |\n";
        documentation += "|------|------|---------|-------------|\n";

        Object.entries(story.parameters.controls).forEach(([prop, config]) => {
          documentation += `| ${prop} | ${config.type} | ${
            config.defaultValue || "-"
          } | ${config.description || "-"} |\n`;
        });
      }

      documentation += "\n";

      if (story.parameters?.docs?.source) {
        documentation += "### Example\n\n";
        documentation += "```typescript\n";
        documentation += story.parameters.docs.source;
        documentation += "\n```\n\n";
      }
    });

    return documentation;
  }
}

// Example story definition
const buttonStory: ComponentStory = {
  title: "CustomButton",
  component: CustomButton,
  parameters: {
    docs: {
      description:
        "A customizable button component with various styles and states.",
      source: `
CustomButton({
  config: {
    text: 'Click Me',
    type: ButtonType.PRIMARY,
    size: ButtonSize.MEDIUM,
    onClick: () => console.log('Button clicked')
  }
})
      `,
    },
    controls: {
      text: {
        type: "text",
        defaultValue: "Button",
        description: "The text displayed on the button",
      },
      type: {
        type: "select",
        options: Object.values(ButtonType),
        defaultValue: ButtonType.DEFAULT,
        description: "The button type/style",
      },
      size: {
        type: "select",
        options: Object.values(ButtonSize),
        defaultValue: ButtonSize.MEDIUM,
        description: "The button size",
      },
      disabled: {
        type: "boolean",
        defaultValue: false,
        description: "Whether the button is disabled",
      },
      loading: {
        type: "boolean",
        defaultValue: false,
        description: "Whether the button shows loading state",
      },
    },
  },
};
````

## Testing Framework

### Component Testing Utilities

```typescript
export interface ComponentTestConfig {
  component: any;
  props?: any;
  context?: any;
}

export class ComponentTester {
  private testResults: TestResult[] = [];

  interface TestResult {
    testName: string;
    passed: boolean;
    error?: string;
    duration: number;
  }

  async runTest(testName: string, testFn: () => Promise<void> | void): Promise<void> {
    const startTime = Date.now();

    try {
      await testFn();
      this.testResults.push({
        testName,
        passed: true,
        duration: Date.now() - startTime
      });
    } catch (error) {
      this.testResults.push({
        testName,
        passed: false,
        error: error.message,
        duration: Date.now() - startTime
      });
    }
  }

  expectToRender(config: ComponentTestConfig): void {
    // Mock implementation - in real scenario would use testing framework
    if (!config.component) {
      throw new Error('Component should render');
    }
  }

  expectPropsToWork(config: ComponentTestConfig, propTests: any[]): void {
    propTests.forEach(test => {
      // Mock implementation - test prop changes
      if (!test.expected) {
        throw new Error(`Prop ${test.prop} test failed`);
      }
    });
  }

  expectEventsToWork(config: ComponentTestConfig, eventTests: any[]): void {
    eventTests.forEach(test => {
      // Mock implementation - test event handlers
      if (!test.triggered) {
        throw new Error(`Event ${test.event} was not triggered`);
      }
    });
  }

  getTestResults(): TestResult[] {
    return [...this.testResults];
  }

  getTestSummary(): { total: number; passed: number; failed: number } {
    const total = this.testResults.length;
    const passed = this.testResults.filter(r => r.passed).length;
    const failed = total - passed;

    return { total, passed, failed };
  }

  clearResults(): void {
    this.testResults = [];
  }
}

// Example test suite
export class ButtonTestSuite {
  private tester = new ComponentTester();

  async runAllTests(): Promise<void> {
    await this.testBasicRendering();
    await this.testPropsHandling();
    await this.testEventHandling();
    await this.testStateManagement();
  }

  private async testBasicRendering(): Promise<void> {
    await this.tester.runTest('Button renders correctly', () => {
      this.tester.expectToRender({
        component: CustomButton,
        props: { config: { text: 'Test Button' } }
      });
    });
  }

  private async testPropsHandling(): Promise<void> {
    await this.tester.runTest('Button handles props correctly', () => {
      this.tester.expectPropsToWork({
        component: CustomButton,
        props: { config: { text: 'Test', disabled: true } }
      }, [
        { prop: 'text', expected: true },
        { prop: 'disabled', expected: true }
      ]);
    });
  }

  private async testEventHandling(): Promise<void> {
    await this.tester.runTest('Button handles click events', () => {
      let clicked = false;
      const onClick = () => { clicked = true; };

      this.tester.expectEventsToWork({
        component: CustomButton,
        props: { config: { onClick } }
      }, [
        { event: 'click', triggered: clicked }
      ]);
    });
  }

  private async testStateManagement(): Promise<void> {
    await this.tester.runTest('Button manages state correctly', () => {
      // Test loading state, disabled state, etc.
    });
  }

  getResults(): any {
    return this.tester.getTestSummary();
  }
}
```

## Component Library Packaging

### Build Configuration

```typescript
// build-config.ts
export interface BuildConfig {
  entry: string;
  output: {
    path: string;
    filename: string;
    libraryName: string;
  };
  externals: string[];
  optimization: {
    minify: boolean;
    treeshaking: boolean;
  };
  typescript: {
    declaration: boolean;
    declarationMap: boolean;
  };
}

export const buildConfig: BuildConfig = {
  entry: "./src/index.ts",
  output: {
    path: "./dist",
    filename: "harmony-ui.js",
    libraryName: "HarmonyUI",
  },
  externals: [
    "@ohos.multimedia.media",
    "@ohos.ability.common",
    "@ohos.file.fs",
  ],
  optimization: {
    minify: true,
    treeshaking: true,
  },
  typescript: {
    declaration: true,
    declarationMap: true,
  },
};

// Component library entry point
// src/index.ts
export { CustomButton, ButtonType, ButtonSize } from "./components/Button";
export { CustomInput, InputType } from "./components/Input";
export { ThemeProvider, useTheme } from "./theme/ThemeProvider";
export { StyleUtils } from "./utils/StyleUtils";
export { ComponentTester, ComponentStorybook } from "./testing/TestUtils";

export type { ButtonConfig, InputConfig, ThemeProviderConfig } from "./types";
```

### Distribution Package

```json
{
  "name": "@your-org/harmony-ui",
  "version": "1.0.0",
  "description": "A comprehensive UI component library for HarmonyOS",
  "main": "dist/harmony-ui.js",
  "types": "dist/index.d.ts",
  "files": ["dist", "src", "README.md", "CHANGELOG.md"],
  "scripts": {
    "build": "tsc && webpack --mode production",
    "dev": "webpack serve --mode development",
    "test": "jest",
    "storybook": "start-storybook -p 6006",
    "docs": "typedoc src --out docs",
    "prepublishOnly": "npm run build && npm run test"
  },
  "keywords": ["harmonyos", "ui", "components", "arkts", "typescript"],
  "peerDependencies": {
    "@ohos.multimedia.media": ">=1.0.0",
    "@ohos.ability.common": ">=1.0.0"
  },
  "devDependencies": {
    "typescript": "^4.9.0",
    "webpack": "^5.0.0",
    "jest": "^29.0.0"
  }
}
```

## Best Practices

### 1. Component Design Principles

- Keep components focused and single-purpose
- Design for reusability and composability
- Provide clear and consistent APIs
- Support theming and customization

### 2. Performance Optimization

- Implement efficient re-rendering strategies
- Use proper state management
- Optimize bundle size with tree-shaking
- Lazy load components when possible

### 3. Documentation and Testing

- Write comprehensive component documentation
- Include usage examples and API references
- Implement thorough testing coverage
- Provide migration guides for version updates

### 4. Accessibility

- Follow HarmonyOS accessibility guidelines
- Support keyboard navigation
- Provide proper ARIA labels and roles
- Test with accessibility tools

## Conclusion

Building a comprehensive component library for HarmonyOS requires careful consideration of architecture, design patterns, and developer experience. By following the patterns and practices outlined in this article, you can create robust, reusable components that enhance productivity and maintain consistency across your applications.

The key to a successful component library is balancing flexibility with ease of use, providing comprehensive documentation and testing, and maintaining a clear vision for the library's purpose and scope. Remember to iterate based on user feedback and evolving platform capabilities to keep your library relevant and valuable.
