# ArkTS Reactive Patterns and Observables

## Introduction

Reactive programming is a paradigm that focuses on asynchronous data streams and the propagation of change. This guide explores advanced reactive patterns, Observable implementations, and stream processing techniques in ArkTS applications.

## Observable Implementation

### Core Observable System

```typescript
interface Observer<T> {
  next(value: T): void;
  error(error: any): void;
  complete(): void;
}

interface Subscription {
  unsubscribe(): void;
  closed: boolean;
}

interface Subscribable<T> {
  subscribe(observer: Observer<T>): Subscription;
}

class Observable<T> implements Subscribable<T> {
  private _subscribe: (observer: Observer<T>) => (() => void) | void;

  constructor(subscribe: (observer: Observer<T>) => (() => void) | void) {
    this._subscribe = subscribe;
  }

  subscribe(observer: Observer<T>): Subscription;
  subscribe(
    next?: (value: T) => void,
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription;
  subscribe(
    observerOrNext?: Observer<T> | ((value: T) => void),
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription {
    const observer = this.normalizeObserver(observerOrNext, error, complete);
    const subscription = new SubscriptionImpl();

    try {
      const teardown = this._subscribe(observer);
      if (teardown) {
        subscription.addTeardown(teardown);
      }
    } catch (err) {
      observer.error(err);
    }

    return subscription;
  }

  // Transformation operators
  map<R>(project: (value: T) => R): Observable<R> {
    return new Observable<R>((observer) => {
      return this.subscribe({
        next: (value) => {
          try {
            observer.next(project(value));
          } catch (err) {
            observer.error(err);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      }).unsubscribe;
    });
  }

  filter(predicate: (value: T) => boolean): Observable<T> {
    return new Observable<T>((observer) => {
      return this.subscribe({
        next: (value) => {
          try {
            if (predicate(value)) {
              observer.next(value);
            }
          } catch (err) {
            observer.error(err);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      }).unsubscribe;
    });
  }

  flatMap<R>(project: (value: T) => Observable<R>): Observable<R> {
    return new Observable<R>((observer) => {
      const subscriptions: Subscription[] = [];
      let completed = false;
      let outerCompleted = false;

      const checkCompletion = () => {
        if (outerCompleted && subscriptions.length === 0 && !completed) {
          completed = true;
          observer.complete();
        }
      };

      const outerSubscription = this.subscribe({
        next: (value) => {
          try {
            const inner = project(value);
            const innerSubscription = inner.subscribe({
              next: (innerValue) => observer.next(innerValue),
              error: (err) => observer.error(err),
              complete: () => {
                const index = subscriptions.indexOf(innerSubscription);
                if (index >= 0) {
                  subscriptions.splice(index, 1);
                }
                checkCompletion();
              },
            });
            subscriptions.push(innerSubscription);
          } catch (err) {
            observer.error(err);
          }
        },
        error: (err) => observer.error(err),
        complete: () => {
          outerCompleted = true;
          checkCompletion();
        },
      });

      return () => {
        outerSubscription.unsubscribe();
        subscriptions.forEach((sub) => sub.unsubscribe());
      };
    });
  }

  debounce(dueTime: number): Observable<T> {
    return new Observable<T>((observer) => {
      let timeout: number | null = null;
      let lastValue: T;
      let hasValue = false;

      return this.subscribe({
        next: (value) => {
          lastValue = value;
          hasValue = true;

          if (timeout) {
            clearTimeout(timeout);
          }

          timeout = setTimeout(() => {
            if (hasValue) {
              observer.next(lastValue);
              hasValue = false;
            }
          }, dueTime);
        },
        error: (err) => observer.error(err),
        complete: () => {
          if (timeout) {
            clearTimeout(timeout);
          }
          if (hasValue) {
            observer.next(lastValue);
          }
          observer.complete();
        },
      }).unsubscribe;
    });
  }

  throttle(duration: number): Observable<T> {
    return new Observable<T>((observer) => {
      let lastEmission = 0;

      return this.subscribe({
        next: (value) => {
          const now = Date.now();
          if (now - lastEmission >= duration) {
            lastEmission = now;
            observer.next(value);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      }).unsubscribe;
    });
  }

  private normalizeObserver(
    observerOrNext?: Observer<T> | ((value: T) => void),
    error?: (error: any) => void,
    complete?: () => void
  ): Observer<T> {
    if (typeof observerOrNext === "object" && observerOrNext !== null) {
      return observerOrNext as Observer<T>;
    }

    return {
      next: (observerOrNext as (value: T) => void) || (() => {}),
      error:
        error ||
        ((err) => {
          throw err;
        }),
      complete: complete || (() => {}),
    };
  }

  // Static creation methods
  static of<T>(...values: T[]): Observable<T> {
    return new Observable<T>((observer) => {
      values.forEach((value) => observer.next(value));
      observer.complete();
    });
  }

  static from<T>(source: T[] | Promise<T> | Subscribable<T>): Observable<T> {
    if (Array.isArray(source)) {
      return Observable.of(...source);
    }

    if (source && typeof (source as Promise<T>).then === "function") {
      return new Observable<T>((observer) => {
        (source as Promise<T>)
          .then((value) => {
            observer.next(value);
            observer.complete();
          })
          .catch((err) => observer.error(err));
      });
    }

    if (source && typeof (source as Subscribable<T>).subscribe === "function") {
      return new Observable<T>((observer) => {
        return (source as Subscribable<T>).subscribe(observer).unsubscribe;
      });
    }

    throw new Error("Invalid source for Observable.from");
  }

  static interval(period: number): Observable<number> {
    return new Observable<number>((observer) => {
      let count = 0;
      const id = setInterval(() => {
        observer.next(count++);
      }, period);

      return () => clearInterval(id);
    });
  }

  static timer(delay: number, period?: number): Observable<number> {
    return new Observable<number>((observer) => {
      let count = 0;
      const timeoutId = setTimeout(() => {
        observer.next(count++);

        if (period !== undefined) {
          const intervalId = setInterval(() => {
            observer.next(count++);
          }, period);

          return () => clearInterval(intervalId);
        } else {
          observer.complete();
        }
      }, delay);

      return () => clearTimeout(timeoutId);
    });
  }

  static merge<T>(...sources: Observable<T>[]): Observable<T> {
    return new Observable<T>((observer) => {
      const subscriptions: Subscription[] = [];
      let completed = 0;

      sources.forEach((source) => {
        const subscription = source.subscribe({
          next: (value) => observer.next(value),
          error: (err) => observer.error(err),
          complete: () => {
            completed++;
            if (completed === sources.length) {
              observer.complete();
            }
          },
        });
        subscriptions.push(subscription);
      });

      return () => {
        subscriptions.forEach((sub) => sub.unsubscribe());
      };
    });
  }
}

class SubscriptionImpl implements Subscription {
  private _closed = false;
  private teardowns: (() => void)[] = [];

  get closed(): boolean {
    return this._closed;
  }

  unsubscribe(): void {
    if (this._closed) return;

    this._closed = true;
    this.teardowns.forEach((teardown) => {
      try {
        teardown();
      } catch (err) {
        console.error("Error during unsubscription:", err);
      }
    });
    this.teardowns = [];
  }

  addTeardown(teardown: () => void): void {
    if (this._closed) {
      teardown();
    } else {
      this.teardowns.push(teardown);
    }
  }
}
```

