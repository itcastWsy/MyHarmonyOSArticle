# 23-State Management Advanced

## Introduction

State management is a crucial aspect of HarmonyOS application development that determines how data flows through your application and how UI components stay synchronized with the underlying data. This article explores advanced state management patterns, best practices, and performance optimization techniques for building scalable ArkTS applications.

## State Management Overview

### State Categories in ArkTS

ArkTS provides several decorators for different state management scenarios:

- `@State`: Local component state
- `@Prop`: One-way data binding from parent to child
- `@Link`: Two-way data binding between components
- `@Provide` and `@Consume`: Cross-component data sharing
- `@Observed` and `@ObjectLink`: Deep observation of complex objects
- `@StorageLink` and `@StorageProp`: Persistent data storage

## Advanced State Patterns

### Global State Management with AppStorage

```typescript
// Global application state
AppStorage.SetOrCreate('userInfo', null);
AppStorage.SetOrCreate('theme', 'light');
AppStorage.SetOrCreate('language', 'en');

// State management service
export class AppStateManager {
  static setUserInfo(user: UserInfo): void {
    AppStorage.Set('userInfo', user);
  }

  static getUserInfo(): UserInfo | null {
    return AppStorage.Get('userInfo');
  }

  static setTheme(theme: 'light' | 'dark'): void {
    AppStorage.Set('theme', theme);
  }

  static getTheme(): string {
    return AppStorage.Get('theme') || 'light';
  }
}

// Component using global state
@Component
struct UserProfile {
  @StorageLink('userInfo') userInfo: UserInfo | null = null;
  @StorageLink('theme') theme: string = 'light';

  build() {
    Column() {
      if (this.userInfo) {
        Text(this.userInfo.name)
          .fontSize(20)
          .fontColor(this.theme === 'dark' ? Color.White : Color.Black)
      } else {
        Text('Please log in')
          .fontColor(Color.Gray)
      }
    }
    .backgroundColor(this.theme === 'dark' ? Color.Black : Color.White)
  }
}
```

### Complex Object State Management

```typescript
// Observable data model
@Observed
class TodoList {
  items: TodoItem[] = [];
  filter: 'all' | 'active' | 'completed' = 'all';

  addItem(text: string): void {
    this.items.push(new TodoItem(text));
  }

  removeItem(id: string): void {
    const index = this.items.findIndex(item => item.id === id);
    if (index > -1) {
      this.items.splice(index, 1);
    }
  }

  toggleItem(id: string): void {
    const item = this.items.find(item => item.id === id);
    if (item) {
      item.completed = !item.completed;
    }
  }

  get filteredItems(): TodoItem[] {
    switch (this.filter) {
      case 'active':
        return this.items.filter(item => !item.completed);
      case 'completed':
        return this.items.filter(item => item.completed);
      default:
        return this.items;
    }
  }
}

@Observed
class TodoItem {
  id: string;
  text: string;
  completed: boolean = false;
  createdAt: Date;

  constructor(text: string) {
    this.id = generateId();
    this.text = text;
    this.createdAt = new Date();
  }
}

// Todo application component
@Component
struct TodoApp {
  @State todoList: TodoList = new TodoList();
  @State newItemText: string = '';

  build() {
    Column() {
      // Add new item
      Row() {
        TextInput({ placeholder: 'Add new task' })
          .layoutWeight(1)
          .onChange((value: string) => {
            this.newItemText = value;
          })

        Button('Add')
          .onClick(() => {
            if (this.newItemText.trim()) {
              this.todoList.addItem(this.newItemText.trim());
              this.newItemText = '';
            }
          })
      }
      .margin({ bottom: 16 })

      // Filter buttons
      Row() {
        Button('All')
          .type(this.todoList.filter === 'all' ? ButtonType.Normal : ButtonType.Outline)
          .onClick(() => {
            this.todoList.filter = 'all';
          })

        Button('Active')
          .type(this.todoList.filter === 'active' ? ButtonType.Normal : ButtonType.Outline)
          .onClick(() => {
            this.todoList.filter = 'active';
          })

        Button('Completed')
          .type(this.todoList.filter === 'completed' ? ButtonType.Normal : ButtonType.Outline)
          .onClick(() => {
            this.todoList.filter = 'completed';
          })
      }
      .justifyContent(FlexAlign.SpaceEvenly)
      .margin({ bottom: 16 })

      // Todo list
      List() {
        ForEach(this.todoList.filteredItems, (item: TodoItem) => {
          ListItem() {
            TodoItemComponent({ item: item, todoList: this.todoList })
          }
        }, (item: TodoItem) => item.id)
      }
    }
    .padding(16)
  }
}

@Component
struct TodoItemComponent {
  @ObjectLink item: TodoItem;
  @Link todoList: TodoList;

  build() {
    Row() {
      Checkbox({ name: 'todo', group: 'todoGroup' })
        .select(this.item.completed)
        .onChange((value: boolean) => {
          this.todoList.toggleItem(this.item.id);
        })

      Text(this.item.text)
        .layoutWeight(1)
        .decoration({
          type: this.item.completed ? TextDecorationType.LineThrough : TextDecorationType.None
        })
        .fontColor(this.item.completed ? Color.Gray : Color.Black)
        .margin({ left: 8 })

      Button('Delete')
        .type(ButtonType.Outline)
        .onClick(() => {
          this.todoList.removeItem(this.item.id);
        })
    }
    .width('100%')
    .padding(12)
    .border({ width: 1, color: Color.Gray })
    .borderRadius(8)
    .margin({ bottom: 8 })
  }
}
```

