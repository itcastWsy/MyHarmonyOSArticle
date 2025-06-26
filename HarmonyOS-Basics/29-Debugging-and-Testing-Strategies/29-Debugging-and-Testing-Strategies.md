# 29-Debugging and Testing Strategies

## Introduction

Effective debugging and testing are essential for developing reliable HarmonyOS applications. This article covers debugging techniques, testing frameworks, error handling strategies, and development tools for ensuring application quality.

## Debugging Techniques

### Logging and Console Output

```typescript
export class Logger {
  private static readonly LOG_LEVELS = {
    DEBUG: 0,
    INFO: 1,
    WARN: 2,
    ERROR: 3
  };

  private static currentLevel = this.LOG_LEVELS.DEBUG;

  static debug(message: string, ...args: any[]): void {
    if (this.currentLevel <= this.LOG_LEVELS.DEBUG) {
      console.debug(`[DEBUG] ${this.formatMessage(message)}`, ...args);
    }
  }

  static info(message: string, ...args: any[]): void {
    if (this.currentLevel <= this.LOG_LEVELS.INFO) {
      console.info(`[INFO] ${this.formatMessage(message)}`, ...args);
    }
  }

  static warn(message: string, ...args: any[]): void {
    if (this.currentLevel <= this.LOG_LEVELS.WARN) {
      console.warn(`[WARN] ${this.formatMessage(message)}`, ...args);
    }
  }

  static error(message: string, error?: Error, ...args: any[]): void {
    if (this.currentLevel <= this.LOG_LEVELS.ERROR) {
      console.error(`[ERROR] ${this.formatMessage(message)}`, error, ...args);
    }
  }

  static setLevel(level: keyof typeof Logger.LOG_LEVELS): void {
    this.currentLevel = this.LOG_LEVELS[level];
  }

  private static formatMessage(message: string): string {
    const timestamp = new Date().toISOString();
    return `${timestamp} - ${message}`;
  }
}

// Usage example
@Component
struct DebuggableComponent {
  @State private data: any[] = [];

  async aboutToAppear(): Promise<void> {
    Logger.info('Component initializing');

    try {
      await this.loadData();
      Logger.info('Data loaded successfully', { count: this.data.length });
    } catch (error) {
      Logger.error('Failed to load data', error as Error);
    }
  }

  private async loadData(): Promise<void> {
    Logger.debug('Starting data load operation');

    // Simulate API call
    const startTime = Date.now();
    this.data = await DataService.fetchData();
    const duration = Date.now() - startTime;

    Logger.debug('Data load completed', { duration: `${duration}ms` });
  }

  build() {
    Column() {
      if (this.data.length === 0) {
        Text('Loading...')
          .fontSize(16)
      } else {
        List() {
          ForEach(this.data, (item: any, index: number) => {
            ListItem() {
              Text(item.title)
                .onClick(() => {
                  Logger.debug('Item clicked', { index, itemId: item.id });
                })
            }
          })
        }
      }
    }
  }
}
```

### Error Boundaries and Error Handling

```typescript
// Global error handler
export class ErrorHandler {
  private static errorReporters: ((error: Error, context?: string) => void)[] = [];

  static addErrorReporter(reporter: (error: Error, context?: string) => void): void {
    this.errorReporters.push(reporter);
  }

  static handleError(error: Error, context?: string): void {
    Logger.error('Unhandled error occurred', error, { context });

    // Report to all registered reporters
    this.errorReporters.forEach(reporter => {
      try {
        reporter(error, context);
      } catch (reporterError) {
        console.error('Error reporter failed:', reporterError);
      }
    });
  }

  static wrapAsync<T>(asyncFn: () => Promise<T>, context?: string): Promise<T> {
    return asyncFn().catch(error => {
      this.handleError(error as Error, context);
      throw error;
    });
  }
}

// Error boundary component
@Component
struct ErrorBoundary {
  @State private hasError: boolean = false;
  @State private errorMessage: string = '';
  @BuilderParam content: () => void = this.defaultContent;

  @Builder
  defaultContent() {
    Text('No content provided')
  }

  build() {
    if (this.hasError) {
      Column() {
        Text('Something went wrong')
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .margin({ bottom: 16 })

        Text(this.errorMessage)
          .fontSize(14)
          .fontColor(Color.Gray)
          .margin({ bottom: 20 })

        Button('Retry')
          .onClick(() => {
            this.resetError();
          })
      }
      .width('100%')
      .height('100%')
      .justifyContent(FlexAlign.Center)
      .padding(20)
    } else {
      try {
        this.content()
      } catch (error) {
        this.catchError(error as Error);
      }
    }
  }

  private catchError(error: Error): void {
    this.hasError = true;
    this.errorMessage = error.message;
    ErrorHandler.handleError(error, 'ErrorBoundary');
  }

  private resetError(): void {
    this.hasError = false;
    this.errorMessage = '';
  }
}

// Safe async operation wrapper
export class SafeAsync {
  static async execute<T>(
    operation: () => Promise<T>,
    fallback?: T,
    onError?: (error: Error) => void
  ): Promise<T | undefined> {
    try {
      return await operation();
    } catch (error) {
      const err = error as Error;

      if (onError) {
        onError(err);
      } else {
        ErrorHandler.handleError(err, 'SafeAsync.execute');
      }

      return fallback;
    }
  }
}
```

