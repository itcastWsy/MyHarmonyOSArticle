# 27-Device Capabilities and Hardware Integration

## Introduction

HarmonyOS provides extensive APIs for accessing device hardware capabilities, including sensors, camera, location services, and system features. This article explores how to integrate with various device capabilities to create rich, hardware-aware applications.

## Permission Management

### Requesting Permissions

```typescript
import abilityAccessCtrl, { Permissions } from "@ohos.abilityAccessCtrl";
import { Context } from "@ohos.ability.common";

export class PermissionManager {
  private static accessManager = abilityAccessCtrl.createAtManager();

  static async requestPermissions(
    context: Context,
    permissions: Permissions[]
  ): Promise<boolean> {
    try {
      const grantStatus = await this.accessManager.requestPermissionsFromUser(
        context,
        permissions
      );

      for (let i = 0; i < grantStatus.permissions.length; i++) {
        if (
          grantStatus.authResults[i] !==
          abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
        ) {
          console.log(`Permission denied: ${grantStatus.permissions[i]}`);
          return false;
        }
      }

      return true;
    } catch (error) {
      console.error("Failed to request permissions:", error);
      return false;
    }
  }

  static async checkPermission(permission: Permissions): Promise<boolean> {
    try {
      const grantStatus = await this.accessManager.checkAccessToken(
        0,
        permission
      );
      return grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED;
    } catch (error) {
      console.error(`Failed to check permission ${permission}:`, error);
      return false;
    }
  }

  static async requestCameraPermission(context: Context): Promise<boolean> {
    return await this.requestPermissions(context, ["ohos.permission.CAMERA"]);
  }

  static async requestLocationPermission(context: Context): Promise<boolean> {
    return await this.requestPermissions(context, [
      "ohos.permission.APPROXIMATELY_LOCATION",
      "ohos.permission.LOCATION",
    ]);
  }

  static async requestMicrophonePermission(context: Context): Promise<boolean> {
    return await this.requestPermissions(context, [
      "ohos.permission.MICROPHONE",
    ]);
  }
}
```

## Camera Integration

### Basic Camera Operations