## Cross-Component Communication

### Provide/Consume Pattern

```typescript
// Shared service
@Observed
class NotificationService {
  notifications: Notification[] = [];

  addNotification(message: string, type: 'info' | 'success' | 'error' = 'info'): void {
    const notification = new Notification(message, type);
    this.notifications.push(notification);

    // Auto remove after 5 seconds
    setTimeout(() => {
      this.removeNotification(notification.id);
    }, 5000);
  }

  removeNotification(id: string): void {
    const index = this.notifications.findIndex(n => n.id === id);
    if (index > -1) {
      this.notifications.splice(index, 1);
    }
  }
}

// Root component providing the service
@Entry
@Component
struct MainApp {
  @Provide notificationService: NotificationService = new NotificationService();

  build() {
    Stack() {
      // Main application content
      Column() {
        HeaderComponent()
        ContentComponent()
        FooterComponent()
      }

      // Notification overlay
      NotificationOverlay()
    }
  }
}

// Component consuming the service
@Component
struct SomeFeatureComponent {
  @Consume notificationService: NotificationService;

  private handleSuccess(): void {
    this.notificationService.addNotification('Operation completed successfully!', 'success');
  }

  private handleError(): void {
    this.notificationService.addNotification('An error occurred', 'error');
  }

  build() {
    Column() {
      Button('Success Action')
        .onClick(() => this.handleSuccess())

      Button('Error Action')
        .onClick(() => this.handleError())
    }
  }
}

// Notification display component
@Component
struct NotificationOverlay {
  @Consume notificationService: NotificationService;

  build() {
    Column() {
      ForEach(this.notificationService.notifications, (notification: Notification) => {
        NotificationItem({
          notification: notification,
          onClose: (id: string) => {
            this.notificationService.removeNotification(id);
          }
        })
      }, (notification: Notification) => notification.id)
    }
    .position({ x: 16, y: 60 })
    .zIndex(1000)
  }
}
```

## Persistent State Management

### Local Storage Integration

```typescript
// Persistent storage service
export class StorageService {
  private static readonly USER_PREFERENCES_KEY = 'user_preferences';
  private static readonly APP_DATA_KEY = 'app_data';

  static async saveUserPreferences(preferences: UserPreferences): Promise<void> {
    try {
      const dataPreferences = await this.getPreferences();
      await dataPreferences.put(this.USER_PREFERENCES_KEY, JSON.stringify(preferences));
      await dataPreferences.flush();
    } catch (error) {
      console.error('Failed to save user preferences:', error);
    }
  }

  static async loadUserPreferences(): Promise<UserPreferences | null> {
    try {
      const dataPreferences = await this.getPreferences();
      const data = await dataPreferences.get(this.USER_PREFERENCES_KEY, '');
      return data ? JSON.parse(data as string) : null;
    } catch (error) {
      console.error('Failed to load user preferences:', error);
      return null;
    }
  }

  private static async getPreferences() {
    return await dataPreferences.getPreferences(getContext(), 'app_storage');
  }
}

// State manager with persistence
@Observed
class AppState {
  userPreferences: UserPreferences = new UserPreferences();
  isLoading: boolean = false;

  async initialize(): Promise<void> {
    this.isLoading = true;
    try {
      const savedPreferences = await StorageService.loadUserPreferences();
      if (savedPreferences) {
        this.userPreferences = savedPreferences;
      }
    } catch (error) {
      console.error('Failed to initialize app state:', error);
    } finally {
      this.isLoading = false;
    }
  }

  async updatePreferences(newPreferences: Partial<UserPreferences>): Promise<void> {
    Object.assign(this.userPreferences, newPreferences);
    await StorageService.saveUserPreferences(this.userPreferences);
  }
}

// Application root with persistent state
@Entry
@Component
struct PersistentApp {
  @Provide appState: AppState = new AppState();

  aboutToAppear(): void {
    this.appState.initialize();
  }

  build() {
    if (this.appState.isLoading) {
      LoadingScreen()
    } else {
      MainContent()
    }
  }
}
```