## Unit Testing

### Test Framework Setup

```typescript
// Test utilities
export class TestUtils {
  static createMockContext(): Context {
    return {
      filesDir: "/mock/files",
      cacheDir: "/mock/cache",
      // Add other required Context properties
    } as Context;
  }

  static createMockUser(overrides?: Partial<User>): User {
    return {
      id: 1,
      name: "Test User",
      email: "test@example.com",
      createdAt: new Date().toISOString(),
      ...overrides,
    };
  }

  static async waitFor(
    condition: () => boolean,
    timeout: number = 5000
  ): Promise<boolean> {
    const startTime = Date.now();

    while (Date.now() - startTime < timeout) {
      if (condition()) {
        return true;
      }
      await new Promise((resolve) => setTimeout(resolve, 100));
    }

    return false;
  }
}

// Example service with testable methods
export class UserService {
  private httpClient: HttpClient;

  constructor(httpClient: HttpClient) {
    this.httpClient = httpClient;
  }

  async getUser(id: number): Promise<User | null> {
    try {
      const response = await this.httpClient.get<ApiResponse<User>>(
        `/users/${id}`
      );
      return response.data;
    } catch (error) {
      Logger.error("Failed to fetch user", error as Error, { userId: id });
      return null;
    }
  }

  async createUser(userData: CreateUserRequest): Promise<User | null> {
    if (!this.validateUserData(userData)) {
      throw new Error("Invalid user data");
    }

    try {
      const response = await this.httpClient.post<ApiResponse<User>>(
        "/users",
        userData
      );
      return response.data;
    } catch (error) {
      Logger.error("Failed to create user", error as Error, { userData });
      return null;
    }
  }

  private validateUserData(userData: CreateUserRequest): boolean {
    return !!(
      userData.name &&
      userData.email &&
      userData.name.trim() &&
      userData.email.includes("@")
    );
  }
}

// Mock HTTP client for testing
export class MockHttpClient implements HttpClient {
  private responses: Map<string, any> = new Map();
  private requestLog: { method: string; url: string; data?: any }[] = [];

  setResponse(url: string, response: any): void {
    this.responses.set(url, response);
  }

  getRequestLog(): { method: string; url: string; data?: any }[] {
    return this.requestLog;
  }

  clearRequestLog(): void {
    this.requestLog = [];
  }

  async get<T>(url: string): Promise<T> {
    this.requestLog.push({ method: "GET", url });

    const response = this.responses.get(url);
    if (!response) {
      throw new Error(`No mock response for GET ${url}`);
    }

    return response as T;
  }

  async post<T>(url: string, data: any): Promise<T> {
    this.requestLog.push({ method: "POST", url, data });

    const response = this.responses.get(url);
    if (!response) {
      throw new Error(`No mock response for POST ${url}`);
    }

    return response as T;
  }
}
```

### Test Cases

