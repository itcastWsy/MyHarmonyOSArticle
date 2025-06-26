# 37. ArkTS Reactive Programming and State Management

## Introduction

Reactive programming in ArkTS revolutionizes how we handle data flow and state management in HarmonyOS applications. This article explores advanced reactive patterns, observable data structures, and efficient state management techniques using ArkTS's reactive programming paradigm.

## Core Reactive Concepts

### Observable Data Structures

```typescript
// Basic Observable Implementation
export interface Observer<T> {
  next(value: T): void;
  error?(error: any): void;
  complete?(): void;
}

export interface Subscription {
  unsubscribe(): void;
}

export class Observable<T> {
  private observers: Observer<T>[] = [];

  constructor(
    private producer: (observer: Observer<T>) => void | (() => void)
  ) {}

  subscribe(observer: Observer<T>): Subscription {
    this.observers.push(observer);

    const cleanup = this.producer(observer);

    return {
      unsubscribe: () => {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
          this.observers.splice(index, 1);
        }
        if (cleanup && typeof cleanup === "function") {
          cleanup();
        }
      },
    };
  }

  static fromArray<T>(array: T[]): Observable<T> {
    return new Observable<T>((observer) => {
      array.forEach((item) => observer.next(item));
      observer.complete?.();
    });
  }

  static fromEvent<T>(target: any, eventName: string): Observable<T> {
    return new Observable<T>((observer) => {
      const handler = (event: T) => observer.next(event);
      target.addEventListener(eventName, handler);

      return () => target.removeEventListener(eventName, handler);
    });
  }

  map<U>(transform: (value: T) => U): Observable<U> {
    return new Observable<U>((observer) => {
      return this.subscribe({
        next: (value) => observer.next(transform(value)),
        error: (error) => observer.error?.(error),
        complete: () => observer.complete?.(),
      });
    });
  }

  filter(predicate: (value: T) => boolean): Observable<T> {
    return new Observable<T>((observer) => {
      return this.subscribe({
        next: (value) => predicate(value) && observer.next(value),
        error: (error) => observer.error?.(error),
        complete: () => observer.complete?.(),
      });
    });
  }

  debounceTime(delay: number): Observable<T> {
    return new Observable<T>((observer) => {
      let timeoutId: number;

      return this.subscribe({
        next: (value) => {
          clearTimeout(timeoutId);
          timeoutId = setTimeout(() => observer.next(value), delay);
        },
        error: (error) => observer.error?.(error),
        complete: () => observer.complete?.(),
      });
    });
  }
}
```

### Reactive State Store

```typescript
export interface StoreState {
  [key: string]: any;
}

export interface ActionBase {
  type: string;
  payload?: any;
}

export type Reducer<T extends StoreState> = (state: T, action: ActionBase) => T;

export class ReactiveStore<T extends StoreState> {
  private state: T;
  private stateSubject: Observable<T>;
  private observers: Observer<T>[] = [];
  private middlewares: Array<
    (action: ActionBase, state: T) => ActionBase | void
  > = [];

  constructor(private initialState: T, private reducer: Reducer<T>) {
    this.state = { ...initialState };
    this.stateSubject = new Observable<T>((observer) => {
      this.observers.push(observer);
      observer.next(this.state);

      return () => {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
          this.observers.splice(index, 1);
        }
      };
    });
  }

  getState(): T {
    return { ...this.state };
  }

  select<K>(selector: (state: T) => K): Observable<K> {
    return this.stateSubject.map(selector).filter((current, index, values) => {
      // Simple distinctUntilChanged implementation
      return index === 0 || current !== values[index - 1];
    });
  }

  dispatch(action: ActionBase): void {
    let processedAction = action;

    // Apply middlewares
    for (const middleware of this.middlewares) {
      const result = middleware(processedAction, this.state);
      if (result) {
        processedAction = result;
      }
    }

    const newState = this.reducer(this.state, processedAction);

    if (newState !== this.state) {
      this.state = newState;
      this.notifyObservers();
    }
  }

  subscribe(observer: Observer<T>): Subscription {
    return this.stateSubject.subscribe(observer);
  }

  addMiddleware(
    middleware: (action: ActionBase, state: T) => ActionBase | void
  ): void {
    this.middlewares.push(middleware);
  }

  private notifyObservers(): void {
    this.observers.forEach((observer) => observer.next(this.state));
  }
}
```