```typescript
import camera from '@ohos.multimedia.camera';
import { Context } from '@ohos.ability.common';

export class CameraManager {
  private cameraManager: camera.CameraManager | null = null;
  private captureSession: camera.CaptureSession | null = null;
  private photoOutput: camera.PhotoOutput | null = null;

  async initialize(context: Context): Promise<void> {
    try {
      this.cameraManager = camera.getCameraManager(context);

      // Get available cameras
      const cameras = await this.cameraManager.getSupportedCameras();
      if (cameras.length === 0) {
        throw new Error('No cameras available');
      }

      console.log(`Found ${cameras.length} cameras`);
    } catch (error) {
      console.error('Failed to initialize camera:', error);
      throw error;
    }
  }

  async startPreview(surfaceId: string): Promise<void> {
    if (!this.cameraManager) {
      throw new Error('Camera manager not initialized');
    }

    try {
      const cameras = await this.cameraManager.getSupportedCameras();
      const cameraInput = this.cameraManager.createCameraInput(cameras[0]);

      await cameraInput.open();

      // Create capture session
      this.captureSession = this.cameraManager.createCaptureSession();
      await this.captureSession.beginConfig();

      // Add camera input
      await this.captureSession.addInput(cameraInput);

      // Create preview output
      const previewProfile = this.cameraManager.getSupportedPreviewProfiles(cameras[0])[0];
      const previewOutput = this.cameraManager.createPreviewOutput(previewProfile, surfaceId);
      await this.captureSession.addOutput(previewOutput);

      // Create photo output
      const photoProfile = this.cameraManager.getSupportedPhotoProfiles(cameras[0])[0];
      this.photoOutput = this.cameraManager.createPhotoOutput(photoProfile);
      await this.captureSession.addOutput(this.photoOutput);

      await this.captureSession.commitConfig();
      await this.captureSession.start();

      console.log('Camera preview started');
    } catch (error) {
      console.error('Failed to start camera preview:', error);
      throw error;
    }
  }

  async takePhoto(): Promise<void> {
    if (!this.photoOutput) {
      throw new Error('Photo output not initialized');
    }

    try {
      const photoCaptureSetting: camera.PhotoCaptureSetting = {
        quality: camera.QualityLevel.QUALITY_LEVEL_HIGH,
        rotation: camera.ImageRotation.ROTATION_0
      };

      await this.photoOutput.capture(photoCaptureSetting);
      console.log('Photo captured');
    } catch (error) {
      console.error('Failed to capture photo:', error);
      throw error;
    }
  }

  async stopPreview(): Promise<void> {
    try {
      if (this.captureSession) {
        await this.captureSession.stop();
        await this.captureSession.release();
        this.captureSession = null;
      }

      this.photoOutput = null;
      console.log('Camera preview stopped');
    } catch (error) {
      console.error('Failed to stop camera preview:', error);
      throw error;
    }
  }
}

// Camera component
@Component
struct CameraPreview {
  @State private cameraManager: CameraManager = new CameraManager();
  @State private isPreviewActive: boolean = false;
  private xComponentController: XComponentController = new XComponentController();

  async aboutToAppear(): Promise<void> {
    try {
      const hasPermission = await PermissionManager.requestCameraPermission(getContext(this));
      if (!hasPermission) {
        console.error('Camera permission denied');
        return;
      }

      await this.cameraManager.initialize(getContext(this));
    } catch (error) {
      console.error('Failed to setup camera:', error);
    }
  }

  private async startCamera(): Promise<void> {
    try {
      const surfaceId = this.xComponentController.getXComponentSurfaceId();
      await this.cameraManager.startPreview(surfaceId);
      this.isPreviewActive = true;
    } catch (error) {
      console.error('Failed to start camera:', error);
    }
  }

  private async capturePhoto(): Promise<void> {
    try {
      await this.cameraManager.takePhoto();
    } catch (error) {
      console.error('Failed to capture photo:', error);
    }
  }

  private async stopCamera(): Promise<void> {
    try {
      await this.cameraManager.stopPreview();
      this.isPreviewActive = false;
    } catch (error) {
      console.error('Failed to stop camera:', error);
    }
  }

  build() {
    Column() {
      // Camera preview
      XComponent({
        id: 'cameraPreview',
        type: 'surface',
        controller: this.xComponentController
      })
        .width('100%')
        .height('70%')
        .onLoad(() => {
          console.log('XComponent loaded');
        })

      // Control buttons
      Row({ space: 20 }) {
        Button(this.isPreviewActive ? 'Stop Camera' : 'Start Camera')
          .onClick(async () => {
            if (this.isPreviewActive) {
              await this.stopCamera();
            } else {
              await this.startCamera();
            }
          })

        Button('Capture Photo')
          .enabled(this.isPreviewActive)
          .onClick(async () => {
            await this.capturePhoto();
          })
      }
      .margin({ top: 20 })
    }
    .padding(16)
  }
}
```

## Location Services

### GPS and Location Tracking

