# ArkTS Advanced Networking and WebSockets

## Introduction

Modern applications require sophisticated networking capabilities for real-time communication, data synchronization, and seamless user experiences. This guide explores advanced networking patterns, WebSocket implementation, and real-time communication strategies in ArkTS.

## WebSocket Implementation

### Advanced WebSocket Manager

```typescript
enum ConnectionState {
  Disconnected = "disconnected",
  Connecting = "connecting",
  Connected = "connected",
  Reconnecting = "reconnecting",
  Failed = "failed",
}

interface WebSocketConfig {
  url: string;
  protocols?: string[];
  reconnect: boolean;
  maxReconnectAttempts: number;
  reconnectInterval: number;
  heartbeatInterval: number;
  messageQueueSize: number;
  binaryType?: "blob" | "arraybuffer";
}

interface WebSocketMessage {
  id: string;
  type: string;
  payload: any;
  timestamp: number;
  priority: MessagePriority;
}

enum MessagePriority {
  Low = "low",
  Normal = "normal",
  High = "high",
  Critical = "critical",
}

class AdvancedWebSocketManager {
  private socket: WebSocket | null = null;
  private config: WebSocketConfig;
  private state: ConnectionState = ConnectionState.Disconnected;
  private reconnectAttempts: number = 0;
  private messageQueue: WebSocketMessage[] = [];
  private heartbeatTimer: number | null = null;
  private reconnectTimer: number | null = null;
  private listeners = new Map<string, Set<Function>>();

  constructor(config: WebSocketConfig) {
    this.config = config;
  }

  async connect(): Promise<void> {
    if (
      this.state === ConnectionState.Connected ||
      this.state === ConnectionState.Connecting
    ) {
      return Promise.resolve();
    }

    return new Promise((resolve, reject) => {
      this.setState(ConnectionState.Connecting);

      try {
        this.socket = new WebSocket(this.config.url, this.config.protocols);
        this.setupSocketHandlers(resolve, reject);
      } catch (error) {
        this.setState(ConnectionState.Failed);
        reject(error);
      }
    });
  }

  disconnect(): void {
    this.clearTimers();
    this.reconnectAttempts = 0;

    if (this.socket) {
      this.socket.close(1000, "Normal closure");
      this.socket = null;
    }

    this.setState(ConnectionState.Disconnected);
  }

  send(message: Omit<WebSocketMessage, "id" | "timestamp">): Promise<void> {
    const fullMessage: WebSocketMessage = {
      id: this.generateMessageId(),
      timestamp: Date.now(),
      ...message,
    };

    if (this.state !== ConnectionState.Connected) {
      return this.queueMessage(fullMessage);
    }

    return this.sendMessage(fullMessage);
  }

  subscribe(eventType: string, handler: Function): () => void {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, new Set());
    }

    this.listeners.get(eventType)!.add(handler);

    return () => {
      this.listeners.get(eventType)?.delete(handler);
    };
  }

  getState(): ConnectionState {
    return this.state;
  }

  private setupSocketHandlers(
    resolve: () => void,
    reject: (error: any) => void
  ): void {
    if (!this.socket) return;

    this.socket.onopen = () => {
      this.setState(ConnectionState.Connected);
      this.reconnectAttempts = 0;
      this.startHeartbeat();
      this.processMessageQueue();
      resolve();
    };

    this.socket.onclose = (event) => {
      this.setState(ConnectionState.Disconnected);
      this.clearTimers();

      if (
        this.config.reconnect &&
        this.reconnectAttempts < this.config.maxReconnectAttempts
      ) {
        this.scheduleReconnect();
      } else {
        this.emit("disconnect", { code: event.code, reason: event.reason });
      }
    };

    this.socket.onerror = (error) => {
      this.setState(ConnectionState.Failed);
      this.emit("error", error);
      reject(error);
    };

    this.socket.onmessage = (event) => {
      this.handleMessage(event.data);
    };
  }

  private handleMessage(data: any): void {
    try {
      const message = typeof data === "string" ? JSON.parse(data) : data;

      // Handle system messages
      if (message.type === "heartbeat") {
        this.handleHeartbeat();
        return;
      }

      // Emit message to subscribers
      this.emit("message", message);
      this.emit(message.type, message.payload);
    } catch (error) {
      console.error("Failed to parse WebSocket message:", error);
      this.emit("error", error);
    }
  }

  private async sendMessage(message: WebSocketMessage): Promise<void> {
    if (!this.socket || this.socket.readyState !== WebSocket.OPEN) {
      throw new Error("WebSocket is not connected");
    }

    try {
      const serialized = JSON.stringify(message);
      this.socket.send(serialized);
      this.emit("messageSent", message);
    } catch (error) {
      this.emit("error", error);
      throw error;
    }
  }

  private queueMessage(message: WebSocketMessage): Promise<void> {
    return new Promise((resolve, reject) => {
      if (this.messageQueue.length >= this.config.messageQueueSize) {
        // Remove lowest priority message
        this.removeLowestPriorityMessage();
      }

      this.messageQueue.push(message);
      this.sortMessageQueue();

      // Store resolve/reject for later execution
      message.resolve = resolve;
      message.reject = reject;
    });
  }

  private async processMessageQueue(): Promise<void> {
    while (
      this.messageQueue.length > 0 &&
      this.state === ConnectionState.Connected
    ) {
      const message = this.messageQueue.shift()!;

      try {
        await this.sendMessage(message);
        message.resolve?.();
      } catch (error) {
        message.reject?.(error);
      }
    }
  }

  private scheduleReconnect(): void {
    this.setState(ConnectionState.Reconnecting);
    this.reconnectAttempts++;

    const delay =
      this.config.reconnectInterval * Math.pow(2, this.reconnectAttempts - 1);

    this.reconnectTimer = setTimeout(() => {
      this.connect().catch(() => {
        if (this.reconnectAttempts >= this.config.maxReconnectAttempts) {
          this.setState(ConnectionState.Failed);
          this.emit("reconnectFailed");
        }
      });
    }, delay);
  }

  private startHeartbeat(): void {
    if (this.config.heartbeatInterval <= 0) return;

    this.heartbeatTimer = setInterval(() => {
      this.send({
        type: "heartbeat",
        payload: { timestamp: Date.now() },
        priority: MessagePriority.High,
      }).catch(() => {
        // Heartbeat failed, connection might be lost
        this.disconnect();
      });
    }, this.config.heartbeatInterval);
  }

  private clearTimers(): void {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
      this.heartbeatTimer = null;
    }

    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
      this.reconnectTimer = null;
    }
  }

  private setState(newState: ConnectionState): void {
    const oldState = this.state;
    this.state = newState;
    this.emit("stateChange", { oldState, newState });
  }

  private emit(eventType: string, data?: any): void {
    const handlers = this.listeners.get(eventType);
    if (handlers) {
      handlers.forEach((handler) => handler(data));
    }
  }

  private generateMessageId(): string {
    return `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### Real-time Chat Implementation

