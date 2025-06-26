# 24-Page Routing and Navigation

## Introduction

Page routing and navigation are fundamental aspects of HarmonyOS application development that determine how users move between different screens and sections of your app. This article covers the routing system, navigation patterns, parameter passing, and best practices for creating seamless user experiences.

## Router Basics

### Basic Navigation

```typescript
import router from '@ohos.router';

// Navigate to a page
@Component
struct HomePage {
  private navigateToDetail(): void {
    router.pushUrl({
      url: 'pages/DetailPage',
      params: {
        id: 123,
        title: 'Sample Item'
      }
    }).catch((error: Error) => {
      console.error('Navigation failed:', error);
    });
  }

  build() {
    Column() {
      Text('Home Page')
        .fontSize(24)
        .margin({ bottom: 20 })

      Button('Go to Detail')
        .onClick(() => {
          this.navigateToDetail();
        })
    }
    .padding(20)
  }
}

// Destination page
@Entry
@Component
struct DetailPage {
  @State private itemId: number = 0;
  @State private itemTitle: string = '';

  aboutToAppear(): void {
    const params = router.getParams() as Record<string, Object>;
    this.itemId = params.id as number;
    this.itemTitle = params.title as string;
  }

  private goBack(): void {
    router.back();
  }

  build() {
    Column() {
      Row() {
        Button('← Back')
          .onClick(() => {
            this.goBack();
          })

        Blank()

        Text('Detail Page')
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
      }
      .width('100%')
      .margin({ bottom: 20 })

      Text(`Item ID: ${this.itemId}`)
        .margin({ bottom: 10 })

      Text(`Title: ${this.itemTitle}`)
        .margin({ bottom: 10 })
    }
    .padding(20)
  }
}
```

### Navigation Methods

```typescript
// Navigation service wrapper
export class NavigationService {

  // Push new page onto stack
  static async pushPage(url: string, params?: Record<string, Object>): Promise<void> {
    try {
      await router.pushUrl({
        url: url,
        params: params
      });
    } catch (error) {
      console.error(`Failed to navigate to ${url}:`, error);
      throw error;
    }
  }

  // Replace current page
  static async replacePage(url: string, params?: Record<string, Object>): Promise<void> {
    try {
      await router.replaceUrl({
        url: url,
        params: params
      });
    } catch (error) {
      console.error(`Failed to replace with ${url}:`, error);
      throw error;
    }
  }

  // Go back to previous page
  static goBack(): void {
    router.back();
  }

  // Clear stack and navigate to page
  static async clearAndNavigate(url: string, params?: Record<string, Object>): Promise<void> {
    try {
      await router.clear();
      await this.pushPage(url, params);
    } catch (error) {
      console.error(`Failed to clear and navigate to ${url}:`, error);
      throw error;
    }
  }

  // Get current route parameters
  static getRouteParams<T = Record<string, Object>>(): T {
    return router.getParams() as T;
  }

  // Get route stack length
  static getStackLength(): number {
    return router.getLength();
  }
}

// Usage example
@Component
struct NavigationExample {
  private async handleLogin(): Promise<void> {
    try {
      // Simulate login process
      const loginSuccess = await this.performLogin();

      if (loginSuccess) {
        // Clear navigation stack and go to main page
        await NavigationService.clearAndNavigate('pages/MainPage', {
          userId: 'user123',
          loginTime: new Date().toISOString()
        });
      }
    } catch (error) {
      console.error('Login failed:', error);
    }
  }

  private async performLogin(): Promise<boolean> {
    // Simulated login logic
    return new Promise((resolve) => {
      setTimeout(() => resolve(true), 1000);
    });
  }

  build() {
    Column() {
      Button('Login')
        .onClick(() => {
          this.handleLogin();
        })
    }
  }
}
```

## Tab Navigation

### Bottom Tab Navigation

