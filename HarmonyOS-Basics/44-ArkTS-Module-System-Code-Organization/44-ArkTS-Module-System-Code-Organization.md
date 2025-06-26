# 44. ArkTS Module System and Code Organization

## Introduction

Effective code organization and module architecture are fundamental to building scalable and maintainable HarmonyOS applications. This article explores ArkTS module system, advanced architectural patterns, and best practices for structuring large-scale applications.

## Module System Fundamentals

### Advanced Module Architecture

```typescript
// Core module interfaces and types
export interface ModuleMetadata {
  name: string;
  version: string;
  dependencies: string[];
  exports: string[];
  lazy?: boolean;
  singleton?: boolean;
}

export interface ModuleDefinition {
  metadata: ModuleMetadata;
  factory: () => any;
  instance?: any;
}

export class ModuleRegistry {
  private static instance: ModuleRegistry;
  private modules: Map<string, ModuleDefinition> = new Map();
  private loadedModules: Map<string, any> = new Map();
  private loadingPromises: Map<string, Promise<any>> = new Map();
  private dependencyGraph: Map<string, Set<string>> = new Map();

  static getInstance(): ModuleRegistry {
    if (!this.instance) {
      this.instance = new ModuleRegistry();
    }
    return this.instance;
  }

  register(definition: ModuleDefinition): void {
    const { name } = definition.metadata;

    if (this.modules.has(name)) {
      throw new Error(`Module '${name}' is already registered`);
    }

    this.modules.set(name, definition);
    this.buildDependencyGraph(definition);
  }

  async load<T>(moduleName: string): Promise<T> {
    // Return cached instance for singletons
    if (this.loadedModules.has(moduleName)) {
      const definition = this.modules.get(moduleName);
      if (definition?.metadata.singleton !== false) {
        return this.loadedModules.get(moduleName);
      }
    }

    // Return existing loading promise if in progress
    if (this.loadingPromises.has(moduleName)) {
      return this.loadingPromises.get(moduleName);
    }

    // Start loading process
    const loadingPromise = this.performLoad<T>(moduleName);
    this.loadingPromises.set(moduleName, loadingPromise);

    try {
      const module = await loadingPromise;

      // Cache singleton instances
      const definition = this.modules.get(moduleName);
      if (definition?.metadata.singleton !== false) {
        this.loadedModules.set(moduleName, module);
      }

      return module;
    } finally {
      this.loadingPromises.delete(moduleName);
    }
  }

  private async performLoad<T>(moduleName: string): Promise<T> {
    const definition = this.modules.get(moduleName);
    if (!definition) {
      throw new Error(`Module '${moduleName}' not found`);
    }

    // Load dependencies first
    await this.loadDependencies(moduleName);

    // Create module instance
    try {
      const instance = await definition.factory();
      console.log(`Module '${moduleName}' loaded successfully`);
      return instance;
    } catch (error) {
      console.error(`Failed to load module '${moduleName}':`, error);
      throw error;
    }
  }

  private async loadDependencies(moduleName: string): Promise<void> {
    const dependencies = this.dependencyGraph.get(moduleName) || new Set();

    if (dependencies.size === 0) return;

    // Check for circular dependencies
    this.detectCircularDependencies(moduleName, new Set());

    // Load all dependencies in parallel
    const loadPromises = Array.from(dependencies).map((dep) => this.load(dep));
    await Promise.all(loadPromises);
  }

  private buildDependencyGraph(definition: ModuleDefinition): void {
    const { name, dependencies } = definition.metadata;
    this.dependencyGraph.set(name, new Set(dependencies));
  }

  private detectCircularDependencies(
    moduleName: string,
    visited: Set<string>,
    path: string[] = []
  ): void {
    if (visited.has(moduleName)) {
      const cycle = path.slice(path.indexOf(moduleName));
      throw new Error(
        `Circular dependency detected: ${cycle.join(" -> ")} -> ${moduleName}`
      );
    }

    visited.add(moduleName);
    path.push(moduleName);

    const dependencies = this.dependencyGraph.get(moduleName) || new Set();
    for (const dep of dependencies) {
      this.detectCircularDependencies(dep, new Set(visited), [...path]);
    }
  }

  getModuleInfo(moduleName: string): ModuleMetadata | null {
    const definition = this.modules.get(moduleName);
    return definition?.metadata || null;
  }

  getAllModules(): string[] {
    return Array.from(this.modules.keys());
  }

  getDependencyTree(moduleName: string): any {
    const buildTree = (name: string, visited: Set<string> = new Set()): any => {
      if (visited.has(name)) return { name, circular: true };

      visited.add(name);
      const dependencies = this.dependencyGraph.get(name) || new Set();

      return {
        name,
        dependencies: Array.from(dependencies).map((dep) =>
          buildTree(dep, new Set(visited))
        ),
      };
    };

    return buildTree(moduleName);
  }

  unload(moduleName: string): boolean {
    if (this.loadedModules.has(moduleName)) {
      this.loadedModules.delete(moduleName);
      console.log(`Module '${moduleName}' unloaded`);
      return true;
    }
    return false;
  }
}
```

