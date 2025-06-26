# ArkUI Smart Home Integration

## Introduction

Smart home integration enables ArkUI applications to control and monitor IoT devices, home automation systems, and connected appliances. This guide covers device discovery, protocol communication, automation rules, and unified control interfaces.

## Device Management System

### IoT Device Manager

```typescript
interface IoTDevice {
  id: string;
  name: string;
  type: DeviceType;
  brand: string;
  model: string;
  status: DeviceStatus;
  capabilities: DeviceCapability[];
  location: string;
  lastUpdate: number;
  metadata: Record<string, any>;
}

enum DeviceType {
  LIGHT = "light",
  SWITCH = "switch",
  THERMOSTAT = "thermostat",
  CAMERA = "camera",
  SENSOR = "sensor",
  LOCK = "lock",
  FAN = "fan",
  SPEAKER = "speaker",
}

enum DeviceStatus {
  ONLINE = "online",
  OFFLINE = "offline",
  CONNECTING = "connecting",
  ERROR = "error",
}

interface DeviceCapability {
  name: string;
  type: "boolean" | "number" | "string" | "enum";
  readable: boolean;
  writable: boolean;
  min?: number;
  max?: number;
  options?: string[];
  unit?: string;
}

class SmartHomeManager {
  private devices = new Map<string, IoTDevice>();
  private protocols = new Map<string, DeviceProtocol>();
  private deviceListeners = new Set<(devices: IoTDevice[]) => void>();
  private automationRules: AutomationRule[] = [];

  constructor() {
    this.initializeProtocols();
    this.startDeviceDiscovery();
  }

  async discoverDevices(): Promise<IoTDevice[]> {
    const discoveredDevices: IoTDevice[] = [];

    for (const [name, protocol] of this.protocols) {
      try {
        const devices = await protocol.discover();
        discoveredDevices.push(...devices);
      } catch (error) {
        console.error(`Discovery failed for ${name}:`, error);
      }
    }

    discoveredDevices.forEach((device) => {
      this.devices.set(device.id, device);
    });

    this.notifyDeviceListeners();
    return discoveredDevices;
  }

  async controlDevice(
    deviceId: string,
    capability: string,
    value: any
  ): Promise<boolean> {
    const device = this.devices.get(deviceId);
    if (!device) return false;

    const protocol = this.getProtocolForDevice(device);
    if (!protocol) return false;

    try {
      const success = await protocol.sendCommand(device, capability, value);
      if (success) {
        this.updateDeviceState(deviceId, capability, value);
      }
      return success;
    } catch (error) {
      console.error(`Command failed for device ${deviceId}:`, error);
      return false;
    }
  }

  async getDeviceState(deviceId: string): Promise<Record<string, any> | null> {
    const device = this.devices.get(deviceId);
    if (!device) return null;

    const protocol = this.getProtocolForDevice(device);
    if (!protocol) return null;

    try {
      return await protocol.getState(device);
    } catch (error) {
      console.error(`State query failed for device ${deviceId}:`, error);
      return null;
    }
  }

  getDevices(): IoTDevice[] {
    return Array.from(this.devices.values());
  }

  getDevicesByType(type: DeviceType): IoTDevice[] {
    return this.getDevices().filter((device) => device.type === type);
  }

  getDevicesByLocation(location: string): IoTDevice[] {
    return this.getDevices().filter((device) => device.location === location);
  }

  onDevicesUpdated(listener: (devices: IoTDevice[]) => void): void {
    this.deviceListeners.add(listener);
  }

  private initializeProtocols(): void {
    this.protocols.set("zigbee", new ZigbeeProtocol());
    this.protocols.set("wifi", new WiFiProtocol());
    this.protocols.set("bluetooth", new BluetoothProtocol());
    this.protocols.set("matter", new MatterProtocol());
  }

  private getProtocolForDevice(device: IoTDevice): DeviceProtocol | null {
    const protocolName = device.metadata.protocol;
    return this.protocols.get(protocolName) || null;
  }

  private updateDeviceState(
    deviceId: string,
    capability: string,
    value: any
  ): void {
    const device = this.devices.get(deviceId);
    if (device) {
      device.metadata[capability] = value;
      device.lastUpdate = Date.now();
      this.notifyDeviceListeners();
    }
  }

  private startDeviceDiscovery(): void {
    // Periodic device discovery
    setInterval(() => {
      this.discoverDevices();
    }, 30000); // Every 30 seconds
  }

  private notifyDeviceListeners(): void {
    const devices = this.getDevices();
    this.deviceListeners.forEach((listener) => listener(devices));
  }
}

interface DeviceProtocol {
  discover(): Promise<IoTDevice[]>;
  sendCommand(
    device: IoTDevice,
    capability: string,
    value: any
  ): Promise<boolean>;
  getState(device: IoTDevice): Promise<Record<string, any>>;
}

class WiFiProtocol implements DeviceProtocol {
  async discover(): Promise<IoTDevice[]> {
    // Implement WiFi device discovery
    return [];
  }

  async sendCommand(
    device: IoTDevice,
    capability: string,
    value: any
  ): Promise<boolean> {
    // Implement WiFi command sending
    return true;
  }

  async getState(device: IoTDevice): Promise<Record<string, any>> {
    // Implement WiFi state querying
    return {};
  }
}

class ZigbeeProtocol implements DeviceProtocol {
  async discover(): Promise<IoTDevice[]> {
    // Implement Zigbee device discovery
    return [];
  }

  async sendCommand(
    device: IoTDevice,
    capability: string,
    value: any
  ): Promise<boolean> {
    // Implement Zigbee command sending
    return true;
  }

  async getState(device: IoTDevice): Promise<Record<string, any>> {
    // Implement Zigbee state querying
    return {};
  }
}

class BluetoothProtocol implements DeviceProtocol {
  async discover(): Promise<IoTDevice[]> {
    // Implement Bluetooth device discovery
    return [];
  }

  async sendCommand(
    device: IoTDevice,
    capability: string,
    value: any
  ): Promise<boolean> {
    // Implement Bluetooth command sending
    return true;
  }

  async getState(device: IoTDevice): Promise<Record<string, any>> {
    // Implement Bluetooth state querying
    return {};
  }
}

class MatterProtocol implements DeviceProtocol {
  async discover(): Promise<IoTDevice[]> {
    // Implement Matter/Thread device discovery
    return [];
  }

  async sendCommand(
    device: IoTDevice,
    capability: string,
    value: any
  ): Promise<boolean> {
    // Implement Matter command sending
    return true;
  }

  async getState(device: IoTDevice): Promise<Record<string, any>> {
    // Implement Matter state querying
    return {};
  }
}
```

