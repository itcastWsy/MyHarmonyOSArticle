# ArkUI Real-Time Analytics

## Introduction

Real-Time Analytics in ArkUI enables instant data processing, live dashboards, streaming analytics, and immediate insights. This guide covers event streaming, real-time aggregation, live visualizations, and performance monitoring.

## Real-Time Analytics Framework

```typescript
interface AnalyticsEvent {
  id: string;
  type: EventType;
  timestamp: number;
  userId?: string;
  sessionId: string;
  data: Record<string, any>;
  metadata: EventMetadata;
}

type EventType =
  | "page_view"
  | "user_action"
  | "performance"
  | "error"
  | "custom";

interface AnalyticsMetric {
  name: string;
  value: number;
  unit: string;
  timestamp: number;
  dimensions: Record<string, string>;
  tags: string[];
}

class RealTimeAnalyticsEngine {
  private eventStream = new EventStream();
  private metricStore = new MetricStore();
  private aggregator = new RealTimeAggregator();
  private alertManager = new AlertManager();

  async trackEvent(event: Partial<AnalyticsEvent>): Promise<void> {
    const completeEvent: AnalyticsEvent = {
      id: this.generateEventId(),
      timestamp: Date.now(),
      sessionId: this.getCurrentSession(),
      metadata: await this.enrichMetadata(),
      ...event,
    } as AnalyticsEvent;

    await this.eventStream.publish(completeEvent);
    await this.processEventRealTime(completeEvent);
  }

  async trackMetric(
    name: string,
    value: number,
    dimensions: Record<string, string> = {}
  ): Promise<void> {
    const metric: AnalyticsMetric = {
      name,
      value,
      unit: this.getMetricUnit(name),
      timestamp: Date.now(),
      dimensions,
      tags: [],
    };

    await this.metricStore.store(metric);
    await this.aggregator.processMetric(metric);
  }

  async getLiveStatistics(): Promise<LiveStatistics> {
    const now = Date.now();
    const lastMinute = { start: now - 60000, end: now };
    const lastHour = { start: now - 3600000, end: now };

    return {
      timestamp: now,
      lastMinute: await this.getMetrics(lastMinute),
      lastHour: await this.getMetrics(lastHour),
      activeUsers: await this.getActiveUserCount(),
      errorRate: await this.getErrorRate(lastMinute),
    };
  }

  async setAlert(condition: AlertCondition): Promise<string> {
    return this.alertManager.createAlert(condition);
  }

  private async processEventRealTime(event: AnalyticsEvent): Promise<void> {
    await this.updateCounters(event);
    await this.detectAnomalies(event);
    await this.alertManager.checkEvent(event);
  }

  private async updateCounters(event: AnalyticsEvent): Promise<void> {
    await this.trackMetric(`events.${event.type}`, 1);

    if (event.userId) {
      await this.trackMetric("active_users", 1, { user_id: event.userId });
    }

    if (event.type === "page_view" && event.data.page) {
      await this.trackMetric("page_views", 1, { page: event.data.page });
    }
  }

  private async detectAnomalies(event: AnalyticsEvent): Promise<void> {
    if (event.type === "error") {
      const errorRate = await this.getRecentErrorRate();
      if (errorRate > 0.05) {
        await this.alertManager.triggerAlert({
          type: "anomaly",
          severity: "high",
          message: `High error rate: ${(errorRate * 100).toFixed(2)}%`,
        });
      }
    }
  }

  private generateEventId(): string {
    return `evt_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private getCurrentSession(): string {
    return `sess_${Date.now()}`;
  }

  private async enrichMetadata(): Promise<EventMetadata> {
    return {
      source: "arkui-app",
      version: "1.0.0",
      environment: "production",
    };
  }

  private getMetricUnit(name: string): string {
    const units: Record<string, string> = {
      "performance.loadTime": "ms",
      "memory.usage": "bytes",
      "cpu.usage": "percent",
    };
    return units[name] || "count";
  }
}

class EventStream {
  private listeners: ((event: AnalyticsEvent) => void)[] = [];
  private buffer: AnalyticsEvent[] = [];

  async publish(event: AnalyticsEvent): Promise<void> {
    this.buffer.push(event);
    if (this.buffer.length > 1000) {
      this.buffer.shift();
    }

    this.listeners.forEach((listener) => {
      try {
        listener(event);
      } catch (error) {
        console.error("Error in event listener:", error);
      }
    });
  }

  onEvent(listener: (event: AnalyticsEvent) => void): void {
    this.listeners.push(listener);
  }
}

class MetricStore {
  private metrics: AnalyticsMetric[] = [];

  async store(metric: AnalyticsMetric): Promise<void> {
    this.metrics.push(metric);
    if (this.metrics.length > 10000) {
      this.metrics.shift();
    }
  }

  async getMetrics(timeRange: TimeRange): Promise<AnalyticsMetric[]> {
    return this.metrics.filter(
      (metric) =>
        metric.timestamp >= timeRange.start && metric.timestamp <= timeRange.end
    );
  }
}

