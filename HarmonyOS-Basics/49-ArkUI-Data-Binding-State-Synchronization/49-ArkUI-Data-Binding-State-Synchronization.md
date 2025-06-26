# ArkUI Data Binding and State Synchronization

## Introduction

ArkUI provides a powerful reactive data binding system that enables automatic UI updates when application state changes. This comprehensive guide explores advanced data binding patterns, state synchronization techniques, and performance optimization strategies for building responsive user interfaces.

## Core Data Binding Concepts

### Reactive State Management

```typescript
// Observable data store
class ObservableStore<T> {
  private _data: T;
  private listeners: Set<(data: T) => void> = new Set();

  constructor(initialData: T) {
    this._data = initialData;
  }

  get data(): T {
    return this._data;
  }

  set data(newData: T) {
    this._data = newData;
    this.notifyListeners();
  }

  update(updater: (data: T) => T): void {
    this.data = updater(this._data);
  }

  subscribe(listener: (data: T) => void): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private notifyListeners(): void {
    this.listeners.forEach((listener) => listener(this._data));
  }
}

// Application state interface
interface AppState {
  user: UserState;
  products: ProductState;
  cart: CartState;
  ui: UIState;
}

interface UserState {
  isAuthenticated: boolean;
  profile: UserProfile | null;
  preferences: UserPreferences;
}

interface ProductState {
  items: Product[];
  categories: Category[];
  loading: boolean;
  filters: ProductFilters;
}

// Global state store
const appStore = new ObservableStore<AppState>({
  user: {
    isAuthenticated: false,
    profile: null,
    preferences: {
      theme: "auto",
      language: "en",
      notifications: true,
    },
  },
  products: {
    items: [],
    categories: [],
    loading: false,
    filters: {
      category: "",
      priceRange: [0, 1000],
      inStock: true,
    },
  },
  cart: {
    items: [],
    total: 0,
    discount: 0,
  },
  ui: {
    activeTab: "home",
    sidebarOpen: false,
    loading: false,
  },
});
```

### Advanced State Decorators

```typescript
// Custom state management decorators
function StateSelector<T, K>(selector: (state: T) => K) {
  return function(target: any, propertyKey: string) {
    const privateKey = `_${propertyKey}`

    Object.defineProperty(target, propertyKey, {
      get() {
        if (!this[privateKey]) {
          this[privateKey] = selector(appStore.data)

          // Subscribe to state changes
          this._unsubscribe = this._unsubscribe || []
          this._unsubscribe.push(
            appStore.subscribe((state) => {
              const newValue = selector(state)
              if (this[privateKey] !== newValue) {
                this[privateKey] = newValue
                // Trigger UI update
                this.markNeedUpdate?.()
              }
            })
          )
        }
        return this[privateKey]
      },
      set(value: K) {
        this[privateKey] = value
      }
    })
  }
}

// Component with state binding
@Component
struct ProductListComponent {
  @StateSelector((state: AppState) => state.products.items)
  products: Product[]

  @StateSelector((state: AppState) => state.products.loading)
  loading: boolean

  @StateSelector((state: AppState) => state.products.filters)
  filters: ProductFilters

  @State private searchText: string = ''

  build() {
    Column() {
      this.buildSearchHeader()
      this.buildFilterBar()
      this.buildProductGrid()
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  buildSearchHeader() {
    Row() {
      TextInput({
        placeholder: 'Search products...',
        text: this.searchText
      })
        .layoutWeight(1)
        .onChange((value: string) => {
          this.searchText = value
          this.filterProducts()
        })

      Button('Search')
        .margin({ left: 8 })
        .onClick(() => this.performSearch())
    }
    .padding(16)
    .width('100%')
  }

  @Builder
  buildFilterBar() {
    Row() {
      Select([
        { value: '', text: 'All Categories' },
        { value: 'electronics', text: 'Electronics' },
        { value: 'clothing', text: 'Clothing' },
        { value: 'books', text: 'Books' }
      ])
        .selected(this.getCategoryIndex())
        .onSelect((index: number) => {
          this.updateFilter('category', this.getCategoryValue(index))
        })

      Checkbox()
        .select(this.filters.inStock)
        .onChange((value: boolean) => {
          this.updateFilter('inStock', value)
        })

      Text('In Stock Only')
        .fontSize(14)
    }
    .padding({ horizontal: 16, vertical: 8 })
    .width('100%')
  }

  @Builder
  buildProductGrid() {
    if (this.loading) {
      LoadingIndicator()
    } else {
      Grid() {
        ForEach(this.getFilteredProducts(), (product: Product) => {
          GridItem() {
            ProductCard({ product: product })
          }
        })
      }
      .columnsTemplate('1fr 1fr')
      .rowsGap(16)
      .columnsGap(16)
      .padding(16)
    }
  }

  private getFilteredProducts(): Product[] {
    return this.products.filter(product => {
      if (this.searchText && !product.name.toLowerCase().includes(this.searchText.toLowerCase())) {
        return false
      }
      if (this.filters.category && product.category !== this.filters.category) {
        return false
      }
      if (this.filters.inStock && !product.inStock) {
        return false
      }
      return true
    })
  }

  private updateFilter(key: keyof ProductFilters, value: any) {
    appStore.update(state => ({
      ...state,
      products: {
        ...state.products,
        filters: {
          ...state.products.filters,
          [key]: value
        }
      }
    }))
  }
}
```