### Subject Implementation

```typescript
abstract class Subject<T> extends Observable<T> implements Observer<T> {
  protected observers: Observer<T>[] = [];
  protected isStopped = false;
  protected hasError = false;
  protected thrownError: any = null;

  constructor() {
    super((observer) => {
      if (this.hasError) {
        observer.error(this.thrownError);
        return;
      }

      if (this.isStopped) {
        observer.complete();
        return;
      }

      this.observers.push(observer);

      return () => {
        const index = this.observers.indexOf(observer);
        if (index >= 0) {
          this.observers.splice(index, 1);
        }
      };
    });
  }

  next(value: T): void {
    if (this.isStopped) return;

    this.observers.forEach((observer) => {
      try {
        observer.next(value);
      } catch (err) {
        console.error("Observer error:", err);
      }
    });
  }

  error(error: any): void {
    if (this.isStopped) return;

    this.hasError = true;
    this.thrownError = error;
    this.isStopped = true;

    this.observers.forEach((observer) => {
      try {
        observer.error(error);
      } catch (err) {
        console.error("Observer error handling failed:", err);
      }
    });

    this.observers = [];
  }

  complete(): void {
    if (this.isStopped) return;

    this.isStopped = true;

    this.observers.forEach((observer) => {
      try {
        observer.complete();
      } catch (err) {
        console.error("Observer completion failed:", err);
      }
    });

    this.observers = [];
  }

  asObservable(): Observable<T> {
    return new Observable<T>(
      (observer) => this.subscribe(observer).unsubscribe
    );
  }
}

class BehaviorSubject<T> extends Subject<T> {
  constructor(private _value: T) {
    super();
  }

  get value(): T {
    return this._value;
  }

  next(value: T): void {
    this._value = value;
    super.next(value);
  }

  subscribe(observer: Observer<T>): Subscription;
  subscribe(
    next?: (value: T) => void,
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription {
    const subscription = super.subscribe(observer || next!, error, complete);

    // Emit current value immediately
    if (!this.isStopped && !this.hasError) {
      const normalizedObserver = this.normalizeObserver(
        observer || next!,
        error,
        complete
      );
      try {
        normalizedObserver.next(this._value);
      } catch (err) {
        normalizedObserver.error(err);
      }
    }

    return subscription;
  }

  private normalizeObserver(
    observerOrNext: Observer<T> | ((value: T) => void),
    error?: (error: any) => void,
    complete?: () => void
  ): Observer<T> {
    if (typeof observerOrNext === "object" && observerOrNext !== null) {
      return observerOrNext as Observer<T>;
    }

    return {
      next: observerOrNext as (value: T) => void,
      error: error || (() => {}),
      complete: complete || (() => {}),
    };
  }
}

class ReplaySubject<T> extends Subject<T> {
  private buffer: T[] = [];

  constructor(private bufferSize = Infinity, private windowTime = Infinity) {
    super();
  }

  next(value: T): void {
    this.addToBuffer(value);
    super.next(value);
  }

  subscribe(observer: Observer<T>): Subscription;
  subscribe(
    next?: (value: T) => void,
    error?: (error: any) => void,
    complete?: () => void
  ): Subscription {
    const subscription = super.subscribe(observer || next!, error, complete);

    // Replay buffered values
    if (!this.isStopped && !this.hasError) {
      const normalizedObserver = this.normalizeObserver(
        observer || next!,
        error,
        complete
      );
      this.buffer.forEach((value) => {
        try {
          normalizedObserver.next(value);
        } catch (err) {
          normalizedObserver.error(err);
        }
      });
    }

    return subscription;
  }

  private addToBuffer(value: T): void {
    this.buffer.push(value);

    if (this.buffer.length > this.bufferSize) {
      this.buffer.shift();
    }

    // Time-based cleanup would be implemented here
    // This is a simplified version
  }

  private normalizeObserver(
    observerOrNext: Observer<T> | ((value: T) => void),
    error?: (error: any) => void,
    complete?: () => void
  ): Observer<T> {
    if (typeof observerOrNext === "object" && observerOrNext !== null) {
      return observerOrNext as Observer<T>;
    }

    return {
      next: observerOrNext as (value: T) => void,
      error: error || (() => {}),
      complete: complete || (() => {}),
    };
  }
}
```