### Service Container and Dependency Injection

```typescript
// Advanced dependency injection container
export interface ServiceDefinition<T = any> {
  identifier: string | symbol;
  factory: (container: ServiceContainer) => T;
  singleton?: boolean;
  dependencies?: (string | symbol)[];
  metadata?: Record<string, any>;
}

export class ServiceContainer {
  private static instance: ServiceContainer;
  private services: Map<string | symbol, ServiceDefinition> = new Map();
  private instances: Map<string | symbol, any> = new Map();
  private resolving: Set<string | symbol> = new Set();

  static getInstance(): ServiceContainer {
    if (!this.instance) {
      this.instance = new ServiceContainer();
    }
    return this.instance;
  }

  register<T>(definition: ServiceDefinition<T>): void {
    this.services.set(definition.identifier, definition);
  }

  resolve<T>(identifier: string | symbol): T {
    // Return cached singleton
    if (this.instances.has(identifier)) {
      return this.instances.get(identifier);
    }

    // Check for circular dependencies
    if (this.resolving.has(identifier)) {
      throw new Error(
        `Circular dependency detected for service: ${String(identifier)}`
      );
    }

    const definition = this.services.get(identifier);
    if (!definition) {
      throw new Error(`Service not registered: ${String(identifier)}`);
    }

    this.resolving.add(identifier);

    try {
      // Resolve dependencies first
      if (definition.dependencies) {
        definition.dependencies.forEach((dep) => this.resolve(dep));
      }

      // Create instance
      const instance = definition.factory(this);

      // Cache singleton
      if (definition.singleton !== false) {
        this.instances.set(identifier, instance);
      }

      return instance;
    } finally {
      this.resolving.delete(identifier);
    }
  }

  // Decorator for automatic service registration
  service<T>(
    identifier?: string | symbol,
    options: { singleton?: boolean; dependencies?: (string | symbol)[] } = {}
  ) {
    return function (target: new (...args: any[]) => T) {
      const serviceId = identifier || target.name;
      const container = ServiceContainer.getInstance();

      container.register({
        identifier: serviceId,
        factory: (container) => {
          // Resolve constructor dependencies
          const dependencies =
            options.dependencies?.map((dep) => container.resolve(dep)) || [];
          return new target(...dependencies);
        },
        singleton: options.singleton !== false,
        dependencies: options.dependencies,
      });

      return target;
    };
  }

  isRegistered(identifier: string | symbol): boolean {
    return this.services.has(identifier);
  }

  unregister(identifier: string | symbol): boolean {
    const removed = this.services.delete(identifier);
    this.instances.delete(identifier);
    return removed;
  }

  clear(): void {
    this.services.clear();
    this.instances.clear();
    this.resolving.clear();
  }

  getRegisteredServices(): (string | symbol)[] {
    return Array.from(this.services.keys());
  }
}

// Service decorators
export function Injectable(identifier?: string | symbol) {
  return ServiceContainer.getInstance().service(identifier);
}

export function Inject(identifier: string | symbol) {
  return function (
    target: any,
    propertyKey: string | symbol | undefined,
    parameterIndex: number
  ) {
    // Store injection metadata
    const container = ServiceContainer.getInstance();
    // Implementation would use reflection metadata
  };
}
```