```typescript
import geoLocationManager from '@ohos.geoLocationManager';

export class LocationManager {
  private static currentLocationRequest: number = -1;

  static async getCurrentLocation(): Promise<geoLocationManager.Location> {
    try {
      const location = await geoLocationManager.getCurrentLocation({
        priority: geoLocationManager.LocationRequestPriority.FIRST_FIX,
        scenario: geoLocationManager.LocationRequestScenario.UNSET,
        timeoutMs: 30000,
        maxAccuracy: 100
      });

      return location;
    } catch (error) {
      console.error('Failed to get current location:', error);
      throw new Error('Unable to retrieve location. Please check GPS settings.');
    }
  }

  static async startLocationTracking(
    callback: (location: geoLocationManager.Location) => void,
    interval: number = 10000
  ): Promise<void> {
    try {
      const requestInfo: geoLocationManager.LocationRequest = {
        priority: geoLocationManager.LocationRequestPriority.ACCURACY,
        scenario: geoLocationManager.LocationRequestScenario.NAVIGATION,
        timeInterval: interval,
        distanceInterval: 10,
        maxAccuracy: 50
      };

      this.currentLocationRequest = geoLocationManager.on('locationChange', requestInfo, callback);
      console.log('Location tracking started');
    } catch (error) {
      console.error('Failed to start location tracking:', error);
      throw error;
    }
  }

  static stopLocationTracking(): void {
    if (this.currentLocationRequest !== -1) {
      geoLocationManager.off('locationChange', this.currentLocationRequest);
      this.currentLocationRequest = -1;
      console.log('Location tracking stopped');
    }
  }

  static async getAddressFromCoordinates(latitude: number, longitude: number): Promise<string> {
    try {
      const reverseGeoCodeRequest: geoLocationManager.ReverseGeoCodeRequest = {
        latitude: latitude,
        longitude: longitude,
        maxItems: 1
      };

      const addresses = await geoLocationManager.getAddressesFromLocation(reverseGeoCodeRequest);

      if (addresses && addresses.length > 0) {
        const address = addresses[0];
        return `${address.addressLine}, ${address.locality}, ${address.countryName}`;
      }

      return 'Address not found';
    } catch (error) {
      console.error('Failed to get address from coordinates:', error);
      return 'Unable to retrieve address';
    }
  }
}

// Location tracking component
@Component
struct LocationTracker {
  @State private currentLocation: geoLocationManager.Location | null = null;
  @State private currentAddress: string = '';
  @State private isTracking: boolean = false;

  async aboutToAppear(): Promise<void> {
    const hasPermission = await PermissionManager.requestLocationPermission(getContext(this));
    if (!hasPermission) {
      console.error('Location permission denied');
    }
  }

  aboutToDisappear(): void {
    this.stopTracking();
  }

  private async getCurrentLocation(): Promise<void> {
    try {
      const location = await LocationManager.getCurrentLocation();
      this.currentLocation = location;

      // Get address
      this.currentAddress = await LocationManager.getAddressFromCoordinates(
        location.latitude,
        location.longitude
      );
    } catch (error) {
      console.error('Failed to get location:', error);
    }
  }

  private async startTracking(): Promise<void> {
    try {
      await LocationManager.startLocationTracking((location) => {
        this.currentLocation = location;
        console.log(`Location updated: ${location.latitude}, ${location.longitude}`);
      }, 5000); // Update every 5 seconds

      this.isTracking = true;
    } catch (error) {
      console.error('Failed to start tracking:', error);
    }
  }

  private stopTracking(): void {
    LocationManager.stopLocationTracking();
    this.isTracking = false;
  }

  build() {
    Column({ space: 16 }) {
      Text('Location Services')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      if (this.currentLocation) {
        Column({ space: 8 }) {
          Text(`Latitude: ${this.currentLocation.latitude.toFixed(6)}`)
          Text(`Longitude: ${this.currentLocation.longitude.toFixed(6)}`)
          Text(`Accuracy: ${this.currentLocation.accuracy}m`)
          Text(`Address: ${this.currentAddress}`)
        }
        .padding(16)
        .backgroundColor('#F5F5F5')
        .borderRadius(8)
      }

      Row({ space: 16 }) {
        Button('Get Current Location')
          .onClick(async () => {
            await this.getCurrentLocation();
          })

        Button(this.isTracking ? 'Stop Tracking' : 'Start Tracking')
          .onClick(async () => {
            if (this.isTracking) {
              this.stopTracking();
            } else {
              await this.startTracking();
            }
          })
      }
    }
    .padding(20)
    .width('100%')
  }
}
```

## Sensor Integration

### Accelerometer and Gyroscope