## Reactive State Management

### State Store with Observables

```typescript
interface Action {
  type: string
  payload?: any
}

interface Reducer<T> {
  (state: T, action: Action): T
}

interface Middleware<T> {
  (store: Store<T>) => (next: (action: Action) => void) => (action: Action) => void
}

class Store<T> {
  private state$: BehaviorSubject<T>
  private reducer: Reducer<T>
  private middlewares: Middleware<T>[] = []

  constructor(initialState: T, reducer: Reducer<T>) {
    this.state$ = new BehaviorSubject(initialState)
    this.reducer = reducer
  }

  getState(): T {
    return this.state$.value
  }

  getState$(): Observable<T> {
    return this.state$.asObservable()
  }

  dispatch(action: Action): void {
    const dispatch = this.buildDispatchChain()
    dispatch(action)
  }

  select<R>(selector: (state: T) => R): Observable<R> {
    return this.state$.map(selector).distinctUntilChanged()
  }

  addMiddleware(middleware: Middleware<T>): void {
    this.middlewares.push(middleware)
  }

  private buildDispatchChain(): (action: Action) => void {
    const dispatch = (action: Action) => {
      const currentState = this.state$.value
      const newState = this.reducer(currentState, action)
      this.state$.next(newState)
    }

    return this.middlewares.reduceRight(
      (next, middleware) => middleware(this)(next),
      dispatch
    )
  }
}

// Distinct until changed operator
declare module './Observable' {
  interface Observable<T> {
    distinctUntilChanged<K>(keySelector?: (value: T) => K): Observable<T>
  }
}

Observable.prototype.distinctUntilChanged = function<T, K>(
  this: Observable<T>,
  keySelector?: (value: T) => K
): Observable<T> {
  return new Observable<T>(observer => {
    let hasValue = false
    let lastKey: K

    return this.subscribe({
      next: value => {
        const key = keySelector ? keySelector(value) : (value as any)

        if (!hasValue || key !== lastKey) {
          hasValue = true
          lastKey = key
          observer.next(value)
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    }).unsubscribe
  })
}

// Example usage
interface AppState {
  user: User | null
  products: Product[]
  cart: CartItem[]
  loading: boolean
}

interface User {
  id: string
  name: string
  email: string
}

interface Product {
  id: string
  name: string
  price: number
}

interface CartItem {
  productId: string
  quantity: number
}

const initialState: AppState = {
  user: null,
  products: [],
  cart: [],
  loading: false
}

function appReducer(state: AppState, action: Action): AppState {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload }

    case 'SET_PRODUCTS':
      return { ...state, products: action.payload }

    case 'ADD_TO_CART':
      const existingItem = state.cart.find(item => item.productId === action.payload.productId)
      if (existingItem) {
        return {
          ...state,
          cart: state.cart.map(item =>
            item.productId === action.payload.productId
              ? { ...item, quantity: item.quantity + action.payload.quantity }
              : item
          )
        }
      } else {
        return {
          ...state,
          cart: [...state.cart, action.payload]
        }
      }

    case 'SET_LOADING':
      return { ...state, loading: action.payload }

    default:
      return state
  }
}

// Middleware examples
const loggingMiddleware: Middleware<AppState> = store => next => action => {
  console.log('Dispatching action:', action)
  console.log('Current state:', store.getState())
  next(action)
  console.log('New state:', store.getState())
}

const asyncMiddleware: Middleware<AppState> = store => next => action => {
  if (typeof action === 'function') {
    return action(store.dispatch, store.getState)
  }
  return next(action)
}

const store = new Store(initialState, appReducer)
store.addMiddleware(loggingMiddleware)
store.addMiddleware(asyncMiddleware)
```

