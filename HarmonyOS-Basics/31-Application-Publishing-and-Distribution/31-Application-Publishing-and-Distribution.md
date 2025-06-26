# 31-Application Publishing and Distribution

## Introduction

Publishing and distributing HarmonyOS applications involves multiple steps from preparation to app store submission. This article covers the complete process including app preparation, signing, testing, store submission, and post-launch considerations.

## Application Preparation

### App Configuration

```json5
// module.json5
{
  module: {
    name: "entry",
    type: "entry",
    description: "$string:module_desc",
    mainElement: "EntryAbility",
    deviceTypes: ["phone", "tablet"],
    deliveryWithInstall: true,
    installationFree: false,
    pages: "$profile:main_pages",
    abilities: [
      {
        name: "EntryAbility",
        srcEntry: "./ets/entryability/EntryAbility.ts",
        description: "$string:EntryAbility_desc",
        icon: "$media:icon",
        label: "$string:EntryAbility_label",
        startWindowIcon: "$media:icon",
        startWindowBackground: "$color:start_window_background",
        exported: true,
        skills: [
          {
            entities: ["entity.system.home"],
            actions: ["action.system.home"],
          },
        ],
      },
    ],
    requestPermissions: [
      {
        name: "ohos.permission.INTERNET",
        reason: "$string:permission_internet_reason",
        usedScene: {
          abilities: ["EntryAbility"],
          when: "inuse",
        },
      },
      {
        name: "ohos.permission.CAMERA",
        reason: "$string:permission_camera_reason",
        usedScene: {
          abilities: ["EntryAbility"],
          when: "inuse",
        },
      },
    ],
  },
}
```

### App Information Configuration

```json5
// app.json5
{
  app: {
    bundleName: "com.example.myapp",
    vendor: "Example Company",
    versionCode: 1000000,
    versionName: "1.0.0",
    icon: "$media:app_icon",
    label: "$string:app_name",
    description: "$string:app_description",
    minCompatibleVersionCode: 1000000,
    targetAPIVersion: 11,
    apiReleaseType: "Release",
  },
}
```

## Build Configuration

### Release Build Setup

```typescript
// build-profile.json5
{
  "apiType": "stageMode",
  "buildOption": {
    "strictMode": {
      "caseSensitiveCheck": true,
      "useNormalizedOHMUrl": true
    }
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [
        {
          "name": "default",
          "applyToProducts": [
            "default"
          ]
        }
      ]
    }
  ],
  "products": [
    {
      "name": "default",
      "signingConfig": "default",
      "compileSdkVersion": 11,
      "compatibleSdkVersion": 11,
      "runtimeOS": "HarmonyOS"
    }
  ]
}
```

### Environment-Specific Configuration

```typescript
// Build configuration service
export class BuildConfigService {
  private static readonly BUILD_CONFIGS = {
    development: {
      apiBaseUrl: "https://dev-api.example.com",
      enableLogging: true,
      enableDebugMode: true,
      analyticsEnabled: false,
    },
    staging: {
      apiBaseUrl: "https://staging-api.example.com",
      enableLogging: true,
      enableDebugMode: false,
      analyticsEnabled: true,
    },
    production: {
      apiBaseUrl: "https://api.example.com",
      enableLogging: false,
      enableDebugMode: false,
      analyticsEnabled: true,
    },
  };

  static getConfig(): BuildConfig {
    const environment = this.getCurrentEnvironment();
    return this.BUILD_CONFIGS[environment];
  }

  private static getCurrentEnvironment(): keyof typeof BuildConfigService.BUILD_CONFIGS {
    // In actual implementation, this would be determined by build flags
    if (__BUILD_TYPE__ === "release") {
      return "production";
    } else if (__BUILD_TYPE__ === "debug") {
      return "development";
    } else {
      return "staging";
    }
  }
}

interface BuildConfig {
  apiBaseUrl: string;
  enableLogging: boolean;
  enableDebugMode: boolean;
  analyticsEnabled: boolean;
}
```

## Code Signing

### Signing Configuration

```json5
// signing-configs.json5
{
  signingConfigs: [
    {
      name: "default",
      type: "HarmonyOS",
      material: {
        certpath: "./certificates/app-release.p12",
        storePassword: "password",
        keyAlias: "key0",
        keyPassword: "password",
        profile: "./certificates/app-release.p7b",
        signAlg: "SHA256withRSA",
        verify: true,
        compatibleVersion: 9,
      },
    },
  ],
}
```

### Signing Process Automation

