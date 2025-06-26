# ArkUI Security Best Practices

## Introduction

Security is paramount in modern application development. This guide covers comprehensive security practices for ArkUI applications, including data protection, secure communication, authentication patterns, and vulnerability prevention strategies.

## Data Security and Protection

### Secure Data Storage

```typescript
enum SecurityLevel {
  Low = "low",
  Medium = "medium",
  High = "high",
  Critical = "critical",
}

interface SecureStorageOptions {
  encryption: boolean;
  securityLevel: SecurityLevel;
  expirationTime?: number;
  accessControl?: AccessControlOptions;
}

interface AccessControlOptions {
  requireAuthentication: boolean;
  allowedOperations: ("read" | "write" | "delete")[];
  deviceBindingRequired: boolean;
}

class SecureStorageManager {
  private encryptionKey: CryptoKey | null = null;
  private iv: Uint8Array = new Uint8Array(16);

  async initialize(): Promise<void> {
    this.encryptionKey = await this.generateEncryptionKey();
    crypto.getRandomValues(this.iv);
  }

  async storeSecureData(
    key: string,
    data: any,
    options: SecureStorageOptions
  ): Promise<boolean> {
    try {
      // Validate access permissions
      if (!(await this.validateAccess(options.accessControl, "write"))) {
        throw new Error("Access denied for write operation");
      }

      let serializedData = JSON.stringify(data);

      // Apply encryption if required
      if (options.encryption && this.encryptionKey) {
        serializedData = await this.encryptData(serializedData);
      }

      // Create storage entry with metadata
      const storageEntry: StorageEntry = {
        data: serializedData,
        securityLevel: options.securityLevel,
        encrypted: options.encryption,
        timestamp: Date.now(),
        expirationTime: options.expirationTime,
        checksum: await this.generateChecksum(serializedData),
      };

      // Store based on security level
      return await this.storeBySecurityLevel(
        key,
        storageEntry,
        options.securityLevel
      );
    } catch (error) {
      console.error("Failed to store secure data:", error);
      return false;
    }
  }

  async retrieveSecureData(
    key: string,
    accessControl?: AccessControlOptions
  ): Promise<any | null> {
    try {
      // Validate access permissions
      if (!(await this.validateAccess(accessControl, "read"))) {
        throw new Error("Access denied for read operation");
      }

      const storageEntry = await this.retrieveByKey(key);
      if (!storageEntry) return null;

      // Check expiration
      if (this.isExpired(storageEntry)) {
        await this.deleteSecureData(key);
        return null;
      }

      // Verify data integrity
      if (
        !(await this.verifyChecksum(storageEntry.data, storageEntry.checksum))
      ) {
        throw new Error("Data integrity check failed");
      }

      let data = storageEntry.data;

      // Decrypt if necessary
      if (storageEntry.encrypted && this.encryptionKey) {
        data = await this.decryptData(data);
      }

      return JSON.parse(data);
    } catch (error) {
      console.error("Failed to retrieve secure data:", error);
      return null;
    }
  }

  private async encryptData(data: string): Promise<string> {
    if (!this.encryptionKey) throw new Error("Encryption key not available");

    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(data);

    const encrypted = await crypto.subtle.encrypt(
      {
        name: "AES-GCM",
        iv: this.iv,
      },
      this.encryptionKey,
      dataBuffer
    );

    const combined = new Uint8Array(this.iv.length + encrypted.byteLength);
    combined.set(this.iv);
    combined.set(new Uint8Array(encrypted), this.iv.length);

    return btoa(String.fromCharCode(...combined));
  }

  private async decryptData(encryptedData: string): Promise<string> {
    if (!this.encryptionKey) throw new Error("Encryption key not available");

    const combined = new Uint8Array(
      atob(encryptedData)
        .split("")
        .map((char) => char.charCodeAt(0))
    );

    const iv = combined.slice(0, 16);
    const encrypted = combined.slice(16);

    const decrypted = await crypto.subtle.decrypt(
      {
        name: "AES-GCM",
        iv: iv,
      },
      this.encryptionKey,
      encrypted
    );

    const decoder = new TextDecoder();
    return decoder.decode(decrypted);
  }

  private async generateEncryptionKey(): Promise<CryptoKey> {
    return crypto.subtle.generateKey(
      {
        name: "AES-GCM",
        length: 256,
      },
      false,
      ["encrypt", "decrypt"]
    );
  }

  private async generateChecksum(data: string): Promise<string> {
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(data);
    const hashBuffer = await crypto.subtle.digest("SHA-256", dataBuffer);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map((b) => b.toString(16).padStart(2, "0")).join("");
  }
}

interface StorageEntry {
  data: string;
  securityLevel: SecurityLevel;
  encrypted: boolean;
  timestamp: number;
  expirationTime?: number;
  checksum: string;
}
```

