# ArkUI Environmental Monitoring

## Introduction

Environmental Monitoring in ArkUI enables real-time tracking of environmental conditions, sensor data processing, and automated alerting systems. This guide covers sensor integration, data analytics, and environmental management.

## Environmental Monitoring Framework

```typescript
interface EnvironmentalSensor {
  id: string;
  type: SensorType;
  location: Location;
  status: SensorStatus;
  lastReading: SensorReading;
  calibration: CalibrationData;
  alertThresholds: AlertThreshold[];
}

type SensorType =
  | "temperature"
  | "humidity"
  | "air_quality"
  | "noise"
  | "light"
  | "pressure"
  | "co2"
  | "radiation";
type SensorStatus = "active" | "inactive" | "error" | "maintenance";

interface SensorReading {
  value: number;
  unit: string;
  timestamp: number;
  quality: number;
  confidence: number;
}

class EnvironmentalMonitoringSystem {
  private sensors = new Map<string, EnvironmentalSensor>();
  private dataStore = new EnvironmentalDataStore();
  private alertSystem = new AlertSystem();
  private analytics = new EnvironmentalAnalytics();

  async addSensor(config: SensorConfig): Promise<string> {
    const sensorId = `sensor_${Date.now()}`;

    const sensor: EnvironmentalSensor = {
      id: sensorId,
      type: config.type,
      location: config.location,
      status: "active",
      lastReading: {
        value: 0,
        unit: this.getUnitForSensorType(config.type),
        timestamp: Date.now(),
        quality: 1.0,
        confidence: 1.0,
      },
      calibration: {
        offset: 0,
        scale: 1.0,
        lastCalibrated: Date.now(),
      },
      alertThresholds: config.alertThresholds || [],
    };

    this.sensors.set(sensorId, sensor);
    await this.startSensorMonitoring(sensor);

    return sensorId;
  }

  async collectReading(sensorId: string): Promise<SensorReading | null> {
    const sensor = this.sensors.get(sensorId);
    if (!sensor || sensor.status !== "active") return null;

    try {
      const rawReading = await this.readSensorValue(sensor);
      const calibratedReading = this.applyCalibratedReading(
        rawReading,
        sensor.calibration
      );

      const reading: SensorReading = {
        value: calibratedReading,
        unit: sensor.lastReading.unit,
        timestamp: Date.now(),
        quality: this.assessReadingQuality(calibratedReading, sensor),
        confidence: this.calculateConfidence(sensor),
      };

      sensor.lastReading = reading;
      await this.dataStore.storeReading(sensorId, reading);

      // Check for alerts
      this.checkAlertThresholds(sensor, reading);

      return reading;
    } catch (error) {
      console.error(
        `Failed to collect reading from sensor ${sensorId}:`,
        error
      );
      sensor.status = "error";
      return null;
    }
  }

  async getEnvironmentalConditions(
    location?: Location
  ): Promise<EnvironmentalConditions> {
    const relevantSensors = location
      ? this.getSensorsNearLocation(location, 1000) // 1km radius
      : Array.from(this.sensors.values());

    const conditions: EnvironmentalConditions = {
      timestamp: Date.now(),
      location: location || { latitude: 0, longitude: 0, altitude: 0 },
      readings: new Map(),
      overallQuality: "good",
      alerts: [],
    };

    for (const sensor of relevantSensors) {
      if (sensor.status === "active" && sensor.lastReading) {
        conditions.readings.set(sensor.type, sensor.lastReading);
      }
    }

    // Calculate overall environmental quality
    conditions.overallQuality = this.calculateOverallQuality(
      conditions.readings
    );

    // Get active alerts
    conditions.alerts = await this.alertSystem.getActiveAlerts(location);

    return conditions;
  }

  async generateEnvironmentalReport(
    timeRange: TimeRange,
    location?: Location
  ): Promise<EnvironmentalReport> {
    const historicalData = await this.dataStore.getHistoricalData(
      timeRange,
      location
    );

    return this.analytics.generateReport(historicalData, {
      includeAverages: true,
      includeTrends: true,
      includeAnomalies: true,
      includeForecasts: true,
    });
  }

  async predictEnvironmentalConditions(
    hours: number,
    location?: Location
  ): Promise<EnvironmentalForecast> {
    const historicalData = await this.dataStore.getHistoricalData(
      {
        start: Date.now() - 30 * 24 * 60 * 60 * 1000, // 30 days
        end: Date.now(),
      },
      location
    );

    return this.analytics.generateForecast(historicalData, hours);
  }

  calibrateSensor(sensorId: string, referenceValue: number): boolean {
    const sensor = this.sensors.get(sensorId);
    if (!sensor) return false;

    const currentReading = sensor.lastReading.value;
    const offset = referenceValue - currentReading;

    sensor.calibration = {
      offset,
      scale: sensor.calibration.scale,
      lastCalibrated: Date.now(),
    };

    return true;
  }

  setAlertThreshold(sensorId: string, threshold: AlertThreshold): boolean {
    const sensor = this.sensors.get(sensorId);
    if (!sensor) return false;

    const existingIndex = sensor.alertThresholds.findIndex(
      (t) => t.type === threshold.type
    );
    if (existingIndex >= 0) {
      sensor.alertThresholds[existingIndex] = threshold;
    } else {
      sensor.alertThresholds.push(threshold);
    }

    return true;
  }

  private async startSensorMonitoring(
    sensor: EnvironmentalSensor
  ): Promise<void> {
    // Start periodic data collection
    setInterval(async () => {
      if (sensor.status === "active") {
        await this.collectReading(sensor.id);
      }
    }, this.getReadingInterval(sensor.type));
  }

  private async readSensorValue(sensor: EnvironmentalSensor): Promise<number> {
    // Simulate sensor reading
    const baseValues = {
      temperature: 22.0,
      humidity: 45.0,
      air_quality: 50.0,
      noise: 40.0,
      light: 300.0,
      pressure: 1013.25,
      co2: 400.0,
      radiation: 0.1,
    };

    const baseValue = baseValues[sensor.type] || 0;
    const variation = baseValue * 0.1 * (Math.random() - 0.5);

    return baseValue + variation;
  }

  private applyCalibratedReading(
    rawValue: number,
    calibration: CalibrationData
  ): number {
    return (rawValue + calibration.offset) * calibration.scale;
  }

  private assessReadingQuality(
    value: number,
    sensor: EnvironmentalSensor
  ): number {
    // Simple quality assessment based on value stability
    const readings = this.dataStore.getRecentReadings(sensor.id, 10);
    if (readings.length < 2) return 1.0;

    const variance = this.calculateVariance(readings.map((r) => r.value));
    const expectedVariance = this.getExpectedVariance(sensor.type);

    return Math.max(0, 1 - variance / expectedVariance);
  }

  private calculateConfidence(sensor: EnvironmentalSensor): number {
    const timeSinceCalibration = Date.now() - sensor.calibration.lastCalibrated;
    const maxAge = 30 * 24 * 60 * 60 * 1000; // 30 days

    return Math.max(0.5, 1 - timeSinceCalibration / maxAge);
  }

  private checkAlertThresholds(
    sensor: EnvironmentalSensor,
    reading: SensorReading
  ): void {
    for (const threshold of sensor.alertThresholds) {
      const isTriggered = this.isThresholdTriggered(reading.value, threshold);

      if (isTriggered) {
        this.alertSystem.triggerAlert({
          sensorId: sensor.id,
          type: threshold.type,
          value: reading.value,
          threshold: threshold.value,
          location: sensor.location,
          timestamp: reading.timestamp,
        });
      }
    }
  }

  private isThresholdTriggered(
    value: number,
    threshold: AlertThreshold
  ): boolean {
    switch (threshold.type) {
      case "above":
        return value > threshold.value;
      case "below":
        return value < threshold.value;
      case "outside_range":
        return value < threshold.min! || value > threshold.max!;
      default:
        return false;
    }
  }

  private getUnitForSensorType(type: SensorType): string {
    const units = {
      temperature: "°C",
      humidity: "%",
      air_quality: "AQI",
      noise: "dB",
      light: "lux",
      pressure: "hPa",
      co2: "ppm",
      radiation: "μSv/h",
    };
    return units[type] || "";
  }

  private getReadingInterval(type: SensorType): number {
    // Reading intervals in milliseconds
    const intervals = {
      temperature: 30000, // 30 seconds
      humidity: 30000, // 30 seconds
      air_quality: 60000, // 1 minute
      noise: 10000, // 10 seconds
      light: 60000, // 1 minute
      pressure: 30000, // 30 seconds
      co2: 60000, // 1 minute
      radiation: 300000, // 5 minutes
    };
    return intervals[type] || 60000;
  }

  getSensors(): EnvironmentalSensor[] {
    return Array.from(this.sensors.values());
  }

  getSensor(sensorId: string): EnvironmentalSensor | undefined {
    return this.sensors.get(sensorId);
  }
}

interface EnvironmentalConditions {
  timestamp: number;
  location: Location;
  readings: Map<SensorType, SensorReading>;
  overallQuality: "excellent" | "good" | "moderate" | "poor" | "hazardous";
  alerts: Alert[];
}

interface AlertThreshold {
  type: "above" | "below" | "outside_range";
  value: number;
  min?: number;
  max?: number;
  enabled: boolean;
}

interface Location {
  latitude: number;
  longitude: number;
  altitude: number;
}

interface CalibrationData {
  offset: number;
  scale: number;
  lastCalibrated: number;
}
```

