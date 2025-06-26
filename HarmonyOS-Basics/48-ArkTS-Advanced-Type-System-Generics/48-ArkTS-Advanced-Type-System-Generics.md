# ArkTS Advanced Type System and Generics

## Introduction

HarmonyOS ArkTS provides a powerful type system that extends TypeScript with enhanced type safety and performance optimizations. This guide explores advanced type concepts, generic programming patterns, and type-safe development practices.

## Understanding ArkTS Type System

### Enhanced Type Safety

```typescript
interface UserProfile {
  readonly id: string;
  username: string;
  email: string;
  preferences: UserPreferences;
}

interface UserPreferences {
  theme: "light" | "dark" | "auto";
  language: string;
  notifications: NotificationSettings;
}

// Generic repository pattern
class BaseRepository<T extends { id: K }, K = string> {
  protected items: Map<K, T> = new Map();

  findById(id: K): T | null {
    return this.items.get(id) || null;
  }

  create(entityData: Omit<T, "id">): T {
    const id = this.generateId();
    const entity = { ...entityData, id } as T;
    this.items.set(id, entity);
    return entity;
  }

  protected generateId(): K {
    return `${Date.now()}` as K;
  }
}
```

## Advanced Generic Patterns

### Conditional Types and Mapped Types

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

type ApiResponse<T> = {
  data: T;
  status: "success" | "error";
  message?: string;
  timestamp: number;
};

// Generic event system
class TypedEventEmitter<T extends Record<string, any>> {
  private listeners: {
    [K in keyof T]?: Array<(payload: T[K]) => void>;
  } = {};

  on<K extends keyof T>(event: K, listener: (payload: T[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }

  emit<K extends keyof T>(event: K, payload: T[K]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach((listener) => listener(payload));
    }
  }
}
```

## ArkUI Integration with Advanced Types

### Generic Component Development

```typescript
interface TableColumn<T> {
  key: keyof T
  title: string
  render?: (value: T[keyof T], record: T) => string
  sortable?: boolean
}

@Component
struct DataTable<T extends Record<string, any>> {
  @Prop data: T[]
  @Prop columns: TableColumn<T>[]
  @State sortColumn?: keyof T
  @State sortDirection: 'asc' | 'desc' = 'asc'

  build() {
    List() {
      ForEach(this.columns, (column: TableColumn<T>) => {
        Text(column.title)
          .fontSize(14)
          .fontWeight(FontWeight.Bold)
      })

      ForEach(this.getSortedData(), (record: T) => {
        Row() {
          ForEach(this.columns, (column: TableColumn<T>) => {
            Text(this.getCellValue(column, record))
              .fontSize(12)
          })
        }
      })
    }
  }

  private getCellValue(column: TableColumn<T>, record: T): string {
    const value = record[column.key]
    return column.render ? column.render(value, record) : String(value)
  }

  private getSortedData(): T[] {
    if (!this.sortColumn) return this.data

    return [...this.data].sort((a, b) => {
      const aVal = a[this.sortColumn!]
      const bVal = b[this.sortColumn!]
      return this.sortDirection === 'asc'
        ? (aVal < bVal ? -1 : 1)
        : (aVal > bVal ? -1 : 1)
    })
  }
}
```

## Performance Considerations

### Type System Optimization

```typescript
// Branded types for type safety without runtime overhead
type UserId = string & { readonly __brand: "UserId" };
type ProductId = string & { readonly __brand: "ProductId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

// Compile-time type checking prevents errors
function linkUserToProduct(userId: UserId, productId: ProductId): void {
  // Type-safe operations without runtime cost
}
```

## Conclusion

ArkTS's advanced type system provides powerful tools for building type-safe applications. By leveraging generics, conditional types, and mapped types, developers can create flexible, reusable code while maintaining compile-time safety and performance.