## ArkUI Integration

### Reactive Component Base

```typescript
@Component
export struct ReactiveComponent {
  @State private storeSubscription?: Subscription;
  @State private componentState: any = {};

  aboutToAppear() {
    this.initializeReactiveBinding();
  }

  aboutToDisappear() {
    this.storeSubscription?.unsubscribe();
  }

  private initializeReactiveBinding(): void {
    // Override in derived components
  }

  protected bindToStore<T>(
    observable: Observable<T>,
    stateKey: string
  ): void {
    this.storeSubscription = observable.subscribe({
      next: (value) => {
        this.componentState = {
          ...this.componentState,
          [stateKey]: value
        };
      },
      error: (error) => console.error('Store binding error:', error)
    });
  }

  build() {
    // Override in derived components
  }
}
```

### Advanced State Management Example

```typescript
// Define application state interface
interface AppState extends StoreState {
  user: {
    id: string;
    name: string;
    preferences: UserPreferences;
  } | null;
  todos: Todo[];
  ui: {
    loading: boolean;
    theme: "light" | "dark";
    notifications: Notification[];
  };
}

interface Todo {
  id: string;
  title: string;
  completed: boolean;
  createdAt: number;
}

interface UserPreferences {
  language: string;
  notifications: boolean;
}

interface Notification {
  id: string;
  type: "info" | "warning" | "error" | "success";
  message: string;
  timestamp: number;
}

// Action definitions
const TodoActions = {
  ADD_TODO: "ADD_TODO",
  TOGGLE_TODO: "TOGGLE_TODO",
  REMOVE_TODO: "REMOVE_TODO",
  LOAD_TODOS: "LOAD_TODOS",
} as const;

const UIActions = {
  SET_LOADING: "SET_LOADING",
  TOGGLE_THEME: "TOGGLE_THEME",
  ADD_NOTIFICATION: "ADD_NOTIFICATION",
  REMOVE_NOTIFICATION: "REMOVE_NOTIFICATION",
} as const;

// Reducer implementation
const appReducer: Reducer<AppState> = (state, action) => {
  switch (action.type) {
    case TodoActions.ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };

    case TodoActions.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id
            ? { ...todo, completed: !todo.completed }
            : todo
        ),
      };

    case TodoActions.REMOVE_TODO:
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload.id),
      };

    case UIActions.SET_LOADING:
      return {
        ...state,
        ui: { ...state.ui, loading: action.payload },
      };

    case UIActions.TOGGLE_THEME:
      return {
        ...state,
        ui: {
          ...state.ui,
          theme: state.ui.theme === "light" ? "dark" : "light",
        },
      };

    case UIActions.ADD_NOTIFICATION:
      return {
        ...state,
        ui: {
          ...state.ui,
          notifications: [...state.ui.notifications, action.payload],
        },
      };

    default:
      return state;
  }
};

// Store initialization
const initialState: AppState = {
  user: null,
  todos: [],
  ui: {
    loading: false,
    theme: "light",
    notifications: [],
  },
};

export const appStore = new ReactiveStore(initialState, appReducer);

// Add logging middleware
appStore.addMiddleware((action, state) => {
  console.log("Action dispatched:", action.type, action.payload);
  console.log("Previous state:", state);
  return action;
});
```

## Reactive UI Components

### Todo List Component with Reactive State

