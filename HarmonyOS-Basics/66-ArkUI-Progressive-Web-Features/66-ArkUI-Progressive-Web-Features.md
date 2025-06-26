# ArkUI Progressive Web Features

## Introduction

Progressive Web Apps (PWA) capabilities enhance ArkUI applications with web-like features including service workers, offline functionality, and native app experiences. This guide explores PWA patterns and web integration techniques.

## Service Worker Integration

### Service Worker Manager

```typescript
class ServiceWorkerManager {
  private worker: ServiceWorker | null = null;
  private registration: ServiceWorkerRegistration | null = null;

  async initialize(): Promise<void> {
    if ("serviceWorker" in navigator) {
      try {
        this.registration = await navigator.serviceWorker.register("/sw.js");
        this.worker = this.registration.active;
        this.setupEventListeners();
      } catch (error) {
        console.error("Service Worker registration failed:", error);
      }
    }
  }

  async postMessage(message: any): Promise<any> {
    return new Promise((resolve, reject) => {
      if (!this.worker) {
        reject(new Error("Service Worker not available"));
        return;
      }

      const channel = new MessageChannel();
      channel.port1.onmessage = (event) => {
        if (event.data.error) {
          reject(new Error(event.data.error));
        } else {
          resolve(event.data);
        }
      };

      this.worker.postMessage(message, [channel.port2]);
    });
  }

  private setupEventListeners(): void {
    navigator.serviceWorker.addEventListener("message", (event) => {
      this.handleMessage(event.data);
    });

    navigator.serviceWorker.addEventListener("controllerchange", () => {
      this.worker = navigator.serviceWorker.controller;
    });
  }

  private handleMessage(data: any): void {
    switch (data.type) {
      case "CACHE_UPDATED":
        this.notifyCacheUpdate(data.payload);
        break;
      case "SYNC_BACKGROUND":
        this.handleBackgroundSync(data.payload);
        break;
    }
  }

  private notifyCacheUpdate(payload: any): void {
    // Handle cache updates
  }

  private handleBackgroundSync(payload: any): void {
    // Handle background sync events
  }
}
```

### Offline Storage Strategy

```typescript
interface CacheStrategy {
  name: string;
  match(request: Request): boolean;
  handle(request: Request): Promise<Response>;
}

class CacheFirstStrategy implements CacheStrategy {
  name = "cache-first";

  match(request: Request): boolean {
    return request.destination === "image" || request.url.includes("/static/");
  }

  async handle(request: Request): Promise<Response> {
    const cache = await caches.open("static-v1");
    const cached = await cache.match(request);

    if (cached) {
      return cached;
    }

    const response = await fetch(request);
    if (response.ok) {
      cache.put(request, response.clone());
    }

    return response;
  }
}

class NetworkFirstStrategy implements CacheStrategy {
  name = "network-first";

  match(request: Request): boolean {
    return request.url.includes("/api/");
  }

  async handle(request: Request): Promise<Response> {
    try {
      const response = await fetch(request);

      if (response.ok) {
        const cache = await caches.open("api-v1");
        cache.put(request, response.clone());
      }

      return response;
    } catch {
      const cache = await caches.open("api-v1");
      const cached = await cache.match(request);

      if (cached) {
        return cached;
      }

      throw new Error("Network and cache failed");
    }
  }
}

class OfflineManager {
  private strategies: CacheStrategy[] = [
    new CacheFirstStrategy(),
    new NetworkFirstStrategy(),
  ];

  async handleRequest(request: Request): Promise<Response> {
    const strategy = this.strategies.find((s) => s.match(request));

    if (strategy) {
      return strategy.handle(request);
    }

    return fetch(request);
  }

  async preCache(urls: string[]): Promise<void> {
    const cache = await caches.open("precache-v1");
    await cache.addAll(urls);
  }
}
```

## Web Integration Features

### Push Notifications

```typescript
interface NotificationPayload {
  title: string;
  body: string;
  icon?: string;
  badge?: string;
  tag?: string;
  data?: any;
}

class PushNotificationManager {
  private registration: ServiceWorkerRegistration | null = null;

  async initialize(): Promise<void> {
    if ("serviceWorker" in navigator && "PushManager" in window) {
      this.registration = await navigator.serviceWorker.ready;
    }
  }

  async requestPermission(): Promise<boolean> {
    const permission = await Notification.requestPermission();
    return permission === "granted";
  }

  async subscribe(publicKey: string): Promise<PushSubscription | null> {
    if (!this.registration) return null;

    try {
      const subscription = await this.registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(publicKey),
      });

      return subscription;
    } catch (error) {
      console.error("Push subscription failed:", error);
      return null;
    }
  }

  async unsubscribe(): Promise<boolean> {
    if (!this.registration) return false;

    const subscription = await this.registration.pushManager.getSubscription();
    if (subscription) {
      return subscription.unsubscribe();
    }

    return false;
  }

  async showNotification(payload: NotificationPayload): Promise<void> {
    if (!this.registration) return;

    await this.registration.showNotification(payload.title, {
      body: payload.body,
      icon: payload.icon,
      badge: payload.badge,
      tag: payload.tag,
      data: payload.data,
    });
  }

  private urlBase64ToUint8Array(base64String: string): Uint8Array {
    const padding = "=".repeat((4 - (base64String.length % 4)) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, "+")
      .replace(/_/g, "/");

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }

    return outputArray;
  }
}
```