## Bidirectional Data Binding

### Form State Management

```typescript
// Form state manager
class FormStateManager<T extends Record<string, any>> {
  private _values: Partial<T> = {}
  private _errors: Partial<Record<keyof T, string>> = {}
  private _touched: Partial<Record<keyof T, boolean>> = {}
  private listeners: Set<() => void> = new Set()

  get values(): Partial<T> {
    return { ...this._values }
  }

  get errors(): Partial<Record<keyof T, string>> {
    return { ...this._errors }
  }

  get touched(): Partial<Record<keyof T, boolean>> {
    return { ...this._touched }
  }

  setValue<K extends keyof T>(field: K, value: T[K]): void {
    this._values[field] = value
    this._touched[field] = true
    this.validateField(field, value)
    this.notifyListeners()
  }

  setError<K extends keyof T>(field: K, error: string | null): void {
    if (error) {
      this._errors[field] = error
    } else {
      delete this._errors[field]
    }
    this.notifyListeners()
  }

  reset(): void {
    this._values = {}
    this._errors = {}
    this._touched = {}
    this.notifyListeners()
  }

  isValid(): boolean {
    return Object.keys(this._errors).length === 0
  }

  subscribe(listener: () => void): () => void {
    this.listeners.add(listener)
    return () => this.listeners.delete(listener)
  }

  private validateField<K extends keyof T>(field: K, value: T[K]): void {
    // Custom validation logic here
    if (typeof value === 'string' && value.trim() === '') {
      this.setError(field, `${String(field)} is required`)
    } else {
      this.setError(field, null)
    }
  }

  private notifyListeners(): void {
    this.listeners.forEach(listener => listener())
  }
}

// Form component with bidirectional binding
@Component
struct UserProfileForm {
  private formManager = new FormStateManager<UserProfile>()
  @State private formState: any = {}

  aboutToAppear() {
    // Subscribe to form state changes
    this.formManager.subscribe(() => {
      this.formState = {
        values: this.formManager.values,
        errors: this.formManager.errors,
        touched: this.formManager.touched
      }
    })
  }

  build() {
    Column({ space: 16 }) {
      Text('User Profile')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      this.buildFormField('username', 'Username', InputType.Normal)
      this.buildFormField('email', 'Email', InputType.Email)
      this.buildFormField('phone', 'Phone', InputType.PhoneNumber)

      Button('Save Profile')
        .width('100%')
        .enabled(this.formManager.isValid())
        .onClick(() => this.handleSubmit())
    }
    .padding(16)
    .width('100%')
  }

  @Builder
  buildFormField(field: keyof UserProfile, label: string, type: InputType) {
    Column({ space: 4 }) {
      Text(label)
        .fontSize(14)
        .alignSelf(ItemAlign.Start)

      TextInput({
        placeholder: label,
        text: String(this.formState.values?.[field] || '')
      })
        .type(type)
        .onChange((value: string) => {
          this.formManager.setValue(field, value as any)
        })
        .borderColor(this.getFieldBorderColor(field))

      if (this.formState.errors?.[field] && this.formState.touched?.[field]) {
        Text(this.formState.errors[field])
          .fontSize(12)
          .fontColor(Color.Red)
          .alignSelf(ItemAlign.Start)
      }
    }
    .alignItems(HorizontalAlign.Start)
    .width('100%')
  }

  private getFieldBorderColor(field: keyof UserProfile): Color {
    if (this.formState.touched?.[field]) {
      return this.formState.errors?.[field] ? Color.Red : Color.Green
    }
    return Color.Gray
  }

  private handleSubmit() {
    if (this.formManager.isValid()) {
      // Submit form data
      console.log('Form submitted:', this.formManager.values)
    }
  }
}
```