```typescript
export class SigningManager {
  private static readonly KEYSTORE_PATH = "./certificates/app-release.p12";
  private static readonly PROFILE_PATH = "./certificates/app-release.p7b";

  static async signApplication(appPath: string): Promise<string> {
    try {
      console.log("Starting application signing process...");

      // Verify certificate validity
      await this.verifyCertificate();

      // Sign the application
      const signedAppPath = await this.performSigning(appPath);

      // Verify signature
      await this.verifySignature(signedAppPath);

      console.log("Application signed successfully");
      return signedAppPath;
    } catch (error) {
      console.error("Application signing failed:", error);
      throw error;
    }
  }

  private static async verifyCertificate(): Promise<void> {
    // Implement certificate verification
    console.log("Verifying certificate...");

    if (!fs.existsSync(this.KEYSTORE_PATH)) {
      throw new Error("Keystore file not found");
    }

    if (!fs.existsSync(this.PROFILE_PATH)) {
      throw new Error("Profile file not found");
    }
  }

  private static async performSigning(appPath: string): Promise<string> {
    // Implement actual signing process
    console.log("Performing application signing...");

    // This would use HarmonyOS signing tools
    return `${appPath}.signed`;
  }

  private static async verifySignature(signedAppPath: string): Promise<void> {
    // Implement signature verification
    console.log("Verifying application signature...");
  }
}
```

## Quality Assurance

### Automated Testing for Release

```typescript
export class ReleaseTestRunner {
  static async runReleaseTests(): Promise<TestResults> {
    console.log("Starting release test suite...");

    const results: TestResults = {
      unitTests: await this.runUnitTests(),
      integrationTests: await this.runIntegrationTests(),
      performanceTests: await this.runPerformanceTests(),
      securityTests: await this.runSecurityTests(),
      uiTests: await this.runUITests(),
    };

    const allPassed = Object.values(results).every((result) => result.passed);

    if (!allPassed) {
      throw new Error(
        "Release tests failed. Application is not ready for distribution."
      );
    }

    console.log("All release tests passed successfully");
    return results;
  }

  private static async runUnitTests(): Promise<TestResult> {
    console.log("Running unit tests...");

    try {
      // Run unit test suite
      const testCount = 150;
      const passedCount = 150;

      return {
        name: "Unit Tests",
        passed: passedCount === testCount,
        totalTests: testCount,
        passedTests: passedCount,
        duration: 5000,
      };
    } catch (error) {
      return {
        name: "Unit Tests",
        passed: false,
        totalTests: 0,
        passedTests: 0,
        duration: 0,
        error: error.message,
      };
    }
  }

  private static async runIntegrationTests(): Promise<TestResult> {
    console.log("Running integration tests...");

    try {
      // Run integration test suite
      const testCount = 25;
      const passedCount = 25;

      return {
        name: "Integration Tests",
        passed: passedCount === testCount,
        totalTests: testCount,
        passedTests: passedCount,
        duration: 15000,
      };
    } catch (error) {
      return {
        name: "Integration Tests",
        passed: false,
        totalTests: 0,
        passedTests: 0,
        duration: 0,
        error: error.message,
      };
    }
  }

  private static async runPerformanceTests(): Promise<TestResult> {
    console.log("Running performance tests...");

    try {
      // Run performance benchmarks
      return {
        name: "Performance Tests",
        passed: true,
        totalTests: 10,
        passedTests: 10,
        duration: 30000,
      };
    } catch (error) {
      return {
        name: "Performance Tests",
        passed: false,
        totalTests: 0,
        passedTests: 0,
        duration: 0,
        error: error.message,
      };
    }
  }

  private static async runSecurityTests(): Promise<TestResult> {
    console.log("Running security tests...");

    try {
      // Run security scans
      return {
        name: "Security Tests",
        passed: true,
        totalTests: 15,
        passedTests: 15,
        duration: 10000,
      };
    } catch (error) {
      return {
        name: "Security Tests",
        passed: false,
        totalTests: 0,
        passedTests: 0,
        duration: 0,
        error: error.message,
      };
    }
  }

  private static async runUITests(): Promise<TestResult> {
    console.log("Running UI tests...");

    try {
      // Run UI automation tests
      return {
        name: "UI Tests",
        passed: true,
        totalTests: 20,
        passedTests: 20,
        duration: 25000,
      };
    } catch (error) {
      return {
        name: "UI Tests",
        passed: false,
        totalTests: 0,
        passedTests: 0,
        duration: 0,
        error: error.message,
      };
    }
  }
}

interface TestResult {
  name: string;
  passed: boolean;
  totalTests: number;
  passedTests: number;
  duration: number;
  error?: string;
}

interface TestResults {
  unitTests: TestResult;
  integrationTests: TestResult;
  performanceTests: TestResult;
  securityTests: TestResult;
  uiTests: TestResult;
}
```