```typescript
@Component
export struct TodoListComponent extends ReactiveComponent {
  @State todos: Todo[] = [];
  @State loading: boolean = false;
  @State newTodoTitle: string = '';

  private initializeReactiveBinding(): void {
    // Bind to todos from store
    this.bindToStore(
      appStore.select(state => state.todos),
      'todos'
    );

    // Bind to loading state
    this.bindToStore(
      appStore.select(state => state.ui.loading),
      'loading'
    );

    // Create derived observables
    const todoStats = appStore.select(state => ({
      total: state.todos.length,
      completed: state.todos.filter(t => t.completed).length,
      pending: state.todos.filter(t => !t.completed).length
    }));

    todoStats.subscribe({
      next: (stats) => {
        console.log('Todo statistics updated:', stats);
      }
    });
  }

  private addTodo(): void {
    if (this.newTodoTitle.trim()) {
      const newTodo: Todo = {
        id: Date.now().toString(),
        title: this.newTodoTitle.trim(),
        completed: false,
        createdAt: Date.now()
      };

      appStore.dispatch({
        type: TodoActions.ADD_TODO,
        payload: newTodo
      });

      this.newTodoTitle = '';
    }
  }

  private toggleTodo(todo: Todo): void {
    appStore.dispatch({
      type: TodoActions.TOGGLE_TODO,
      payload: { id: todo.id }
    });
  }

  private removeTodo(todo: Todo): void {
    appStore.dispatch({
      type: TodoActions.REMOVE_TODO,
      payload: { id: todo.id }
    });
  }

  build() {
    Column({ space: 16 }) {
      // Header
      Row() {
        Text('Reactive Todo List')
          .fontSize(24)
          .fontWeight(FontWeight.Bold)

        if (this.loading) {
          LoadingProgress()
            .width(20)
            .height(20)
        }
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)

      // Add new todo
      Row({ space: 8 }) {
        TextInput({
          placeholder: 'Enter new todo...',
          text: this.newTodoTitle
        })
          .onChange((value) => {
            this.newTodoTitle = value;
          })
          .onSubmit(() => this.addTodo())
          .layoutWeight(1)

        Button('Add')
          .onClick(() => this.addTodo())
          .enabled(this.newTodoTitle.trim().length > 0)
      }
      .width('100%')

      // Todo list
      List({ space: 8 }) {
        ForEach(this.todos, (todo: Todo) => {
          ListItem() {
            Row({ space: 12 }) {
              Checkbox({
                name: todo.id,
                group: 'todos'
              })
                .select(todo.completed)
                .onChange((isChecked) => {
                  this.toggleTodo(todo);
                })

              Text(todo.title)
                .fontSize(16)
                .decoration({
                  type: todo.completed
                    ? TextDecorationType.LineThrough
                    : TextDecorationType.None
                })
                .fontColor(todo.completed ? Color.Gray : Color.Black)
                .layoutWeight(1)

              Button('Delete')
                .type(ButtonType.Circle)
                .width(32)
                .height(32)
                .fontSize(12)
                .backgroundColor(Color.Red)
                .onClick(() => this.removeTodo(todo))
            }
            .width('100%')
            .padding(8)
            .backgroundColor(Color.White)
            .borderRadius(8)
          }
        }, (todo: Todo) => todo.id)
      }
      .layoutWeight(1)
      .width('100%')
    }
    .width('100%')
    .height('100%')
    .padding(16)
    .backgroundColor('#f5f5f5')
  }
}
```

## Advanced Reactive Patterns

### Reactive Form Validation