```typescript
@Entry
@Component
struct MainTabPage {
  @State private currentTab: number = 0;
  private readonly tabs = [
    { name: 'Home', icon: $r('app.media.icon_home') },
    { name: 'Search', icon: $r('app.media.icon_search') },
    { name: 'Profile', icon: $r('app.media.icon_profile') },
    { name: 'Settings', icon: $r('app.media.icon_settings') }
  ];

  @Builder
  TabBuilder(title: string, targetIndex: number, selectedImg: Resource, normalImg: Resource) {
    Column() {
      Image(this.currentTab === targetIndex ? selectedImg : normalImg)
        .size({ width: 25, height: 25 })
      Text(title)
        .fontColor(this.currentTab === targetIndex ? '#1698CE' : '#6B6B6B')
        .fontSize(10)
        .margin({ top: 4 })
    }
    .width('100%')
    .height(50)
    .justifyContent(FlexAlign.Center)
  }

  build() {
    Tabs({ barPosition: BarPosition.End, controller: new TabsController() }) {
      TabContent() {
        HomeTabContent()
      }
      .tabBar(this.TabBuilder('Home', 0, $r('app.media.icon_home_selected'), $r('app.media.icon_home')))

      TabContent() {
        SearchTabContent()
      }
      .tabBar(this.TabBuilder('Search', 1, $r('app.media.icon_search_selected'), $r('app.media.icon_search')))

      TabContent() {
        ProfileTabContent()
      }
      .tabBar(this.TabBuilder('Profile', 2, $r('app.media.icon_profile_selected'), $r('app.media.icon_profile')))

      TabContent() {
        SettingsTabContent()
      }
      .tabBar(this.TabBuilder('Settings', 3, $r('app.media.icon_settings_selected'), $r('app.media.icon_settings')))
    }
    .onChange((index: number) => {
      this.currentTab = index;
    })
    .width('100%')
    .backgroundColor('#F1F3F5')
  }
}

// Tab content components
@Component
struct HomeTabContent {
  build() {
    Column() {
      Text('Home Content')
        .fontSize(24)
        .textAlign(TextAlign.Center)

      Button('Go to Detail')
        .margin({ top: 20 })
        .onClick(() => {
          NavigationService.pushPage('pages/DetailPage', {
            from: 'home',
            timestamp: Date.now()
          });
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

### Drawer Navigation

```typescript
@Component
struct DrawerNavigation {
  @State private drawerOpen: boolean = false;

  @Builder
  DrawerContent() {
    Column() {
      // User profile section
      Row() {
        Image($r('app.media.default_avatar'))
          .width(50)
          .height(50)
          .borderRadius(25)

        Column() {
          Text('John Doe')
            .fontSize(16)
            .fontWeight(FontWeight.Bold)
          Text('john.doe@example.com')
            .fontSize(12)
            .fontColor(Color.Gray)
        }
        .alignItems(HorizontalAlign.Start)
        .margin({ left: 12 })
      }
      .width('100%')
      .padding(16)
      .margin({ bottom: 20 })

      // Navigation items
      this.DrawerMenuItem('Dashboard', $r('app.media.icon_dashboard'), () => {
        this.navigateToPage('pages/DashboardPage');
      })

      this.DrawerMenuItem('Orders', $r('app.media.icon_orders'), () => {
        this.navigateToPage('pages/OrdersPage');
      })

      this.DrawerMenuItem('Settings', $r('app.media.icon_settings'), () => {
        this.navigateToPage('pages/SettingsPage');
      })

      Divider()
        .margin({ vertical: 16 })

      this.DrawerMenuItem('Logout', $r('app.media.icon_logout'), () => {
        this.handleLogout();
      })
    }
    .width(280)
    .height('100%')
    .backgroundColor(Color.White)
    .padding({ top: 60, left: 20, right: 20 })
  }

  @Builder
  DrawerMenuItem(title: string, icon: Resource, onClick: () => void) {
    Row() {
      Image(icon)
        .width(24)
        .height(24)

      Text(title)
        .fontSize(16)
        .margin({ left: 16 })
    }
    .width('100%')
    .height(48)
    .onClick(() => {
      this.drawerOpen = false;
      onClick();
    })
    .padding({ horizontal: 12 })
    .borderRadius(8)
    .stateStyles({
      pressed: {
        .backgroundColor('#E5E5E5')
      }
    })
  }

  private navigateToPage(url: string): void {
    NavigationService.pushPage(url);
  }

  private handleLogout(): void {
    // Implement logout logic
    NavigationService.clearAndNavigate('pages/LoginPage');
  }

  build() {
    Stack() {
      // Main content
      Column() {
        // App bar
        Row() {
          Button() {
            Image($r('app.media.icon_menu'))
              .width(24)
              .height(24)
          }
          .type(ButtonType.Circle)
          .backgroundColor(Color.Transparent)
          .onClick(() => {
            this.drawerOpen = true;
          })

          Text('App Title')
            .fontSize(18)
            .fontWeight(FontWeight.Bold)
            .margin({ left: 16 })

          Blank()
        }
        .width('100%')
        .height(56)
        .padding({ horizontal: 16 })
        .backgroundColor('#1976D2')

        // Main content area
        Column() {
          Text('Main Content')
            .fontSize(24)
        }
        .layoutWeight(1)
        .justifyContent(FlexAlign.Center)
      }

      // Drawer overlay
      if (this.drawerOpen) {
        Row() {
          this.DrawerContent()

          // Overlay to close drawer
          Column()
            .layoutWeight(1)
            .height('100%')
            .backgroundColor('rgba(0, 0, 0, 0.3)')
            .onClick(() => {
              this.drawerOpen = false;
            })
        }
        .width('100%')
        .height('100%')
        .transition({
          type: TransitionType.Insert,
          opacity: 0,
          translate: { x: -280 }
        })
      }
    }
  }
}
```

## Deep Linking and URL Handling

### URL Scheme Registration

```typescript
// Configure URL schemes in module.json5
{
  "abilities": [
    {
      "name": "EntryAbility",
      "skills": [
        {
          "entities": ["entity.system.default"],
          "actions": ["action.system.home"]
        },
        {
          "entities": ["entity.system.browsable"],
          "actions": ["action.system.browsable"],
          "uris": [
            {
              "scheme": "myapp",
              "host": "open"
            }
          ]
        }
      ]
    }
  ]
}