class RealTimeAggregator {
  private counters = new Map<string, number>();
  private timeSeriesData = new Map<
    string,
    Array<{ timestamp: number; value: number }>
  >();

  async processMetric(metric: AnalyticsMetric): Promise<void> {
    const key = `${metric.name}:${JSON.stringify(metric.dimensions)}`;
    this.counters.set(key, (this.counters.get(key) || 0) + metric.value);

    if (!this.timeSeriesData.has(metric.name)) {
      this.timeSeriesData.set(metric.name, []);
    }

    const timeSeries = this.timeSeriesData.get(metric.name)!;
    timeSeries.push({ timestamp: metric.timestamp, value: metric.value });

    const oneHourAgo = Date.now() - 3600000;
    this.timeSeriesData.set(
      metric.name,
      timeSeries.filter((point) => point.timestamp > oneHourAgo)
    );
  }

  async getCount(metricName: string, timeRange: TimeRange): Promise<number> {
    const timeSeries = this.timeSeriesData.get(metricName);
    if (!timeSeries) return 0;

    return timeSeries
      .filter(
        (point) =>
          point.timestamp >= timeRange.start && point.timestamp <= timeRange.end
      )
      .reduce((sum, point) => sum + point.value, 0);
  }
}

class AlertManager {
  private alerts = new Map<string, AlertCondition>();
  private alertHandlers: ((alert: Alert) => void)[] = [];

  async createAlert(condition: AlertCondition): Promise<string> {
    const alertId = `alert_${Date.now()}`;
    this.alerts.set(alertId, condition);
    return alertId;
  }

  async checkEvent(event: AnalyticsEvent): Promise<void> {
    // Implementation for checking events against alert conditions
  }

  async triggerAlert(alert: Alert): Promise<void> {
    this.alertHandlers.forEach((handler) => {
      try {
        handler(alert);
      } catch (error) {
        console.error("Error in alert handler:", error);
      }
    });
  }

  onAlert(handler: (alert: Alert) => void): void {
    this.alertHandlers.push(handler);
  }
}

interface EventMetadata {
  source: string;
  version: string;
  environment: string;
}

interface TimeRange {
  start: number;
  end: number;
}

interface LiveStatistics {
  timestamp: number;
  lastMinute: MetricsSummary;
  lastHour: MetricsSummary;
  activeUsers: number;
  errorRate: number;
}

interface MetricsSummary {
  totalEvents: number;
  activeUsers: number;
  metrics: Record<string, number>;
}

interface AlertCondition {
  metric: string;
  operator: "gt" | "lt" | "eq";
  threshold: number;
  timeWindow: number;
}

interface Alert {
  type: string;
  severity: "low" | "medium" | "high" | "critical";
  message: string;
}
```

## ArkUI Analytics Component

```typescript
@Component
export struct RealTimeAnalyticsDashboard {
  @State private analytics: RealTimeAnalyticsEngine = new RealTimeAnalyticsEngine()
  @State private liveStats: LiveStatistics | null = null
  @State private isRunning: boolean = false
  @State private recentEvents: AnalyticsEvent[] = []
  @State private alerts: Alert[] = []

  aboutToAppear() {
    this.startRealTimeUpdates()
  }