### Input Validation and Sanitization

```typescript
class InputValidator {
  private static readonly XSS_PATTERNS = [
    /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi,
    /javascript:/gi,
    /on\w+\s*=/gi,
    /<iframe\b[^<]*(?:(?!<\/iframe>)<[^<]*)*<\/iframe>/gi
  ]

  private static readonly SQL_INJECTION_PATTERNS = [
    /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION)\b)/gi,
    /(--|#|\/\*|\*\/)/g,
    /(\b(OR|AND)\b\s+\d+\s*=\s*\d+)/gi
  ]

  static validateAndSanitize(input: string, type: InputType): ValidationResult {
    const result: ValidationResult = {
      isValid: true,
      sanitizedValue: input,
      errors: []
    }

    // Basic null/undefined check
    if (!input || typeof input !== 'string') {
      result.isValid = false
      result.errors.push('Input is required and must be a string')
      return result
    }

    // Length validation
    if (!this.validateLength(input, type)) {
      result.isValid = false
      result.errors.push(`Input length exceeds maximum allowed for ${type}`)
    }

    // XSS protection
    const xssResult = this.checkForXSS(input)
    if (!xssResult.safe) {
      result.isValid = false
      result.errors.push('Potential XSS attack detected')
      result.sanitizedValue = xssResult.sanitized
    }

    // SQL injection protection
    if (!this.checkForSQLInjection(input)) {
      result.isValid = false
      result.errors.push('Potential SQL injection detected')
    }

    // Type-specific validation
    switch (type) {
      case 'email':
        if (!this.validateEmail(input)) {
          result.isValid = false
          result.errors.push('Invalid email format')
        }
        break
      case 'url':
        if (!this.validateURL(input)) {
          result.isValid = false
          result.errors.push('Invalid URL format')
        }
        break
      case 'phone':
        if (!this.validatePhone(input)) {
          result.isValid = false
          result.errors.push('Invalid phone number format')
        }
        break
    }

    return result
  }

  private static checkForXSS(input: string): { safe: boolean; sanitized: string } {
    let sanitized = input
    let safe = true

    for (const pattern of this.XSS_PATTERNS) {
      if (pattern.test(input)) {
        safe = false
        sanitized = sanitized.replace(pattern, '')
      }
    }

    // HTML encode remaining tags
    sanitized = sanitized
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;')
      .replace(/\//g, '&#x2F;')

    return { safe, sanitized }
  }

  private static checkForSQLInjection(input: string): boolean {
    for (const pattern of this.SQL_INJECTION_PATTERNS) {
      if (pattern.test(input)) {
        return false
      }
    }
    return true
  }

  private static validateEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    return emailRegex.test(email)
  }

  private static validateURL(url: string): boolean {
    try {
      new URL(url)
      return url.startsWith('https://') || url.startsWith('http://')
    } catch {
      return false
    }
  }

  private static validatePhone(phone: string): boolean {
    const phoneRegex = /^\+?[\d\s\-\(\)]+$/
    return phoneRegex.test(phone) && phone.replace(/\D/g, '').length >= 10
  }
}

type InputType = 'text' | 'email' | 'url' | 'phone' | 'password'

interface ValidationResult {
  isValid: boolean
  sanitizedValue: string
  errors: string[]
}

// Secure input component
@Component
struct SecureTextInput {
  @Prop type: InputType = 'text'
  @Prop placeholder: string = ''
  @Prop onValidChange?: (value: string, isValid: boolean) => void
  @State private value: string = ''
  @State private errors: string[] = []
  @State private isValid: boolean = true

  build() {
    Column({ space: 4 }) {
      TextInput({
        placeholder: this.placeholder,
        text: this.value
      })
        .type(this.getInputType())
        .onChange((value: string) => {
          this.validateInput(value)
        })
        .borderColor(this.isValid ? Color.Gray : Color.Red)

      if (!this.isValid && this.errors.length > 0) {
        ForEach(this.errors, (error: string) => {
          Text(error)
            .fontSize(12)
            .fontColor(Color.Red)
        })
      }
    }
    .alignItems(HorizontalAlign.Start)
    .width('100%')
  }

  private validateInput(value: string): void {
    const result = InputValidator.validateAndSanitize(value, this.type)

    this.value = result.sanitizedValue
    this.isValid = result.isValid
    this.errors = result.errors

    if (this.onValidChange) {
      this.onValidChange(this.value, this.isValid)
    }
  }

  private getInputType(): InputType {
    switch (this.type) {
      case 'email': return InputType.Email
      case 'password': return InputType.Password
      case 'phone': return InputType.PhoneNumber
      default: return InputType.Normal
    }
  }
}
```

