# 21-ArkTS Best Practices and Code Quality

## Introduction

Writing high-quality ArkTS code is essential for building maintainable, performant, and robust HarmonyOS applications. This article explores best practices, coding standards, and quality guidelines specifically tailored for ArkTS development. Following these practices will help you create applications that are not only functional but also efficient and easy to maintain.

## Code Organization and Structure

### Folder Structure Best Practices

Organize your ArkTS project with a clear and logical folder structure:

```
src/
├── main/
│   ├── ets/
│   │   ├── components/          # Reusable UI components
│   │   ├── pages/              # Page-level components
│   │   ├── model/              # Data models and types
│   │   ├── common/             # Common utilities and constants
│   │   ├── manager/            # Business logic managers
│   │   └── entryability/       # Application entry points
│   └── resources/              # Static resources
```

### File Naming Conventions

Follow consistent naming conventions:

```typescript
// Component files - PascalCase
UserProfile.ets;
NavigationBar.ets;

// Utility files - camelCase
httpUtils.ets;
dateFormatter.ets;

// Model files - PascalCase
UserModel.ets;
ApiResponse.ets;

// Constants files - camelCase
appConstants.ets;
```

## Type Safety and Declaration Best Practices

### Explicit Type Annotations

Always use explicit type annotations for better code clarity and compile-time checking:

**Good Practice:**

```typescript
// Explicit function signatures
function calculateTotal(price: number, tax: number): number {
  return price + price * tax;
}

// Explicit variable types for complex objects
interface UserPreferences {
  theme: string;
  language: string;
  notifications: boolean;
}

const userPrefs: UserPreferences = {
  theme: "dark",
  language: "en",
  notifications: true,
};
```

**Avoid:**

```typescript
// Implicit types can be unclear
function calculateTotal(price, tax) {
  return price + price * tax;
}

const userPrefs = {
  theme: "dark",
  language: "en",
  notifications: true,
};
```

### Interface vs Type Aliases

Use interfaces for object shapes and type aliases for union types:

```typescript
// Use interfaces for object structures
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Use type aliases for unions and computed types
type Status = "loading" | "success" | "error";
type EventHandler = (event: ClickEvent) => void;
```

## Component Development Best Practices

### Component Composition and Reusability

Create composable and reusable components:

```typescript
// Good: Reusable button component
@Component
export struct CustomButton {
  @Prop label: string = '';
  @Prop disabled: boolean = false;
  @Prop type: 'primary' | 'secondary' = 'primary';
  private onClickHandler?: () => void;

  build() {
    Button(this.label)
      .enabled(!this.disabled)
      .backgroundColor(this.type === 'primary' ? Color.Blue : Color.Gray)
      .onClick(() => {
        if (this.onClickHandler) {
          this.onClickHandler();
        }
      })
  }

  setClickHandler(handler: () => void): CustomButton {
    this.onClickHandler = handler;
    return this;
  }
}
```

### State Management Best Practices

Use appropriate state management for different scenarios:

```typescript
// Local state for component-specific data
@Component
struct CounterComponent {
  @State private count: number = 0;

  build() {
    Column() {
      Text(`Count: ${this.count}`)
      Button('Increment')
        .onClick(() => {
          this.count++;
        })
    }
  }
}

// Shared state for data that needs to be accessed across components
@ObservedV2
class AppState {
  @Trace user: UserInfo | null = null;
  @Trace isLoggedIn: boolean = false;
}

// Global state instance
export const appState = new AppState();
```

## Error Handling and Defensive Programming

### Proper Error Handling

Implement comprehensive error handling:

```typescript
// Good: Comprehensive error handling
async function fetchUserData(userId: string): Promise<UserInfo | null> {
  try {
    if (!userId || userId.trim() === "") {
      throw new Error("User ID is required");
    }

    const response = await httpRequest({
      method: http.RequestMethod.GET,
      url: `${API_BASE_URL}/users/${userId}`,
      expectDataType: http.DataType.JSON,
    });

    if (response.responseCode !== 200) {
      throw new Error(`HTTP ${response.responseCode}: ${response.result}`);
    }

    return response.result as UserInfo;
  } catch (error) {
    console.error("Failed to fetch user data:", error);
    promptAction.showToast({
      message: "Failed to load user information",
      duration: 2000,
    });
    return null;
  }
}
```

