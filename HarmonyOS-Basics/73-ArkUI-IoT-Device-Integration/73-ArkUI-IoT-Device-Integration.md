# ArkUI IoT Device Integration

## Introduction

IoT device integration enables ArkUI applications to communicate with smart devices, sensors, and IoT ecosystems. This guide covers device discovery, communication protocols, and real-time data visualization for IoT applications.

## Device Discovery and Management

### IoT Device Manager

```typescript
interface IoTDevice {
  id: string;
  name: string;
  type: "sensor" | "actuator" | "controller" | "gateway";
  protocol: "bluetooth" | "wifi" | "zigbee" | "mqtt";
  status: "online" | "offline" | "error";
  lastSeen: number;
  capabilities: string[];
  data?: Record<string, any>;
}

class IoTDeviceManager {
  private devices = new Map<string, IoTDevice>();
  private listeners = new Set<(devices: IoTDevice[]) => void>();
  private scanTimeout?: number;

  startDeviceScanning(): void {
    this.scanBluetooth();
    this.scanWiFi();
    this.connectMQTT();

    this.scanTimeout = setInterval(() => {
      this.updateDeviceStatus();
    }, 5000);
  }

  stopDeviceScanning(): void {
    if (this.scanTimeout) {
      clearInterval(this.scanTimeout);
    }
  }

  addDevice(device: IoTDevice): void {
    this.devices.set(device.id, device);
    this.notifyListeners();
  }

  removeDevice(deviceId: string): void {
    this.devices.delete(deviceId);
    this.notifyListeners();
  }

  getDevices(): IoTDevice[] {
    return Array.from(this.devices.values());
  }

  getDevice(deviceId: string): IoTDevice | undefined {
    return this.devices.get(deviceId);
  }

  onDevicesChanged(listener: (devices: IoTDevice[]) => void): void {
    this.listeners.add(listener);
  }

  private scanBluetooth(): void {
    // Simulate Bluetooth device discovery
    setTimeout(() => {
      this.addDevice({
        id: "ble_temp_001",
        name: "Temperature Sensor",
        type: "sensor",
        protocol: "bluetooth",
        status: "online",
        lastSeen: Date.now(),
        capabilities: ["temperature", "humidity"],
        data: { temperature: 22.5, humidity: 45 },
      });
    }, 1000);
  }

  private scanWiFi(): void {
    // Simulate WiFi device discovery
    setTimeout(() => {
      this.addDevice({
        id: "wifi_light_001",
        name: "Smart Light",
        type: "actuator",
        protocol: "wifi",
        status: "online",
        lastSeen: Date.now(),
        capabilities: ["brightness", "color"],
        data: { brightness: 80, color: "#ffffff", power: true },
      });
    }, 1500);
  }

  private connectMQTT(): void {
    // Simulate MQTT broker connection
    setTimeout(() => {
      this.addDevice({
        id: "mqtt_gateway_001",
        name: "IoT Gateway",
        type: "gateway",
        protocol: "mqtt",
        status: "online",
        lastSeen: Date.now(),
        capabilities: ["relay", "data_collection"],
        data: { connectedDevices: 5, uptime: 86400 },
      });
    }, 2000);
  }

  private updateDeviceStatus(): void {
    this.devices.forEach((device) => {
      // Simulate data updates
      if (device.type === "sensor") {
        this.updateSensorData(device);
      }
      device.lastSeen = Date.now();
    });
    this.notifyListeners();
  }

  private updateSensorData(device: IoTDevice): void {
    if (device.capabilities.includes("temperature")) {
      device.data!.temperature = 20 + Math.random() * 10;
    }
    if (device.capabilities.includes("humidity")) {
      device.data!.humidity = 40 + Math.random() * 20;
    }
  }

  private notifyListeners(): void {
    const devices = this.getDevices();
    this.listeners.forEach((listener) => listener(devices));
  }
}
```

### Device Control Interface

