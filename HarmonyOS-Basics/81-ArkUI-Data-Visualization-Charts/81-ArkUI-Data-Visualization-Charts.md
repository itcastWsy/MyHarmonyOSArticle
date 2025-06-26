# ArkUI Data Visualization and Charts

## Introduction

Data visualization in ArkUI enables interactive charts, graphs, and visual analytics. This guide covers chart libraries, custom visualizations, real-time data rendering, and interactive dashboard components.

## Chart System Architecture

### Core Chart Manager

```typescript
interface ChartData {
  labels: string[];
  datasets: ChartDataset[];
  metadata?: Record<string, any>;
}

interface ChartDataset {
  label: string;
  data: number[];
  backgroundColor?: string | string[];
  borderColor?: string;
  borderWidth?: number;
  tension?: number;
}

interface ChartOptions {
  responsive: boolean;
  animation: boolean;
  legend: {
    display: boolean;
    position: "top" | "bottom" | "left" | "right";
  };
  scales?: Record<string, any>;
  plugins?: Record<string, any>;
}

class ChartRenderer {
  private canvas: HTMLCanvasElement;
  private context: CanvasRenderingContext2D;
  private data: ChartData;
  private options: ChartOptions;

  constructor(canvas: HTMLCanvasElement) {
    this.canvas = canvas;
    this.context = canvas.getContext("2d")!;
  }

  renderLineChart(data: ChartData, options: ChartOptions): void {
    this.data = data;
    this.options = options;
    this.clearCanvas();
    this.drawAxes();
    this.drawLineChart();
    this.drawLegend();
  }

  renderBarChart(data: ChartData, options: ChartOptions): void {
    this.data = data;
    this.options = options;
    this.clearCanvas();
    this.drawAxes();
    this.drawBarChart();
    this.drawLegend();
  }

  renderPieChart(data: ChartData, options: ChartOptions): void {
    this.data = data;
    this.options = options;
    this.clearCanvas();
    this.drawPieChart();
    this.drawLegend();
  }

  private clearCanvas(): void {
    this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
  }

  private drawAxes(): void {
    const padding = 50;
    const width = this.canvas.width - 2 * padding;
    const height = this.canvas.height - 2 * padding;

    this.context.strokeStyle = "#ddd";
    this.context.lineWidth = 1;

    // X-axis
    this.context.beginPath();
    this.context.moveTo(padding, this.canvas.height - padding);
    this.context.lineTo(
      this.canvas.width - padding,
      this.canvas.height - padding
    );
    this.context.stroke();

    // Y-axis
    this.context.beginPath();
    this.context.moveTo(padding, padding);
    this.context.lineTo(padding, this.canvas.height - padding);
    this.context.stroke();
  }

  private drawLineChart(): void {
    const padding = 50;
    const width = this.canvas.width - 2 * padding;
    const height = this.canvas.height - 2 * padding;

    this.data.datasets.forEach((dataset, index) => {
      this.context.strokeStyle = dataset.borderColor || "#007AFF";
      this.context.lineWidth = dataset.borderWidth || 2;
      this.context.beginPath();

      dataset.data.forEach((value, i) => {
        const x = padding + (i / (dataset.data.length - 1)) * width;
        const maxValue = Math.max(...dataset.data);
        const y = this.canvas.height - padding - (value / maxValue) * height;

        if (i === 0) {
          this.context.moveTo(x, y);
        } else {
          this.context.lineTo(x, y);
        }
      });

      this.context.stroke();
    });
  }

  private drawBarChart(): void {
    const padding = 50;
    const width = this.canvas.width - 2 * padding;
    const height = this.canvas.height - 2 * padding;
    const barWidth = (width / this.data.labels.length) * 0.8;

    this.data.datasets.forEach((dataset, datasetIndex) => {
      dataset.data.forEach((value, i) => {
        const x =
          padding +
          (i / this.data.labels.length) * width +
          datasetIndex * (barWidth / this.data.datasets.length);
        const maxValue = Math.max(...dataset.data);
        const barHeight = (value / maxValue) * height;

        this.context.fillStyle = Array.isArray(dataset.backgroundColor)
          ? dataset.backgroundColor[i]
          : dataset.backgroundColor || "#007AFF";

        this.context.fillRect(
          x,
          this.canvas.height - padding - barHeight,
          barWidth / this.data.datasets.length,
          barHeight
        );
      });
    });
  }

  private drawPieChart(): void {
    const centerX = this.canvas.width / 2;
    const centerY = this.canvas.height / 2;
    const radius = Math.min(centerX, centerY) - 50;

    let totalValue = 0;
    this.data.datasets[0].data.forEach((value) => (totalValue += value));

    let currentAngle = -Math.PI / 2;
    const colors = ["#FF6B6B", "#4ECDC4", "#45B7D1", "#96CEB4", "#FFEAA7"];

    this.data.datasets[0].data.forEach((value, index) => {
      const sliceAngle = (value / totalValue) * 2 * Math.PI;

      this.context.fillStyle = colors[index % colors.length];
      this.context.beginPath();
      this.context.moveTo(centerX, centerY);
      this.context.arc(
        centerX,
        centerY,
        radius,
        currentAngle,
        currentAngle + sliceAngle
      );
      this.context.fill();

      currentAngle += sliceAngle;
    });
  }

  private drawLegend(): void {
    if (!this.options.legend.display) return;

    const legendY =
      this.options.legend.position === "bottom" ? this.canvas.height - 30 : 20;
    let legendX = 20;

    this.data.datasets.forEach((dataset, index) => {
      this.context.fillStyle = dataset.borderColor || "#007AFF";
      this.context.fillRect(legendX, legendY, 15, 15);

      this.context.fillStyle = "#000";
      this.context.font = "12px Arial";
      this.context.fillText(dataset.label, legendX + 20, legendY + 12);

      legendX += 100;
    });
  }
}
```