```typescript
import sensor from '@ohos.sensor';

export class SensorManager {
  private static accelerometerCallback: sensor.SensorId.ACCELEROMETER | null = null;
  private static gyroscopeCallback: sensor.SensorId.GYROSCOPE | null = null;

  static startAccelerometer(callback: (data: sensor.AccelerometerResponse) => void): void {
    try {
      sensor.on(sensor.SensorId.ACCELEROMETER, callback, {
        interval: 100000000 // 100ms in nanoseconds
      });

      this.accelerometerCallback = sensor.SensorId.ACCELEROMETER;
      console.log('Accelerometer started');
    } catch (error) {
      console.error('Failed to start accelerometer:', error);
    }
  }

  static stopAccelerometer(): void {
    if (this.accelerometerCallback) {
      sensor.off(sensor.SensorId.ACCELEROMETER);
      this.accelerometerCallback = null;
      console.log('Accelerometer stopped');
    }
  }

  static startGyroscope(callback: (data: sensor.GyroscopeResponse) => void): void {
    try {
      sensor.on(sensor.SensorId.GYROSCOPE, callback, {
        interval: 100000000 // 100ms in nanoseconds
      });

      this.gyroscopeCallback = sensor.SensorId.GYROSCOPE;
      console.log('Gyroscope started');
    } catch (error) {
      console.error('Failed to start gyroscope:', error);
    }
  }

  static stopGyroscope(): void {
    if (this.gyroscopeCallback) {
      sensor.off(sensor.SensorId.GYROSCOPE);
      this.gyroscopeCallback = null;
      console.log('Gyroscope stopped');
    }
  }

  static async getSensorInfo(sensorId: sensor.SensorId): Promise<sensor.Sensor[]> {
    try {
      return await sensor.getSensorList();
    } catch (error) {
      console.error('Failed to get sensor info:', error);
      return [];
    }
  }
}

// Sensor monitoring component
@Component
struct SensorMonitor {
  @State private accelerometerData: sensor.AccelerometerResponse | null = null;
  @State private gyroscopeData: sensor.GyroscopeResponse | null = null;
  @State private isMonitoring: boolean = false;

  aboutToDisappear(): void {
    this.stopMonitoring();
  }

  private startMonitoring(): void {
    SensorManager.startAccelerometer((data) => {
      this.accelerometerData = data;
    });

    SensorManager.startGyroscope((data) => {
      this.gyroscopeData = data;
    });

    this.isMonitoring = true;
  }

  private stopMonitoring(): void {
    SensorManager.stopAccelerometer();
    SensorManager.stopGyroscope();
    this.isMonitoring = false;
  }

  build() {
    Column({ space: 16 }) {
      Text('Sensor Monitor')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // Accelerometer data
      if (this.accelerometerData) {
        Column({ space: 8 }) {
          Text('Accelerometer')
            .fontSize(18)
            .fontWeight(FontWeight.Medium)
          Text(`X: ${this.accelerometerData.x.toFixed(3)}`)
          Text(`Y: ${this.accelerometerData.y.toFixed(3)}`)
          Text(`Z: ${this.accelerometerData.z.toFixed(3)}`)
        }
        .padding(16)
        .backgroundColor('#E3F2FD')
        .borderRadius(8)
        .width('100%')
      }

      // Gyroscope data
      if (this.gyroscopeData) {
        Column({ space: 8 }) {
          Text('Gyroscope')
            .fontSize(18)
            .fontWeight(FontWeight.Medium)
          Text(`X: ${this.gyroscopeData.x.toFixed(3)}`)
          Text(`Y: ${this.gyroscopeData.y.toFixed(3)}`)
          Text(`Z: ${this.gyroscopeData.z.toFixed(3)}`)
        }
        .padding(16)
        .backgroundColor('#F3E5F5')
        .borderRadius(8)
        .width('100%')
      }

      Button(this.isMonitoring ? 'Stop Monitoring' : 'Start Monitoring')
        .onClick(() => {
          if (this.isMonitoring) {
            this.stopMonitoring();
          } else {
            this.startMonitoring();
          }
        })
    }
    .padding(20)
    .width('100%')
  }
}
```

