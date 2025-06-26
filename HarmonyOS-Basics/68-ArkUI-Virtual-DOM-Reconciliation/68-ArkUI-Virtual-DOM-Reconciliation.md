# ArkUI Virtual DOM and Reconciliation

## Introduction

Virtual DOM and reconciliation algorithms are core concepts for optimizing UI rendering performance. This guide explores virtual DOM implementation, diffing algorithms, and reconciliation strategies in ArkUI applications.

## Virtual DOM Implementation

### Virtual Node Structure

```typescript
interface VNode {
  type: string | ComponentType;
  props: Record<string, any>;
  children: VNode[];
  key?: string | number;
  ref?: any;
  element?: any;
}

interface ComponentType {
  new (props: any): Component;
  displayName?: string;
}

class VNodeFactory {
  static createElement(
    type: string | ComponentType,
    props: Record<string, any> = {},
    ...children: (VNode | string | number)[]
  ): VNode {
    return {
      type,
      props: { ...props },
      children: children.map((child) => this.normalizeChild(child)),
      key: props.key,
      ref: props.ref,
    };
  }

  private static normalizeChild(child: VNode | string | number): VNode {
    if (typeof child === "string" || typeof child === "number") {
      return {
        type: "text",
        props: { content: String(child) },
        children: [],
      };
    }
    return child;
  }

  static createTextNode(content: string): VNode {
    return {
      type: "text",
      props: { content },
      children: [],
    };
  }

  static createFragment(children: VNode[]): VNode {
    return {
      type: "fragment",
      props: {},
      children,
    };
  }
}

// JSX-like helper
function h(
  type: string | ComponentType,
  props?: Record<string, any>,
  ...children: any[]
): VNode {
  return VNodeFactory.createElement(type, props, ...children);
}
```

### Diffing Algorithm