## Interactive Chart Components

### Dynamic Chart Component

```typescript
@Component
struct InteractiveChart {
  @State private chartType: 'line' | 'bar' | 'pie' = 'line'
  @State private chartData: ChartData = {
    labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
    datasets: [{
      label: 'Sales',
      data: [12, 19, 3, 5, 2, 3],
      borderColor: '#007AFF',
      backgroundColor: '#007AFF20'
    }]
  }
  @State private isAnimating: boolean = false

  private chartRenderer: ChartRenderer | null = null
  private canvasRef: HTMLCanvasElement | null = null

  build() {
    Column() {
      this.buildChartTypeSelector()
      this.buildChartContainer()
      this.buildDataControls()
      this.buildAnimationControls()
    }
    .width('100%')
    .height('100%')
    .padding(16)
  }

  @Builder
  private buildChartTypeSelector() {
    Row() {
      Text('Chart Type:')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ right: 16 })

      ['line', 'bar', 'pie'].forEach(type => {
        Button(type.toUpperCase())
          .onClick(() => this.changeChartType(type as any))
          .backgroundColor(this.chartType === type ? '#007AFF' : '#E0E0E0')
          .fontColor(this.chartType === type ? '#FFFFFF' : '#000000')
          .margin({ right: 8 })
      })
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildChartContainer() {
    Stack() {
      Canvas(this.canvasRef)
        .width('100%')
        .height(300)
        .backgroundColor('#FFFFFF')
        .borderRadius(8)
        .onReady(() => this.initializeChart())

      if (this.isAnimating) {
        LoadingProgress()
          .color('#007AFF')
          .width(50)
          .height(50)
      }
    }
    .margin({ bottom: 20 })
    .shadow({ radius: 4, color: '#00000020', offsetY: 2 })
  }

  @Builder
  private buildDataControls() {
    Column() {
      Text('Data Controls')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Button('Add Data')
          .onClick(() => this.addRandomData())
          .backgroundColor('#34C759')
          .fontColor('#FFFFFF')
          .margin({ right: 8 })

        Button('Remove Data')
          .onClick(() => this.removeLastData())
          .backgroundColor('#FF3B30')
          .fontColor('#FFFFFF')
          .margin({ right: 8 })

        Button('Randomize')
          .onClick(() => this.randomizeData())
          .backgroundColor('#FF9500')
          .fontColor('#FFFFFF')
      }
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 20 })
  }

  @Builder
  private buildAnimationControls() {
    Row() {
      Text('Animate Chart')
        .fontSize(14)
        .margin({ right: 16 })

      Toggle({ type: ToggleType.Switch, isOn: this.isAnimating })
        .onChange((isOn) => {
          this.isAnimating = isOn
          if (isOn) {
            this.startAnimation()
          }
        })
    }
  }

  private initializeChart(): void {
    if (this.canvasRef) {
      this.chartRenderer = new ChartRenderer(this.canvasRef)
      this.renderChart()
    }
  }

  private changeChartType(type: 'line' | 'bar' | 'pie'): void {
    this.chartType = type
    this.renderChart()
  }

  private renderChart(): void {
    if (!this.chartRenderer) return

    const options: ChartOptions = {
      responsive: true,
      animation: true,
      legend: {
        display: true,
        position: 'top'
      }
    }

    switch (this.chartType) {
      case 'line':
        this.chartRenderer.renderLineChart(this.chartData, options)
        break
      case 'bar':
        this.chartRenderer.renderBarChart(this.chartData, options)
        break
      case 'pie':
        this.chartRenderer.renderPieChart(this.chartData, options)
        break
    }
  }

  private addRandomData(): void {
    const newValue = Math.floor(Math.random() * 20) + 1
    const newLabel = `Item ${this.chartData.labels.length + 1}`

    this.chartData = {
      ...this.chartData,
      labels: [...this.chartData.labels, newLabel],
      datasets: this.chartData.datasets.map(dataset => ({
        ...dataset,
        data: [...dataset.data, newValue]
      }))
    }

    this.renderChart()
  }

  private removeLastData(): void {
    if (this.chartData.labels.length > 1) {
      this.chartData = {
        ...this.chartData,
        labels: this.chartData.labels.slice(0, -1),
        datasets: this.chartData.datasets.map(dataset => ({
          ...dataset,
          data: dataset.data.slice(0, -1)
        }))
      }

      this.renderChart()
    }
  }

  private randomizeData(): void {
    this.chartData = {
      ...this.chartData,
      datasets: this.chartData.datasets.map(dataset => ({
        ...dataset,
        data: dataset.data.map(() => Math.floor(Math.random() * 20) + 1)
      }))
    }

    this.renderChart()
  }

  private startAnimation(): void {
    const animateFrame = () => {
      if (!this.isAnimating) return

      this.randomizeData()
      setTimeout(animateFrame, 1000)
    }

    animateFrame()
  }
}
```