## System Information

### Device and System Info

```typescript
import deviceInfo from "@ohos.deviceInfo";
import batteryInfo from "@ohos.batteryInfo";
import display from "@ohos.display";

export class SystemInfoManager {
  static getDeviceInfo(): DeviceInformation {
    return {
      deviceType: deviceInfo.deviceType,
      manufacture: deviceInfo.manufacture,
      brand: deviceInfo.brand,
      marketName: deviceInfo.marketName,
      productModel: deviceInfo.productModel,
      productSeries: deviceInfo.productSeries,
      osFullName: deviceInfo.osFullName,
      osReleaseType: deviceInfo.osReleaseType,
      sdkApiVersion: deviceInfo.sdkApiVersion.toString(),
    };
  }

  static getBatteryInfo(): BatteryInformation {
    return {
      batterySOC: batteryInfo.batterySOC,
      chargingStatus: batteryInfo.chargingStatus,
      healthStatus: batteryInfo.healthStatus,
      pluggedType: batteryInfo.pluggedType,
      voltage: batteryInfo.voltage,
      technology: batteryInfo.technology,
      batteryTemperature: batteryInfo.batteryTemperature,
    };
  }

  static async getDisplayInfo(): Promise<DisplayInformation> {
    try {
      const defaultDisplay = await display.getDefaultDisplaySync();

      return {
        id: defaultDisplay.id,
        width: defaultDisplay.width,
        height: defaultDisplay.height,
        densityDPI: defaultDisplay.densityDPI,
        densityPixels: defaultDisplay.densityPixels,
        scaledDensity: defaultDisplay.scaledDensity,
        rotation: defaultDisplay.rotation,
      };
    } catch (error) {
      console.error("Failed to get display info:", error);
      throw error;
    }
  }
}

interface DeviceInformation {
  deviceType: string;
  manufacture: string;
  brand: string;
  marketName: string;
  productModel: string;
  productSeries: string;
  osFullName: string;
  osReleaseType: string;
  sdkApiVersion: string;
}

interface BatteryInformation {
  batterySOC: number;
  chargingStatus: batteryInfo.BatteryChargeState;
  healthStatus: batteryInfo.BatteryHealthState;
  pluggedType: batteryInfo.BatteryPluggedType;
  voltage: number;
  technology: string;
  batteryTemperature: number;
}

interface DisplayInformation {
  id: number;
  width: number;
  height: number;
  densityDPI: number;
  densityPixels: number;
  scaledDensity: number;
  rotation: number;
}
```

## Best Practices

### 1. Permission Management

- Request permissions at the appropriate time
- Explain why permissions are needed
- Handle permission denials gracefully
- Use minimal necessary permissions

### 2. Hardware Integration

- Check hardware availability before use
- Handle hardware errors gracefully
- Optimize battery usage
- Clean up resources properly

### 3. Performance Considerations

- Use sensors efficiently to preserve battery
- Implement proper data throttling
- Cache frequently accessed system information
- Handle background/foreground transitions

### 4. User Experience

- Provide feedback for hardware operations
- Implement fallback mechanisms
- Handle hardware unavailability
- Respect user privacy settings

## Conclusion

HarmonyOS provides comprehensive APIs for integrating with device hardware capabilities. By properly managing permissions, implementing efficient hardware access patterns, and following best practices, you can create applications that leverage device capabilities while maintaining good performance and user experience.

Key takeaways:

1. **Permission Strategy**: Implement proper permission management workflows
2. **Hardware Integration**: Use appropriate APIs for different hardware components
3. **Resource Management**: Clean up hardware resources and listeners properly
4. **Error Handling**: Handle hardware unavailability and errors gracefully
5. **Performance**: Optimize hardware usage for battery life and performance