### Background Sync

```typescript
interface SyncTask {
  id: string;
  data: any;
  timestamp: number;
  retries: number;
}

class BackgroundSyncManager {
  private pendingTasks: SyncTask[] = [];
  private maxRetries = 3;

  async scheduleSync(tag: string, data: any): Promise<void> {
    const task: SyncTask = {
      id: this.generateId(),
      data,
      timestamp: Date.now(),
      retries: 0,
    };

    this.pendingTasks.push(task);
    this.saveToStorage();

    if (
      "serviceWorker" in navigator &&
      "sync" in window.ServiceWorkerRegistration.prototype
    ) {
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register(tag);
    } else {
      // Fallback for unsupported browsers
      this.processTasksWhenOnline();
    }
  }

  async processPendingTasks(): Promise<void> {
    const tasks = [...this.pendingTasks];

    for (const task of tasks) {
      try {
        await this.executeTask(task);
        this.removeTask(task.id);
      } catch (error) {
        task.retries++;
        if (task.retries >= this.maxRetries) {
          this.removeTask(task.id);
        }
      }
    }

    this.saveToStorage();
  }

  private async executeTask(task: SyncTask): Promise<void> {
    // Execute the actual sync operation
    await fetch("/api/sync", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(task.data),
    });
  }

  private removeTask(id: string): void {
    this.pendingTasks = this.pendingTasks.filter((task) => task.id !== id);
  }

  private saveToStorage(): void {
    localStorage.setItem("pendingTasks", JSON.stringify(this.pendingTasks));
  }

  private loadFromStorage(): void {
    const stored = localStorage.getItem("pendingTasks");
    if (stored) {
      this.pendingTasks = JSON.parse(stored);
    }
  }

  private processTasksWhenOnline(): void {
    const processWhenOnline = () => {
      if (navigator.onLine) {
        this.processPendingTasks();
      } else {
        setTimeout(processWhenOnline, 5000);
      }
    };

    processWhenOnline();
  }

  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
}
```

## PWA Component Integration

### Install Prompt Component

```typescript
@Component
struct PWAInstallPrompt {
  @State private showInstallPrompt: boolean = false
  @State private deferredPrompt: any = null

  build() {
    if (this.showInstallPrompt) {
      Column() {
        Row() {
          Column() {
            Text('Install App')
              .fontSize(16)
              .fontWeight(FontWeight.Bold)

            Text('Add this app to your home screen for easy access')
              .fontSize(14)
              .fontColor('#666')
              .margin({ top: 4 })
          }
          .alignItems(HorizontalAlign.Start)
          .flexGrow(1)

          Row() {
            Button('Install')
              .fontSize(14)
              .backgroundColor('#007AFF')
              .onClick(() => this.installApp())

            Button('×')
              .fontSize(16)
              .backgroundColor(Color.Transparent)
              .fontColor('#666')
              .margin({ left: 8 })
              .onClick(() => this.dismissPrompt())
          }
        }
        .padding(16)
        .backgroundColor('#f8f9fa')
        .borderRadius(8)
        .border({ width: 1, color: '#e9ecef' })
      }
      .margin(16)
    }
  }

  aboutToAppear() {
    this.setupInstallPrompt()
  }

  private setupInstallPrompt(): void {
    window.addEventListener('beforeinstallprompt', (event) => {
      event.preventDefault()
      this.deferredPrompt = event
      this.showInstallPrompt = true
    })

    window.addEventListener('appinstalled', () => {
      this.showInstallPrompt = false
      this.deferredPrompt = null
    })
  }

  private async installApp(): Promise<void> {
    if (this.deferredPrompt) {
      this.deferredPrompt.prompt()
      const { outcome } = await this.deferredPrompt.userChoice

      if (outcome === 'accepted') {
        this.showInstallPrompt = false
      }

      this.deferredPrompt = null
    }
  }

  private dismissPrompt(): void {
    this.showInstallPrompt = false
    this.deferredPrompt = null
  }
}
```