  build() {
    Scroll() {
      Column({ space: 16 }) {
        this.buildHeader()
        this.buildLiveMetrics()
        this.buildEventTracking()
        this.buildAlertsSection()
      }
      .width('100%')
      .padding(16)
    }
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Real-Time Analytics')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Spacer()

      Button(this.isRunning ? 'Stop' : 'Start')
        .backgroundColor(this.isRunning ? '#FF3B30' : '#34C759')
        .onClick(() => {
          this.isRunning = !this.isRunning
        })
    }
    .width('100%')
  }

  @Builder
  private buildLiveMetrics() {
    if (!this.liveStats) return

    Column() {
      Text('Live Metrics')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        this.buildMetricCard('Active Users', this.liveStats.activeUsers.toString(), '#34C759')
        this.buildMetricCard('Events/min', this.liveStats.lastMinute.totalEvents.toString(), '#007AFF')
        this.buildMetricCard('Error Rate', `${(this.liveStats.errorRate * 100).toFixed(1)}%`, '#FF3B30')
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)
    }
  }

  @Builder
  private buildMetricCard(title: string, value: string, color: string) {
    Column() {
      Text(value)
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .fontColor(color)

      Text(title)
        .fontSize(12)
        .fontColor('#666666')
    }
    .width('30%')
    .padding(12)
    .backgroundColor('#F8F9FA')
    .borderRadius(8)
    .alignItems(HorizontalAlign.Center)
  }

  @Builder
  private buildEventTracking() {
    Column() {
      Text('Event Tracking')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Button('Page View')
          .flexGrow(1)
          .margin({ right: 4 })
          .onClick(async () => {
            await this.analytics.trackEvent({
              type: 'page_view',
              data: { page: '/dashboard' }
            })
          })

        Button('User Action')
          .flexGrow(1)
          .margin({ horizontal: 4 })
          .onClick(async () => {
            await this.analytics.trackEvent({
              type: 'user_action',
              data: { action: 'click', target: 'button' }
            })
          })

        Button('Error')
          .flexGrow(1)
          .margin({ left: 4 })
          .backgroundColor('#FF3B30')
          .onClick(async () => {
            await this.analytics.trackEvent({
              type: 'error',
              data: { message: 'Test error' }
            })
          })
      }
      .width('100%')
    }
  }

  @Builder
  private buildAlertsSection() {
    Column() {
      Text('Active Alerts')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      if (this.alerts.length === 0) {
        Text('No active alerts')
          .fontSize(14)
          .fontColor('#8E8E93')
          .textAlign(TextAlign.Center)
      } else {
        ForEach(this.alerts, (alert: Alert) => {
          this.buildAlertItem(alert)
        })
      }
    }
  }

  @Builder
  private buildAlertItem(alert: Alert) {
    Row() {
      Text(`${alert.type.toUpperCase()}: ${alert.message}`)
        .fontSize(14)
        .fontColor('#FFFFFF')
        .flexGrow(1)

      Text('×')
        .fontSize(18)
        .fontColor('#FFFFFF')
        .onClick(() => {
          const index = this.alerts.indexOf(alert)
          if (index >= 0) {
            this.alerts.splice(index, 1)
          }
        })
    }
    .width('100%')
    .padding(12)
    .backgroundColor(this.getSeverityColor(alert.severity))
    .borderRadius(8)
    .margin({ bottom: 8 })
  }

  private startRealTimeUpdates(): void {
    setInterval(async () => {
      if (this.isRunning) {
        this.liveStats = await this.analytics.getLiveStatistics()
      }
    }, 2000)
  }

  private getSeverityColor(severity: string): string {
    const colors = {
      low: '#34C759',
      medium: '#FF9500',
      high: '#FF3B30',
      critical: '#AF52DE',
    }
    return colors[severity] || '#8E8E93'
  }
}
```

## Advanced Features

```typescript
class AdvancedAnalytics extends RealTimeAnalyticsEngine {
  async trackFunnel(steps: FunnelStep[]): Promise<FunnelAnalysis> {
    const analysis: FunnelAnalysis = {
      steps: [],
      conversionRate: 0,
      dropOffPoints: [],
    };

    for (let i = 0; i < steps.length; i++) {
      const step = steps[i];
      const count = await this.getEventCount(step.eventType, step.filters);

      analysis.steps.push({
        name: step.name,
        count,
        conversionRate: i > 0 ? count / analysis.steps[0].count : 1,
      });

      if (i > 0 && analysis.steps[i].conversionRate < 0.5) {
        analysis.dropOffPoints.push(i);
      }
    }

    analysis.conversionRate =
      analysis.steps.length > 0
        ? analysis.steps[analysis.steps.length - 1].conversionRate
        : 0;

    return analysis;
  }

  async trackCohort(
    users: string[],
    timeframe: string
  ): Promise<CohortAnalysis> {
    const cohortData: CohortAnalysis = {
      users: users.length,
      retentionRates: [],
      timeframe,
    };

    // Calculate retention for each time period
    for (let period = 1; period <= 12; period++) {
      const retained = await this.getRetainedUsers(users, period);
      cohortData.retentionRates.push({
        period,
        rate: retained / users.length,
      });
    }

    return cohortData;
  }

  async performABTest(
    testName: string,
    variants: ABTestVariant[]
  ): Promise<ABTestResult> {
    const results: ABTestResult = {
      testName,
      variants: [],
      winner: null,
      confidence: 0,
    };

    for (const variant of variants) {
      const metrics = await this.getVariantMetrics(variant.id);
      results.variants.push({
        ...variant,
        metrics,
        conversionRate: metrics.conversions / metrics.users,
      });
    }

    // Determine statistical significance and winner
    results.winner = this.calculateWinner(results.variants);
    results.confidence = this.calculateConfidence(results.variants);

    return results;
  }
}

interface FunnelStep {
  name: string;
  eventType: string;
  filters?: Record<string, any>;
}

interface FunnelAnalysis {
  steps: Array<{
    name: string;
    count: number;
    conversionRate: number;
  }>;
  conversionRate: number;
  dropOffPoints: number[];
}

interface CohortAnalysis {
  users: number;
  retentionRates: Array<{
    period: number;
    rate: number;
  }>;
  timeframe: string;
}

interface ABTestVariant {
  id: string;
  name: string;
  traffic: number;
}

interface ABTestResult {
  testName: string;
  variants: Array<
    ABTestVariant & {
      metrics: VariantMetrics;
      conversionRate: number;
    }
  >;
  winner: string | null;
  confidence: number;
}

interface VariantMetrics {
  users: number;
  conversions: number;
  revenue: number;
}
```

## Conclusion

Real-Time Analytics in ArkUI provides:

- Instant event tracking and processing
- Live dashboard updates and monitoring
- Real-time metric aggregation and analysis
- Automated alert systems and anomaly detection
- Advanced analytics features like funnels and A/B testing

These capabilities enable immediate insights and data-driven decision making for HarmonyOS applications.