### Input Validation

Always validate input parameters:

```typescript
// Input validation utility
class ValidationUtils {
  static isValidEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  static isValidPhoneNumber(phone: string): boolean {
    const phoneRegex = /^1[3-9]\d{9}$/;
    return phoneRegex.test(phone);
  }

  static isNotEmpty(value: string): boolean {
    return value.trim().length > 0;
  }
}

// Usage in component
@Component
struct RegistrationForm {
  @State private email: string = '';
  @State private emailError: string = '';

  private validateEmail(): boolean {
    if (!ValidationUtils.isNotEmpty(this.email)) {
      this.emailError = 'Email is required';
      return false;
    }

    if (!ValidationUtils.isValidEmail(this.email)) {
      this.emailError = 'Please enter a valid email address';
      return false;
    }

    this.emailError = '';
    return true;
  }
}
```

## Performance Optimization

### Efficient State Updates

Minimize unnecessary re-renders by using proper state management:

```typescript
// Good: Granular state updates
@Component
struct UserList {
  @State private users: UserInfo[] = [];
  @State private loading: boolean = false;
  @State private selectedUserId: string = '';

  // Only update specific properties when needed
  private selectUser(userId: string): void {
    this.selectedUserId = userId; // Only triggers re-render for selection state
  }

  private addUser(newUser: UserInfo): void {
    this.users = [...this.users, newUser]; // Immutable update
  }
}
```

### Lazy Loading and Virtual Scrolling

Use lazy loading for large datasets:

```typescript
@Component
struct LazyUserList {
  @State private users: UserInfo[] = [];
  private readonly PAGE_SIZE: number = 20;
  private currentPage: number = 0;

  private async loadMoreUsers(): Promise<void> {
    try {
      const newUsers = await this.fetchUsers(this.currentPage, this.PAGE_SIZE);
      this.users = [...this.users, ...newUsers];
      this.currentPage++;
    } catch (error) {
      console.error('Failed to load more users:', error);
    }
  }

  build() {
    List() {
      LazyForEach(this.users, (user: UserInfo) => {
        ListItem() {
          UserItemComponent({ user: user })
        }
      })
    }
    .onReachEnd(() => {
      this.loadMoreUsers();
    })
  }
}
```

## Memory Management

### Resource Cleanup

Properly manage resources and prevent memory leaks:

```typescript
@Component
struct TimerComponent {
  @State private counter: number = 0;
  private timerId: number = -1;

  aboutToAppear(): void {
    this.startTimer();
  }

  aboutToDisappear(): void {
    this.cleanup();
  }

  private startTimer(): void {
    this.timerId = setInterval(() => {
      this.counter++;
    }, 1000);
  }

  private cleanup(): void {
    if (this.timerId !== -1) {
      clearInterval(this.timerId);
      this.timerId = -1;
    }
  }
}
```

## Code Documentation and Comments

### Effective Documentation

Write clear and meaningful documentation:

````typescript
/**
 * Manages user authentication and session handling
 *
 * @example
 * ```typescript
 * const authManager = new AuthManager();
 * const result = await authManager.login('user@example.com', 'password');
 * if (result.success) {
 *   // Handle successful login
 * }
 * ```
 */
class AuthManager {
  /**
   * Authenticates a user with email and password
   *
   * @param email - User's email address
   * @param password - User's password
   * @returns Promise resolving to authentication result
   * @throws {AuthError} When authentication fails
   */
  async login(email: string, password: string): Promise<AuthResult> {
    // Implementation here
  }

  /**
   * Logs out the current user and clears session data
   *
   * @returns Promise that resolves when logout is complete
   */
  async logout(): Promise<void> {
    // Implementation here
  }
}
````

## Testing Best Practices

### Unit Testing Guidelines

Write comprehensive unit tests:

