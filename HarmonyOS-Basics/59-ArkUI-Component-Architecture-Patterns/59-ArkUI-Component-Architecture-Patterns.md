# ArkUI Component Architecture Patterns

## Introduction

Building scalable and maintainable ArkUI applications requires well-designed component architectures. This guide explores advanced architectural patterns, design principles, and best practices for creating robust component systems in HarmonyOS applications.

## Component Design Patterns

### Composite Pattern Implementation

```typescript
interface ComponentNode {
  id: string
  type: string
  render(): void
  addChild(child: ComponentNode): void
  removeChild(id: string): void
  getChildren(): ComponentNode[]
  dispose(): void
}

abstract class BaseComponent implements ComponentNode {
  protected children: Map<string, ComponentNode> = new Map()
  protected parent: ComponentNode | null = null
  protected props: Record<string, any> = {}
  protected state: Record<string, any> = {}

  constructor(
    public readonly id: string,
    public readonly type: string,
    props: Record<string, any> = {}
  ) {
    this.props = { ...props }
  }

  abstract render(): void

  addChild(child: ComponentNode): void {
    this.children.set(child.id, child)
    if (child instanceof BaseComponent) {
      child.parent = this
    }
  }

  removeChild(id: string): void {
    const child = this.children.get(id)
    if (child) {
      child.dispose()
      this.children.delete(id)
      if (child instanceof BaseComponent) {
        child.parent = null
      }
    }
  }

  getChildren(): ComponentNode[] {
    return Array.from(this.children.values())
  }

  updateProps(newProps: Record<string, any>): void {
    const oldProps = { ...this.props }
    this.props = { ...this.props, ...newProps }
    this.onPropsChanged(oldProps, this.props)
  }

  setState(newState: Record<string, any>): void {
    const oldState = { ...this.state }
    this.state = { ...this.state, ...newState }
    this.onStateChanged(oldState, this.state)
    this.scheduleUpdate()
  }

  dispose(): void {
    this.children.forEach(child => child.dispose())
    this.children.clear()
    this.onDispose()
  }

  protected onPropsChanged(oldProps: any, newProps: any): void {
    // Override in subclasses
  }

  protected onStateChanged(oldState: any, newState: any): void {
    // Override in subclasses
  }

  protected onDispose(): void {
    // Override in subclasses
  }

  protected scheduleUpdate(): void {
    requestAnimationFrame(() => {
      this.render()
    })
  }
}

// Container component example
class ContainerComponent extends BaseComponent {
  private layoutEngine: LayoutEngine

  constructor(id: string, props: ContainerProps) {
    super(id, 'container', props)
    this.layoutEngine = new LayoutEngine(props.layout || 'flex')
  }

  render(): void {
    const layout = this.layoutEngine.calculateLayout(
      this.getChildren(),
      this.props.constraints
    )

    this.renderContainer(layout)
    this.renderChildren()
  }

  private renderContainer(layout: LayoutResult): void {
    // Render container with calculated layout
    Column() {
      // Container implementation
    }
    .width(layout.width)
    .height(layout.height)
    .padding(this.props.padding || 0)
    .margin(this.props.margin || 0)
  }

  private renderChildren(): void {
    this.children.forEach(child => {
      child.render()
    })
  }
}

interface ContainerProps {
  layout?: 'flex' | 'grid' | 'stack'
  constraints?: LayoutConstraints
  padding?: number
  margin?: number
}

interface LayoutConstraints {
  minWidth?: number
  maxWidth?: number
  minHeight?: number
  maxHeight?: number
}

interface LayoutResult {
  width: number
  height: number
  children: ChildLayout[]
}

interface ChildLayout {
  id: string
  x: number
  y: number
  width: number
  height: number
}
```

### Observer Pattern for State Management