```typescript
interface ChatMessage {
  id: string
  userId: string
  username: string
  content: string
  timestamp: number
  type: 'text' | 'image' | 'file'
  metadata?: any
}

interface ChatRoom {
  id: string
  name: string
  users: ChatUser[]
  lastMessage?: ChatMessage
}

interface ChatUser {
  id: string
  username: string
  status: 'online' | 'offline' | 'away'
  avatar?: string
}

class ChatService {
  private wsManager: AdvancedWebSocketManager
  private currentRoom: string | null = null
  private messageHistory = new Map<string, ChatMessage[]>()
  private typingUsers = new Set<string>()
  private eventListeners = new Map<string, Set<Function>>()

  constructor(serverUrl: string) {
    this.wsManager = new AdvancedWebSocketManager({
      url: serverUrl,
      reconnect: true,
      maxReconnectAttempts: 5,
      reconnectInterval: 1000,
      heartbeatInterval: 30000,
      messageQueueSize: 100
    })

    this.setupEventHandlers()
  }

  async connect(): Promise<void> {
    await this.wsManager.connect()
  }

  async joinRoom(roomId: string): Promise<void> {
    this.currentRoom = roomId

    await this.wsManager.send({
      type: 'join_room',
      payload: { roomId },
      priority: MessagePriority.High
    })
  }

  async sendMessage(content: string, type: 'text' | 'image' | 'file' = 'text'): Promise<void> {
    if (!this.currentRoom) {
      throw new Error('Not in a chat room')
    }

    const message: Omit<ChatMessage, 'id'> = {
      userId: this.getCurrentUserId(),
      username: this.getCurrentUsername(),
      content,
      timestamp: Date.now(),
      type
    }

    await this.wsManager.send({
      type: 'chat_message',
      payload: { roomId: this.currentRoom, message },
      priority: MessagePriority.Normal
    })
  }

  async sendTypingIndicator(isTyping: boolean): Promise<void> {
    if (!this.currentRoom) return

    await this.wsManager.send({
      type: 'typing_indicator',
      payload: {
        roomId: this.currentRoom,
        isTyping,
        userId: this.getCurrentUserId()
      },
      priority: MessagePriority.Low
    })
  }

  getMessageHistory(roomId: string): ChatMessage[] {
    return this.messageHistory.get(roomId) || []
  }

  getTypingUsers(): string[] {
    return Array.from(this.typingUsers)
  }

  subscribe(eventType: string, handler: Function): () => void {
    if (!this.eventListeners.has(eventType)) {
      this.eventListeners.set(eventType, new Set())
    }

    this.eventListeners.get(eventType)!.add(handler)

    return () => {
      this.eventListeners.get(eventType)?.delete(handler)
    }
  }

  private setupEventHandlers(): void {
    this.wsManager.subscribe('chat_message', (payload: any) => {
      const message: ChatMessage = payload.message
      this.addMessageToHistory(payload.roomId, message)
      this.emit('messageReceived', message)
    })

    this.wsManager.subscribe('typing_indicator', (payload: any) => {
      if (payload.isTyping) {
        this.typingUsers.add(payload.username)
      } else {
        this.typingUsers.delete(payload.username)
      }
      this.emit('typingChanged', Array.from(this.typingUsers))
    })

    this.wsManager.subscribe('user_joined', (payload: any) => {
      this.emit('userJoined', payload.user)
    })

    this.wsManager.subscribe('user_left', (payload: any) => {
      this.emit('userLeft', payload.user)
    })

    this.wsManager.subscribe('stateChange', (data: any) => {
      this.emit('connectionStateChanged', data.newState)
    })
  }

  private addMessageToHistory(roomId: string, message: ChatMessage): void {
    if (!this.messageHistory.has(roomId)) {
      this.messageHistory.set(roomId, [])
    }

    const history = this.messageHistory.get(roomId)!
    history.push(message)

    // Keep only last 1000 messages
    if (history.length > 1000) {
      history.splice(0, history.length - 1000)
    }
  }

  private emit(eventType: string, data?: any): void {
    const handlers = this.eventListeners.get(eventType)
    if (handlers) {
      handlers.forEach(handler => handler(data))
    }
  }

  private getCurrentUserId(): string {
    // Get from authentication service
    return 'user123'
  }

  private getCurrentUsername(): string {
    // Get from authentication service
    return 'John Doe'
  }
}

// Chat UI Component
@Component
struct ChatInterface {
  @State private messages: ChatMessage[] = []
  @State private currentMessage: string = ''
  @State private typingUsers: string[] = []
  @State private connectionState: ConnectionState = ConnectionState.Disconnected
  private chatService: ChatService
  private scrollController = new Scroller()

  aboutToAppear() {
    this.chatService = new ChatService('wss://chat.example.com')
    this.setupChatListeners()
    this.chatService.connect()
  }

  build() {
    Column() {
      // Connection status
      this.buildConnectionStatus()

      // Message list
      List({ scroller: this.scrollController }) {
        ForEach(this.messages, (message: ChatMessage) => {
          ListItem() {
            this.buildMessage(message)
          }
        })

        // Typing indicator
        if (this.typingUsers.length > 0) {
          ListItem() {
            this.buildTypingIndicator()
          }
        }
      }
      .layoutWeight(1)
      .padding(16)

      // Message input
      this.buildMessageInput()
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  buildMessage(message: ChatMessage) {
    Row() {
      Column() {
        Text(message.username)
          .fontSize(12)
          .fontColor('#666')

        Text(message.content)
          .fontSize(14)
          .padding(8)
          .backgroundColor('#f0f0f0')
          .borderRadius(8)

        Text(this.formatTime(message.timestamp))
          .fontSize(10)
          .fontColor('#999')
      }
      .alignItems(HorizontalAlign.Start)
    }
    .width('100%')
    .margin({ bottom: 8 })
  }

  @Builder
  buildTypingIndicator() {
    Text(`${this.typingUsers.join(', ')} ${this.typingUsers.length === 1 ? 'is' : 'are'} typing...`)
      .fontSize(12)
      .fontColor('#666')
      .fontStyle(FontStyle.Italic)
  }

  @Builder
  buildMessageInput() {
    Row() {
      TextInput({
        placeholder: 'Type a message...',
        text: this.currentMessage
      })
        .layoutWeight(1)
        .onChange((value: string) => {
          this.currentMessage = value
          this.handleTyping()
        })
        .onSubmit(() => this.sendMessage())

      Button('Send')
        .margin({ left: 8 })
        .onClick(() => this.sendMessage())
    }
    .padding(16)
    .width('100%')
  }

  private setupChatListeners(): void {
    this.chatService.subscribe('messageReceived', (message: ChatMessage) => {
      this.messages.push(message)
      this.scrollToBottom()
    })

    this.chatService.subscribe('typingChanged', (users: string[]) => {
      this.typingUsers = users
    })

    this.chatService.subscribe('connectionStateChanged', (state: ConnectionState) => {
      this.connectionState = state
    })
  }

  private async sendMessage(): Promise<void> {
    if (this.currentMessage.trim()) {
      await this.chatService.sendMessage(this.currentMessage.trim())
      this.currentMessage = ''
      await this.chatService.sendTypingIndicator(false)
    }
  }

  private async handleTyping(): Promise<void> {
    await this.chatService.sendTypingIndicator(true)

    // Stop typing indicator after 3 seconds
    setTimeout(() => {
      this.chatService.sendTypingIndicator(false)
    }, 3000)
  }

  private scrollToBottom(): void {
    this.scrollController.scrollEdge(Edge.Bottom)
  }

  private formatTime(timestamp: number): string {
    return new Date(timestamp).toLocaleTimeString()
  }
}
```