## State Performance Optimization

### Efficient State Updates

```typescript
// Optimized list state management
@Observed
class OptimizedListState {
  private _items: ListItem[] = [];
  private _selectedIds: Set<string> = new Set();
  private _searchQuery: string = '';

  // Getter with memoization
  private _filteredItems: ListItem[] | null = null;
  private _lastSearchQuery: string = '';

  get items(): ListItem[] {
    return this._items;
  }

  get filteredItems(): ListItem[] {
    if (this._filteredItems === null || this._lastSearchQuery !== this._searchQuery) {
      this._filteredItems = this.computeFilteredItems();
      this._lastSearchQuery = this._searchQuery;
    }
    return this._filteredItems;
  }

  get selectedIds(): Set<string> {
    return this._selectedIds;
  }

  get searchQuery(): string {
    return this._searchQuery;
  }

  setSearchQuery(query: string): void {
    if (this._searchQuery !== query) {
      this._searchQuery = query;
      this._filteredItems = null; // Invalidate cache
    }
  }

  addItem(item: ListItem): void {
    this._items.push(item);
    this._filteredItems = null; // Invalidate cache
  }

  removeItem(id: string): void {
    const index = this._items.findIndex(item => item.id === id);
    if (index > -1) {
      this._items.splice(index, 1);
      this._selectedIds.delete(id);
      this._filteredItems = null; // Invalidate cache
    }
  }

  toggleSelection(id: string): void {
    if (this._selectedIds.has(id)) {
      this._selectedIds.delete(id);
    } else {
      this._selectedIds.add(id);
    }
  }

  private computeFilteredItems(): ListItem[] {
    if (!this._searchQuery.trim()) {
      return this._items;
    }

    const query = this._searchQuery.toLowerCase();
    return this._items.filter(item =>
      item.title.toLowerCase().includes(query) ||
      item.description.toLowerCase().includes(query)
    );
  }
}

// Component with optimized rendering
@Component
struct OptimizedList {
  @State listState: OptimizedListState = new OptimizedListState();

  build() {
    Column() {
      // Search input - only updates search query
      SearchInput({
        query: this.listState.searchQuery,
        onQueryChange: (query: string) => {
          this.listState.setSearchQuery(query);
        }
      })

      // List with minimal re-renders
      List() {
        LazyForEach(
          new OptimizedDataSource(this.listState.filteredItems),
          (item: ListItem) => {
            ListItem() {
              OptimizedListItem({
                item: item,
                isSelected: this.listState.selectedIds.has(item.id),
                onToggleSelection: () => {
                  this.listState.toggleSelection(item.id);
                }
              })
            }
          },
          (item: ListItem) => item.id
        )
      }
    }
  }
}
```

## Best Practices

### 1. State Structure Design

- Keep state flat and normalized
- Separate UI state from business logic
- Use immutable update patterns
- Implement proper data validation

### 2. Performance Guidelines

- Minimize state updates frequency
- Use computed properties for derived state
- Implement proper key functions for lists
- Avoid unnecessary component re-renders

### 3. Memory Management

- Clean up subscriptions and timers
- Implement proper disposal patterns
- Use weak references where appropriate
- Monitor memory usage in large applications

## Conclusion

Advanced state management in ArkTS requires understanding various patterns and choosing the right approach for each scenario. By implementing proper state architecture, utilizing performance optimization techniques, and following best practices, you can build scalable and maintainable HarmonyOS applications.

Key takeaways:

1. **Choose the Right Pattern**: Use appropriate state management decorators for different scenarios
2. **Optimize Performance**: Implement efficient update strategies and minimize re-renders
3. **Maintain Data Integrity**: Use type-safe approaches and proper validation
4. **Plan for Scale**: Design state architecture that can grow with your application
5. **Handle Persistence**: Implement proper data persistence strategies for user experience