```typescript
// User service tests
describe("UserService", () => {
  let userService: UserService;
  let mockHttpClient: MockHttpClient;

  beforeEach(() => {
    mockHttpClient = new MockHttpClient();
    userService = new UserService(mockHttpClient);
  });

  describe("getUser", () => {
    it("should return user data when API call succeeds", async () => {
      // Arrange
      const userId = 1;
      const expectedUser = TestUtils.createMockUser({ id: userId });
      const apiResponse = { success: true, data: expectedUser };

      mockHttpClient.setResponse(`/users/${userId}`, apiResponse);

      // Act
      const result = await userService.getUser(userId);

      // Assert
      expect(result).toEqual(expectedUser);
      expect(mockHttpClient.getRequestLog()).toHaveLength(1);
      expect(mockHttpClient.getRequestLog()[0]).toEqual({
        method: "GET",
        url: `/users/${userId}`,
      });
    });

    it("should return null when API call fails", async () => {
      // Arrange
      const userId = 1;
      mockHttpClient.setResponse(`/users/${userId}`, null);

      // Act
      const result = await userService.getUser(userId);

      // Assert
      expect(result).toBeNull();
    });
  });

  describe("createUser", () => {
    it("should create user successfully with valid data", async () => {
      // Arrange
      const userData = {
        name: "John Doe",
        email: "john@example.com",
      };
      const expectedUser = TestUtils.createMockUser(userData);
      const apiResponse = { success: true, data: expectedUser };

      mockHttpClient.setResponse("/users", apiResponse);

      // Act
      const result = await userService.createUser(userData);

      // Assert
      expect(result).toEqual(expectedUser);
      expect(mockHttpClient.getRequestLog()).toHaveLength(1);
      expect(mockHttpClient.getRequestLog()[0]).toEqual({
        method: "POST",
        url: "/users",
        data: userData,
      });
    });

    it("should throw error with invalid data", async () => {
      // Arrange
      const invalidUserData = {
        name: "",
        email: "invalid-email",
      };

      // Act & Assert
      await expect(userService.createUser(invalidUserData)).rejects.toThrow(
        "Invalid user data"
      );
    });
  });
});

// Component testing
describe("UserListComponent", () => {
  let component: UserListComponent;
  let mockUserService: jest.Mocked<UserService>;

  beforeEach(() => {
    mockUserService = {
      getUsers: jest.fn(),
      getUser: jest.fn(),
      createUser: jest.fn(),
    } as jest.Mocked<UserService>;

    component = new UserListComponent();
    component.setUserService(mockUserService);
  });

  it("should load users on initialization", async () => {
    // Arrange
    const mockUsers = [
      TestUtils.createMockUser({ id: 1 }),
      TestUtils.createMockUser({ id: 2 }),
    ];
    mockUserService.getUsers.mockResolvedValue(mockUsers);

    // Act
    await component.loadUsers();

    // Assert
    expect(mockUserService.getUsers).toHaveBeenCalledTimes(1);
    expect(component.users).toEqual(mockUsers);
  });

  it("should handle loading error gracefully", async () => {
    // Arrange
    mockUserService.getUsers.mockRejectedValue(new Error("Network error"));

    // Act
    await component.loadUsers();

    // Assert
    expect(component.users).toEqual([]);
    expect(component.hasError).toBe(true);
  });
});
```

## Integration Testing

### API Integration Tests

```typescript
export class IntegrationTestHelper {
  private static baseUrl = "http://localhost:3000";
  private static authToken: string = "";

  static async setupTests(): Promise<void> {
    // Login and get auth token
    const loginResponse = await fetch(`${this.baseUrl}/auth/login`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        email: "test@example.com",
        password: "testpassword",
      }),
    });

    const loginData = await loginResponse.json();
    this.authToken = loginData.token;
  }

  static async createTestUser(userData: Partial<User>): Promise<User> {
    const response = await fetch(`${this.baseUrl}/users`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${this.authToken}`,
      },
      body: JSON.stringify({
        name: "Test User",
        email: `test${Date.now()}@example.com`,
        ...userData,
      }),
    });

    return await response.json();
  }

  static async cleanupTestData(): Promise<void> {
    // Clean up test data
    await fetch(`${this.baseUrl}/test/cleanup`, {
      method: "DELETE",
      headers: {
        Authorization: `Bearer ${this.authToken}`,
      },
    });
  }
}

// Integration test example
describe("User API Integration", () => {
  beforeAll(async () => {
    await IntegrationTestHelper.setupTests();
  });

  afterAll(async () => {
    await IntegrationTestHelper.cleanupTestData();
  });

  it("should create, read, update, and delete user", async () => {
    // Create
    const userData = {
      name: "Integration Test User",
      email: "integration@test.com",
    };
    const createdUser = await IntegrationTestHelper.createTestUser(userData);

    expect(createdUser.id).toBeDefined();
    expect(createdUser.name).toBe(userData.name);

    // Read
    const fetchedUser = await UserService.getUser(createdUser.id);
    expect(fetchedUser).toEqual(createdUser);

    // Update
    const updatedData = { name: "Updated Name" };
    const updatedUser = await UserService.updateUser(
      createdUser.id,
      updatedData
    );
    expect(updatedUser.name).toBe(updatedData.name);

    // Delete
    await UserService.deleteUser(createdUser.id);
    const deletedUser = await UserService.getUser(createdUser.id);
    expect(deletedUser).toBeNull();
  });
});
```

## Performance Testing

### Performance Benchmarks

```typescript
export class PerformanceTester {
  static async benchmarkFunction<T>(
    name: string,
    fn: () => Promise<T>,
    iterations: number = 100
  ): Promise<BenchmarkResult> {
    const durations: number[] = [];
    let errors = 0;

    for (let i = 0; i < iterations; i++) {
      const startTime = performance.now();

      try {
        await fn();
        const duration = performance.now() - startTime;
        durations.push(duration);
      } catch (error) {
        errors++;
        Logger.error(`Benchmark error in iteration ${i}`, error as Error);
      }
    }

    const successfulRuns = durations.length;
    const averageDuration =
      durations.reduce((sum, d) => sum + d, 0) / successfulRuns;
    const minDuration = Math.min(...durations);
    const maxDuration = Math.max(...durations);

    const sortedDurations = durations.sort((a, b) => a - b);
    const p95Duration = sortedDurations[Math.floor(successfulRuns * 0.95)];

    const result: BenchmarkResult = {
      name,
      iterations,
      successfulRuns,
      errors,
      averageDuration,
      minDuration,
      maxDuration,
      p95Duration,
    };

    Logger.info(`Benchmark: ${name}`, result);
    return result;
  }