### Offline Indicator

```typescript
@Component
struct OfflineIndicator {
  @State private isOnline: boolean = navigator.onLine

  build() {
    if (!this.isOnline) {
      Row() {
        Text('📶')
          .fontSize(16)
          .margin({ right: 8 })

        Text('You are offline')
          .fontSize(14)
          .fontColor(Color.White)
      }
      .padding({ horizontal: 16, vertical: 8 })
      .backgroundColor('#FF6B6B')
      .width('100%')
      .justifyContent(FlexAlign.Center)
    }
  }

  aboutToAppear() {
    this.setupOnlineListeners()
  }

  private setupOnlineListeners(): void {
    window.addEventListener('online', () => {
      this.isOnline = true
    })

    window.addEventListener('offline', () => {
      this.isOnline = false
    })
  }
}
```

### Web Share Integration

```typescript
class WebShareManager {
  async share(data: { title?: string; text?: string; url?: string }): Promise<boolean> {
    if ('share' in navigator) {
      try {
        await navigator.share(data)
        return true
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Sharing failed:', error)
        }
        return false
      }
    } else {
      return this.fallbackShare(data)
    }
  }

  private fallbackShare(data: { title?: string; text?: string; url?: string }): boolean {
    if ('clipboard' in navigator) {
      const text = `${data.title || ''}\n${data.text || ''}\n${data.url || ''}`
      navigator.clipboard.writeText(text.trim())
      return true
    }
    return false
  }

  canShare(): boolean {
    return 'share' in navigator
  }
}

@Component
struct ShareButton {
  private shareManager = new WebShareManager()
  @Prop shareData: { title?: string; text?: string; url?: string } = {}

  build() {
    Button('Share')
      .onClick(() => this.handleShare())
  }

  private async handleShare(): Promise<void> {
    const success = await this.shareManager.share(this.shareData)

    if (!success) {
      // Show fallback share options or copy to clipboard
      this.showShareOptions()
    }
  }

  private showShareOptions(): void {
    // Show custom share dialog
  }
}
```

## PWA App Shell

```typescript
@Component
struct PWAAppShell {
  @State private isLoading: boolean = true
  @State private hasUpdate: boolean = false
  private serviceWorkerManager = new ServiceWorkerManager()
  private pushManager = new PushNotificationManager()

  build() {
    Column() {
      PWAInstallPrompt()
      OfflineIndicator()

      if (this.hasUpdate) {
        this.buildUpdateBanner()
      }

      if (this.isLoading) {
        this.buildLoadingScreen()
      } else {
        this.buildMainContent()
      }
    }
  }

  aboutToAppear() {
    this.initializePWA()
  }

  @Builder
  private buildUpdateBanner() {
    Row() {
      Text('New version available')
        .fontSize(14)
        .fontColor(Color.White)
        .flexGrow(1)

      Button('Update')
        .fontSize(12)
        .backgroundColor(Color.White)
        .fontColor('#007AFF')
        .onClick(() => this.updateApp())
    }
    .padding(12)
    .backgroundColor('#007AFF')
    .width('100%')
  }

  @Builder
  private buildLoadingScreen() {
    Column() {
      LoadingProgress()
        .width(40)
        .height(40)
        .color('#007AFF')

      Text('Loading...')
        .fontSize(16)
        .margin({ top: 16 })
    }
    .justifyContent(FlexAlign.Center)
    .alignItems(HorizontalAlign.Center)
    .width('100%')
    .height('100%')
  }

  @Builder
  private buildMainContent() {
    // Main app content
    Text('PWA App Content')
      .fontSize(20)
      .textAlign(TextAlign.Center)
      .width('100%')
      .height('100%')
  }

  private async initializePWA(): Promise<void> {
    try {
      await this.serviceWorkerManager.initialize()
      await this.pushManager.initialize()

      // Check for updates
      this.checkForUpdates()

      this.isLoading = false
    } catch (error) {
      console.error('PWA initialization failed:', error)
      this.isLoading = false
    }
  }

  private checkForUpdates(): void {
    // Check for service worker updates
    navigator.serviceWorker.addEventListener('controllerchange', () => {
      this.hasUpdate = true
    })
  }

  private updateApp(): void {
    window.location.reload()
  }
}
```

## Conclusion

Progressive Web Features in ArkUI provide:

- Service worker integration for offline functionality
- Push notification support for user engagement
- Background sync for reliable data synchronization
- Web share API integration for native sharing
- Install prompts for app-like experiences
- Offline indicators and caching strategies

These features enable creating app-like experiences that work reliably across different network conditions and device capabilities.