```typescript
// UserService.test.ets
import { describe, it, expect, beforeEach } from "@ohos/hypium";
import { UserService } from "../model/UserService";

describe("UserService", () => {
  let userService: UserService;

  beforeEach(() => {
    userService = new UserService();
  });

  it("should validate user email correctly", () => {
    // Valid email tests
    expect(userService.isValidEmail("user@example.com")).toBe(true);
    expect(userService.isValidEmail("test.email+tag@domain.co.uk")).toBe(true);

    // Invalid email tests
    expect(userService.isValidEmail("invalid-email")).toBe(false);
    expect(userService.isValidEmail("")).toBe(false);
    expect(userService.isValidEmail("user@")).toBe(false);
  });

  it("should handle user creation with validation", async () => {
    const validUser = {
      email: "test@example.com",
      name: "Test User",
      age: 25,
    };

    const result = await userService.createUser(validUser);
    expect(result.success).toBe(true);
    expect(result.user.id).toBeDefined();
  });
});
```

## Security Considerations

### Data Sanitization

Always sanitize user input:

```typescript
class SecurityUtils {
  /**
   * Sanitizes user input to prevent XSS attacks
   */
  static sanitizeInput(input: string): string {
    return input
      .replace(/[<>]/g, "") // Remove potential script tags
      .replace(/['"]/g, "") // Remove quotes
      .trim();
  }

  /**
   * Validates and sanitizes file paths
   */
  static sanitizeFilePath(path: string): string {
    return path
      .replace(/\.\./g, "") // Prevent directory traversal
      .replace(/[<>:"|?*]/g, "") // Remove invalid characters
      .trim();
  }
}
```

### Secure Data Storage

Use appropriate storage methods for sensitive data:

```typescript
class SecureStorageManager {
  private static readonly SENSITIVE_DATA_KEY = "user_credentials";

  /**
   * Stores sensitive data using encrypted preferences
   */
  static async storeSensitiveData(data: object): Promise<void> {
    try {
      const preferences = await dataPreferences.getPreferences(
        getContext(this),
        "encrypted_prefs"
      );

      await preferences.put(this.SENSITIVE_DATA_KEY, JSON.stringify(data));
      await preferences.flush();
    } catch (error) {
      console.error("Failed to store sensitive data:", error);
      throw new Error("Storage operation failed");
    }
  }

  /**
   * Retrieves sensitive data with proper error handling
   */
  static async getSensitiveData(): Promise<object | null> {
    try {
      const preferences = await dataPreferences.getPreferences(
        getContext(this),
        "encrypted_prefs"
      );

      const data = await preferences.get(this.SENSITIVE_DATA_KEY, "");
      return data ? JSON.parse(data as string) : null;
    } catch (error) {
      console.error("Failed to retrieve sensitive data:", error);
      return null;
    }
  }
}
```

## Code Review Checklist

When reviewing ArkTS code, check for:

### 1. Type Safety

- [ ] All variables have explicit types where needed
- [ ] No use of `any` type without justification
- [ ] Proper interface definitions for complex objects

### 2. Error Handling

- [ ] All async operations have proper error handling
- [ ] User-facing errors have meaningful messages
- [ ] Network failures are handled gracefully

### 3. Performance

- [ ] No unnecessary re-renders or state updates
- [ ] Proper use of lazy loading for large datasets
- [ ] Resources are properly cleaned up

### 4. Security

- [ ] User input is validated and sanitized
- [ ] Sensitive data is stored securely
- [ ] No hardcoded secrets or credentials

### 5. Maintainability

- [ ] Code is well-documented and commented
- [ ] Functions are focused and have single responsibilities
- [ ] Consistent naming conventions are followed

## Conclusion

Following these best practices will help you write high-quality ArkTS code that is maintainable, performant, and secure. Remember that code quality is not just about functionality—it's about creating code that other developers (including your future self) can easily understand, modify, and extend.

Key takeaways:

1. **Type Safety First**: Always use explicit types and leverage ArkTS's static typing capabilities
2. **Component Design**: Create reusable, composable components with clear interfaces
3. **Error Handling**: Implement comprehensive error handling and input validation
4. **Performance**: Consider performance implications in state management and rendering
5. **Security**: Always validate and sanitize user input and handle sensitive data properly
6. **Documentation**: Write clear, helpful documentation and comments
7. **Testing**: Write comprehensive tests to ensure code reliability

By incorporating these practices into your development workflow, you'll build more robust and maintainable HarmonyOS applications with ArkTS.