```typescript
interface DeviceCommand {
  deviceId: string;
  command: string;
  parameters?: Record<string, any>;
}

interface DeviceResponse {
  success: boolean;
  data?: any;
  error?: string;
}

class DeviceController {
  private deviceManager: IoTDeviceManager;

  constructor(deviceManager: IoTDeviceManager) {
    this.deviceManager = deviceManager;
  }

  async sendCommand(command: DeviceCommand): Promise<DeviceResponse> {
    const device = this.deviceManager.getDevice(command.deviceId);
    if (!device) {
      return { success: false, error: "Device not found" };
    }

    try {
      switch (device.protocol) {
        case "bluetooth":
          return await this.sendBluetoothCommand(command);
        case "wifi":
          return await this.sendWiFiCommand(command);
        case "mqtt":
          return await this.sendMQTTCommand(command);
        default:
          return { success: false, error: "Unsupported protocol" };
      }
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  private async sendBluetoothCommand(
    command: DeviceCommand
  ): Promise<DeviceResponse> {
    // Simulate Bluetooth command
    await new Promise((resolve) => setTimeout(resolve, 500));
    return { success: true, data: { status: "command_executed" } };
  }

  private async sendWiFiCommand(
    command: DeviceCommand
  ): Promise<DeviceResponse> {
    // Simulate WiFi HTTP request
    await new Promise((resolve) => setTimeout(resolve, 300));
    return { success: true, data: { status: "command_executed" } };
  }

  private async sendMQTTCommand(
    command: DeviceCommand
  ): Promise<DeviceResponse> {
    // Simulate MQTT publish
    await new Promise((resolve) => setTimeout(resolve, 200));
    return { success: true, data: { status: "command_published" } };
  }
}
```

## Real-time Data Visualization

### IoT Dashboard Component