```typescript
interface Observer<T> {
  update(data: T): void
}

interface Subject<T> {
  subscribe(observer: Observer<T>): () => void
  unsubscribe(observer: Observer<T>): void
  notify(data: T): void
}

class ObservableState<T> implements Subject<T> {
  private observers: Set<Observer<T>> = new Set()
  private _value: T

  constructor(initialValue: T) {
    this._value = initialValue
  }

  get value(): T {
    return this._value
  }

  set value(newValue: T) {
    if (this._value !== newValue) {
      this._value = newValue
      this.notify(newValue)
    }
  }

  subscribe(observer: Observer<T>): () => void {
    this.observers.add(observer)
    return () => this.unsubscribe(observer)
  }

  unsubscribe(observer: Observer<T>): void {
    this.observers.delete(observer)
  }

  notify(data: T): void {
    this.observers.forEach(observer => {
      try {
        observer.update(data)
      } catch (error) {
        console.error('Observer update failed:', error)
      }
    })
  }
}

// State-aware component
abstract class StatefulComponent<TState> extends BaseComponent implements Observer<TState> {
  protected observableState: ObservableState<TState>
  private unsubscribe: (() => void) | null = null

  constructor(
    id: string,
    type: string,
    initialState: TState,
    props: Record<string, any> = {}
  ) {
    super(id, type, props)
    this.observableState = new ObservableState(initialState)
    this.unsubscribe = this.observableState.subscribe(this)
  }

  update(data: TState): void {
    this.onStateUpdate(data)
    this.scheduleUpdate()
  }

  protected updateState(newState: Partial<TState>): void {
    this.observableState.value = { ...this.observableState.value, ...newState }
  }

  protected getState(): TState {
    return this.observableState.value
  }

  protected abstract onStateUpdate(state: TState): void

  dispose(): void {
    if (this.unsubscribe) {
      this.unsubscribe()
      this.unsubscribe = null
    }
    super.dispose()
  }
}

// Example: Counter component
interface CounterState {
  count: number
  step: number
}

class CounterComponent extends StatefulComponent<CounterState> {
  constructor(id: string, props: { initialCount?: number; step?: number }) {
    super(id, 'counter', {
      count: props.initialCount || 0,
      step: props.step || 1
    }, props)
  }

  render(): void {
    const state = this.getState()

    Column() {
      Text(`Count: ${state.count}`)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Row() {
        Button('-')
          .onClick(() => this.decrement())

        Button('+')
          .onClick(() => this.increment())
          .margin({ left: 16 })
      }
      .margin({ top: 16 })
    }
    .padding(16)
  }

  private increment(): void {
    const state = this.getState()
    this.updateState({ count: state.count + state.step })
  }

  private decrement(): void {
    const state = this.getState()
    this.updateState({ count: state.count - state.step })
  }

  protected onStateUpdate(state: CounterState): void {
    console.log('Counter state updated:', state)
  }
}
```

### Factory Pattern for Component Creation

```typescript
interface ComponentFactory {
  createComponent(type: string, id: string, props: any): ComponentNode;
  registerComponentType(type: string, creator: ComponentCreator): void;
  getAvailableTypes(): string[];
}

type ComponentCreator = (id: string, props: any) => ComponentNode;

class ComponentRegistry implements ComponentFactory {
  private creators = new Map<string, ComponentCreator>();
  private instances = new Map<string, ComponentNode>();

  registerComponentType(type: string, creator: ComponentCreator): void {
    this.creators.set(type, creator);
  }

  createComponent(type: string, id: string, props: any): ComponentNode {
    const creator = this.creators.get(type);
    if (!creator) {
      throw new Error(`Unknown component type: ${type}`);
    }

    const component = creator(id, props);
    this.instances.set(id, component);
    return component;
  }

  getComponent(id: string): ComponentNode | undefined {
    return this.instances.get(id);
  }

  destroyComponent(id: string): void {
    const component = this.instances.get(id);
    if (component) {
      component.dispose();
      this.instances.delete(id);
    }
  }

  getAvailableTypes(): string[] {
    return Array.from(this.creators.keys());
  }

  getInstanceCount(): number {
    return this.instances.size;
  }
}

// Component builder for complex creation
class ComponentBuilder {
  private registry: ComponentRegistry;
  private componentSpec: ComponentSpec;

  constructor(registry: ComponentRegistry) {
    this.registry = registry;
    this.componentSpec = {
      type: "",
      id: "",
      props: {},
      children: [],
    };
  }

  ofType(type: string): ComponentBuilder {
    this.componentSpec.type = type;
    return this;
  }

  withId(id: string): ComponentBuilder {
    this.componentSpec.id = id;
    return this;
  }

  withProps(props: Record<string, any>): ComponentBuilder {
    this.componentSpec.props = { ...this.componentSpec.props, ...props };
    return this;
  }

  addChild(childSpec: ComponentSpec): ComponentBuilder {
    this.componentSpec.children.push(childSpec);
    return this;
  }

  build(): ComponentNode {
    if (!this.componentSpec.type || !this.componentSpec.id) {
      throw new Error("Component type and id are required");
    }

    const component = this.registry.createComponent(
      this.componentSpec.type,
      this.componentSpec.id,
      this.componentSpec.props
    );

    // Build and add children
    this.componentSpec.children.forEach((childSpec) => {
      const child = new ComponentBuilder(this.registry)
        .ofType(childSpec.type)
        .withId(childSpec.id)
        .withProps(childSpec.props)
        .build();

      component.addChild(child);
    });

    return component;
  }

  reset(): ComponentBuilder {
    this.componentSpec = {
      type: "",
      id: "",
      props: {},
      children: [],
    };
    return this;
  }
}

interface ComponentSpec {
  type: string;
  id: string;
  props: Record<string, any>;
  children: ComponentSpec[];
}

// Register built-in components
function registerBuiltinComponents(registry: ComponentRegistry): void {
  registry.registerComponentType(
    "container",
    (id, props) => new ContainerComponent(id, props)
  );

  registry.registerComponentType(
    "counter",
    (id, props) => new CounterComponent(id, props)
  );

  registry.registerComponentType(
    "button",
    (id, props) => new ButtonComponent(id, props)
  );

  registry.registerComponentType(
    "text",
    (id, props) => new TextComponent(id, props)
  );
}
```