## Architectural Patterns

### Domain-Driven Design Structure

```typescript
// Domain layer interfaces
export interface Entity<T = string> {
  id: T;
  equals(other: Entity<T>): boolean;
}

export interface ValueObject {
  equals(other: ValueObject): boolean;
}

export interface Repository<T extends Entity, ID = string> {
  findById(id: ID): Promise<T | null>;
  save(entity: T): Promise<void>;
  delete(id: ID): Promise<void>;
  findAll(): Promise<T[]>;
}

export interface DomainEvent {
  eventType: string;
  occurredOn: Date;
  aggregateId: string;
}

// Base implementations
export abstract class BaseEntity<T = string> implements Entity<T> {
  constructor(public readonly id: T) {}

  equals(other: Entity<T>): boolean {
    return this.id === other.id;
  }
}

export abstract class BaseValueObject implements ValueObject {
  equals(other: ValueObject): boolean {
    return JSON.stringify(this) === JSON.stringify(other);
  }
}

// Domain service example
@Injectable("UserDomainService")
export class UserDomainService {
  constructor(
    private userRepository: Repository<User>,
    private emailService: EmailService
  ) {}

  async createUser(userData: CreateUserCommand): Promise<User> {
    // Domain logic
    const existingUser = await this.userRepository.findById(userData.email);
    if (existingUser) {
      throw new Error("User already exists");
    }

    const user = new User(userData.email, userData.name);
    await this.userRepository.save(user);

    // Domain event
    await this.publishEvent(new UserCreatedEvent(user.id));

    return user;
  }

  private async publishEvent(event: DomainEvent): Promise<void> {
    // Event publishing logic
  }
}

// Domain entities
export class User extends BaseEntity<string> {
  private _name: string;
  private _email: Email;
  private _createdAt: Date;

  constructor(email: string, name: string) {
    super(email);
    this._email = new Email(email);
    this._name = name;
    this._createdAt = new Date();
  }

  get name(): string {
    return this._name;
  }

  get email(): Email {
    return this._email;
  }

  get createdAt(): Date {
    return this._createdAt;
  }

  changeName(newName: string): void {
    if (!newName || newName.trim().length === 0) {
      throw new Error("Name cannot be empty");
    }
    this._name = newName.trim();
  }
}

// Value objects
export class Email extends BaseValueObject {
  private readonly _value: string;

  constructor(email: string) {
    super();
    this.validate(email);
    this._value = email.toLowerCase();
  }

  get value(): string {
    return this._value;
  }

  private validate(email: string): void {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      throw new Error("Invalid email format");
    }
  }
}

// Events
export class UserCreatedEvent implements DomainEvent {
  readonly eventType = "UserCreated";
  readonly occurredOn = new Date();

  constructor(public readonly aggregateId: string) {}
}

// Commands
export interface CreateUserCommand {
  email: string;
  name: string;
}
```

### Clean Architecture Implementation