```typescript
enum PatchType {
  CREATE = "CREATE",
  REMOVE = "REMOVE",
  REPLACE = "REPLACE",
  UPDATE = "UPDATE",
  REORDER = "REORDER",
}

interface Patch {
  type: PatchType;
  vnode?: VNode;
  element?: any;
  props?: Record<string, any>;
  children?: Patch[];
}

class VirtualDOMDiffer {
  diff(oldVNode: VNode | null, newVNode: VNode | null): Patch[] {
    const patches: Patch[] = [];

    if (!oldVNode && newVNode) {
      patches.push({ type: PatchType.CREATE, vnode: newVNode });
    } else if (oldVNode && !newVNode) {
      patches.push({ type: PatchType.REMOVE, vnode: oldVNode });
    } else if (oldVNode && newVNode) {
      if (this.shouldReplace(oldVNode, newVNode)) {
        patches.push({ type: PatchType.REPLACE, vnode: newVNode });
      } else {
        const propPatches = this.diffProps(oldVNode.props, newVNode.props);
        if (propPatches.length > 0) {
          patches.push({
            type: PatchType.UPDATE,
            props: this.mergePropPatches(propPatches),
          });
        }

        const childPatches = this.diffChildren(
          oldVNode.children,
          newVNode.children
        );
        if (childPatches.length > 0) {
          patches.push({ type: PatchType.UPDATE, children: childPatches });
        }
      }
    }

    return patches;
  }

  private shouldReplace(oldVNode: VNode, newVNode: VNode): boolean {
    return oldVNode.type !== newVNode.type || oldVNode.key !== newVNode.key;
  }

  private diffProps(
    oldProps: Record<string, any>,
    newProps: Record<string, any>
  ): Patch[] {
    const patches: Patch[] = [];
    const allKeys = new Set([
      ...Object.keys(oldProps),
      ...Object.keys(newProps),
    ]);

    allKeys.forEach((key) => {
      const oldValue = oldProps[key];
      const newValue = newProps[key];

      if (oldValue !== newValue) {
        patches.push({
          type: PatchType.UPDATE,
          props: { [key]: newValue },
        });
      }
    });

    return patches;
  }

  private diffChildren(oldChildren: VNode[], newChildren: VNode[]): Patch[] {
    if (this.hasKeys(oldChildren) || this.hasKeys(newChildren)) {
      return this.diffChildrenWithKeys(oldChildren, newChildren);
    } else {
      return this.diffChildrenWithoutKeys(oldChildren, newChildren);
    }
  }

  private diffChildrenWithKeys(
    oldChildren: VNode[],
    newChildren: VNode[]
  ): Patch[] {
    const patches: Patch[] = [];
    const oldKeyMap = this.createKeyMap(oldChildren);
    const newKeyMap = this.createKeyMap(newChildren);

    // Handle moves and updates
    newChildren.forEach((newChild, newIndex) => {
      const key = newChild.key || newIndex;
      const oldChild = oldKeyMap.get(key);

      if (oldChild) {
        const childPatches = this.diff(oldChild, newChild);
        if (childPatches.length > 0) {
          patches.push(...childPatches);
        }
      } else {
        patches.push({ type: PatchType.CREATE, vnode: newChild });
      }
    });

    // Handle removals
    oldChildren.forEach((oldChild) => {
      const key = oldChild.key || oldChildren.indexOf(oldChild);
      if (!newKeyMap.has(key)) {
        patches.push({ type: PatchType.REMOVE, vnode: oldChild });
      }
    });

    return patches;
  }

  private diffChildrenWithoutKeys(
    oldChildren: VNode[],
    newChildren: VNode[]
  ): Patch[] {
    const patches: Patch[] = [];
    const maxLength = Math.max(oldChildren.length, newChildren.length);

    for (let i = 0; i < maxLength; i++) {
      const oldChild = oldChildren[i];
      const newChild = newChildren[i];
      const childPatches = this.diff(oldChild || null, newChild || null);
      patches.push(...childPatches);
    }

    return patches;
  }

  private hasKeys(children: VNode[]): boolean {
    return children.some((child) => child.key !== undefined);
  }

  private createKeyMap(children: VNode[]): Map<string | number, VNode> {
    const keyMap = new Map<string | number, VNode>();
    children.forEach((child, index) => {
      const key = child.key !== undefined ? child.key : index;
      keyMap.set(key, child);
    });
    return keyMap;
  }

  private mergePropPatches(patches: Patch[]): Record<string, any> {
    const merged: Record<string, any> = {};
    patches.forEach((patch) => {
      if (patch.props) {
        Object.assign(merged, patch.props);
      }
    });
    return merged;
  }
}
```

### Reconciler Implementation

