# 36. Distributed Caching and Data Consistency

## Introduction

Distributed caching and data consistency are crucial for building scalable HarmonyOS applications. This article explores implementing caching strategies, data synchronization patterns, and maintaining consistency across distributed systems.

## Cache Implementation

### Multi-Level Cache System

```typescript
export interface CacheItem<T> {
  key: string;
  value: T;
  expiresAt: number;
  version: number;
  metadata: Record<string, any>;
}

export interface CacheOptions {
  ttl?: number;
  maxSize?: number;
  compressionEnabled?: boolean;
}

export abstract class BaseCache<T> {
  protected options: Required<CacheOptions>;

  constructor(options: CacheOptions = {}) {
    this.options = {
      ttl: options.ttl || 300000,
      maxSize: options.maxSize || 1000,
      compressionEnabled: options.compressionEnabled || false,
    };
  }

  abstract get(key: string): Promise<T | null>;
  abstract set(key: string, value: T, ttl?: number): Promise<void>;
  abstract delete(key: string): Promise<boolean>;
  abstract clear(): Promise<void>;
  abstract size(): Promise<number>;

  protected isExpired(item: CacheItem<T>): boolean {
    return Date.now() > item.expiresAt;
  }

  protected createCacheItem<T>(
    key: string,
    value: T,
    ttl?: number
  ): CacheItem<T> {
    return {
      key,
      value,
      expiresAt: Date.now() + (ttl || this.options.ttl),
      version: 1,
      metadata: {},
    };
  }
}

export class MemoryCache<T> extends BaseCache<T> {
  private cache: Map<string, CacheItem<T>> = new Map();

  async get(key: string): Promise<T | null> {
    const item = this.cache.get(key);

    if (!item || this.isExpired(item)) {
      if (item) this.cache.delete(key);
      return null;
    }

    return item.value;
  }

  async set(key: string, value: T, ttl?: number): Promise<void> {
    if (this.cache.size >= this.options.maxSize && !this.cache.has(key)) {
      await this.evictLRU();
    }

    const item = this.createCacheItem(key, value, ttl);
    this.cache.set(key, item);
  }

  async delete(key: string): Promise<boolean> {
    return this.cache.delete(key);
  }

  async clear(): Promise<void> {
    this.cache.clear();
  }

  async size(): Promise<number> {
    return this.cache.size;
  }

  private async evictLRU(): Promise<void> {
    let oldestKey: string | null = null;
    let oldestTime = Date.now();

    for (const [key, item] of this.cache.entries()) {
      if (item.expiresAt < oldestTime) {
        oldestTime = item.expiresAt;
        oldestKey = key;
      }
    }

    if (oldestKey) this.cache.delete(oldestKey);
  }
}
```

## Data Consistency Patterns

### Eventual Consistency Manager

```typescript
export interface ConsistencyEvent {
  entityId: string;
  entityType: string;
  operation: "create" | "update" | "delete";
  data: any;
  version: number;
  timestamp: number;
}

export class EventualConsistencyManager {
  private eventLog: ConsistencyEvent[] = [];
  private versionVector: Map<string, number> = new Map();

  async applyEvent(event: ConsistencyEvent): Promise<void> {
    const conflicts = this.detectConflicts(event);

    if (conflicts.length > 0) {
      await this.resolveConflicts(event, conflicts);
    } else {
      await this.applyEventDirectly(event);
    }

    this.eventLog.push(event);
    this.updateVersionVector(event);
  }

  private detectConflicts(event: ConsistencyEvent): ConsistencyEvent[] {
    return this.eventLog.filter(
      (existing) =>
        existing.entityId === event.entityId &&
        existing.version === event.version &&
        existing.timestamp !== event.timestamp
    );
  }

  private async resolveConflicts(
    event: ConsistencyEvent,
    conflicts: ConsistencyEvent[]
  ): Promise<void> {
    // Last write wins strategy
    const allEvents = [event, ...conflicts];
    const winningEvent = allEvents.reduce((latest, current) =>
      current.timestamp > latest.timestamp ? current : latest
    );

    await this.applyEventDirectly(winningEvent);
  }

  private async applyEventDirectly(event: ConsistencyEvent): Promise<void> {
    switch (event.operation) {
      case "create":
        console.log(`Creating entity ${event.entityId}`);
        break;
      case "update":
        console.log(`Updating entity ${event.entityId}`);
        break;
      case "delete":
        console.log(`Deleting entity ${event.entityId}`);
        break;
    }
  }

  private updateVersionVector(event: ConsistencyEvent): void {
    const currentVersion = this.versionVector.get(event.entityId) || 0;
    this.versionVector.set(
      event.entityId,
      Math.max(currentVersion, event.version)
    );
  }
}
```

## Cache Management

```typescript
export class CacheManager {
  private static instance: CacheManager;
  private caches: Map<string, BaseCache<any>> = new Map();

  static getInstance(): CacheManager {
    if (!this.instance) {
      this.instance = new CacheManager();
    }
    return this.instance;
  }

  createCache<T>(name: string, options: CacheOptions = {}): BaseCache<T> {
    const cache = new MemoryCache<T>(options);
    this.caches.set(name, cache);
    return cache;
  }

  getCache<T>(name: string): BaseCache<T> | null {
    return this.caches.get(name) || null;
  }

  async invalidatePattern(pattern: string): Promise<void> {
    for (const cache of this.caches.values()) {
      const keys = (await cache.keys?.()) || [];
      const matchingKeys = keys.filter((key) => new RegExp(pattern).test(key));

      for (const key of matchingKeys) {
        await cache.delete(key);
      }
    }
  }
}
```

## Usage Example

```typescript
// Service implementation with caching
class UserService {
  private cache = CacheManager.getInstance().createCache<User>("users", {
    ttl: 600000, // 10 minutes
    maxSize: 1000,
  });

  async getUser(id: string): Promise<User | null> {
    const cacheKey = `user:${id}`;

    // Try cache first
    let user = await this.cache.get(cacheKey);
    if (user) return user;

    // Fetch from database
    user = await this.fetchUserFromDatabase(id);
    if (user) {
      await this.cache.set(cacheKey, user);
    }

    return user;
  }

  async updateUser(id: string, updates: Partial<User>): Promise<void> {
    await this.updateUserInDatabase(id, updates);
    await this.cache.delete(`user:${id}`);
  }

  private async fetchUserFromDatabase(id: string): Promise<User | null> {
    // Database implementation
    return null;
  }

  private async updateUserInDatabase(
    id: string,
    updates: Partial<User>
  ): Promise<void> {
    // Database implementation
  }
}

interface User {
  id: string;
  name: string;
  email: string;
  lastActive: number;
}
```

## Best Practices

1. **Cache Strategy**: Choose appropriate TTL based on data volatility
2. **Memory Management**: Implement LRU eviction for memory-constrained environments
3. **Consistency**: Use eventual consistency for distributed scenarios
4. **Monitoring**: Track cache hit rates and performance metrics
5. **Error Handling**: Gracefully handle cache failures with fallback mechanisms

## Conclusion

Distributed caching and data consistency enable building scalable HarmonyOS applications with optimal performance while maintaining data integrity across distributed systems.