## HTTP Client Enhancements

### Advanced Request Manager

```typescript
interface RequestConfig {
  url: string;
  method: "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
  headers?: Record<string, string>;
  params?: Record<string, any>;
  data?: any;
  timeout?: number;
  retries?: number;
  cache?: boolean;
  priority?: RequestPriority;
}

enum RequestPriority {
  Low = "low",
  Normal = "normal",
  High = "high",
  Critical = "critical",
}

class AdvancedHttpClient {
  private requestQueue: PriorityQueue<QueuedRequest> = new PriorityQueue();
  private cache = new Map<string, CachedResponse>();
  private interceptors: RequestInterceptor[] = [];
  private concurrentRequests = 0;
  private maxConcurrentRequests = 6;

  async request<T>(config: RequestConfig): Promise<HttpResponse<T>> {
    // Apply interceptors
    const processedConfig = await this.applyRequestInterceptors(config);

    // Check cache
    if (config.cache) {
      const cached = this.getCachedResponse<T>(processedConfig);
      if (cached) return cached;
    }

    // Queue or execute request
    if (this.concurrentRequests >= this.maxConcurrentRequests) {
      return this.queueRequest<T>(processedConfig);
    }

    return this.executeRequest<T>(processedConfig);
  }

  addInterceptor(interceptor: RequestInterceptor): void {
    this.interceptors.push(interceptor);
  }

  private async executeRequest<T>(
    config: RequestConfig
  ): Promise<HttpResponse<T>> {
    this.concurrentRequests++;

    try {
      const response = await this.performRequest<T>(config);

      // Cache successful responses
      if (config.cache && response.status >= 200 && response.status < 300) {
        this.cacheResponse(config, response);
      }

      return response;
    } finally {
      this.concurrentRequests--;
      this.processQueue();
    }
  }

  private async performRequest<T>(
    config: RequestConfig
  ): Promise<HttpResponse<T>> {
    const url = this.buildUrl(config.url, config.params);
    const headers = this.buildHeaders(config.headers);

    const requestInit: RequestInit = {
      method: config.method,
      headers,
      body: config.data ? JSON.stringify(config.data) : undefined,
      signal: AbortSignal.timeout(config.timeout || 10000),
    };

    const response = await fetch(url, requestInit);
    const data = await response.json();

    return {
      data,
      status: response.status,
      statusText: response.statusText,
      headers: response.headers,
    };
  }

  private buildUrl(baseUrl: string, params?: Record<string, any>): string {
    if (!params) return baseUrl;

    const url = new URL(baseUrl);
    Object.entries(params).forEach(([key, value]) => {
      url.searchParams.append(key, String(value));
    });

    return url.toString();
  }

  private buildHeaders(
    customHeaders?: Record<string, string>
  ): Record<string, string> {
    return {
      "Content-Type": "application/json",
      Accept: "application/json",
      ...customHeaders,
    };
  }
}
```

## Conclusion

Advanced networking in ArkTS applications enables:

- Real-time communication through WebSockets
- Robust connection management with reconnection logic
- Message queuing and priority handling
- Advanced HTTP client features
- Performance optimization through caching and request pooling
- Error handling and fallback strategies

These patterns provide the foundation for building modern, connected applications that maintain performance and reliability across various network conditions.