## Performance Optimization

### Efficient State Updates

```typescript
// Optimized state update patterns
class OptimizedStateManager {
  private state: AppState
  private updateQueue: Array<(state: AppState) => AppState> = []
  private isUpdating = false

  constructor(initialState: AppState) {
    this.state = initialState
  }

  // Batch multiple updates for performance
  batchUpdate(updates: Array<(state: AppState) => AppState>): void {
    this.updateQueue.push(...updates)

    if (!this.isUpdating) {
      this.isUpdating = true
      requestAnimationFrame(() => {
        this.processBatchedUpdates()
      })
    }
  }

  // Process all queued updates in a single frame
  private processBatchedUpdates(): void {
    let newState = this.state

    while (this.updateQueue.length > 0) {
      const update = this.updateQueue.shift()!
      newState = update(newState)
    }

    if (newState !== this.state) {
      this.state = newState
      this.notifyComponents()
    }

    this.isUpdating = false
  }

  // Memoized selectors for performance
  private selectorCache = new Map<string, any>()

  select<T>(selector: (state: AppState) => T, cacheKey?: string): T {
    if (cacheKey && this.selectorCache.has(cacheKey)) {
      return this.selectorCache.get(cacheKey)
    }

    const result = selector(this.state)

    if (cacheKey) {
      this.selectorCache.set(cacheKey, result)
    }

    return result
  }

  private notifyComponents(): void {
    // Clear selector cache when state changes
    this.selectorCache.clear()
    // Notify subscribed components
  }
}

// Component with optimized rendering
@Component
struct OptimizedProductList {
  @State private products: Product[] = []
  @State private visibleProducts: Product[] = []
  private virtualScrollController = new Scroller()

  build() {
    List({ scroller: this.virtualScrollController }) {
      LazyForEach(
        this.getVirtualizedData(),
        (product: Product) => product.id,
        (product: Product) => {
          ListItem() {
            ProductCard({ product: product })
          }
          .height(120)
        }
      )
    }
    .width('100%')
    .height('100%')
    .onScrollIndex((start: number, end: number) => {
      this.updateVisibleRange(start, end)
    })
  }

  private getVirtualizedData(): DataSource<Product> {
    // Implement virtual scrolling data source
    return new ProductDataSource(this.products)
  }

  private updateVisibleRange(start: number, end: number): void {
    // Only render visible items for performance
    this.visibleProducts = this.products.slice(start, end + 1)
  }
}

// Virtual data source for large lists
class ProductDataSource implements IDataSource {
  private products: Product[]
  private listeners: DataChangeListener[] = []

  constructor(products: Product[]) {
    this.products = products
  }

  totalCount(): number {
    return this.products.length
  }

  getData(index: number): Product {
    return this.products[index]
  }

  registerDataChangeListener(listener: DataChangeListener): void {
    this.listeners.push(listener)
  }

  unregisterDataChangeListener(listener: DataChangeListener): void {
    const index = this.listeners.indexOf(listener)
    if (index >= 0) {
      this.listeners.splice(index, 1)
    }
  }
}
```

## Conclusion

ArkUI's data binding and state synchronization provide powerful tools for building reactive applications. Key benefits include:

- Automatic UI updates through reactive state management
- Efficient bidirectional data binding for forms
- Performance optimization through batched updates
- Virtual scrolling for large data sets
- Type-safe state management patterns

These patterns enable developers to build responsive, performant applications with clean, maintainable code architecture.
