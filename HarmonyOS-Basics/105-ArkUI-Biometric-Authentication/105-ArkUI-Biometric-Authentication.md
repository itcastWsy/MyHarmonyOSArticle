# ArkUI Biometric Authentication

## Introduction

Biometric Authentication in ArkUI provides secure user verification through fingerprint, face recognition, and voice authentication. This guide covers biometric APIs, security protocols, and multi-modal authentication systems.

## Biometric Framework

```typescript
interface BiometricConfig {
  types: BiometricType[];
  fallbackEnabled: boolean;
  securityLevel: SecurityLevel;
  timeout: number;
  maxAttempts: number;
}

type BiometricType = "fingerprint" | "face" | "voice" | "iris" | "palm";
type SecurityLevel = "basic" | "strong" | "high";

class BiometricAuthenticator {
  private availableTypes = new Set<BiometricType>();
  private authHistory: AuthAttempt[] = [];
  private securityManager = new SecurityManager();

  async initialize(): Promise<void> {
    await this.detectAvailableTypes();
    await this.securityManager.initialize();
  }

  async authenticate(config: BiometricConfig): Promise<AuthResult> {
    const authId = `auth_${Date.now()}`;
    const attempt: AuthAttempt = {
      id: authId,
      timestamp: Date.now(),
      types: config.types,
      status: "pending",
    };

    this.authHistory.push(attempt);

    try {
      // Validate security level
      if (!this.validateSecurityLevel(config.securityLevel)) {
        throw new Error("Insufficient security level");
      }

      // Try authentication with each type
      for (const type of config.types) {
        if (!this.availableTypes.has(type)) {
          continue;
        }

        const result = await this.authenticateWithType(type, config);

        if (result.success) {
          attempt.status = "success";
          attempt.authenticatedType = type;

          return {
            success: true,
            authId,
            type,
            confidence: result.confidence,
            biometricData: this.securityManager.encryptBiometricData(
              result.data
            ),
          };
        }
      }

      // If all types failed
      attempt.status = "failed";

      if (config.fallbackEnabled) {
        return await this.fallbackAuthentication(authId);
      }

      return {
        success: false,
        authId,
        error: "Authentication failed",
        canRetry: attempt.attemptCount < config.maxAttempts,
      };
    } catch (error) {
      attempt.status = "error";
      throw error;
    }
  }

  async enrollBiometric(type: BiometricType): Promise<EnrollmentResult> {
    if (!this.availableTypes.has(type)) {
      return {
        success: false,
        error: `${type} authentication not available`,
      };
    }

    const enrollmentId = `enroll_${Date.now()}`;

    try {
      const biometricData = await this.captureBiometric(type);
      const template = await this.createTemplate(biometricData, type);

      await this.securityManager.storeBiometricTemplate(enrollmentId, template);

      return {
        success: true,
        enrollmentId,
        type,
        quality: this.assessQuality(biometricData),
      };
    } catch (error) {
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async deleteBiometric(enrollmentId: string): Promise<boolean> {
    return await this.securityManager.deleteBiometricTemplate(enrollmentId);
  }

  getAvailableTypes(): BiometricType[] {
    return Array.from(this.availableTypes);
  }

  getAuthHistory(): AuthAttempt[] {
    return this.authHistory.slice(-10); // Last 10 attempts
  }

  private async detectAvailableTypes(): Promise<void> {
    // Simulate hardware detection
    const hardwareCapabilities = await this.checkHardwareCapabilities();

    if (hardwareCapabilities.fingerprint) {
      this.availableTypes.add("fingerprint");
    }

    if (hardwareCapabilities.camera) {
      this.availableTypes.add("face");
    }

    if (hardwareCapabilities.microphone) {
      this.availableTypes.add("voice");
    }
  }

  private async authenticateWithType(
    type: BiometricType,
    config: BiometricConfig
  ): Promise<TypeAuthResult> {
    const startTime = Date.now();

    try {
      switch (type) {
        case "fingerprint":
          return await this.authenticateFingerprint(config.timeout);
        case "face":
          return await this.authenticateFace(config.timeout);
        case "voice":
          return await this.authenticateVoice(config.timeout);
        default:
          throw new Error(`Unsupported authentication type: ${type}`);
      }
    } catch (error) {
      return {
        success: false,
        confidence: 0,
        error: error.message,
        processingTime: Date.now() - startTime,
      };
    }
  }

  private async authenticateFingerprint(
    timeout: number
  ): Promise<TypeAuthResult> {
    return new Promise((resolve) => {
      // Simulate fingerprint scanning
      setTimeout(() => {
        const success = Math.random() > 0.2; // 80% success rate
        resolve({
          success,
          confidence: success ? 0.95 + Math.random() * 0.05 : 0,
          data: success ? this.generateFingerprintData() : null,
          processingTime: 1000 + Math.random() * 500,
        });
      }, 1000);
    });
  }

  private async authenticateFace(timeout: number): Promise<TypeAuthResult> {
    return new Promise((resolve) => {
      // Simulate face recognition
      setTimeout(() => {
        const success = Math.random() > 0.15; // 85% success rate
        resolve({
          success,
          confidence: success ? 0.92 + Math.random() * 0.08 : 0,
          data: success ? this.generateFaceData() : null,
          processingTime: 1500 + Math.random() * 800,
        });
      }, 1500);
    });
  }

  private async authenticateVoice(timeout: number): Promise<TypeAuthResult> {
    return new Promise((resolve) => {
      // Simulate voice recognition
      setTimeout(() => {
        const success = Math.random() > 0.25; // 75% success rate
        resolve({
          success,
          confidence: success ? 0.88 + Math.random() * 0.12 : 0,
          data: success ? this.generateVoiceData() : null,
          processingTime: 2000 + Math.random() * 1000,
        });
      }, 2000);
    });
  }

  private async captureBiometric(type: BiometricType): Promise<BiometricData> {
    // Simulate biometric capture
    await new Promise((resolve) => setTimeout(resolve, 1000));

    return {
      type,
      data: new Uint8Array(256), // Mock biometric data
      timestamp: Date.now(),
      quality: 0.8 + Math.random() * 0.2,
    };
  }

  private async createTemplate(
    data: BiometricData,
    type: BiometricType
  ): Promise<BiometricTemplate> {
    // Simulate template creation
    await new Promise((resolve) => setTimeout(resolve, 500));

    return {
      id: `template_${Date.now()}`,
      type,
      template: new Uint8Array(128), // Mock template
      createdAt: Date.now(),
      quality: data.quality,
    };
  }

  private assessQuality(data: BiometricData): number {
    return data.quality;
  }

  private validateSecurityLevel(level: SecurityLevel): boolean {
    // Implement security level validation
    return true;
  }

  private async fallbackAuthentication(authId: string): Promise<AuthResult> {
    // Implement PIN/Password fallback
    return {
      success: false,
      authId,
      error: "Fallback authentication not implemented",
    };
  }

  private async checkHardwareCapabilities(): Promise<HardwareCapabilities> {
    // Simulate hardware capability detection
    return {
      fingerprint: true,
      camera: true,
      microphone: true,
      iris: false,
    };
  }

  private generateFingerprintData(): any {
    return {
      minutiae: Array(20)
        .fill(0)
        .map(() => Math.random()),
    };
  }

  private generateFaceData(): any {
    return {
      landmarks: Array(68)
        .fill(0)
        .map(() => ({ x: Math.random(), y: Math.random() })),
    };
  }

  private generateVoiceData(): any {
    return {
      mfcc: Array(13)
        .fill(0)
        .map(() => Math.random()),
    };
  }
}

class SecurityManager {
  private encryptionKey: CryptoKey | null = null;

  async initialize(): Promise<void> {
    this.encryptionKey = await this.generateEncryptionKey();
  }

  async encryptBiometricData(data: any): Promise<string> {
    if (!this.encryptionKey)
      throw new Error("Security manager not initialized");

    const dataString = JSON.stringify(data);
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(dataString);

    // Simulate encryption
    const encrypted = await crypto.subtle.encrypt(
      { name: "AES-GCM", iv: crypto.getRandomValues(new Uint8Array(12)) },
      this.encryptionKey,
      dataBuffer
    );

    return btoa(String.fromCharCode(...new Uint8Array(encrypted)));
  }

  async storeBiometricTemplate(
    id: string,
    template: BiometricTemplate
  ): Promise<void> {
    // Implement secure template storage
    console.log(`Storing template ${id}`);
  }

  async deleteBiometricTemplate(id: string): Promise<boolean> {
    // Implement template deletion
    console.log(`Deleting template ${id}`);
    return true;
  }

  private async generateEncryptionKey(): Promise<CryptoKey> {
    return await crypto.subtle.generateKey(
      { name: "AES-GCM", length: 256 },
      false,
      ["encrypt", "decrypt"]
    );
  }
}

interface AuthResult {
  success: boolean;
  authId: string;
  type?: BiometricType;
  confidence?: number;
  biometricData?: string;
  error?: string;
  canRetry?: boolean;
}

interface AuthAttempt {
  id: string;
  timestamp: number;
  types: BiometricType[];
  status: "pending" | "success" | "failed" | "error";
  authenticatedType?: BiometricType;
  attemptCount?: number;
}

interface BiometricData {
  type: BiometricType;
  data: Uint8Array;
  timestamp: number;
  quality: number;
}

interface BiometricTemplate {
  id: string;
  type: BiometricType;
  template: Uint8Array;
  createdAt: number;
  quality: number;
}
```