// Deep link handler
export class DeepLinkHandler {
  static handleDeepLink(uri: string): void {
    try {
      const url = new URL(uri);
      const path = url.pathname;
      const params = this.parseQueryParams(url.search);

      switch (path) {
        case '/product':
          this.handleProductLink(params);
          break;
        case '/user':
          this.handleUserLink(params);
          break;
        case '/order':
          this.handleOrderLink(params);
          break;
        default:
          this.handleDefaultLink();
          break;
      }
    } catch (error) {
      console.error('Invalid deep link:', error);
      this.handleDefaultLink();
    }
  }

  private static parseQueryParams(search: string): Record<string, string> {
    const params: Record<string, string> = {};
    const urlParams = new URLSearchParams(search);

    for (const [key, value] of urlParams.entries()) {
      params[key] = value;
    }

    return params;
  }

  private static handleProductLink(params: Record<string, string>): void {
    const productId = params.id;
    if (productId) {
      NavigationService.pushPage('pages/ProductDetailPage', {
        productId: productId,
        source: 'deeplink'
      });
    }
  }

  private static handleUserLink(params: Record<string, string>): void {
    const userId = params.id;
    if (userId) {
      NavigationService.pushPage('pages/UserProfilePage', {
        userId: userId,
        source: 'deeplink'
      });
    }
  }

  private static handleOrderLink(params: Record<string, string>): void {
    const orderId = params.id;
    if (orderId) {
      NavigationService.pushPage('pages/OrderDetailPage', {
        orderId: orderId,
        source: 'deeplink'
      });
    }
  }

  private static handleDefaultLink(): void {
    NavigationService.clearAndNavigate('pages/MainPage');
  }
}

// Usage in EntryAbility
export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // Handle deep link on app start
    if (want.uri) {
      DeepLinkHandler.handleDeepLink(want.uri);
    }
  }

  onNewWant(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // Handle deep link when app is already running
    if (want.uri) {
      DeepLinkHandler.handleDeepLink(want.uri);
    }
  }
}
```

## Navigation Guards and Middleware

### Route Protection

```typescript
// Authentication service
export class AuthService {
  private static isAuthenticated: boolean = false;

  static login(token: string): void {
    this.isAuthenticated = true;
    // Store token securely
  }

  static logout(): void {
    this.isAuthenticated = false;
    // Clear stored token
  }

  static isLoggedIn(): boolean {
    return this.isAuthenticated;
  }
}

// Navigation guard
export class NavigationGuard {
  private static protectedRoutes = new Set([
    'pages/ProfilePage',
    'pages/OrdersPage',
    'pages/SettingsPage',
    'pages/AdminPage'
  ]);

  static canNavigate(url: string): boolean {
    if (this.protectedRoutes.has(url)) {
      return AuthService.isLoggedIn();
    }
    return true;
  }

  static async navigateWithGuard(url: string, params?: Record<string, Object>): Promise<void> {
    if (this.canNavigate(url)) {
      await NavigationService.pushPage(url, params);
    } else {
      // Redirect to login with return URL
      await NavigationService.pushPage('pages/LoginPage', {
        returnUrl: url,
        returnParams: params
      });
    }
  }
}

// Protected navigation component
@Component
struct ProtectedNavigation {
  private navigateToProfile(): void {
    NavigationGuard.navigateWithGuard('pages/ProfilePage');
  }

  private navigateToOrders(): void {
    NavigationGuard.navigateWithGuard('pages/OrdersPage');
  }

  build() {
    Column() {
      Button('View Profile')
        .onClick(() => {
          this.navigateToProfile();
        })
        .margin({ bottom: 16 })

      Button('View Orders')
        .onClick(() => {
          this.navigateToOrders();
        })
    }
    .padding(20)
  }
}

// Login page with return navigation
@Entry
@Component
struct LoginPage {
  @State private returnUrl: string = '';
  @State private returnParams: Record<string, Object> = {};