```typescript
@Component
struct IoTDashboard {
  @State private devices: IoTDevice[] = []
  @State private selectedDevice: IoTDevice | null = null
  private deviceManager = new IoTDeviceManager()
  private deviceController = new DeviceController(this.deviceManager)

  build() {
    Column() {
      this.buildHeader()
      this.buildDeviceGrid()
      if (this.selectedDevice) {
        this.buildDeviceDetails()
      }
    }
    .padding(16)
  }

  aboutToAppear() {
    this.deviceManager.onDevicesChanged((devices) => {
      this.devices = devices
    })
    this.deviceManager.startDeviceScanning()
  }

  aboutToDisappear() {
    this.deviceManager.stopDeviceScanning()
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('IoT Dashboard')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Text(`${this.devices.length} devices`)
        .fontSize(14)
        .fontColor('#666')
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildDeviceGrid() {
    Grid() {
      ForEach(this.devices, (device: IoTDevice) => {
        GridItem() {
          this.buildDeviceCard(device)
        }
      })
    }
    .columnsTemplate('1fr 1fr')
    .columnsGap(16)
    .rowsGap(16)
  }

  @Builder
  private buildDeviceCard(device: IoTDevice) {
    Column() {
      Row() {
        Text(this.getDeviceIcon(device.type))
          .fontSize(32)

        Column() {
          Text(device.name)
            .fontSize(16)
            .fontWeight(FontWeight.Bold)
            .maxLines(1)

          Text(device.type)
            .fontSize(12)
            .fontColor('#666')
        }
        .alignItems(HorizontalAlign.Start)
        .flexGrow(1)
        .margin({ left: 12 })
      }
      .width('100%')
      .margin({ bottom: 12 })

      if (device.data) {
        this.buildDeviceData(device)
      }

      Row() {
        Text(device.protocol.toUpperCase())
          .fontSize(10)
          .fontColor('#666')
          .flexGrow(1)

        Circle({ width: 8, height: 8 })
          .fill(this.getStatusColor(device.status))
      }
      .width('100%')
      .margin({ top: 8 })
    }
    .padding(16)
    .backgroundColor('#ffffff')
    .borderRadius(12)
    .shadow({ radius: 4, color: '#00000020', offsetY: 2 })
    .onClick(() => this.selectedDevice = device)
  }

  @Builder
  private buildDeviceData(device: IoTDevice) {
    Column() {
      ForEach(Object.entries(device.data!), ([key, value]) => {
        Row() {
          Text(key)
            .fontSize(12)
            .fontColor('#666')
            .flexGrow(1)

          Text(this.formatValue(value))
            .fontSize(14)
            .fontWeight(FontWeight.Medium)
        }
        .margin({ bottom: 4 })
      })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildDeviceDetails() {
    if (!this.selectedDevice) return

    Column() {
      Row() {
        Text(`${this.selectedDevice.name} Controls`)
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Button('×')
          .backgroundColor(Color.Transparent)
          .fontColor('#666')
          .onClick(() => this.selectedDevice = null)
      }
      .margin({ bottom: 16 })

      if (this.selectedDevice.type === 'actuator') {
        this.buildActuatorControls(this.selectedDevice)
      } else if (this.selectedDevice.type === 'sensor') {
        this.buildSensorChart(this.selectedDevice)
      }
    }
    .backgroundColor('#ffffff')
    .borderRadius(12)
    .padding(20)
    .margin({ top: 20 })
    .shadow({ radius: 8, color: '#00000020', offsetY: 4 })
  }

  @Builder
  private buildActuatorControls(device: IoTDevice) {
    Column() {
      if (device.capabilities.includes('brightness')) {
        this.buildSliderControl('Brightness', device.data?.brightness || 0, (value) => {
          this.sendDeviceCommand(device.id, 'setBrightness', { brightness: value })
        })
      }

      if (device.capabilities.includes('power')) {
        this.buildSwitchControl('Power', device.data?.power || false, (value) => {
          this.sendDeviceCommand(device.id, 'setPower', { power: value })
        })
      }
    }
  }

  @Builder
  private buildSliderControl(label: string, value: number, onChange: (value: number) => void) {
    Column() {
      Row() {
        Text(label)
          .fontSize(16)
          .flexGrow(1)

        Text(`${value}%`)
          .fontSize(16)
          .fontWeight(FontWeight.Bold)
      }
      .margin({ bottom: 8 })

      Slider({
        value: value,
        min: 0,
        max: 100,
        style: SliderStyle.OutSet
      })
      .trackColor('#E0E0E0')
      .selectedColor('#007AFF')
      .onChange((value) => onChange(value))
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildSwitchControl(label: string, value: boolean, onChange: (value: boolean) => void) {
    Row() {
      Text(label)
        .fontSize(16)
        .flexGrow(1)

      Toggle({ type: ToggleType.Switch, isOn: value })
        .onChange((isOn) => onChange(isOn))
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildSensorChart(device: IoTDevice) {
    Column() {
      Text('Real-time Data')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 16 })

      // Simplified chart representation
      ForEach(Object.entries(device.data!), ([key, value]) => {
        Column() {
          Row() {
            Text(key)
              .fontSize(14)
              .flexGrow(1)

            Text(this.formatValue(value))
              .fontSize(18)
              .fontWeight(FontWeight.Bold)
              .fontColor('#007AFF')
          }
          .margin({ bottom: 8 })

          // Visual representation of the value
          Row() {
            Progress({
              value: this.normalizeValue(key, value as number),
              total: 100,
              style: ProgressStyle.Linear
            })
            .color('#007AFF')
            .backgroundColor('#E0E0E0')
          }
          .width('100%')
        }
        .margin({ bottom: 16 })
      })
    }
  }

  private getDeviceIcon(type: string): string {
    switch (type) {
      case 'sensor': return '🌡️'
      case 'actuator': return '💡'
      case 'controller': return '🎛️'
      case 'gateway': return '📡'
      default: return '📱'
    }
  }

  private getStatusColor(status: string): string {
    switch (status) {
      case 'online': return '#34C759'
      case 'offline': return '#8E8E93'
      case 'error': return '#FF3B30'
      default: return '#8E8E93'
    }
  }

  private formatValue(value: any): string {
    if (typeof value === 'number') {
      return value.toFixed(1)
    }
    return String(value)
  }

  private normalizeValue(key: string, value: number): number {
    switch (key) {
      case 'temperature':
        return Math.max(0, Math.min(100, (value / 50) * 100))
      case 'humidity':
      case 'brightness':
        return Math.max(0, Math.min(100, value))
      default:
        return Math.max(0, Math.min(100, value))
    }
  }

  private async sendDeviceCommand(deviceId: string, command: string, parameters: any): Promise<void> {
    try {
      const response = await this.deviceController.sendCommand({
        deviceId,
        command,
        parameters
      })

      if (!response.success) {
        console.error('Command failed:', response.error)
      }
    } catch (error) {
      console.error('Failed to send command:', error)
    }
  }
}
```

## Sensor Data Processing

### Data Analytics Engine