  static async stressTest(
    name: string,
    fn: () => Promise<void>,
    concurrency: number = 10,
    duration: number = 30000
  ): Promise<StressTestResult> {
    const startTime = Date.now();
    const endTime = startTime + duration;

    let completedRequests = 0;
    let errors = 0;
    const durations: number[] = [];

    const workers = Array(concurrency)
      .fill(null)
      .map(async () => {
        while (Date.now() < endTime) {
          const requestStart = Date.now();

          try {
            await fn();
            completedRequests++;
            durations.push(Date.now() - requestStart);
          } catch (error) {
            errors++;
          }
        }
      });

    await Promise.all(workers);

    const actualDuration = Date.now() - startTime;
    const throughput = (completedRequests / actualDuration) * 1000; // requests per second
    const errorRate = errors / (completedRequests + errors);
    const averageResponseTime =
      durations.reduce((sum, d) => sum + d, 0) / durations.length;

    const result: StressTestResult = {
      name,
      duration: actualDuration,
      concurrency,
      completedRequests,
      errors,
      throughput,
      errorRate,
      averageResponseTime,
    };

    Logger.info(`Stress test: ${name}`, result);
    return result;
  }
}

interface BenchmarkResult {
  name: string;
  iterations: number;
  successfulRuns: number;
  errors: number;
  averageDuration: number;
  minDuration: number;
  maxDuration: number;
  p95Duration: number;
}

interface StressTestResult {
  name: string;
  duration: number;
  concurrency: number;
  completedRequests: number;
  errors: number;
  throughput: number;
  errorRate: number;
  averageResponseTime: number;
}

// Usage example
describe("Performance Tests", () => {
  it("should benchmark data processing", async () => {
    const result = await PerformanceTester.benchmarkFunction(
      "processLargeDataset",
      async () => {
        const data = Array(1000)
          .fill(null)
          .map((_, i) => ({ id: i, value: Math.random() }));
        return AsyncProcessor.processLargeDataset(
          data,
          (item) => item.value * 2
        );
      },
      50
    );

    expect(result.averageDuration).toBeLessThan(1000); // Should complete in under 1 second
    expect(result.errors).toBe(0);
  });

  it("should stress test API endpoints", async () => {
    const result = await PerformanceTester.stressTest(
      "getUserAPI",
      async () => {
        await UserService.getUser(1);
      },
      20, // 20 concurrent requests
      10000 // for 10 seconds
    );

    expect(result.throughput).toBeGreaterThan(10); // At least 10 requests per second
    expect(result.errorRate).toBeLessThan(0.01); // Less than 1% error rate
  });
});
```

## Best Practices

### 1. Debugging Strategy

- Use structured logging with appropriate log levels
- Implement comprehensive error boundaries
- Use debugging tools and IDE features effectively
- Set up proper development vs production logging

### 2. Testing Strategy

- Write unit tests for business logic
- Use integration tests for API interactions
- Implement end-to-end tests for critical user flows
- Mock external dependencies appropriately

### 3. Test Organization

- Organize tests by feature or component
- Use descriptive test names and descriptions
- Maintain test data and fixtures properly
- Keep tests independent and repeatable

### 4. Performance Testing

- Establish performance baselines
- Test under realistic load conditions
- Monitor key performance metrics
- Set up automated performance testing

## Conclusion

Effective debugging and testing strategies are crucial for maintaining code quality and ensuring reliable HarmonyOS applications. By implementing comprehensive logging, error handling, testing frameworks, and performance monitoring, you can catch issues early and maintain high application quality.

Key takeaways:

1. **Implement Comprehensive Logging**: Use structured logging for effective debugging
2. **Write Testable Code**: Design code with testing in mind
3. **Use Multiple Testing Types**: Combine unit, integration, and performance tests
4. **Automate Testing**: Set up continuous testing in your development pipeline
5. **Monitor Performance**: Establish baselines and monitor key metrics continuously