## Advanced Component Patterns

### Higher-Order Components

```typescript
interface ComponentEnhancer<TProps, TEnhancedProps> {
  (component: ComponentConstructor<TProps>): ComponentConstructor<TEnhancedProps>
}

type ComponentConstructor<TProps> = new (id: string, props: TProps) => ComponentNode

function withLogging<TProps>(
  logLevel: 'info' | 'debug' | 'warn' = 'info'
): ComponentEnhancer<TProps, TProps & { enableLogging?: boolean }> {
  return (WrappedComponent) => {
    return class LoggingComponent extends BaseComponent {
      private wrappedInstance: ComponentNode

      constructor(id: string, props: TProps & { enableLogging?: boolean }) {
        super(id, 'logging-wrapper', props)

        if (props.enableLogging !== false) {
          console.log(`[${logLevel.toUpperCase()}] Creating component: ${id}`)
        }

        this.wrappedInstance = new WrappedComponent(id, props)
      }

      render(): void {
        if (this.props.enableLogging !== false) {
          console.log(`[${logLevel.toUpperCase()}] Rendering component: ${this.id}`)
        }
        this.wrappedInstance.render()
      }

      addChild(child: ComponentNode): void {
        this.wrappedInstance.addChild(child)
      }

      removeChild(id: string): void {
        this.wrappedInstance.removeChild(id)
      }

      getChildren(): ComponentNode[] {
        return this.wrappedInstance.getChildren()
      }

      dispose(): void {
        if (this.props.enableLogging !== false) {
          console.log(`[${logLevel.toUpperCase()}] Disposing component: ${this.id}`)
        }
        this.wrappedInstance.dispose()
        super.dispose()
      }
    }
  }
}

function withErrorBoundary<TProps>(
  fallbackComponent?: ComponentConstructor<any>
): ComponentEnhancer<TProps, TProps & { onError?: (error: Error) => void }> {
  return (WrappedComponent) => {
    return class ErrorBoundaryComponent extends BaseComponent {
      private wrappedInstance: ComponentNode | null = null
      private hasError: boolean = false
      private error: Error | null = null

      constructor(id: string, props: TProps & { onError?: (error: Error) => void }) {
        super(id, 'error-boundary', props)
        this.createWrappedInstance()
      }

      render(): void {
        if (this.hasError) {
          if (fallbackComponent) {
            const fallback = new fallbackComponent(this.id + '-fallback', {
              error: this.error
            })
            fallback.render()
          } else {
            this.renderErrorFallback()
          }
        } else if (this.wrappedInstance) {
          try {
            this.wrappedInstance.render()
          } catch (error) {
            this.handleError(error as Error)
          }
        }
      }

      private createWrappedInstance(): void {
        try {
          this.wrappedInstance = new WrappedComponent(this.id, this.props)
        } catch (error) {
          this.handleError(error as Error)
        }
      }

      private handleError(error: Error): void {
        this.hasError = true
        this.error = error

        if (this.props.onError) {
          this.props.onError(error)
        }

        console.error('Component error boundary caught error:', error)
      }

      private renderErrorFallback(): void {
        Column() {
          Text('Something went wrong')
            .fontSize(16)
            .fontColor(Color.Red)

          Text(this.error?.message || 'Unknown error')
            .fontSize(12)
            .fontColor('#666')
            .margin({ top: 8 })

          Button('Retry')
            .margin({ top: 16 })
            .onClick(() => this.retry())
        }
        .padding(16)
        .justifyContent(FlexAlign.Center)
      }

      private retry(): void {
        this.hasError = false
        this.error = null
        this.createWrappedInstance()
        this.scheduleUpdate()
      }

      addChild(child: ComponentNode): void {
        this.wrappedInstance?.addChild(child)
      }

      removeChild(id: string): void {
        this.wrappedInstance?.removeChild(id)
      }

      getChildren(): ComponentNode[] {
        return this.wrappedInstance?.getChildren() || []
      }

      dispose(): void {
        this.wrappedInstance?.dispose()
        super.dispose()
      }
    }
  }
}

// Usage example
const EnhancedCounter = withErrorBoundary()(withLogging('debug')(CounterComponent))

// Create enhanced component
function createEnhancedCounter(id: string, props: any): ComponentNode {
  return new EnhancedCounter(id, {
    ...props,
    enableLogging: true,
    onError: (error) => console.error('Counter error:', error)
  })
}
```