## Smart Home Dashboard

### Home Control Interface

```typescript
@Component
struct SmartHomeDashboard {
  @State private devices: IoTDevice[] = []
  @State private selectedLocation: string = 'all'
  @State private selectedDeviceType: DeviceType | 'all' = 'all'
  @State private isDiscovering: boolean = false

  private smartHomeManager = new SmartHomeManager()

  build() {
    Column() {
      this.buildHeader()
      this.buildFilters()
      this.buildDeviceGrid()
      this.buildQuickActions()
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }

  aboutToAppear() {
    this.loadDevices()
    this.smartHomeManager.onDevicesUpdated((devices) => {
      this.devices = devices
    })
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Smart Home')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Button('Discover')
        .onClick(() => this.discoverDevices())
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .fontSize(14)
        .height(36)
        .margin({ right: 8 })

      if (this.isDiscovering) {
        LoadingProgress()
          .width(24)
          .height(24)
          .color('#007AFF')
      }
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .shadow({ radius: 2, color: '#00000010', offsetY: 1 })
  }

  @Builder
  private buildFilters() {
    Row() {
      Text('Location:')
        .fontSize(14)
        .margin({ right: 8 })

      Select([
        { value: 'all', text: 'All Rooms' },
        { value: 'living-room', text: 'Living Room' },
        { value: 'bedroom', text: 'Bedroom' },
        { value: 'kitchen', text: 'Kitchen' },
        { value: 'bathroom', text: 'Bathroom' }
      ])
        .selected(this.selectedLocation === 'all' ? 0 : 1)
        .value(this.selectedLocation)
        .onSelect((index, value) => {
          this.selectedLocation = value
        })
        .width(120)
        .margin({ right: 16 })

      Text('Type:')
        .fontSize(14)
        .margin({ right: 8 })

      Select([
        { value: 'all', text: 'All Types' },
        { value: DeviceType.LIGHT, text: 'Lights' },
        { value: DeviceType.SWITCH, text: 'Switches' },
        { value: DeviceType.THERMOSTAT, text: 'Climate' },
        { value: DeviceType.SENSOR, text: 'Sensors' }
      ])
        .selected(this.selectedDeviceType === 'all' ? 0 : 1)
        .value(this.selectedDeviceType as string)
        .onSelect((index, value) => {
          this.selectedDeviceType = value as DeviceType | 'all'
        })
        .width(120)
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .margin({ top: 1 })
  }

  @Builder
  private buildDeviceGrid() {
    Grid() {
      ForEach(this.getFilteredDevices(), (device: IoTDevice) => {
        GridItem() {
          this.buildDeviceCard(device)
        }
      })
    }
    .columnsTemplate('1fr 1fr')
    .rowsGap(12)
    .columnsGap(12)
    .padding(16)
  }

  @Builder
  private buildDeviceCard(device: IoTDevice) {
    Column() {
      Row() {
        Text(this.getDeviceIcon(device.type))
          .fontSize(24)
          .margin({ right: 12 })

        Column() {
          Text(device.name)
            .fontSize(16)
            .fontWeight(FontWeight.Bold)
            .textAlign(TextAlign.Start)

          Text(device.location)
            .fontSize(12)
            .fontColor('#666666')
            .textAlign(TextAlign.Start)
        }
        .alignItems(HorizontalAlign.Start)
        .flexGrow(1)

        Circle({ width: 8, height: 8 })
          .fill(this.getStatusColor(device.status))
      }
      .width('100%')
      .margin({ bottom: 16 })

      this.buildDeviceControls(device)
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .borderRadius(12)
    .shadow({ radius: 4, color: '#00000010', offsetY: 2 })
    .width('100%')
    .onClick(() => this.openDeviceDetails(device))
  }

  @Builder
  private buildDeviceControls(device: IoTDevice) {
    switch (device.type) {
      case DeviceType.LIGHT:
        this.buildLightControls(device)
        break
      case DeviceType.SWITCH:
        this.buildSwitchControls(device)
        break
      case DeviceType.THERMOSTAT:
        this.buildThermostatControls(device)
        break
      case DeviceType.SENSOR:
        this.buildSensorDisplay(device)
        break
      default:
        this.buildGenericControls(device)
    }
  }

  @Builder
  private buildLightControls(device: IoTDevice) {
    Column() {
      Row() {
        Text('Brightness')
          .fontSize(14)
          .flexGrow(1)

        Text(`${device.metadata.brightness || 0}%`)
          .fontSize(12)
          .fontColor('#666666')
      }
      .margin({ bottom: 8 })

      Slider({
        value: device.metadata.brightness || 0,
        min: 0,
        max: 100,
        step: 1
      })
        .trackColor('#E0E0E0')
        .selectedColor('#007AFF')
        .onChange((value) => {
          this.controlDevice(device.id, 'brightness', value)
        })

      Row() {
        Button(device.metadata.power ? 'ON' : 'OFF')
          .onClick(() => this.controlDevice(device.id, 'power', !device.metadata.power))
          .backgroundColor(device.metadata.power ? '#34C759' : '#8E8E93')
          .fontColor('#FFFFFF')
          .flexGrow(1)
          .margin({ right: 8 })

        Button('Color')
          .onClick(() => this.openColorPicker(device))
          .backgroundColor('#FF9500')
          .fontColor('#FFFFFF')
          .width(60)
      }
      .margin({ top: 12 })
    }
  }

  @Builder
  private buildSwitchControls(device: IoTDevice) {
    Toggle({ type: ToggleType.Switch, isOn: device.metadata.power || false })
      .onChange((isOn) => {
        this.controlDevice(device.id, 'power', isOn)
      })
      .selectedColor('#34C759')
      .width('100%')
  }

  @Builder
  private buildThermostatControls(device: IoTDevice) {
    Column() {
      Row() {
        Text('Target')
          .fontSize(14)
          .flexGrow(1)

        Text(`${device.metadata.targetTemperature || 20}°C`)
          .fontSize(16)
          .fontWeight(FontWeight.Bold)
      }
      .margin({ bottom: 8 })

      Row() {
        Button('-')
          .onClick(() => this.adjustTemperature(device, -1))
          .backgroundColor('#FF3B30')
          .fontColor('#FFFFFF')
          .width(40)
          .height(40)

        Text(`${device.metadata.currentTemperature || 22}°C`)
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)
          .textAlign(TextAlign.Center)

        Button('+')
          .onClick(() => this.adjustTemperature(device, 1))
          .backgroundColor('#007AFF')
          .fontColor('#FFFFFF')
          .width(40)
          .height(40)
      }
    }
  }

  @Builder
  private buildSensorDisplay(device: IoTDevice) {
    Column() {
      ForEach(Object.entries(device.metadata), ([key, value]) => {
        if (key !== 'protocol') {
          Row() {
            Text(key.charAt(0).toUpperCase() + key.slice(1))
              .fontSize(12)
              .fontColor('#666666')
              .flexGrow(1)

            Text(value.toString())
              .fontSize(14)
              .fontWeight(FontWeight.Bold)
          }
          .margin({ bottom: 4 })
        }
      })
    }
  }

  @Builder
  private buildGenericControls(device: IoTDevice) {
    Text('Device controls not available')
      .fontSize(12)
      .fontColor('#666666')
      .textAlign(TextAlign.Center)
  }

  @Builder
  private buildQuickActions() {
    Column() {
      Text('Quick Actions')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Button('All Lights On')
          .onClick(() => this.controlAllDevices(DeviceType.LIGHT, 'power', true))
          .backgroundColor('#34C759')
          .fontColor('#FFFFFF')
          .flexGrow(1)
          .margin({ right: 8 })

        Button('All Lights Off')
          .onClick(() => this.controlAllDevices(DeviceType.LIGHT, 'power', false))
          .backgroundColor('#8E8E93')
          .fontColor('#FFFFFF')
          .flexGrow(1)
          .margin({ right: 8 })

        Button('Away Mode')
          .onClick(() => this.activateAwayMode())
          .backgroundColor('#FF9500')
          .fontColor('#FFFFFF')
          .flexGrow(1)
      }
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .margin(16)
    .borderRadius(12)
    .shadow({ radius: 4, color: '#00000010', offsetY: 2 })
  }

  private getFilteredDevices(): IoTDevice[] {
    return this.devices.filter(device => {
      const locationMatch = this.selectedLocation === 'all' || device.location === this.selectedLocation
      const typeMatch = this.selectedDeviceType === 'all' || device.type === this.selectedDeviceType
      return locationMatch && typeMatch
    })
  }

  private getDeviceIcon(type: DeviceType): string {
    const icons = {
      [DeviceType.LIGHT]: '💡',
      [DeviceType.SWITCH]: '🔌',
      [DeviceType.THERMOSTAT]: '🌡️',
      [DeviceType.CAMERA]: '📷',
      [DeviceType.SENSOR]: '📊',
      [DeviceType.LOCK]: '🔒',
      [DeviceType.FAN]: '🌀',
      [DeviceType.SPEAKER]: '🔊'
    }
    return icons[type] || '📱'
  }

  private getStatusColor(status: DeviceStatus): string {
    switch (status) {
      case DeviceStatus.ONLINE: return '#34C759'
      case DeviceStatus.OFFLINE: return '#8E8E93'
      case DeviceStatus.CONNECTING: return '#FF9500'
      case DeviceStatus.ERROR: return '#FF3B30'
      default: return '#8E8E93'
    }
  }

  private async loadDevices(): Promise<void> {
    this.devices = this.smartHomeManager.getDevices()
  }

  private async discoverDevices(): Promise<void> {
    this.isDiscovering = true
    try {
      await this.smartHomeManager.discoverDevices()
    } finally {
      this.isDiscovering = false
    }
  }

  private async controlDevice(deviceId: string, capability: string, value: any): Promise<void> {
    await this.smartHomeManager.controlDevice(deviceId, capability, value)
  }

  private async controlAllDevices(type: DeviceType, capability: string, value: any): Promise<void> {
    const devices = this.smartHomeManager.getDevicesByType(type)
    for (const device of devices) {
      await this.smartHomeManager.controlDevice(device.id, capability, value)
    }
  }

  private adjustTemperature(device: IoTDevice, delta: number): void {
    const currentTarget = device.metadata.targetTemperature || 20
    const newTarget = Math.max(10, Math.min(30, currentTarget + delta))
    this.controlDevice(device.id, 'targetTemperature', newTarget)
  }

  private openDeviceDetails(device: IoTDevice): void {
    // Navigate to device details page
    console.log('Opening details for device:', device.id)
  }

  private openColorPicker(device: IoTDevice): void {
    // Open color picker dialog
    console.log('Opening color picker for device:', device.id)
  }

  private activateAwayMode(): void {
    // Turn off all lights, set thermostat to eco mode, etc.
    this.controlAllDevices(DeviceType.LIGHT, 'power', false)
    // Additional away mode logic
  }
}
```