```typescript
// Use cases (Application layer)
export interface UseCase<TRequest, TResponse> {
  execute(request: TRequest): Promise<TResponse>;
}

export class CreateUserUseCase
  implements UseCase<CreateUserRequest, CreateUserResponse>
{
  constructor(
    private userRepository: Repository<User>,
    private userDomainService: UserDomainService,
    private logger: Logger
  ) {}

  async execute(request: CreateUserRequest): Promise<CreateUserResponse> {
    try {
      this.logger.info("Creating user", { email: request.email });

      const user = await this.userDomainService.createUser({
        email: request.email,
        name: request.name,
      });

      this.logger.info("User created successfully", { userId: user.id });

      return {
        success: true,
        userId: user.id,
        message: "User created successfully",
      };
    } catch (error) {
      this.logger.error("Failed to create user", { error, request });

      return {
        success: false,
        error: error.message,
      };
    }
  }
}

// Infrastructure layer
@Injectable("UserRepositoryImpl")
export class UserRepositoryImpl implements Repository<User> {
  constructor(private database: Database) {}

  async findById(id: string): Promise<User | null> {
    const userData = await this.database.query(
      "SELECT * FROM users WHERE id = ?",
      [id]
    );

    if (!userData) return null;

    return new User(userData.email, userData.name);
  }

  async save(user: User): Promise<void> {
    await this.database.execute(
      "INSERT OR REPLACE INTO users (id, email, name, created_at) VALUES (?, ?, ?, ?)",
      [user.id, user.email.value, user.name, user.createdAt.toISOString()]
    );
  }

  async delete(id: string): Promise<void> {
    await this.database.execute("DELETE FROM users WHERE id = ?", [id]);
  }

  async findAll(): Promise<User[]> {
    const usersData = await this.database.query("SELECT * FROM users");
    return usersData.map((data) => new User(data.email, data.name));
  }
}

// Presentation layer (Controllers)
export class UserController {
  constructor(
    private createUserUseCase: CreateUserUseCase,
    private getUserUseCase: GetUserUseCase
  ) {}

  async createUser(request: any): Promise<any> {
    const response = await this.createUserUseCase.execute({
      email: request.body.email,
      name: request.body.name,
    });

    return {
      status: response.success ? 201 : 400,
      body: response,
    };
  }

  async getUser(request: any): Promise<any> {
    const response = await this.getUserUseCase.execute({
      userId: request.params.id,
    });

    return {
      status: response.success ? 200 : 404,
      body: response,
    };
  }
}

// Request/Response models
export interface CreateUserRequest {
  email: string;
  name: string;
}

export interface CreateUserResponse {
  success: boolean;
  userId?: string;
  message?: string;
  error?: string;
}
```

### Feature-Based Module Organization

```typescript
// Feature module structure
export interface FeatureModule {
  name: string;
  components: ComponentDefinition[];
  services: ServiceDefinition[];
  routes?: RouteDefinition[];
  initialize(): Promise<void>;
  dispose(): Promise<void>;
}

export class UserManagementModule implements FeatureModule {
  name = "UserManagement";

  components = [
    {
      name: "UserListComponent",
      factory: () => import("./components/UserList"),
    },
    {
      name: "UserFormComponent",
      factory: () => import("./components/UserForm"),
    },
    {
      name: "UserDetailComponent",
      factory: () => import("./components/UserDetail"),
    },
  ];

  services = [
    {
      identifier: "UserService",
      factory: (container) =>
        new UserService(
          container.resolve("UserRepository"),
          container.resolve("Logger")
        ),
      singleton: true,
    },
    {
      identifier: "UserRepository",
      factory: (container) =>
        new UserRepositoryImpl(container.resolve("Database")),
      singleton: true,
    },
  ];

  routes = [
    { path: "/users", component: "UserListComponent" },
    { path: "/users/:id", component: "UserDetailComponent" },
    { path: "/users/create", component: "UserFormComponent" },
  ];

  async initialize(): Promise<void> {
    const container = ServiceContainer.getInstance();

    // Register services
    this.services.forEach((service) => container.register(service));

    // Register components
    const moduleRegistry = ModuleRegistry.getInstance();
    this.components.forEach((component) => {
      moduleRegistry.register({
        metadata: {
          name: component.name,
          version: "1.0.0",
          dependencies: [],
          exports: [component.name],
        },
        factory: component.factory,
      });
    });

    console.log(`Module '${this.name}' initialized`);
  }

  async dispose(): Promise<void> {
    const container = ServiceContainer.getInstance();

    // Unregister services
    this.services.forEach((service) => {
      container.unregister(service.identifier);
    });

    console.log(`Module '${this.name}' disposed`);
  }
}

// Module loader
export class FeatureModuleLoader {
  private static instance: FeatureModuleLoader;
  private loadedModules: Map<string, FeatureModule> = new Map();

  static getInstance(): FeatureModuleLoader {
    if (!this.instance) {
      this.instance = new FeatureModuleLoader();
    }
    return this.instance;
  }

  async loadModule(module: FeatureModule): Promise<void> {
    if (this.loadedModules.has(module.name)) {
      console.warn(`Module '${module.name}' is already loaded`);
      return;
    }

    try {
      await module.initialize();
      this.loadedModules.set(module.name, module);
      console.log(`Module '${module.name}' loaded successfully`);
    } catch (error) {
      console.error(`Failed to load module '${module.name}':`, error);
      throw error;
    }
  }

  async unloadModule(moduleName: string): Promise<void> {
    const module = this.loadedModules.get(moduleName);
    if (!module) {
      console.warn(`Module '${moduleName}' is not loaded`);
      return;
    }

    try {
      await module.dispose();
      this.loadedModules.delete(moduleName);
      console.log(`Module '${moduleName}' unloaded successfully`);
    } catch (error) {
      console.error(`Failed to unload module '${moduleName}':`, error);
      throw error;
    }
  }

  getLoadedModules(): string[] {
    return Array.from(this.loadedModules.keys());
  }

  isModuleLoaded(moduleName: string): boolean {
    return this.loadedModules.has(moduleName);
  }
}

// Component definitions
interface ComponentDefinition {
  name: string;
  factory: () => Promise<any>;
}

interface RouteDefinition {
  path: string;
  component: string;
  guards?: string[];
}
```