## Store Submission

### AppGallery Submission Process

```typescript
export class AppGallerySubmissionManager {
  private static readonly SUBMISSION_ENDPOINTS = {
    upload: "https://connect-api.cloud.huawei.com/api/publish/v2/upload-url",
    submit: "https://connect-api.cloud.huawei.com/api/publish/v2/app-submit",
    status: "https://connect-api.cloud.huawei.com/api/publish/v2/app-status",
  };

  static async submitToAppGallery(
    submissionData: AppSubmissionData
  ): Promise<string> {
    try {
      console.log("Starting AppGallery submission process...");

      // Step 1: Upload application package
      const uploadResult = await this.uploadApplicationPackage(
        submissionData.packagePath
      );

      // Step 2: Submit application metadata
      const submissionId = await this.submitApplicationMetadata({
        ...submissionData,
        packageId: uploadResult.packageId,
      });

      // Step 3: Monitor submission status
      await this.monitorSubmissionStatus(submissionId);

      console.log("Application submitted successfully to AppGallery");
      return submissionId;
    } catch (error) {
      console.error("AppGallery submission failed:", error);
      throw error;
    }
  }

  private static async uploadApplicationPackage(
    packagePath: string
  ): Promise<UploadResult> {
    console.log("Uploading application package...");

    // Implement package upload
    return {
      packageId: "pkg_" + Date.now(),
      uploadUrl: "https://example.com/upload/123",
      status: "uploaded",
    };
  }

  private static async submitApplicationMetadata(
    data: AppSubmissionDataWithPackage
  ): Promise<string> {
    console.log("Submitting application metadata...");

    const submissionPayload = {
      appName: data.appName,
      appDescription: data.appDescription,
      version: data.version,
      packageId: data.packageId,
      category: data.category,
      tags: data.tags,
      screenshots: data.screenshots,
      privacyPolicy: data.privacyPolicy,
      supportContact: data.supportContact,
    };

    // Submit to AppGallery API
    return "sub_" + Date.now();
  }

  private static async monitorSubmissionStatus(
    submissionId: string
  ): Promise<void> {
    console.log("Monitoring submission status...");

    let attempts = 0;
    const maxAttempts = 30;

    while (attempts < maxAttempts) {
      const status = await this.getSubmissionStatus(submissionId);

      if (status === "approved") {
        console.log("Submission approved");
        return;
      } else if (status === "rejected") {
        throw new Error("Submission was rejected");
      }

      await new Promise((resolve) => setTimeout(resolve, 10000)); // Wait 10 seconds
      attempts++;
    }

    throw new Error("Submission status check timed out");
  }

  private static async getSubmissionStatus(
    submissionId: string
  ): Promise<string> {
    // Get submission status from API
    return "pending";
  }
}

interface AppSubmissionData {
  appName: string;
  appDescription: string;
  version: string;
  packagePath: string;
  category: string;
  tags: string[];
  screenshots: string[];
  privacyPolicy: string;
  supportContact: string;
}

interface AppSubmissionDataWithPackage extends AppSubmissionData {
  packageId: string;
}

interface UploadResult {
  packageId: string;
  uploadUrl: string;
  status: string;
}
```

## Release Pipeline

### Automated Release Process