## Network Security

### Secure HTTP Communication

```typescript
class SecureHttpClient {
  private static readonly TIMEOUT = 10000;
  private static readonly MAX_RETRIES = 3;

  private baseURL: string;
  private defaultHeaders: Record<string, string>;
  private certificatePinning: CertificatePinning;

  constructor(baseURL: string, options: HttpClientOptions = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      "Content-Type": "application/json",
      "X-Requested-With": "XMLHttpRequest",
      ...options.headers,
    };
    this.certificatePinning =
      options.certificatePinning || new CertificatePinning();
  }

  async request<T>(config: SecureRequestConfig): Promise<SecureResponse<T>> {
    // Validate URL
    if (!this.isSecureURL(config.url)) {
      throw new SecurityError("Only HTTPS URLs are allowed", "INSECURE_URL");
    }

    // Prepare headers
    const headers = {
      ...this.defaultHeaders,
      ...config.headers,
      "X-Request-ID": this.generateRequestId(),
      "X-Timestamp": Date.now().toString(),
    };

    // Add CSRF token if available
    const csrfToken = await this.getCSRFToken();
    if (csrfToken) {
      headers["X-CSRF-Token"] = csrfToken;
    }

    // Sanitize request body
    const sanitizedBody = config.body
      ? this.sanitizeRequestBody(config.body)
      : undefined;

    const requestConfig: RequestInit = {
      method: config.method || "GET",
      headers,
      body: sanitizedBody ? JSON.stringify(sanitizedBody) : undefined,
      signal: AbortSignal.timeout(config.timeout || SecureHttpClient.TIMEOUT),
    };

    // Perform request with retries
    return this.executeWithRetry(config.url, requestConfig, config.retries);
  }

  private async executeWithRetry<T>(
    url: string,
    config: RequestInit,
    maxRetries: number = SecureHttpClient.MAX_RETRIES
  ): Promise<SecureResponse<T>> {
    let lastError: Error;

    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const response = await fetch(url, config);

        // Verify certificate pinning
        await this.certificatePinning.verify(url, response);

        // Check rate limiting
        this.checkRateLimit(response);

        if (!response.ok) {
          throw new HttpError(
            `HTTP ${response.status}: ${response.statusText}`,
            response.status
          );
        }

        const data = await this.parseResponse<T>(response);

        return {
          data,
          status: response.status,
          headers: response.headers,
          isSuccess: true,
        };
      } catch (error) {
        lastError = error as Error;

        if (attempt < maxRetries && this.shouldRetry(error as Error)) {
          await this.delay(Math.pow(2, attempt) * 1000); // Exponential backoff
          continue;
        }

        throw error;
      }
    }

    throw lastError!;
  }

  private isSecureURL(url: string): boolean {
    try {
      const urlObj = new URL(url, this.baseURL);
      return urlObj.protocol === "https:";
    } catch {
      return false;
    }
  }

  private sanitizeRequestBody(body: any): any {
    if (typeof body === "string") {
      const validation = InputValidator.validateAndSanitize(body, "text");
      return validation.sanitizedValue;
    }

    if (typeof body === "object" && body !== null) {
      const sanitized: any = {};
      for (const [key, value] of Object.entries(body)) {
        if (typeof value === "string") {
          const validation = InputValidator.validateAndSanitize(value, "text");
          sanitized[key] = validation.sanitizedValue;
        } else {
          sanitized[key] = value;
        }
      }
      return sanitized;
    }

    return body;
  }

  private async parseResponse<T>(response: Response): Promise<T> {
    const contentType = response.headers.get("content-type");

    if (contentType?.includes("application/json")) {
      const text = await response.text();

      // Validate JSON response for potential XSS
      if (this.containsXSS(text)) {
        throw new SecurityError(
          "Response contains potential XSS content",
          "XSS_DETECTED"
        );
      }

      return JSON.parse(text);
    }

    return response.text() as T;
  }

  private containsXSS(text: string): boolean {
    const xssPatterns = [/<script/i, /javascript:/i, /on\w+\s*=/i, /<iframe/i];

    return xssPatterns.some((pattern) => pattern.test(text));
  }
}

interface SecureRequestConfig {
  url: string;
  method?: string;
  headers?: Record<string, string>;
  body?: any;
  timeout?: number;
  retries?: number;
}

interface SecureResponse<T> {
  data: T;
  status: number;
  headers: Headers;
  isSuccess: boolean;
}

class SecurityError extends Error {
  constructor(message: string, public code: string) {
    super(message);
    this.name = "SecurityError";
  }
}

// Certificate pinning implementation
class CertificatePinning {
  private pinnedCertificates = new Map<string, string[]>();

  addPin(hostname: string, certificates: string[]): void {
    this.pinnedCertificates.set(hostname, certificates);
  }

  async verify(url: string, response: Response): Promise<void> {
    const hostname = new URL(url).hostname;
    const pins = this.pinnedCertificates.get(hostname);

    if (!pins || pins.length === 0) {
      return; // No pinning configured for this host
    }

    // In a real implementation, you would verify the certificate chain
    // This is a simplified example
    const certificateHeader = response.headers.get("x-certificate-fingerprint");

    if (!certificateHeader || !pins.includes(certificateHeader)) {
      throw new SecurityError(
        "Certificate pinning validation failed",
        "CERT_PIN_FAILURE"
      );
    }
  }
}
```