```typescript
interface SensorReading {
  timestamp: number;
  deviceId: string;
  sensorType: string;
  value: number;
  unit: string;
}

class SensorDataAnalytics {
  private readings: SensorReading[] = [];
  private maxReadings = 1000;

  addReading(reading: SensorReading): void {
    this.readings.push(reading);

    if (this.readings.length > this.maxReadings) {
      this.readings = this.readings.slice(-this.maxReadings);
    }
  }

  getAverageValue(
    deviceId: string,
    sensorType: string,
    timeWindow: number
  ): number {
    const cutoff = Date.now() - timeWindow;
    const relevantReadings = this.readings.filter(
      (r) =>
        r.deviceId === deviceId &&
        r.sensorType === sensorType &&
        r.timestamp >= cutoff
    );

    if (relevantReadings.length === 0) return 0;

    const sum = relevantReadings.reduce(
      (acc, reading) => acc + reading.value,
      0
    );
    return sum / relevantReadings.length;
  }

  detectAnomalies(deviceId: string, sensorType: string): SensorReading[] {
    const readings = this.readings.filter(
      (r) => r.deviceId === deviceId && r.sensorType === sensorType
    );

    if (readings.length < 10) return [];

    const values = readings.map((r) => r.value);
    const mean = values.reduce((a, b) => a + b, 0) / values.length;
    const variance =
      values.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / values.length;
    const stdDev = Math.sqrt(variance);

    const threshold = 2 * stdDev;

    return readings.filter(
      (reading) => Math.abs(reading.value - mean) > threshold
    );
  }

  getTrend(
    deviceId: string,
    sensorType: string,
    timeWindow: number
  ): "increasing" | "decreasing" | "stable" {
    const cutoff = Date.now() - timeWindow;
    const readings = this.readings
      .filter(
        (r) =>
          r.deviceId === deviceId &&
          r.sensorType === sensorType &&
          r.timestamp >= cutoff
      )
      .sort((a, b) => a.timestamp - b.timestamp);

    if (readings.length < 2) return "stable";

    const firstHalf = readings.slice(0, Math.floor(readings.length / 2));
    const secondHalf = readings.slice(Math.floor(readings.length / 2));

    const firstAvg =
      firstHalf.reduce((sum, r) => sum + r.value, 0) / firstHalf.length;
    const secondAvg =
      secondHalf.reduce((sum, r) => sum + r.value, 0) / secondHalf.length;

    const diff = secondAvg - firstAvg;
    const threshold = 0.1;

    if (diff > threshold) return "increasing";
    if (diff < -threshold) return "decreasing";
    return "stable";
  }
}
```

## Home Automation Integration

### Smart Home Controller

```typescript
interface HomeScene {
  id: string;
  name: string;
  devices: { deviceId: string; state: any }[];
  triggers?: SceneTrigger[];
}

interface SceneTrigger {
  type: "time" | "sensor" | "location";
  condition: any;
}

class SmartHomeController {
  private scenes: HomeScene[] = [];
  private deviceManager: IoTDeviceManager;
  private deviceController: DeviceController;

  constructor(
    deviceManager: IoTDeviceManager,
    deviceController: DeviceController
  ) {
    this.deviceManager = deviceManager;
    this.deviceController = deviceController;
  }

  createScene(scene: HomeScene): void {
    this.scenes.push(scene);
  }

  async activateScene(sceneId: string): Promise<boolean> {
    const scene = this.scenes.find((s) => s.id === sceneId);
    if (!scene) return false;

    try {
      for (const deviceState of scene.devices) {
        await this.deviceController.sendCommand({
          deviceId: deviceState.deviceId,
          command: "setState",
          parameters: deviceState.state,
        });
      }
      return true;
    } catch (error) {
      console.error("Failed to activate scene:", error);
      return false;
    }
  }

  getScenes(): HomeScene[] {
    return this.scenes;
  }

  initializeDefaultScenes(): void {
    this.scenes = [
      {
        id: "morning",
        name: "Good Morning",
        devices: [
          {
            deviceId: "wifi_light_001",
            state: { brightness: 80, power: true },
          },
          { deviceId: "smart_thermostat", state: { temperature: 22 } },
        ],
        triggers: [{ type: "time", condition: { hour: 7, minute: 0 } }],
      },
      {
        id: "evening",
        name: "Evening Relaxation",
        devices: [
          {
            deviceId: "wifi_light_001",
            state: { brightness: 30, color: "#ff8800" },
          },
        ],
        triggers: [{ type: "time", condition: { hour: 20, minute: 0 } }],
      },
    ];
  }
}
```

## Conclusion

IoT device integration in ArkUI applications enables:

- Comprehensive device discovery and management across multiple protocols
- Real-time sensor data visualization and analytics
- Smart device control and automation
- Home automation scene management
- Anomaly detection and trend analysis
- Cross-platform IoT ecosystem integration

These capabilities allow developers to create sophisticated IoT applications that seamlessly integrate with smart devices and provide intelligent automation features for users.