## Component Integration

### Reactive Component Base

```typescript
abstract class ReactiveComponent<TState> extends BaseComponent {
  protected state$: BehaviorSubject<TState>
  private subscriptions: Subscription[] = []

  constructor(
    id: string,
    type: string,
    initialState: TState,
    props: Record<string, any> = {}
  ) {
    super(id, type, props)
    this.state$ = new BehaviorSubject(initialState)
    this.setupStateSubscription()
  }

  protected getState(): TState {
    return this.state$.value
  }

  protected getState$(): Observable<TState> {
    return this.state$.asObservable()
  }

  protected setState(newState: Partial<TState>): void {
    const currentState = this.state$.value
    const updatedState = { ...currentState, ...newState }
    this.state$.next(updatedState)
  }

  protected select<R>(selector: (state: TState) => R): Observable<R> {
    return this.state$.map(selector).distinctUntilChanged()
  }

  protected addSubscription(subscription: Subscription): void {
    this.subscriptions.push(subscription)
  }

  protected onStateChange(state: TState): void {
    // Override in subclasses
    this.scheduleUpdate()
  }

  private setupStateSubscription(): void {
    const subscription = this.state$.subscribe(state => {
      this.onStateChange(state)
    })
    this.addSubscription(subscription)
  }

  dispose(): void {
    this.subscriptions.forEach(sub => sub.unsubscribe())
    this.subscriptions = []
    super.dispose()
  }
}

// Example reactive component
interface TodoState {
  todos: Todo[]
  filter: 'all' | 'active' | 'completed'
  loading: boolean
}

interface Todo {
  id: string
  text: string
  completed: boolean
  createdAt: Date
}

class TodoListComponent extends ReactiveComponent<TodoState> {
  private todoService: TodoService

  constructor(id: string, props: any) {
    super(id, 'todo-list', {
      todos: [],
      filter: 'all',
      loading: false
    }, props)

    this.todoService = new TodoService()
    this.setupEffects()
    this.loadTodos()
  }

  render(): void {
    const state = this.getState()
    const filteredTodos = this.getFilteredTodos(state.todos, state.filter)

    Column() {
      this.renderHeader()

      if (state.loading) {
        this.renderLoading()
      } else {
        this.renderTodoList(filteredTodos)
      }

      this.renderFilter(state.filter)
    }
    .padding(16)
  }

  private setupEffects(): void {
    // Effect: Auto-save todos when they change
    const saveEffect = this.select(state => state.todos)
      .debounce(500)
      .subscribe(todos => {
        this.todoService.saveTodos(todos)
      })

    this.addSubscription(saveEffect)

    // Effect: Update document title
    const titleEffect = this.select(state => ({
      total: state.todos.length,
      completed: state.todos.filter(t => t.completed).length
    }))
    .subscribe(({ total, completed }) => {
      document.title = `Todos (${completed}/${total})`
    })

    this.addSubscription(titleEffect)
  }

  private async loadTodos(): Promise<void> {
    this.setState({ loading: true })

    try {
      const todos = await this.todoService.loadTodos()
      this.setState({ todos, loading: false })
    } catch (error) {
      this.setState({ loading: false })
      console.error('Failed to load todos:', error)
    }
  }

  private addTodo(text: string): void {
    const newTodo: Todo = {
      id: Date.now().toString(),
      text,
      completed: false,
      createdAt: new Date()
    }

    const currentTodos = this.getState().todos
    this.setState({ todos: [...currentTodos, newTodo] })
  }

  private toggleTodo(id: string): void {
    const currentTodos = this.getState().todos
    const updatedTodos = currentTodos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
    this.setState({ todos: updatedTodos })
  }

  private deleteTodo(id: string): void {
    const currentTodos = this.getState().todos
    const updatedTodos = currentTodos.filter(todo => todo.id !== id)
    this.setState({ todos: updatedTodos })
  }

  private setFilter(filter: 'all' | 'active' | 'completed'): void {
    this.setState({ filter })
  }

  private getFilteredTodos(todos: Todo[], filter: string): Todo[] {
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed)
      case 'completed':
        return todos.filter(todo => todo.completed)
      default:
        return todos
    }
  }

  private renderHeader(): void {
    Row() {
      TextInput({ placeholder: 'Add new todo...' })
        .flexGrow(1)
        .onSubmit((value) => {
          if (value.trim()) {
            this.addTodo(value.trim())
          }
        })
    }
    .margin({ bottom: 16 })
  }

  private renderLoading(): void {
    Text('Loading todos...')
      .fontSize(16)
      .textAlign(TextAlign.Center)
  }

  private renderTodoList(todos: Todo[]): void {
    Column() {
      ForEach(todos, (todo: Todo) => {
        this.renderTodoItem(todo)
      })
    }
  }

  private renderTodoItem(todo: Todo): void {
    Row() {
      Checkbox({ name: todo.id, group: 'todos' })
        .select(todo.completed)
        .onChange((checked) => this.toggleTodo(todo.id))

      Text(todo.text)
        .fontSize(16)
        .decoration({
          type: todo.completed ? TextDecorationType.LineThrough : TextDecorationType.None
        })
        .flexGrow(1)
        .margin({ left: 8 })

      Button('Delete')
        .type(ButtonType.Circle)
        .onClick(() => this.deleteTodo(todo.id))
    }
    .width('100%')
    .padding(8)
    .margin({ bottom: 4 })
  }

  private renderFilter(currentFilter: string): void {
    Row() {
      ['all', 'active', 'completed'].forEach(filter => {
        Button(filter)
          .type(currentFilter === filter ? ButtonType.Emphasized : ButtonType.Normal)
          .onClick(() => this.setFilter(filter as any))
          .margin({ right: 8 })
      })
    }
    .margin({ top: 16 })
    .justifyContent(FlexAlign.Center)
  }
}

class TodoService {
  async loadTodos(): Promise<Todo[]> {
    // Simulate API call
    return new Promise(resolve => {
      setTimeout(() => {
        resolve([
          {
            id: '1',
            text: 'Learn ArkTS',
            completed: false,
            createdAt: new Date()
          },
          {
            id: '2',
            text: 'Build HarmonyOS app',
            completed: true,
            createdAt: new Date()
          }
        ])
      }, 1000)
    })
  }

  async saveTodos(todos: Todo[]): Promise<void> {
    // Simulate API call
    console.log('Saving todos:', todos)
  }
}
```

## Conclusion

Reactive programming with Observables in ArkTS provides:

- Powerful stream processing capabilities
- Declarative handling of asynchronous data
- Elegant composition of complex data flows
- Automatic memory management through subscriptions
- Integration with component lifecycle
- Type-safe reactive state management

These patterns enable building responsive, scalable applications with clean separation of concerns and predictable data flow.