## Environmental Dashboard Component

```typescript
@Component
struct EnvironmentalDashboard {
  @State private sensors: EnvironmentalSensor[] = []
  @State private currentConditions: EnvironmentalConditions | null = null
  @State private activeAlerts: Alert[] = []
  @State private selectedSensor: string = ''

  private monitoringSystem = new EnvironmentalMonitoringSystem()

  aboutToAppear() {
    this.initializeMonitoring()
    this.startRealTimeUpdates()
  }

  build() {
    Column() {
      this.buildHeader()
      this.buildConditionsOverview()
      this.buildSensorGrid()
      this.buildAlertsPanel()
    }
    .width('100%')
    .padding(16)
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Environmental Monitoring')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Text(`${this.sensors.filter(s => s.status === 'active').length}/${this.sensors.length} Active`)
        .fontSize(14)
        .fontColor('#34C759')
        .backgroundColor('#E8F5E8')
        .padding({ horizontal: 8, vertical: 4 })
        .borderRadius(4)
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildConditionsOverview() {
    if (!this.currentConditions) return

    Column() {
      Text('Current Conditions')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Text('Overall Quality:')
          .fontSize(14)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Text(this.currentConditions.overallQuality.toUpperCase())
          .fontSize(14)
          .fontColor(this.getQualityColor(this.currentConditions.overallQuality))
          .backgroundColor(this.getQualityBackgroundColor(this.currentConditions.overallQuality))
          .padding({ horizontal: 8, vertical: 4 })
          .borderRadius(4)
      }
      .margin({ bottom: 16 })

      Grid() {
        ForEach(Array.from(this.currentConditions.readings.entries()), ([type, reading]: [SensorType, SensorReading]) => {
          GridItem() {
            this.buildConditionCard(type, reading)
          }
        })
      }
      .columnsTemplate('1fr 1fr')
      .columnsGap(12)
      .rowsGap(12)
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 20 })
  }

  @Builder
  private buildConditionCard(type: SensorType, reading: SensorReading) {
    Column() {
      Text(this.formatSensorType(type))
        .fontSize(14)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 4 })

      Text(`${reading.value.toFixed(1)} ${reading.unit}`)
        .fontSize(18)
        .fontColor('#007AFF')
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 4 })

      Progress({
        value: reading.quality * 100,
        total: 100
      })
        .color('#34C759')
        .backgroundColor('#E0E0E0')
        .style({ strokeWidth: 3 })

      Text(`Quality: ${(reading.quality * 100).toFixed(0)}%`)
        .fontSize(10)
        .fontColor('#666666')
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .alignItems(HorizontalAlign.Center)
  }

  @Builder
  private buildSensorGrid() {
    Column() {
      Text('Sensor Status')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Grid() {
        ForEach(this.sensors, (sensor: EnvironmentalSensor) => {
          GridItem() {
            this.buildSensorCard(sensor)
          }
        })
      }
      .columnsTemplate('1fr 1fr')
      .columnsGap(12)
      .rowsGap(12)
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 20 })
  }

  @Builder
  private buildSensorCard(sensor: EnvironmentalSensor) {
    Column() {
      Row() {
        Text(this.formatSensorType(sensor.type))
          .fontSize(14)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Text(sensor.status)
          .fontSize(10)
          .fontColor(this.getStatusColor(sensor.status))
          .backgroundColor(this.getStatusBackgroundColor(sensor.status))
          .padding({ horizontal: 6, vertical: 2 })
          .borderRadius(3)
      }
      .margin({ bottom: 8 })

      Text(`${sensor.lastReading.value.toFixed(1)} ${sensor.lastReading.unit}`)
        .fontSize(16)
        .fontColor('#007AFF')
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 4 })

      Text(`Location: ${sensor.location.latitude.toFixed(3)}, ${sensor.location.longitude.toFixed(3)}`)
        .fontSize(10)
        .fontColor('#666666')
        .margin({ bottom: 4 })

      Text(`Updated: ${this.formatTime(sensor.lastReading.timestamp)}`)
        .fontSize(10)
        .fontColor('#999999')
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .onClick(() => this.selectedSensor = sensor.id)
    .border({
      width: this.selectedSensor === sensor.id ? 2 : 1,
      color: this.selectedSensor === sensor.id ? '#007AFF' : '#E0E0E0'
    })
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildAlertsPanel() {
    if (this.activeAlerts.length === 0) return

    Column() {
      Text('Active Alerts')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      ForEach(this.activeAlerts, (alert: Alert) => {
        this.buildAlertItem(alert)
      })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildAlertItem(alert: Alert) {
    Row() {
      Column() {
        Text(alert.title)
          .fontSize(14)
          .fontWeight(FontWeight.Bold)
          .fontColor('#FF3B30')

        Text(alert.message)
          .fontSize(12)
          .fontColor('#666666')

        Text(this.formatTime(alert.timestamp))
          .fontSize(10)
          .fontColor('#999999')
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)

      Text(alert.severity)
        .fontSize(10)
        .fontColor('#FFFFFF')
        .backgroundColor(this.getSeverityColor(alert.severity))
        .padding({ horizontal: 6, vertical: 2 })
        .borderRadius(3)
    }
    .width('100%')
    .padding(10)
    .backgroundColor('#FFEBEE')
    .borderRadius(6)
    .margin({ bottom: 6 })
  }

  private async initializeMonitoring(): Promise<void> {
    // Add sample sensors
    const sensorConfigs: SensorConfig[] = [
      {
        type: 'temperature',
        location: { latitude: 40.7128, longitude: -74.0060, altitude: 10 },
        alertThresholds: [
          { type: 'above', value: 30, enabled: true },
          { type: 'below', value: 0, enabled: true }
        ]
      },
      {
        type: 'humidity',
        location: { latitude: 40.7128, longitude: -74.0060, altitude: 10 },
        alertThresholds: [
          { type: 'above', value: 80, enabled: true }
        ]
      },
      {
        type: 'air_quality',
        location: { latitude: 40.7128, longitude: -74.0060, altitude: 10 },
        alertThresholds: [
          { type: 'above', value: 100, enabled: true }
        ]
      }
    ]

    for (const config of sensorConfigs) {
      await this.monitoringSystem.addSensor(config)
    }

    this.sensors = this.monitoringSystem.getSensors()
  }

  private startRealTimeUpdates(): void {
    setInterval(async () => {
      this.currentConditions = await this.monitoringSystem.getEnvironmentalConditions()
      this.sensors = this.monitoringSystem.getSensors()

      // Update active alerts
      if (this.currentConditions) {
        this.activeAlerts = this.currentConditions.alerts
      }
    }, 5000)
  }

  private formatSensorType(type: SensorType): string {
    const names = {
      temperature: 'Temperature',
      humidity: 'Humidity',
      air_quality: 'Air Quality',
      noise: 'Noise Level',
      light: 'Light Level',
      pressure: 'Pressure',
      co2: 'CO2',
      radiation: 'Radiation'
    }
    return names[type] || type
  }

  private formatTime(timestamp: number): string {
    return new Date(timestamp).toLocaleTimeString()
  }

  private getQualityColor(quality: string): string {
    const colors = {
      excellent: '#34C759',
      good: '#34C759',
      moderate: '#FF9500',
      poor: '#FF3B30',
      hazardous: '#FF3B30'
    }
    return colors[quality] || '#8E8E93'
  }

  private getQualityBackgroundColor(quality: string): string {
    const colors = {
      excellent: '#E8F5E8',
      good: '#E8F5E8',
      moderate: '#FFF3E0',
      poor: '#FFEBEE',
      hazardous: '#FFEBEE'
    }
    return colors[quality] || '#F0F0F0'
  }

  private getStatusColor(status: string): string {
    const colors = {
      active: '#34C759',
      inactive: '#8E8E93',
      error: '#FF3B30',
      maintenance: '#FF9500'
    }
    return colors[status] || '#8E8E93'
  }

  private getStatusBackgroundColor(status: string): string {
    const colors = {
      active: '#E8F5E8',
      inactive: '#F0F0F0',
      error: '#FFEBEE',
      maintenance: '#FFF3E0'
    }
    return colors[status] || '#F0F0F0'
  }

  private getSeverityColor(severity: string): string {
    const colors = {
      low: '#34C759',
      medium: '#FF9500',
      high: '#FF3B30',
      critical: '#FF3B30'
    }
    return colors[severity] || '#8E8E93'
  }
}

interface SensorConfig {
  type: SensorType
  location: Location
  alertThresholds?: AlertThreshold[]
}

interface Alert {
  id: string
  title: string
  message: string
  severity: 'low' | 'medium' | 'high' | 'critical'
  timestamp: number
  sensorId: string
}
```

## Conclusion

Environmental Monitoring in ArkUI provides:

- Real-time sensor data collection and processing
- Automated alert systems and threshold monitoring
- Environmental condition analysis and reporting
- Predictive analytics and forecasting
- Comprehensive sensor management and calibration

These capabilities enable effective environmental monitoring for smart cities, industrial facilities, and research applications.