```typescript
interface Reconciler {
  render(vnode: VNode, container: any): void;
  patch(patches: Patch[], container: any): void;
}

class ArkUIReconciler implements Reconciler {
  private componentInstances = new Map<VNode, Component>();
  private elementCache = new Map<VNode, any>();

  render(vnode: VNode, container: any): void {
    const element = this.createElementFromVNode(vnode);
    this.mountElement(element, container);
  }

  patch(patches: Patch[], container: any): void {
    patches.forEach((patch) => {
      this.applyPatch(patch, container);
    });
  }

  private createElementFromVNode(vnode: VNode): any {
    if (this.elementCache.has(vnode)) {
      return this.elementCache.get(vnode);
    }

    let element: any;

    if (typeof vnode.type === "string") {
      element = this.createDOMElement(vnode);
    } else if (typeof vnode.type === "function") {
      element = this.createComponentElement(vnode);
    } else {
      throw new Error(`Unknown vnode type: ${vnode.type}`);
    }

    this.elementCache.set(vnode, element);
    return element;
  }

  private createDOMElement(vnode: VNode): any {
    switch (vnode.type) {
      case "text":
        return this.createTextElement(vnode.props.content);
      case "fragment":
        return this.createFragmentElement(vnode.children);
      default:
        return this.createArkUIElement(vnode);
    }
  }

  private createComponentElement(vnode: VNode): any {
    const ComponentClass = vnode.type as ComponentType;
    const instance = new ComponentClass(vnode.props);

    this.componentInstances.set(vnode, instance);

    if (instance.render) {
      const childVNode = instance.render();
      return this.createElementFromVNode(childVNode);
    }

    return null;
  }

  private createArkUIElement(vnode: VNode): any {
    // Map virtual DOM to ArkUI components
    switch (vnode.type) {
      case "Column":
        return this.createColumnElement(vnode);
      case "Row":
        return this.createRowElement(vnode);
      case "Text":
        return this.createTextComponent(vnode);
      case "Button":
        return this.createButtonComponent(vnode);
      default:
        throw new Error(`Unsupported ArkUI component: ${vnode.type}`);
    }
  }

  private createColumnElement(vnode: VNode): any {
    return Column(() => {
      vnode.children.forEach((child) => {
        this.createElementFromVNode(child);
      });
    })
      .width(vnode.props.width || "100%")
      .height(vnode.props.height || "auto")
      .padding(vnode.props.padding || 0)
      .margin(vnode.props.margin || 0);
  }

  private createRowElement(vnode: VNode): any {
    return Row(() => {
      vnode.children.forEach((child) => {
        this.createElementFromVNode(child);
      });
    })
      .width(vnode.props.width || "100%")
      .height(vnode.props.height || "auto")
      .padding(vnode.props.padding || 0)
      .margin(vnode.props.margin || 0);
  }

  private createTextComponent(vnode: VNode): any {
    return Text(vnode.props.content || "")
      .fontSize(vnode.props.fontSize || 16)
      .fontColor(vnode.props.color || "#000000")
      .fontWeight(vnode.props.fontWeight || FontWeight.Normal);
  }

  private createButtonComponent(vnode: VNode): any {
    return Button(vnode.props.text || "Button")
      .onClick(() => {
        if (vnode.props.onClick) {
          vnode.props.onClick();
        }
      })
      .backgroundColor(vnode.props.backgroundColor || "#007AFF")
      .fontColor(vnode.props.fontColor || "#FFFFFF");
  }

  private createTextElement(content: string): any {
    return Text(content);
  }

  private createFragmentElement(children: VNode[]): any {
    return Column(() => {
      children.forEach((child) => {
        this.createElementFromVNode(child);
      });
    });
  }

  private mountElement(element: any, container: any): void {
    // Mount element to container
    if (container && element) {
      // Implementation depends on ArkUI mounting system
    }
  }

  private applyPatch(patch: Patch, container: any): void {
    switch (patch.type) {
      case PatchType.CREATE:
        if (patch.vnode) {
          const element = this.createElementFromVNode(patch.vnode);
          this.mountElement(element, container);
        }
        break;

      case PatchType.REMOVE:
        if (patch.vnode) {
          this.unmountElement(patch.vnode);
        }
        break;

      case PatchType.REPLACE:
        if (patch.vnode) {
          const element = this.createElementFromVNode(patch.vnode);
          this.replaceElement(element, container);
        }
        break;

      case PatchType.UPDATE:
        this.updateElement(patch, container);
        break;
    }
  }

  private unmountElement(vnode: VNode): void {
    const instance = this.componentInstances.get(vnode);
    if (instance && instance.componentWillUnmount) {
      instance.componentWillUnmount();
    }

    this.componentInstances.delete(vnode);
    this.elementCache.delete(vnode);
  }

  private replaceElement(newElement: any, container: any): void {
    // Replace existing element with new one
  }

  private updateElement(patch: Patch, container: any): void {
    if (patch.props) {
      this.updateElementProps(patch.props, container);
    }

    if (patch.children) {
      patch.children.forEach((childPatch) => {
        this.applyPatch(childPatch, container);
      });
    }
  }

  private updateElementProps(props: Record<string, any>, element: any): void {
    // Update element properties
    Object.entries(props).forEach(([key, value]) => {
      this.setElementProperty(element, key, value);
    });
  }

  private setElementProperty(element: any, key: string, value: any): void {
    // Set property on ArkUI element
    switch (key) {
      case "width":
        element.width(value);
        break;
      case "height":
        element.height(value);
        break;
      case "backgroundColor":
        element.backgroundColor(value);
        break;
      case "onClick":
        element.onClick(value);
        break;
      // Add more property mappings as needed
    }
  }
}
```