## Real-time Data Dashboard

### Dashboard Component

```typescript
@Component
struct RealTimeDashboard {
  @State private metrics: DashboardMetric[] = []
  @State private selectedTimeRange: '1h' | '24h' | '7d' | '30d' = '24h'
  @State private autoRefresh: boolean = true
  @State private refreshInterval: number = 5000

  private dataSource = new RealTimeDataSource()

  build() {
    Column() {
      this.buildDashboardHeader()
      this.buildMetricsGrid()
      this.buildChartsSection()
      this.buildControlPanel()
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }

  aboutToAppear() {
    this.initializeDashboard()
  }

  @Builder
  private buildDashboardHeader() {
    Row() {
      Text('Real-Time Dashboard')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      ['1h', '24h', '7d', '30d'].forEach(range => {
        Button(range)
          .onClick(() => this.changeTimeRange(range as any))
          .backgroundColor(this.selectedTimeRange === range ? '#007AFF' : '#E0E0E0')
          .fontColor(this.selectedTimeRange === range ? '#FFFFFF' : '#000000')
          .fontSize(12)
          .height(32)
          .margin({ left: 4 })
      })
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .shadow({ radius: 2, color: '#00000010', offsetY: 1 })
  }

  @Builder
  private buildMetricsGrid() {
    Grid() {
      ForEach(this.metrics, (metric: DashboardMetric) => {
        GridItem() {
          this.buildMetricCard(metric)
        }
      })
    }
    .columnsTemplate('1fr 1fr')
    .rowsGap(12)
    .columnsGap(12)
    .padding(16)
  }

  @Builder
  private buildMetricCard(metric: DashboardMetric) {
    Column() {
      Row() {
        Text(metric.name)
          .fontSize(14)
          .fontColor('#666666')
          .flexGrow(1)

        Text(metric.trend >= 0 ? '↗' : '↘')
          .fontSize(16)
          .fontColor(metric.trend >= 0 ? '#34C759' : '#FF3B30')
      }
      .width('100%')
      .margin({ bottom: 8 })

      Text(metric.value.toString())
        .fontSize(28)
        .fontWeight(FontWeight.Bold)
        .fontColor('#000000')

      Text(`${metric.trend >= 0 ? '+' : ''}${metric.trend.toFixed(1)}%`)
        .fontSize(12)
        .fontColor(metric.trend >= 0 ? '#34C759' : '#FF3B30')
        .margin({ top: 4 })
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .alignItems(HorizontalAlign.Start)
    .width('100%')
    .shadow({ radius: 2, color: '#00000010', offsetY: 1 })
  }

  @Builder
  private buildChartsSection() {
    Column() {
      Text('Charts')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 16 })

      // Multiple chart components would go here
      InteractiveChart()
        .height(300)
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .margin({ horizontal: 16, bottom: 16 })
    .borderRadius(8)
    .shadow({ radius: 2, color: '#00000010', offsetY: 1 })
  }

  @Builder
  private buildControlPanel() {
    Row() {
      Text('Auto Refresh')
        .fontSize(14)
        .margin({ right: 16 })

      Toggle({ type: ToggleType.Switch, isOn: this.autoRefresh })
        .onChange((isOn) => {
          this.autoRefresh = isOn
          if (isOn) {
            this.startAutoRefresh()
          }
        })

      Blank()

      Button('Refresh Now')
        .onClick(() => this.refreshData())
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .fontSize(12)
        .height(32)
    }
    .padding(16)
    .backgroundColor('#FFFFFF')
    .margin({ horizontal: 16 })
    .borderRadius(8)
    .shadow({ radius: 2, color: '#00000010', offsetY: 1 })
  }

  private initializeDashboard(): void {
    this.loadMetrics()
    if (this.autoRefresh) {
      this.startAutoRefresh()
    }
  }

  private loadMetrics(): void {
    this.metrics = [
      { name: 'Total Users', value: 12567, trend: 5.2 },
      { name: 'Revenue', value: 89340, trend: -2.1 },
      { name: 'Conversion Rate', value: 3.47, trend: 1.8 },
      { name: 'Page Views', value: 45623, trend: 12.5 }
    ]
  }

  private changeTimeRange(range: '1h' | '24h' | '7d' | '30d'): void {
    this.selectedTimeRange = range
    this.refreshData()
  }

  private refreshData(): void {
    this.loadMetrics()
    // Refresh chart data as well
  }

  private startAutoRefresh(): void {
    const refresh = () => {
      if (!this.autoRefresh) return

      this.refreshData()
      setTimeout(refresh, this.refreshInterval)
    }

    refresh()
  }
}

interface DashboardMetric {
  name: string
  value: number
  trend: number
}

class RealTimeDataSource {
  async getMetrics(timeRange: string): Promise<DashboardMetric[]> {
    // Simulate API call
    return []
  }

  async getChartData(timeRange: string): Promise<ChartData> {
    // Simulate API call
    return {
      labels: [],
      datasets: []
    }
  }
}
```

## Conclusion

Data visualization in ArkUI applications provides:

- Comprehensive charting capabilities with multiple chart types
- Real-time data updates and interactive visualizations
- Customizable dashboard components with metrics display
- Animation and transition effects for engaging user experiences
- Responsive design for various screen sizes
- Integration with real-time data sources

These features enable developers to create powerful analytics and monitoring applications with rich visual data representations.
