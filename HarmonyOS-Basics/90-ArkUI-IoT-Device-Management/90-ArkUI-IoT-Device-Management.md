# ArkUI IoT Device Management

## Introduction

This guide covers IoT device integration in ArkUI applications, including device discovery, connection management, protocol handling, and real-time data visualization for smart home and industrial IoT scenarios.

## IoT Device Framework

### Device Discovery and Management

```typescript
interface IoTDevice {
  id: string;
  name: string;
  type: "sensor" | "actuator" | "gateway" | "controller";
  protocol: "wifi" | "bluetooth" | "zigbee" | "mqtt" | "coap";
  status: "online" | "offline" | "connecting" | "error";
  lastSeen: number;
  capabilities: string[];
  properties: Record<string, any>;
}

interface DeviceReading {
  deviceId: string;
  sensor: string;
  value: number;
  unit: string;
  timestamp: number;
  quality: "good" | "poor" | "bad";
}

interface DeviceCommand {
  deviceId: string;
  action: string;
  parameters: Record<string, any>;
  timestamp: number;
  status: "pending" | "sent" | "acknowledged" | "failed";
}

class IoTDeviceManager {
  private devices = new Map<string, IoTDevice>();
  private readings = new Map<string, DeviceReading[]>();
  private commands = new Map<string, DeviceCommand>();
  private connectionHandlers = new Map<string, any>();

  async discoverDevices(protocol?: string): Promise<IoTDevice[]> {
    const discovered: IoTDevice[] = [];

    try {
      switch (protocol) {
        case "wifi":
          discovered.push(...(await this.discoverWiFiDevices()));
          break;
        case "bluetooth":
          discovered.push(...(await this.discoverBluetoothDevices()));
          break;
        case "mqtt":
          discovered.push(...(await this.discoverMQTTDevices()));
          break;
        default:
          discovered.push(...(await this.discoverAllProtocols()));
      }

      discovered.forEach((device) => {
        this.devices.set(device.id, device);
      });

      return discovered;
    } catch (error) {
      console.error("Device discovery failed:", error);
      return [];
    }
  }

  async connectDevice(deviceId: string): Promise<boolean> {
    const device = this.devices.get(deviceId);
    if (!device) return false;

    try {
      device.status = "connecting";

      const handler = await this.createConnectionHandler(device);
      this.connectionHandlers.set(deviceId, handler);

      await handler.connect();

      device.status = "online";
      device.lastSeen = Date.now();

      this.startDataCollection(device);

      return true;
    } catch (error) {
      console.error(`Failed to connect to device ${deviceId}:`, error);
      device.status = "error";
      return false;
    }
  }

  async disconnectDevice(deviceId: string): Promise<void> {
    const device = this.devices.get(deviceId);
    const handler = this.connectionHandlers.get(deviceId);

    if (device && handler) {
      await handler.disconnect();
      device.status = "offline";
      this.connectionHandlers.delete(deviceId);
    }
  }

  async sendCommand(command: DeviceCommand): Promise<boolean> {
    const device = this.devices.get(command.deviceId);
    const handler = this.connectionHandlers.get(command.deviceId);

    if (!device || !handler || device.status !== "online") {
      command.status = "failed";
      return false;
    }

    try {
      command.status = "sent";
      this.commands.set(command.deviceId, command);

      await handler.sendCommand(command.action, command.parameters);

      command.status = "acknowledged";
      return true;
    } catch (error) {
      console.error(`Command failed for device ${command.deviceId}:`, error);
      command.status = "failed";
      return false;
    }
  }

  getDevices(): IoTDevice[] {
    return Array.from(this.devices.values());
  }

  getDeviceReadings(deviceId: string, limit: number = 100): DeviceReading[] {
    const readings = this.readings.get(deviceId) || [];
    return readings.slice(-limit);
  }

  private async discoverWiFiDevices(): Promise<IoTDevice[]> {
    // Simulate WiFi device discovery
    return [
      {
        id: "temp-sensor-001",
        name: "Living Room Temp Sensor",
        type: "sensor",
        protocol: "wifi",
        status: "offline",
        lastSeen: Date.now(),
        capabilities: ["temperature", "humidity"],
        properties: { location: "living-room", model: "TempSens-Pro" },
      },
      {
        id: "smart-light-002",
        name: "Kitchen Smart Light",
        type: "actuator",
        protocol: "wifi",
        status: "offline",
        lastSeen: Date.now(),
        capabilities: ["dimming", "color-change"],
        properties: { location: "kitchen", brightness: 80, color: "#FFFFFF" },
      },
    ];
  }

  private async discoverBluetoothDevices(): Promise<IoTDevice[]> {
    // Simulate Bluetooth device discovery
    return [
      {
        id: "fitness-tracker-003",
        name: "Fitness Tracker",
        type: "sensor",
        protocol: "bluetooth",
        status: "offline",
        lastSeen: Date.now(),
        capabilities: ["heart-rate", "steps", "sleep"],
        properties: { batteryLevel: 75, version: "2.1.0" },
      },
    ];
  }

  private async discoverMQTTDevices(): Promise<IoTDevice[]> {
    // Simulate MQTT device discovery
    return [
      {
        id: "security-camera-004",
        name: "Front Door Camera",
        type: "sensor",
        protocol: "mqtt",
        status: "offline",
        lastSeen: Date.now(),
        capabilities: ["video", "motion-detection", "night-vision"],
        properties: { location: "front-door", resolution: "1080p" },
      },
    ];
  }

  private async discoverAllProtocols(): Promise<IoTDevice[]> {
    const allDevices = await Promise.all([
      this.discoverWiFiDevices(),
      this.discoverBluetoothDevices(),
      this.discoverMQTTDevices(),
    ]);

    return allDevices.flat();
  }

  private async createConnectionHandler(device: IoTDevice): Promise<any> {
    switch (device.protocol) {
      case "wifi":
        return new WiFiDeviceHandler(device);
      case "bluetooth":
        return new BluetoothDeviceHandler(device);
      case "mqtt":
        return new MQTTDeviceHandler(device);
      default:
        throw new Error(`Unsupported protocol: ${device.protocol}`);
    }
  }

  private startDataCollection(device: IoTDevice): void {
    const handler = this.connectionHandlers.get(device.id);
    if (!handler) return;

    handler.onDataReceived = (data: any) => {
      const reading: DeviceReading = {
        deviceId: device.id,
        sensor: data.sensor,
        value: data.value,
        unit: data.unit,
        timestamp: Date.now(),
        quality: data.quality || "good",
      };

      let deviceReadings = this.readings.get(device.id) || [];
      deviceReadings.push(reading);

      // Keep only last 1000 readings
      if (deviceReadings.length > 1000) {
        deviceReadings = deviceReadings.slice(-1000);
      }

      this.readings.set(device.id, deviceReadings);
    };

    handler.startDataCollection();
  }
}

// Protocol-specific handlers
class WiFiDeviceHandler {
  constructor(private device: IoTDevice) {}

  async connect(): Promise<void> {
    // WiFi connection implementation
    console.log(`Connecting to WiFi device: ${this.device.name}`);
  }

  async disconnect(): Promise<void> {
    console.log(`Disconnecting WiFi device: ${this.device.name}`);
  }

  async sendCommand(
    action: string,
    parameters: Record<string, any>
  ): Promise<void> {
    console.log(`Sending command ${action} to ${this.device.name}`, parameters);
  }

  startDataCollection(): void {
    // Simulate periodic data collection
    setInterval(() => {
      if (this.onDataReceived) {
        this.onDataReceived({
          sensor: "temperature",
          value: 20 + Math.random() * 10,
          unit: "°C",
          quality: "good",
        });
      }
    }, 5000);
  }

  onDataReceived?: (data: any) => void;
}

class BluetoothDeviceHandler {
  constructor(private device: IoTDevice) {}

  async connect(): Promise<void> {
    console.log(`Connecting to Bluetooth device: ${this.device.name}`);
  }

  async disconnect(): Promise<void> {
    console.log(`Disconnecting Bluetooth device: ${this.device.name}`);
  }

  async sendCommand(
    action: string,
    parameters: Record<string, any>
  ): Promise<void> {
    console.log(
      `Sending BLE command ${action} to ${this.device.name}`,
      parameters
    );
  }

  startDataCollection(): void {
    setInterval(() => {
      if (this.onDataReceived) {
        this.onDataReceived({
          sensor: "heart-rate",
          value: 60 + Math.random() * 40,
          unit: "bpm",
          quality: "good",
        });
      }
    }, 1000);
  }

  onDataReceived?: (data: any) => void;
}

class MQTTDeviceHandler {
  constructor(private device: IoTDevice) {}

  async connect(): Promise<void> {
    console.log(`Connecting to MQTT device: ${this.device.name}`);
  }

  async disconnect(): Promise<void> {
    console.log(`Disconnecting MQTT device: ${this.device.name}`);
  }

  async sendCommand(
    action: string,
    parameters: Record<string, any>
  ): Promise<void> {
    console.log(
      `Publishing MQTT command ${action} to ${this.device.name}`,
      parameters
    );
  }

  startDataCollection(): void {
    setInterval(() => {
      if (this.onDataReceived) {
        this.onDataReceived({
          sensor: "motion",
          value: Math.random() > 0.8 ? 1 : 0,
          unit: "detected",
          quality: "good",
        });
      }
    }, 2000);
  }

  onDataReceived?: (data: any) => void;
}
```