## Component Lifecycle Integration

### Component Base Class

```typescript
interface ComponentState {
  [key: string]: any;
}

interface ComponentProps {
  [key: string]: any;
}

abstract class Component<P = ComponentProps, S = ComponentState> {
  protected props: P;
  protected state: S;
  private shouldUpdateFlag = true;
  private vnode: VNode | null = null;

  constructor(props: P) {
    this.props = props;
    this.state = {} as S;
  }

  abstract render(): VNode;

  setState(partialState: Partial<S>, callback?: () => void): void {
    const prevState = { ...this.state };
    this.state = { ...this.state, ...partialState };

    if (this.shouldComponentUpdate(this.props, this.state, prevState)) {
      this.scheduleUpdate();
    }

    if (callback) {
      callback();
    }
  }

  shouldComponentUpdate(nextProps: P, nextState: S, prevState: S): boolean {
    return this.shouldUpdateFlag;
  }

  componentDidMount?(): void;
  componentDidUpdate?(prevProps: P, prevState: S): void;
  componentWillUnmount?(): void;

  private scheduleUpdate(): void {
    // Schedule re-render
    requestAnimationFrame(() => {
      this.update();
    });
  }

  private update(): void {
    const newVNode = this.render();
    const patches = new VirtualDOMDiffer().diff(this.vnode, newVNode);

    if (patches.length > 0) {
      const reconciler = new ArkUIReconciler();
      reconciler.patch(patches, this.getContainer());

      this.vnode = newVNode;

      if (this.componentDidUpdate) {
        this.componentDidUpdate(this.props, this.state);
      }
    }
  }

  private getContainer(): any {
    // Get the container for this component
    return null;
  }
}

// Functional component support
type FunctionalComponent<P = {}> = (props: P) => VNode;

function createFunctionalComponent<P>(
  fn: FunctionalComponent<P>
): ComponentType {
  return class extends Component<P> {
    render(): VNode {
      return fn(this.props);
    }
  };
}
```

### Hooks Implementation

```typescript
interface Hook {
  state: any;
  queue: any[];
}

class HooksManager {
  private static instance: HooksManager;
  private currentComponent: Component | null = null;
  private currentHookIndex = 0;
  private hooks: Hook[] = [];

  static getInstance(): HooksManager {
    if (!HooksManager.instance) {
      HooksManager.instance = new HooksManager();
    }
    return HooksManager.instance;
  }

  setCurrentComponent(component: Component): void {
    this.currentComponent = component;
    this.currentHookIndex = 0;
  }

  useState<T>(initialState: T): [T, (newState: T) => void] {
    const hookIndex = this.currentHookIndex++;

    if (!this.hooks[hookIndex]) {
      this.hooks[hookIndex] = {
        state: initialState,
        queue: [],
      };
    }

    const hook = this.hooks[hookIndex];
    const setState = (newState: T) => {
      hook.state = newState;
      if (this.currentComponent) {
        (this.currentComponent as any).forceUpdate();
      }
    };

    return [hook.state, setState];
  }

  useEffect(effect: () => void | (() => void), deps?: any[]): void {
    const hookIndex = this.currentHookIndex++;

    if (!this.hooks[hookIndex]) {
      this.hooks[hookIndex] = {
        state: { cleanup: null, deps: null },
        queue: [],
      };
    }

    const hook = this.hooks[hookIndex];
    const hasChanged =
      !deps ||
      !hook.state.deps ||
      deps.some((dep, i) => dep !== hook.state.deps[i]);

    if (hasChanged) {
      if (hook.state.cleanup) {
        hook.state.cleanup();
      }

      hook.state.cleanup = effect() || null;
      hook.state.deps = deps;
    }
  }

  useMemo<T>(factory: () => T, deps: any[]): T {
    const hookIndex = this.currentHookIndex++;

    if (!this.hooks[hookIndex]) {
      this.hooks[hookIndex] = {
        state: { value: factory(), deps },
        queue: [],
      };
    }

    const hook = this.hooks[hookIndex];
    const hasChanged = deps.some((dep, i) => dep !== hook.state.deps[i]);

    if (hasChanged) {
      hook.state.value = factory();
      hook.state.deps = deps;
    }

    return hook.state.value;
  }

  useCallback<T extends (...args: any[]) => any>(callback: T, deps: any[]): T {
    return this.useMemo(() => callback, deps);
  }

  cleanup(): void {
    this.hooks.forEach((hook) => {
      if (hook.state.cleanup) {
        hook.state.cleanup();
      }
    });
    this.hooks = [];
  }
}

// Hook functions
function useState<T>(initialState: T): [T, (newState: T) => void] {
  return HooksManager.getInstance().useState(initialState);
}

function useEffect(effect: () => void | (() => void), deps?: any[]): void {
  return HooksManager.getInstance().useEffect(effect, deps);
}

function useMemo<T>(factory: () => T, deps: any[]): T {
  return HooksManager.getInstance().useMemo(factory, deps);
}

function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: any[]
): T {
  return HooksManager.getInstance().useCallback(callback, deps);
}
```