## Authentication and Authorization

### Secure Authentication Flow

```typescript
interface AuthCredentials {
  username: string;
  password: string;
  twoFactorCode?: string;
}

interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
  tokenType: string;
}

interface UserSession {
  userId: string;
  roles: string[];
  permissions: string[];
  sessionId: string;
  expiresAt: number;
}

class SecureAuthManager {
  private session: UserSession | null = null;
  private tokens: AuthTokens | null = null;
  private refreshTimer: number | null = null;
  private secureStorage = new SecureStorageManager();

  async login(credentials: AuthCredentials): Promise<AuthResult> {
    try {
      // Validate credentials
      const validation = this.validateCredentials(credentials);
      if (!validation.isValid) {
        throw new AuthError(
          "Invalid credentials format",
          "INVALID_CREDENTIALS"
        );
      }

      // Hash password for transmission
      const hashedPassword = await this.hashPassword(credentials.password);

      // Perform authentication
      const response = await this.performAuthentication({
        ...credentials,
        password: hashedPassword,
      });

      // Store tokens securely
      await this.storeTokens(response.tokens);

      // Setup session
      this.session = response.session;
      this.setupTokenRefresh();

      return {
        success: true,
        session: this.session,
        message: "Authentication successful",
      };
    } catch (error) {
      return {
        success: false,
        session: null,
        message: error.message,
        error: error.code,
      };
    }
  }

  async logout(): Promise<void> {
    try {
      // Revoke tokens on server
      if (this.tokens) {
        await this.revokeTokens(this.tokens.accessToken);
      }
    } catch (error) {
      console.error("Failed to revoke tokens:", error);
    } finally {
      // Clear local session data
      this.clearSession();
    }
  }

  isAuthenticated(): boolean {
    return this.session !== null && this.session.expiresAt > Date.now();
  }

  hasPermission(permission: string): boolean {
    return this.session?.permissions.includes(permission) || false;
  }

  hasRole(role: string): boolean {
    return this.session?.roles.includes(role) || false;
  }

  private async storeTokens(tokens: AuthTokens): Promise<void> {
    this.tokens = tokens;

    await this.secureStorage.storeSecureData("auth_tokens", tokens, {
      encryption: true,
      securityLevel: SecurityLevel.Critical,
      expirationTime: tokens.expiresIn * 1000,
      accessControl: {
        requireAuthentication: true,
        allowedOperations: ["read", "write", "delete"],
        deviceBindingRequired: true,
      },
    });
  }

  private async hashPassword(password: string): Promise<string> {
    const encoder = new TextEncoder();
    const data = encoder.encode(password);
    const hashBuffer = await crypto.subtle.digest("SHA-256", data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map((b) => b.toString(16).padStart(2, "0")).join("");
  }

  private validateCredentials(credentials: AuthCredentials): {
    isValid: boolean;
    errors: string[];
  } {
    const errors: string[] = [];

    // Username validation
    if (!credentials.username || credentials.username.length < 3) {
      errors.push("Username must be at least 3 characters long");
    }

    // Password strength validation
    if (!this.validatePasswordStrength(credentials.password)) {
      errors.push("Password does not meet security requirements");
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  private validatePasswordStrength(password: string): boolean {
    const minLength = 8;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

    return (
      password.length >= minLength &&
      hasUpperCase &&
      hasLowerCase &&
      hasNumbers &&
      hasSpecialChar
    );
  }

  private setupTokenRefresh(): void {
    if (this.refreshTimer) {
      clearTimeout(this.refreshTimer);
    }

    if (this.tokens) {
      const refreshTime = (this.tokens.expiresIn - 300) * 1000; // Refresh 5 minutes before expiry
      this.refreshTimer = setTimeout(() => {
        this.refreshAccessToken();
      }, refreshTime);
    }
  }

  private async refreshAccessToken(): Promise<void> {
    try {
      if (!this.tokens?.refreshToken) {
        throw new Error("No refresh token available");
      }

      const response = await this.performTokenRefresh(this.tokens.refreshToken);
      await this.storeTokens(response.tokens);
      this.setupTokenRefresh();
    } catch (error) {
      console.error("Token refresh failed:", error);
      this.clearSession();
    }
  }

  private clearSession(): void {
    this.session = null;
    this.tokens = null;

    if (this.refreshTimer) {
      clearTimeout(this.refreshTimer);
      this.refreshTimer = null;
    }

    // Clear stored tokens
    this.secureStorage.deleteSecureData("auth_tokens");
  }
}

interface AuthResult {
  success: boolean;
  session: UserSession | null;
  message: string;
  error?: string;
}

class AuthError extends Error {
  constructor(message: string, public code: string) {
    super(message);
    this.name = "AuthError";
  }
}
```

## Conclusion

Security best practices in ArkUI applications encompass:

- Comprehensive data encryption and secure storage
- Robust input validation and sanitization
- Secure network communication with certificate pinning
- Strong authentication and authorization mechanisms
- Regular security audits and vulnerability assessments
- Proper error handling without information disclosure
- Secure coding practices and dependency management

Implementing these security measures ensures applications protect user data and maintain trust while providing excellent user experiences across all HarmonyOS devices.