### Component Composition Utilities

```typescript
interface CompositionConfig {
  mergeProps?: boolean
  validateProps?: (props: any) => boolean
  defaultProps?: Record<string, any>
}

class ComponentComposer {
  static compose<T1, T2, TResult>(
    component1: ComponentConstructor<T1>,
    component2: ComponentConstructor<T2>,
    config: CompositionConfig = {}
  ): ComponentConstructor<T1 & T2> {
    return class ComposedComponent extends BaseComponent {
      private instance1: ComponentNode
      private instance2: ComponentNode

      constructor(id: string, props: T1 & T2) {
        const mergedProps = config.mergeProps
          ? { ...config.defaultProps, ...props }
          : props

        if (config.validateProps && !config.validateProps(mergedProps)) {
          throw new Error('Invalid props for composed component')
        }

        super(id, 'composed', mergedProps)

        this.instance1 = new component1(id + '-1', mergedProps)
        this.instance2 = new component2(id + '-2', mergedProps)
      }

      render(): void {
        Column() {
          this.instance1.render()
          this.instance2.render()
        }
      }

      addChild(child: ComponentNode): void {
        // Add to both instances or implement custom logic
        this.instance1.addChild(child)
      }

      removeChild(id: string): void {
        this.instance1.removeChild(id)
        this.instance2.removeChild(id)
      }

      getChildren(): ComponentNode[] {
        return [
          ...this.instance1.getChildren(),
          ...this.instance2.getChildren()
        ]
      }

      dispose(): void {
        this.instance1.dispose()
        this.instance2.dispose()
        super.dispose()
      }
    }
  }

  static mixin<TBase, TMixin>(
    baseComponent: ComponentConstructor<TBase>,
    mixin: ComponentMixin<TMixin>
  ): ComponentConstructor<TBase & TMixin> {
    return class MixedComponent extends BaseComponent {
      private baseInstance: ComponentNode

      constructor(id: string, props: TBase & TMixin) {
        super(id, 'mixed', props)
        this.baseInstance = new baseComponent(id, props)

        // Apply mixin
        Object.assign(this, mixin)
      }

      render(): void {
        this.baseInstance.render()
      }

      addChild(child: ComponentNode): void {
        this.baseInstance.addChild(child)
      }

      removeChild(id: string): void {
        this.baseInstance.removeChild(id)
      }

      getChildren(): ComponentNode[] {
        return this.baseInstance.getChildren()
      }

      dispose(): void {
        this.baseInstance.dispose()
        super.dispose()
      }
    }
  }
}

interface ComponentMixin<T> {
  [key: string]: any
}

// Example mixins
const draggableMixin = {
  isDragging: false,
  dragOffset: { x: 0, y: 0 },

  startDrag(x: number, y: number): void {
    this.isDragging = true
    this.dragOffset = { x, y }
  },

  updateDrag(x: number, y: number): void {
    if (this.isDragging) {
      this.dragOffset = { x, y }
    }
  },

  endDrag(): void {
    this.isDragging = false
    this.dragOffset = { x: 0, y: 0 }
  }
}

const resizableMixin = {
  isResizing: false,
  size: { width: 100, height: 100 },

  startResize(): void {
    this.isResizing = true
  },

  updateSize(width: number, height: number): void {
    this.size = { width, height }
  },

  endResize(): void {
    this.isResizing = false
  }
}
```

## Conclusion

Advanced component architecture patterns in ArkUI enable:

- Scalable and maintainable component hierarchies
- Flexible composition and reusability patterns
- Robust state management and data flow
- Error handling and resilience mechanisms
- Performance optimization through smart rendering
- Extensible component ecosystems

These patterns provide the foundation for building complex, enterprise-grade applications while maintaining code quality and developer productivity.