## Automation System

### Rule Engine

```typescript
interface AutomationRule {
  id: string;
  name: string;
  description: string;
  enabled: boolean;
  trigger: AutomationTrigger;
  conditions: AutomationCondition[];
  actions: AutomationAction[];
  createdAt: number;
  lastTriggered?: number;
}

interface AutomationTrigger {
  type: "time" | "device" | "sensor" | "location" | "manual";
  config: Record<string, any>;
}

interface AutomationCondition {
  type: "device" | "time" | "sensor" | "weather";
  deviceId?: string;
  capability?: string;
  operator: "=" | "!=" | ">" | "<" | ">=" | "<=";
  value: any;
}

interface AutomationAction {
  type: "device" | "notification" | "delay";
  deviceId?: string;
  capability?: string;
  value?: any;
  message?: string;
  delay?: number;
}

class AutomationEngine {
  private rules: AutomationRule[] = [];
  private isRunning = false;
  private smartHomeManager: SmartHomeManager;

  constructor(smartHomeManager: SmartHomeManager) {
    this.smartHomeManager = smartHomeManager;
  }

  start(): void {
    this.isRunning = true;
    this.startRuleEvaluation();
  }

  stop(): void {
    this.isRunning = false;
  }

  addRule(rule: AutomationRule): void {
    this.rules.push(rule);
  }

  removeRule(ruleId: string): void {
    this.rules = this.rules.filter((rule) => rule.id !== ruleId);
  }

  updateRule(ruleId: string, updates: Partial<AutomationRule>): void {
    const ruleIndex = this.rules.findIndex((rule) => rule.id === ruleId);
    if (ruleIndex !== -1) {
      this.rules[ruleIndex] = { ...this.rules[ruleIndex], ...updates };
    }
  }

  getRules(): AutomationRule[] {
    return this.rules;
  }

  async executeRule(ruleId: string): Promise<boolean> {
    const rule = this.rules.find((r) => r.id === ruleId);
    if (!rule || !rule.enabled) return false;

    try {
      // Check conditions
      const conditionsMet = await this.evaluateConditions(rule.conditions);
      if (!conditionsMet) return false;

      // Execute actions
      await this.executeActions(rule.actions);

      // Update last triggered time
      rule.lastTriggered = Date.now();
      return true;
    } catch (error) {
      console.error(`Failed to execute rule ${ruleId}:`, error);
      return false;
    }
  }

  private startRuleEvaluation(): void {
    const evaluate = async () => {
      if (!this.isRunning) return;

      for (const rule of this.rules.filter((r) => r.enabled)) {
        await this.evaluateRuleTrigger(rule);
      }

      setTimeout(evaluate, 1000); // Check every second
    };

    evaluate();
  }

  private async evaluateRuleTrigger(rule: AutomationRule): Promise<void> {
    let shouldTrigger = false;

    switch (rule.trigger.type) {
      case "time":
        shouldTrigger = this.evaluateTimeTrigger(rule.trigger.config);
        break;
      case "device":
        shouldTrigger = await this.evaluateDeviceTrigger(rule.trigger.config);
        break;
      case "sensor":
        shouldTrigger = await this.evaluateSensorTrigger(rule.trigger.config);
        break;
    }

    if (shouldTrigger) {
      await this.executeRule(rule.id);
    }
  }

  private evaluateTimeTrigger(config: Record<string, any>): boolean {
    const now = new Date();
    const currentTime = now.getHours() * 60 + now.getMinutes();
    const triggerTime = config.hour * 60 + config.minute;

    // Check if it's the right time (within 1 minute)
    return Math.abs(currentTime - triggerTime) < 1;
  }

  private async evaluateDeviceTrigger(
    config: Record<string, any>
  ): Promise<boolean> {
    const deviceState = await this.smartHomeManager.getDeviceState(
      config.deviceId
    );
    if (!deviceState) return false;

    const currentValue = deviceState[config.capability];
    return this.compareValues(currentValue, config.operator, config.value);
  }

  private async evaluateSensorTrigger(
    config: Record<string, any>
  ): Promise<boolean> {
    const deviceState = await this.smartHomeManager.getDeviceState(
      config.sensorId
    );
    if (!deviceState) return false;

    const currentValue = deviceState[config.measurement];
    return this.compareValues(currentValue, config.operator, config.threshold);
  }

  private async evaluateConditions(
    conditions: AutomationCondition[]
  ): Promise<boolean> {
    for (const condition of conditions) {
      const result = await this.evaluateCondition(condition);
      if (!result) return false;
    }
    return true;
  }

  private async evaluateCondition(
    condition: AutomationCondition
  ): Promise<boolean> {
    switch (condition.type) {
      case "device":
        if (!condition.deviceId || !condition.capability) return false;
        const deviceState = await this.smartHomeManager.getDeviceState(
          condition.deviceId
        );
        if (!deviceState) return false;
        const deviceValue = deviceState[condition.capability];
        return this.compareValues(
          deviceValue,
          condition.operator,
          condition.value
        );

      case "time":
        const now = new Date();
        const currentHour = now.getHours();
        return this.compareValues(
          currentHour,
          condition.operator,
          condition.value
        );

      case "sensor":
        if (!condition.deviceId || !condition.capability) return false;
        const sensorState = await this.smartHomeManager.getDeviceState(
          condition.deviceId
        );
        if (!sensorState) return false;
        const sensorValue = sensorState[condition.capability];
        return this.compareValues(
          sensorValue,
          condition.operator,
          condition.value
        );

      default:
        return true;
    }
  }

  private async executeActions(actions: AutomationAction[]): Promise<void> {
    for (const action of actions) {
      await this.executeAction(action);
    }
  }

  private async executeAction(action: AutomationAction): Promise<void> {
    switch (action.type) {
      case "device":
        if (
          action.deviceId &&
          action.capability !== undefined &&
          action.value !== undefined
        ) {
          await this.smartHomeManager.controlDevice(
            action.deviceId,
            action.capability,
            action.value
          );
        }
        break;

      case "notification":
        if (action.message) {
          // Send notification
          console.log("Automation notification:", action.message);
        }
        break;

      case "delay":
        if (action.delay) {
          await new Promise((resolve) => setTimeout(resolve, action.delay));
        }
        break;
    }
  }

  private compareValues(value1: any, operator: string, value2: any): boolean {
    switch (operator) {
      case "=":
        return value1 === value2;
      case "!=":
        return value1 !== value2;
      case ">":
        return value1 > value2;
      case "<":
        return value1 < value2;
      case ">=":
        return value1 >= value2;
      case "<=":
        return value1 <= value2;
      default:
        return false;
    }
  }
}
```

## Conclusion

Smart home integration in ArkUI applications provides:

- Comprehensive IoT device discovery and management across multiple protocols
- Unified control interface for diverse smart home devices
- Advanced automation engine with rules, conditions, and actions
- Real-time device monitoring and status updates
- Location-based device organization and filtering
- Quick action capabilities for common scenarios

These features enable developers to create sophisticated smart home applications that provide seamless control and automation of connected devices and systems.