## IoT Dashboard Component

```typescript
@Component
struct IoTDashboard {
  @State private devices: IoTDevice[] = []
  @State private selectedDevice: string = ''
  @State private isScanning: boolean = false
  @State private deviceReadings: Map<string, DeviceReading[]> = new Map()

  private iotManager = new IoTDeviceManager()

  aboutToAppear() {
    this.loadDevices()
  }

  build() {
    Column() {
      this.buildHeader()
      this.buildDeviceGrid()

      if (this.selectedDevice) {
        this.buildDeviceDetails()
      }
    }
    .width('100%')
    .height('100%')
    .padding(16)
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('IoT Dashboard')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Button(this.isScanning ? 'Scanning...' : 'Scan')
        .onClick(() => this.scanDevices())
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .enabled(!this.isScanning)

      if (this.isScanning) {
        LoadingProgress()
          .width(20)
          .height(20)
          .margin({ left: 8 })
      }
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildDeviceGrid() {
    Text(`${this.devices.length} Device(s)`)
      .fontSize(16)
      .fontWeight(FontWeight.Medium)
      .margin({ bottom: 12 })
      .alignSelf(ItemAlign.Start)

    Grid() {
      ForEach(this.devices, (device: IoTDevice) => {
        GridItem() {
          this.buildDeviceCard(device)
        }
      })
    }
    .columnsTemplate('1fr 1fr')
    .columnsGap(12)
    .rowsGap(12)
    .height(300)
  }

  @Builder
  private buildDeviceCard(device: IoTDevice) {
    Column() {
      Row() {
        Text(this.getDeviceIcon(device.type))
          .fontSize(24)
          .margin({ right: 8 })

        Column() {
          Text(device.name)
            .fontSize(14)
            .fontWeight(FontWeight.Bold)
            .maxLines(1)
            .textOverflow({ overflow: TextOverflow.Ellipsis })

          Text(device.protocol.toUpperCase())
            .fontSize(10)
            .fontColor('#666666')
        }
        .alignItems(HorizontalAlign.Start)
        .flexGrow(1)
      }
      .width('100%')
      .margin({ bottom: 8 })

      Row() {
        Text(device.status)
          .fontSize(12)
          .fontColor(this.getStatusColor(device.status))
          .backgroundColor(this.getStatusBackgroundColor(device.status))
          .padding({ horizontal: 6, vertical: 2 })
          .borderRadius(4)
          .flexGrow(1)

        if (device.status === 'offline') {
          Button('Connect')
            .onClick(() => this.connectDevice(device.id))
            .backgroundColor('#34C759')
            .fontColor('#FFFFFF')
            .fontSize(10)
            .height(24)
        } else if (device.status === 'online') {
          Button('Details')
            .onClick(() => this.selectedDevice = device.id)
            .backgroundColor('#007AFF')
            .fontColor('#FFFFFF')
            .fontSize(10)
            .height(24)
        }
      }
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .border({
      width: 1,
      color: device.status === 'online' ? '#34C759' : '#E0E0E0',
      style: BorderStyle.Solid
    })
  }

  @Builder
  private buildDeviceDetails() {
    const device = this.devices.find(d => d.id === this.selectedDevice)
    if (!device) return

    Column() {
      Row() {
        Text(`${device.name} Details`)
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Button('Close')
          .onClick(() => this.selectedDevice = '')
          .backgroundColor('#8E8E93')
          .fontColor('#FFFFFF')
          .fontSize(12)
          .height(32)
      }
      .margin({ bottom: 16 })

      this.buildDeviceInfo(device)
      this.buildDeviceReadings(device)
      this.buildDeviceControls(device)
    }
    .width('100%')
    .padding(16)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
    .margin({ top: 20 })
  }

  @Builder
  private buildDeviceInfo(device: IoTDevice) {
    Column() {
      Text('Device Information')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Row() {
        Text('Type:')
          .fontSize(14)
          .width(80)
        Text(device.type)
          .fontSize(14)
          .fontColor('#666666')
      }
      .margin({ bottom: 4 })

      Row() {
        Text('Protocol:')
          .fontSize(14)
          .width(80)
        Text(device.protocol)
          .fontSize(14)
          .fontColor('#666666')
      }
      .margin({ bottom: 4 })

      Row() {
        Text('Status:')
          .fontSize(14)
          .width(80)
        Text(device.status)
          .fontSize(14)
          .fontColor(this.getStatusColor(device.status))
      }
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 16 })
  }

  @Builder
  private buildDeviceReadings(device: IoTDevice) {
    const readings = this.deviceReadings.get(device.id) || []

    if (readings.length === 0) return

    Column() {
      Text('Recent Readings')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      ForEach(readings.slice(-5), (reading: DeviceReading) => {
        Row() {
          Text(reading.sensor)
            .fontSize(14)
            .flexGrow(1)

          Text(`${reading.value.toFixed(1)} ${reading.unit}`)
            .fontSize(14)
            .fontColor('#007AFF')
            .fontWeight(FontWeight.Medium)

          Text(new Date(reading.timestamp).toLocaleTimeString())
            .fontSize(12)
            .fontColor('#666666')
            .margin({ left: 8 })
        }
        .width('100%')
        .padding({ vertical: 4 })
        .border({
          width: { bottom: 1 },
          color: '#E0E0E0',
          style: BorderStyle.Solid
        })
      })
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 16 })
  }

  @Builder
  private buildDeviceControls(device: IoTDevice) {
    if (device.type !== 'actuator') return

    Column() {
      Text('Device Controls')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Row() {
        Button('Turn On')
          .onClick(() => this.sendDeviceCommand(device.id, 'turn_on'))
          .backgroundColor('#34C759')
          .fontColor('#FFFFFF')
          .flexGrow(1)
          .margin({ right: 8 })

        Button('Turn Off')
          .onClick(() => this.sendDeviceCommand(device.id, 'turn_off'))
          .backgroundColor('#FF3B30')
          .fontColor('#FFFFFF')
          .flexGrow(1)
      }
    }
    .alignItems(HorizontalAlign.Start)
  }

  private async loadDevices(): Promise<void> {
    this.devices = await this.iotManager.discoverDevices()
  }

  private async scanDevices(): Promise<void> {
    this.isScanning = true

    try {
      const newDevices = await this.iotManager.discoverDevices()
      this.devices = newDevices
    } finally {
      this.isScanning = false
    }
  }

  private async connectDevice(deviceId: string): Promise<void> {
    const success = await this.iotManager.connectDevice(deviceId)

    if (success) {
      // Update device status
      const device = this.devices.find(d => d.id === deviceId)
      if (device) {
        device.status = 'online'
      }

      // Start monitoring readings
      this.startReadingsUpdate(deviceId)
    }
  }

  private async sendDeviceCommand(deviceId: string, action: string): Promise<void> {
    const command: DeviceCommand = {
      deviceId,
      action,
      parameters: {},
      timestamp: Date.now(),
      status: 'pending'
    }

    await this.iotManager.sendCommand(command)
  }

  private startReadingsUpdate(deviceId: string): void {
    const updateInterval = setInterval(() => {
      const readings = this.iotManager.getDeviceReadings(deviceId, 10)
      this.deviceReadings.set(deviceId, readings)

      // Check if device is still connected
      const device = this.devices.find(d => d.id === deviceId)
      if (!device || device.status !== 'online') {
        clearInterval(updateInterval)
      }
    }, 1000)
  }

  private getDeviceIcon(type: string): string {
    const icons = {
      sensor: '📊',
      actuator: '🔌',
      gateway: '🌐',
      controller: '🎮'
    }
    return icons[type] || '📱'
  }

  private getStatusColor(status: string): string {
    const colors = {
      online: '#34C759',
      offline: '#8E8E93',
      connecting: '#FF9500',
      error: '#FF3B30'
    }
    return colors[status] || '#8E8E93'
  }

  private getStatusBackgroundColor(status: string): string {
    const colors = {
      online: '#E8F5E8',
      offline: '#F0F0F0',
      connecting: '#FFF3E0',
      error: '#FFEBEE'
    }
    return colors[status] || '#F0F0F0'
  }
}
```

## Conclusion

IoT device management in ArkUI provides:

- Multi-protocol device discovery and connection
- Real-time sensor data collection and visualization
- Remote device control and command execution
- Comprehensive device status monitoring
- Scalable architecture for large device networks

These capabilities enable developers to create powerful IoT applications for smart homes, industrial automation, and connected device ecosystems.