## Biometric Authentication Component

```typescript
@Component
struct BiometricAuthPanel {
  @State private availableTypes: BiometricType[] = []
  @State private selectedTypes: BiometricType[] = []
  @State private authStatus: AuthStatus = 'idle'
  @State private authHistory: AuthAttempt[] = []
  @State private enrolledBiometrics: EnrolledBiometric[] = []

  private authenticator = new BiometricAuthenticator()

  aboutToAppear() {
    this.initializeBiometrics()
  }

  build() {
    Column() {
      this.buildHeader()
      this.buildAuthenticationPanel()
      this.buildEnrollmentPanel()
      this.buildHistoryPanel()
    }
    .width('100%')
    .padding(16)
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Biometric Authentication')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Text(this.authStatus)
        .fontSize(14)
        .fontColor(this.getStatusColor(this.authStatus))
        .backgroundColor(this.getStatusBackgroundColor(this.authStatus))
        .padding({ horizontal: 8, vertical: 4 })
        .borderRadius(4)
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildAuthenticationPanel() {
    Column() {
      Text('Authentication')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Text('Select Authentication Methods:')
        .fontSize(14)
        .margin({ bottom: 8 })

      ForEach(this.availableTypes, (type: BiometricType) => {
        Row() {
          Checkbox()
            .select(this.selectedTypes.includes(type))
            .onChange((isSelected) => this.toggleType(type, isSelected))

          Text(this.formatBiometricType(type))
            .fontSize(14)
            .margin({ left: 8 })
            .flexGrow(1)

          Text(this.getTypeStatus(type))
            .fontSize(12)
            .fontColor('#666666')
        }
        .width('100%')
        .margin({ bottom: 8 })
      })

      Button('Authenticate')
        .onClick(() => this.performAuthentication())
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .width('100%')
        .margin({ top: 12 })
        .enabled(this.selectedTypes.length > 0 && this.authStatus === 'idle')
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 20 })
  }

  @Builder
  private buildEnrollmentPanel() {
    Column() {
      Text('Biometric Enrollment')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      ForEach(this.enrolledBiometrics, (biometric: EnrolledBiometric) => {
        this.buildEnrolledBiometricCard(biometric)
      })

      Row() {
        ForEach(this.availableTypes, (type: BiometricType) => {
          Button(`Enroll ${this.formatBiometricType(type)}`)
            .onClick(() => this.enrollBiometric(type))
            .backgroundColor('#34C759')
            .fontColor('#FFFFFF')
            .fontSize(12)
            .flexGrow(1)
            .margin({ right: 4 })
        })
      }
      .margin({ top: 12 })
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 20 })
  }

  @Builder
  private buildEnrolledBiometricCard(biometric: EnrolledBiometric) {
    Row() {
      Column() {
        Text(this.formatBiometricType(biometric.type))
          .fontSize(14)
          .fontWeight(FontWeight.Bold)

        Text(`Quality: ${(biometric.quality * 100).toFixed(0)}%`)
          .fontSize(12)
          .fontColor('#666666')

        Text(`Enrolled: ${new Date(biometric.enrolledAt).toLocaleDateString()}`)
          .fontSize(10)
          .fontColor('#999999')
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)

      Button('Delete')
        .onClick(() => this.deleteBiometric(biometric.id))
        .backgroundColor('#FF3B30')
        .fontColor('#FFFFFF')
        .fontSize(12)
        .height(32)
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(6)
    .margin({ bottom: 8 })
  }

  @Builder
  private buildHistoryPanel() {
    Column() {
      Text('Authentication History')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      ForEach(this.authHistory.slice(0, 5), (attempt: AuthAttempt) => {
        this.buildHistoryItem(attempt)
      })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildHistoryItem(attempt: AuthAttempt) {
    Row() {
      Column() {
        Text(`Authentication Attempt`)
          .fontSize(14)
          .fontWeight(FontWeight.Bold)

        Text(`Methods: ${attempt.types.map(t => this.formatBiometricType(t)).join(', ')}`)
          .fontSize(12)
          .fontColor('#666666')

        if (attempt.authenticatedType) {
          Text(`Authenticated with: ${this.formatBiometricType(attempt.authenticatedType)}`)
            .fontSize(10)
            .fontColor('#34C759')
        }
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)

      Column() {
        Text(attempt.status)
          .fontSize(12)
          .fontColor(this.getStatusColor(attempt.status))

        Text(new Date(attempt.timestamp).toLocaleTimeString())
          .fontSize(10)
          .fontColor('#999999')
      }
      .alignItems(HorizontalAlign.End)
    }
    .width('100%')
    .padding(10)
    .backgroundColor('#F8F9FA')
    .borderRadius(4)
    .margin({ bottom: 6 })
  }

  private async initializeBiometrics(): Promise<void> {
    await this.authenticator.initialize()
    this.availableTypes = this.authenticator.getAvailableTypes()
    this.authHistory = this.authenticator.getAuthHistory()

    // Load enrolled biometrics
    this.loadEnrolledBiometrics()
  }

  private toggleType(type: BiometricType, isSelected: boolean): void {
    if (isSelected) {
      if (!this.selectedTypes.includes(type)) {
        this.selectedTypes.push(type)
      }
    } else {
      this.selectedTypes = this.selectedTypes.filter(t => t !== type)
    }
  }

  private async performAuthentication(): Promise<void> {
    if (this.selectedTypes.length === 0) return

    this.authStatus = 'authenticating'

    try {
      const config: BiometricConfig = {
        types: this.selectedTypes,
        fallbackEnabled: true,
        securityLevel: 'strong',
        timeout: 30000,
        maxAttempts: 3
      }

      const result = await this.authenticator.authenticate(config)

      if (result.success) {
        this.authStatus = 'success'
        console.log('Authentication successful:', result)
      } else {
        this.authStatus = 'failed'
        console.log('Authentication failed:', result.error)
      }

      // Update history
      this.authHistory = this.authenticator.getAuthHistory()
    } catch (error) {
      this.authStatus = 'error'
      console.error('Authentication error:', error)
    }

    // Reset status after 3 seconds
    setTimeout(() => {
      this.authStatus = 'idle'
    }, 3000)
  }

  private async enrollBiometric(type: BiometricType): Promise<void> {
    try {
      const result = await this.authenticator.enrollBiometric(type)

      if (result.success) {
        console.log('Enrollment successful:', result)
        this.loadEnrolledBiometrics()
      } else {
        console.error('Enrollment failed:', result.error)
      }
    } catch (error) {
      console.error('Enrollment error:', error)
    }
  }

  private async deleteBiometric(id: string): Promise<void> {
    const success = await this.authenticator.deleteBiometric(id)
    if (success) {
      this.loadEnrolledBiometrics()
    }
  }

  private loadEnrolledBiometrics(): void {
    // Load from storage - simulation
    this.enrolledBiometrics = [
      {
        id: 'fingerprint_001',
        type: 'fingerprint',
        quality: 0.95,
        enrolledAt: Date.now() - 86400000
      },
      {
        id: 'face_001',
        type: 'face',
        quality: 0.88,
        enrolledAt: Date.now() - 172800000
      }
    ]
  }

  private formatBiometricType(type: BiometricType): string {
    const names = {
      fingerprint: 'Fingerprint',
      face: 'Face Recognition',
      voice: 'Voice Recognition',
      iris: 'Iris Scan',
      palm: 'Palm Print'
    }
    return names[type] || type
  }

  private getTypeStatus(type: BiometricType): string {
    const isEnrolled = this.enrolledBiometrics.some(b => b.type === type)
    return isEnrolled ? 'Enrolled' : 'Not Enrolled'
  }

  private getStatusColor(status: string): string {
    const colors = {
      idle: '#8E8E93',
      authenticating: '#007AFF',
      success: '#34C759',
      failed: '#FF3B30',
      error: '#FF3B30',
      pending: '#FF9500'
    }
    return colors[status] || '#8E8E93'
  }

  private getStatusBackgroundColor(status: string): string {
    const colors = {
      idle: '#F0F0F0',
      authenticating: '#E3F2FD',
      success: '#E8F5E8',
      failed: '#FFEBEE',
      error: '#FFEBEE',
      pending: '#FFF3E0'
    }
    return colors[status] || '#F0F0F0'
  }
}

interface EnrolledBiometric {
  id: string
  type: BiometricType
  quality: number
  enrolledAt: number
}

type AuthStatus = 'idle' | 'authenticating' | 'success' | 'failed' | 'error'
```

## Conclusion

Biometric Authentication in ArkUI provides:

- Multi-modal biometric authentication support
- Secure biometric data encryption and storage
- Fallback authentication mechanisms
- Real-time authentication status monitoring
- Comprehensive enrollment and management system

These capabilities enable secure, convenient user authentication while maintaining privacy and security standards for sensitive applications.