```typescript
export class ReleasePipelineManager {
  static async executeReleasePipeline(version: string): Promise<void> {
    try {
      console.log(`Starting release pipeline for version ${version}...`);

      // Step 1: Code quality checks
      await this.runCodeQualityChecks();

      // Step 2: Run test suites
      await ReleaseTestRunner.runReleaseTests();

      // Step 3: Build release package
      const packagePath = await this.buildReleasePackage(version);

      // Step 4: Sign application
      const signedPackagePath = await SigningManager.signApplication(
        packagePath
      );

      // Step 5: Create release notes
      const releaseNotes = await this.generateReleaseNotes(version);

      // Step 6: Submit to app stores
      await this.submitToAppStores(signedPackagePath, version, releaseNotes);

      // Step 7: Update version tracking
      await this.updateVersionTracking(version);

      console.log("Release pipeline completed successfully");
    } catch (error) {
      console.error("Release pipeline failed:", error);
      await this.handlePipelineFailure(error, version);
      throw error;
    }
  }

  private static async runCodeQualityChecks(): Promise<void> {
    console.log("Running code quality checks...");

    // Run linting
    console.log("Running ESLint...");

    // Run security scans
    console.log("Running security scans...");

    // Check code coverage
    console.log("Checking code coverage...");
  }

  private static async buildReleasePackage(version: string): Promise<string> {
    console.log("Building release package...");

    // Build the application for release
    return `./dist/app-${version}.hap`;
  }

  private static async generateReleaseNotes(version: string): Promise<string> {
    console.log("Generating release notes...");

    // Generate release notes from git commits, JIRA tickets, etc.
    return `Release ${version}\n\n- Bug fixes and improvements\n- Performance optimizations`;
  }

  private static async submitToAppStores(
    packagePath: string,
    version: string,
    releaseNotes: string
  ): Promise<void> {
    console.log("Submitting to app stores...");

    const submissionData: AppSubmissionData = {
      appName: "My Application",
      appDescription: "A great HarmonyOS application",
      version: version,
      packagePath: packagePath,
      category: "Productivity",
      tags: ["productivity", "utility"],
      screenshots: ["./assets/screenshots/1.png", "./assets/screenshots/2.png"],
      privacyPolicy: "https://example.com/privacy",
      supportContact: "support@example.com",
    };

    // Submit to AppGallery
    await AppGallerySubmissionManager.submitToAppGallery(submissionData);
  }

  private static async updateVersionTracking(version: string): Promise<void> {
    console.log("Updating version tracking...");

    // Update internal version tracking systems
    // Tag git repository
    // Update changelog
  }

  private static async handlePipelineFailure(
    error: Error,
    version: string
  ): Promise<void> {
    console.error(`Release pipeline failed for version ${version}:`, error);

    // Send notifications
    // Create failure reports
    // Rollback if necessary
  }
}
```

## Post-Launch Monitoring

### Analytics and Crash Reporting

```typescript
export class PostLaunchMonitor {
  private static readonly MONITORING_ENDPOINTS = {
    analytics: "https://analytics.example.com/api/events",
    crashes: "https://crashes.example.com/api/reports",
    performance: "https://performance.example.com/api/metrics",
  };

  static async initializeMonitoring(): Promise<void> {
    try {
      await this.setupAnalytics();
      await this.setupCrashReporting();
      await this.setupPerformanceMonitoring();

      console.log("Post-launch monitoring initialized");
    } catch (error) {
      console.error("Failed to initialize monitoring:", error);
    }
  }

  private static async setupAnalytics(): Promise<void> {
    console.log("Setting up analytics...");

    // Initialize analytics SDK
    // Configure event tracking
  }

  private static async setupCrashReporting(): Promise<void> {
    console.log("Setting up crash reporting...");

    // Initialize crash reporting SDK
    // Configure crash handlers
  }

  private static async setupPerformanceMonitoring(): Promise<void> {
    console.log("Setting up performance monitoring...");

    // Initialize performance monitoring
    // Configure performance metrics collection
  }

  static trackAppLaunch(): void {
    this.trackEvent("app_launch", {
      timestamp: Date.now(),
      version: BuildConfigService.getConfig().version,
    });
  }

  static trackFeatureUsage(
    feature: string,
    properties?: Record<string, any>
  ): void {
    this.trackEvent("feature_usage", {
      feature: feature,
      timestamp: Date.now(),
      ...properties,
    });
  }

  private static trackEvent(
    eventName: string,
    properties: Record<string, any>
  ): void {
    // Send event to analytics service
    console.log(`Tracking event: ${eventName}`, properties);
  }
}
```

## Best Practices

### 1. Pre-Release Checklist

- ✅ All tests passing
- ✅ Code review completed
- ✅ Security scan passed
- ✅ Performance benchmarks met
- ✅ Accessibility requirements satisfied
- ✅ App store guidelines compliance verified

### 2. Version Management

- Use semantic versioning (MAJOR.MINOR.PATCH)
- Maintain version compatibility matrix
- Document breaking changes clearly
- Plan deprecation timeline for old versions

### 3. Release Strategy

- Implement staged rollouts
- Monitor key metrics during rollout
- Have rollback plan ready
- Communicate releases to stakeholders

### 4. Quality Assurance

- Automate testing pipelines
- Test on multiple device configurations
- Verify app store compliance
- Monitor crash rates and performance

## Conclusion

Successfully publishing and distributing HarmonyOS applications requires careful planning, automated processes, and continuous monitoring. By following these guidelines and implementing proper release pipelines, you can ensure smooth deployments and maintain high-quality applications in production.

Key takeaways:

1. **Automate Everything**: Build comprehensive CI/CD pipelines for reliable releases
2. **Test Thoroughly**: Implement multiple testing layers before release
3. **Monitor Continuously**: Set up proper monitoring and analytics from day one
4. **Plan for Scale**: Design release processes that can handle growing complexity
5. **Learn and Iterate**: Use post-launch data to improve future releases