  aboutToAppear(): void {
    const params = NavigationService.getRouteParams();
    this.returnUrl = params.returnUrl as string || 'pages/MainPage';
    this.returnParams = params.returnParams as Record<string, Object> || {};
  }

  private async handleLogin(): Promise<void> {
    try {
      // Simulate login
      await this.performLogin();

      // Navigate to originally requested page or main page
      if (this.returnUrl && this.returnUrl !== 'pages/MainPage') {
        await NavigationService.replacePage(this.returnUrl, this.returnParams);
      } else {
        await NavigationService.replacePage('pages/MainPage');
      }
    } catch (error) {
      console.error('Login failed:', error);
    }
  }

  private async performLogin(): Promise<void> {
    // Simulate API call
    return new Promise((resolve) => {
      setTimeout(() => {
        AuthService.login('sample_token');
        resolve();
      }, 1000);
    });
  }

  build() {
    Column() {
      Text('Login Required')
        .fontSize(24)
        .margin({ bottom: 20 })

      if (this.returnUrl !== 'pages/MainPage') {
        Text(`You need to login to access ${this.returnUrl}`)
          .fontSize(14)
          .fontColor(Color.Gray)
          .margin({ bottom: 20 })
      }

      Button('Login')
        .onClick(() => {
          this.handleLogin();
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .padding(20)
  }
}
```

## Navigation State Management

### Navigation History Tracking

```typescript
// Navigation history service
@Observed
class NavigationHistory {
  history: NavigationEntry[] = [];
  currentIndex: number = -1;

  addEntry(url: string, params?: Record<string, Object>): void {
    const entry: NavigationEntry = {
      url: url,
      params: params || {},
      timestamp: Date.now(),
      id: this.generateId(),
    };

    // Remove any entries after current index
    this.history = this.history.slice(0, this.currentIndex + 1);

    // Add new entry
    this.history.push(entry);
    this.currentIndex = this.history.length - 1;

    // Limit history size
    if (this.history.length > 50) {
      this.history.shift();
      this.currentIndex--;
    }
  }

  canGoBack(): boolean {
    return this.currentIndex > 0;
  }

  canGoForward(): boolean {
    return this.currentIndex < this.history.length - 1;
  }

  goBack(): NavigationEntry | null {
    if (this.canGoBack()) {
      this.currentIndex--;
      return this.history[this.currentIndex];
    }
    return null;
  }

  goForward(): NavigationEntry | null {
    if (this.canGoForward()) {
      this.currentIndex++;
      return this.history[this.currentIndex];
    }
    return null;
  }

  getCurrentEntry(): NavigationEntry | null {
    return this.currentIndex >= 0 ? this.history[this.currentIndex] : null;
  }

  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
}

interface NavigationEntry {
  id: string;
  url: string;
  params: Record<string, Object>;
  timestamp: number;
}

// Enhanced navigation service with history
export class EnhancedNavigationService {
  private static history = new NavigationHistory();

  static async pushPage(
    url: string,
    params?: Record<string, Object>
  ): Promise<void> {
    try {
      await router.pushUrl({ url, params });
      this.history.addEntry(url, params);
    } catch (error) {
      console.error(`Failed to navigate to ${url}:`, error);
      throw error;
    }
  }

  static goBack(): void {
    const previousEntry = this.history.goBack();
    if (previousEntry) {
      router.back();
    }
  }

  static getNavigationHistory(): NavigationHistory {
    return this.history;
  }

  static canGoBack(): boolean {
    return this.history.canGoBack();
  }
}
```

## Best Practices

### 1. Navigation Architecture

- Design consistent navigation patterns
- Implement proper back button handling
- Use appropriate navigation methods for different scenarios
- Plan deep linking strategy early

### 2. Performance Optimization

- Lazy load pages when possible
- Implement proper page lifecycle management
- Cache frequently accessed pages
- Optimize parameter passing

### 3. User Experience

- Provide visual feedback during navigation
- Implement smooth transitions
- Handle navigation errors gracefully
- Maintain navigation state across app lifecycle

## Conclusion

Effective page routing and navigation are crucial for creating intuitive HarmonyOS applications. By implementing proper navigation patterns, handling deep links, and managing navigation state, you can create seamless user experiences that guide users through your application efficiently.

Key takeaways:

1. **Choose Appropriate Navigation Patterns**: Use tabs, drawers, or stack navigation based on your app structure
2. **Implement Route Protection**: Guard sensitive pages with authentication checks
3. **Handle Deep Links**: Support external navigation into your application
4. **Optimize Performance**: Implement efficient navigation and page management
5. **Maintain State**: Track navigation history and handle app lifecycle properly
   </rewritten_file>