## Code Organization Patterns

### Plugin Architecture

```typescript
// Plugin system for extensible applications
export interface Plugin {
  name: string;
  version: string;
  activate(context: PluginContext): Promise<void>;
  deactivate(): Promise<void>;
  getCapabilities(): PluginCapability[];
}

export interface PluginCapability {
  type: string;
  name: string;
  description: string;
}

export interface PluginContext {
  registerCommand(command: Command): void;
  registerService(service: ServiceDefinition): void;
  registerComponent(component: ComponentDefinition): void;
  getService<T>(identifier: string | symbol): T;
  emitEvent(event: PluginEvent): void;
  onEvent(eventType: string, handler: (event: PluginEvent) => void): void;
}

export class PluginManager {
  private static instance: PluginManager;
  private plugins: Map<string, Plugin> = new Map();
  private activePlugins: Set<string> = new Set();
  private eventHandlers: Map<string, Array<(event: PluginEvent) => void>> =
    new Map();

  static getInstance(): PluginManager {
    if (!this.instance) {
      this.instance = new PluginManager();
    }
    return this.instance;
  }

  registerPlugin(plugin: Plugin): void {
    if (this.plugins.has(plugin.name)) {
      throw new Error(`Plugin '${plugin.name}' is already registered`);
    }

    this.plugins.set(plugin.name, plugin);
    console.log(`Plugin '${plugin.name}' registered`);
  }

  async activatePlugin(pluginName: string): Promise<void> {
    const plugin = this.plugins.get(pluginName);
    if (!plugin) {
      throw new Error(`Plugin '${pluginName}' not found`);
    }

    if (this.activePlugins.has(pluginName)) {
      console.warn(`Plugin '${pluginName}' is already active`);
      return;
    }

    const context = this.createPluginContext(pluginName);

    try {
      await plugin.activate(context);
      this.activePlugins.add(pluginName);
      console.log(`Plugin '${pluginName}' activated`);
    } catch (error) {
      console.error(`Failed to activate plugin '${pluginName}':`, error);
      throw error;
    }
  }

  async deactivatePlugin(pluginName: string): Promise<void> {
    const plugin = this.plugins.get(pluginName);
    if (!plugin) {
      throw new Error(`Plugin '${pluginName}' not found`);
    }

    if (!this.activePlugins.has(pluginName)) {
      console.warn(`Plugin '${pluginName}' is not active`);
      return;
    }

    try {
      await plugin.deactivate();
      this.activePlugins.delete(pluginName);
      console.log(`Plugin '${pluginName}' deactivated`);
    } catch (error) {
      console.error(`Failed to deactivate plugin '${pluginName}':`, error);
      throw error;
    }
  }

  private createPluginContext(pluginName: string): PluginContext {
    return {
      registerCommand: (command) => {
        // Register command with command system
        console.log(
          `Plugin '${pluginName}' registered command: ${command.name}`
        );
      },
      registerService: (service) => {
        const container = ServiceContainer.getInstance();
        container.register(service);
      },
      registerComponent: (component) => {
        const registry = ModuleRegistry.getInstance();
        registry.register({
          metadata: {
            name: component.name,
            version: "1.0.0",
            dependencies: [],
            exports: [component.name],
          },
          factory: component.factory,
        });
      },
      getService: <T>(identifier: string | symbol): T => {
        const container = ServiceContainer.getInstance();
        return container.resolve<T>(identifier);
      },
      emitEvent: (event) => {
        this.emitEvent(event);
      },
      onEvent: (eventType, handler) => {
        this.addEventListener(eventType, handler);
      },
    };
  }

  private emitEvent(event: PluginEvent): void {
    const handlers = this.eventHandlers.get(event.type) || [];
    handlers.forEach((handler) => {
      try {
        handler(event);
      } catch (error) {
        console.error("Plugin event handler error:", error);
      }
    });
  }

  private addEventListener(
    eventType: string,
    handler: (event: PluginEvent) => void
  ): void {
    if (!this.eventHandlers.has(eventType)) {
      this.eventHandlers.set(eventType, []);
    }
    this.eventHandlers.get(eventType)!.push(handler);
  }

  getActivePlugins(): string[] {
    return Array.from(this.activePlugins);
  }

  getRegisteredPlugins(): string[] {
    return Array.from(this.plugins.keys());
  }

  getPluginCapabilities(pluginName: string): PluginCapability[] {
    const plugin = this.plugins.get(pluginName);
    return plugin?.getCapabilities() || [];
  }
}

// Example plugin implementation
export class ThemePlugin implements Plugin {
  name = "ThemePlugin";
  version = "1.0.0";

  async activate(context: PluginContext): Promise<void> {
    // Register theme service
    context.registerService({
      identifier: "ThemeService",
      factory: () => new ThemeService(),
      singleton: true,
    });

    // Register theme commands
    context.registerCommand({
      name: "setTheme",
      execute: (themeName: string) => {
        const themeService = context.getService<ThemeService>("ThemeService");
        themeService.setTheme(themeName);
        context.emitEvent({
          type: "themeChanged",
          data: { themeName },
          timestamp: Date.now(),
        });
      },
    });

    console.log("Theme plugin activated");
  }

  async deactivate(): Promise<void> {
    console.log("Theme plugin deactivated");
  }

  getCapabilities(): PluginCapability[] {
    return [
      {
        type: "service",
        name: "ThemeService",
        description: "Provides theme management functionality",
      },
      {
        type: "command",
        name: "setTheme",
        description: "Changes the application theme",
      },
    ];
  }
}

// Supporting interfaces
interface Command {
  name: string;
  execute(...args: any[]): any;
}

interface PluginEvent {
  type: string;
  data?: any;
  timestamp: number;
}

class ThemeService {
  private currentTheme = "default";

  setTheme(themeName: string): void {
    this.currentTheme = themeName;
    console.log(`Theme changed to: ${themeName}`);
  }

  getCurrentTheme(): string {
    return this.currentTheme;
  }
}
```

## Best Practices

1. **Module Boundaries**: Define clear boundaries and interfaces between modules
2. **Dependency Management**: Use dependency injection to manage module dependencies
3. **Lazy Loading**: Implement lazy loading for non-critical modules
4. **Error Isolation**: Ensure module failures don't cascade to other modules
5. **Testing**: Write unit tests for individual modules and integration tests for module interactions
6. **Documentation**: Maintain clear documentation for module APIs and dependencies
7. **Versioning**: Implement versioning strategy for module updates

## Conclusion

Effective module system design and code organization are crucial for building scalable HarmonyOS applications. By implementing proper module architecture, dependency injection, and organizational patterns like clean architecture and feature modules, developers can create maintainable codebases that scale effectively with team size and application complexity.

The key to successful modular architecture lies in defining clear boundaries, managing dependencies effectively, and maintaining consistency in organizational patterns throughout the application lifecycle.