```typescript
interface FormField<T> {
  value: T;
  validators: Array<(value: T) => string | null>;
  errors: string[];
  touched: boolean;
  valid: boolean;
}

export class ReactiveForm<T extends Record<string, any>> {
  private fieldsSubject: Observable<Record<keyof T, FormField<any>>>;
  private fields: Record<keyof T, FormField<any>>;
  private observers: Observer<Record<keyof T, FormField<any>>>[] = [];

  constructor(initialData: T) {
    this.fields = {} as Record<keyof T, FormField<any>>;

    Object.keys(initialData).forEach((key) => {
      this.fields[key as keyof T] = {
        value: initialData[key as keyof T],
        validators: [],
        errors: [],
        touched: false,
        valid: true,
      };
    });

    this.fieldsSubject = new Observable<Record<keyof T, FormField<any>>>(
      (observer) => {
        this.observers.push(observer);
        observer.next(this.fields);

        return () => {
          const index = this.observers.indexOf(observer);
          if (index > -1) {
            this.observers.splice(index, 1);
          }
        };
      }
    );
  }

  addValidator<K extends keyof T>(
    fieldName: K,
    validator: (value: T[K]) => string | null
  ): void {
    this.fields[fieldName].validators.push(validator);
    this.validateField(fieldName);
  }

  setValue<K extends keyof T>(fieldName: K, value: T[K]): void {
    this.fields[fieldName].value = value;
    this.fields[fieldName].touched = true;
    this.validateField(fieldName);
    this.notifyObservers();
  }

  getField<K extends keyof T>(fieldName: K): Observable<FormField<T[K]>> {
    return this.fieldsSubject.map((fields) => fields[fieldName]);
  }

  isFormValid(): Observable<boolean> {
    return this.fieldsSubject.map((fields) =>
      Object.values(fields).every((field) => field.valid)
    );
  }

  getFormData(): T {
    const data: Partial<T> = {};
    Object.keys(this.fields).forEach((key) => {
      data[key as keyof T] = this.fields[key as keyof T].value;
    });
    return data as T;
  }

  private validateField<K extends keyof T>(fieldName: K): void {
    const field = this.fields[fieldName];
    const errors: string[] = [];

    field.validators.forEach((validator) => {
      const error = validator(field.value);
      if (error) {
        errors.push(error);
      }
    });

    field.errors = errors;
    field.valid = errors.length === 0;
  }

  private notifyObservers(): void {
    this.observers.forEach((observer) => observer.next(this.fields));
  }

  subscribe(observer: Observer<Record<keyof T, FormField<any>>>): Subscription {
    return this.fieldsSubject.subscribe(observer);
  }
}
```

## Performance Optimization

### Memoization and Selective Updates

```typescript
export class MemoizedObservable<T> extends Observable<T> {
  private lastValue: T | undefined;
  private hasValue = false;

  constructor(
    producer: (observer: Observer<T>) => void | (() => void),
    private compareFn: (a: T, b: T) => boolean = (a, b) => a === b
  ) {
    super((observer) => {
      if (this.hasValue) {
        observer.next(this.lastValue!);
      }

      return producer({
        next: (value) => {
          if (!this.hasValue || !this.compareFn(value, this.lastValue!)) {
            this.lastValue = value;
            this.hasValue = true;
            observer.next(value);
          }
        },
        error: observer.error?.bind(observer),
        complete: observer.complete?.bind(observer),
      });
    });
  }
}

// Usage with complex objects
const complexDataStream = new MemoizedObservable<UserProfile>(
  (observer) => {
    // Producer logic here
  },
  (a, b) => JSON.stringify(a) === JSON.stringify(b) // Deep comparison
);
```

## Best Practices

1. **Unsubscribe Properly**: Always unsubscribe from observables in component lifecycle methods
2. **Use Selectors**: Leverage store selectors to prevent unnecessary re-renders
3. **Immutable Updates**: Ensure state updates are immutable to enable proper change detection
4. **Error Handling**: Implement comprehensive error handling in reactive streams
5. **Memory Management**: Monitor subscription lifecycles to prevent memory leaks
6. **Performance**: Use memoization and selective updates for complex data transformations

## Conclusion

Reactive programming in ArkTS provides powerful tools for building responsive, maintainable HarmonyOS applications. By leveraging observables, reactive state management, and proper integration with ArkUI, developers can create applications that efficiently handle complex data flows while maintaining excellent performance and user experience.

The reactive paradigm enables declarative programming patterns that make code more predictable and easier to test, ultimately leading to more robust HarmonyOS applications.