## Virtual DOM Application

### Virtual Component Example

```typescript
interface CounterProps {
  initialValue?: number;
  step?: number;
}

class VirtualCounter extends Component<CounterProps> {
  constructor(props: CounterProps) {
    super(props);
    this.state = {
      count: props.initialValue || 0,
    };
  }

  render(): VNode {
    return h("Column", { padding: 16 }, [
      h("Text", {
        content: `Count: ${this.state.count}`,
        fontSize: 24,
        fontWeight: FontWeight.Bold,
      }),
      h("Row", { margin: { top: 16 } }, [
        h("Button", {
          text: "-",
          onClick: () => this.decrement(),
          margin: { right: 8 },
        }),
        h("Button", {
          text: "+",
          onClick: () => this.increment(),
        }),
      ]),
    ]);
  }

  private increment(): void {
    this.setState({
      count: this.state.count + (this.props.step || 1),
    });
  }

  private decrement(): void {
    this.setState({
      count: this.state.count - (this.props.step || 1),
    });
  }
}

// Functional component with hooks
function TodoList({ todos }: { todos: string[] }) {
  const [filter, setFilter] = useState("all");
  const [newTodo, setNewTodo] = useState("");

  const filteredTodos = useMemo(() => {
    return todos.filter((todo) => {
      if (filter === "completed") return todo.includes("✓");
      if (filter === "active") return !todo.includes("✓");
      return true;
    });
  }, [todos, filter]);

  useEffect(() => {
    console.log(`Showing ${filteredTodos.length} todos`);
  }, [filteredTodos]);

  return h("Column", { padding: 16 }, [
    h("Text", {
      content: "Todo List",
      fontSize: 20,
      fontWeight: FontWeight.Bold,
    }),
    h("Row", { margin: { vertical: 16 } }, [
      h("Button", {
        text: "All",
        onClick: () => setFilter("all"),
        backgroundColor: filter === "all" ? "#007AFF" : "#E0E0E0",
      }),
      h("Button", {
        text: "Active",
        onClick: () => setFilter("active"),
        backgroundColor: filter === "active" ? "#007AFF" : "#E0E0E0",
      }),
      h("Button", {
        text: "Completed",
        onClick: () => setFilter("completed"),
        backgroundColor: filter === "completed" ? "#007AFF" : "#E0E0E0",
      }),
    ]),
    h(
      "Column",
      {},
      filteredTodos.map((todo, index) =>
        h("Text", {
          key: index,
          content: todo,
          margin: { bottom: 8 },
        })
      )
    ),
  ]);
}
```

## Conclusion

Virtual DOM and reconciliation in ArkUI provide:

- Efficient UI update mechanisms through virtual representation
- Smart diffing algorithms to minimize actual DOM operations
- Component lifecycle integration for predictable updates
- Hooks support for functional programming patterns
- Performance optimization through batch updates
- Declarative UI programming model

These concepts enable building highly performant and maintainable user interfaces with predictable update behavior and optimal rendering performance.
