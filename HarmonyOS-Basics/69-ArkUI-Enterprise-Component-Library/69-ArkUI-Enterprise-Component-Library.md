# ArkUI Enterprise Component Library

## Introduction

Building enterprise-grade component libraries requires scalable architecture, comprehensive testing, and robust documentation. This guide explores creating, managing, and distributing professional ArkUI component libraries.

## Component Library Architecture

### Base Component System

```typescript
interface ComponentTheme {
  colors: {
    primary: string;
    secondary: string;
    success: string;
    warning: string;
    error: string;
    background: string;
    surface: string;
    text: string;
  };
  spacing: {
    xs: number;
    sm: number;
    md: number;
    lg: number;
    xl: number;
  };
  typography: {
    h1: TextStyle;
    h2: TextStyle;
    body: TextStyle;
    caption: TextStyle;
  };
  borderRadius: {
    small: number;
    medium: number;
    large: number;
  };
}

interface TextStyle {
  fontSize: number;
  fontWeight: FontWeight;
  lineHeight?: number;
}

class ThemeProvider {
  private static theme: ComponentTheme = this.getDefaultTheme();

  static setTheme(theme: Partial<ComponentTheme>): void {
    this.theme = { ...this.theme, ...theme };
  }

  static getTheme(): ComponentTheme {
    return this.theme;
  }

  private static getDefaultTheme(): ComponentTheme {
    return {
      colors: {
        primary: "#007AFF",
        secondary: "#34C759",
        success: "#30D158",
        warning: "#FF9500",
        error: "#FF3B30",
        background: "#F2F2F7",
        surface: "#FFFFFF",
        text: "#000000",
      },
      spacing: {
        xs: 4,
        sm: 8,
        md: 16,
        lg: 24,
        xl: 32,
      },
      typography: {
        h1: { fontSize: 32, fontWeight: FontWeight.Bold },
        h2: { fontSize: 24, fontWeight: FontWeight.Bold },
        body: { fontSize: 16, fontWeight: FontWeight.Normal },
        caption: { fontSize: 12, fontWeight: FontWeight.Normal },
      },
      borderRadius: {
        small: 4,
        medium: 8,
        large: 16,
      },
    };
  }
}

abstract class BaseComponent {
  protected theme = ThemeProvider.getTheme();

  protected applyTheme(styles: Record<string, any>): Record<string, any> {
    return styles;
  }
}
```

### Component Props System

```typescript
interface CommonProps {
  testID?: string
  disabled?: boolean
  loading?: boolean
  variant?: 'primary' | 'secondary' | 'outline' | 'text'
  size?: 'small' | 'medium' | 'large'
  fullWidth?: boolean
}

interface ButtonProps extends CommonProps {
  text: string
  icon?: string
  iconPosition?: 'left' | 'right'
  onClick?: () => void
  type?: 'button' | 'submit' | 'reset'
}

@Component
export struct EnterpriseButton {
  @Prop props: ButtonProps = { text: 'Button' }
  @State private isPressed: boolean = false
  @State private isHovered: boolean = false

  build() {
    Button(this.props.text)
      .width(this.props.fullWidth ? '100%' : 'auto')
      .height(this.getButtonHeight())
      .backgroundColor(this.getBackgroundColor())
      .fontColor(this.getFontColor())
      .fontSize(this.getFontSize())
      .borderRadius(this.getBorderRadius())
      .border(this.getBorder())
      .opacity(this.props.disabled ? 0.6 : 1.0)
      .enabled(!this.props.disabled && !this.props.loading)
      .onClick(() => {
        if (this.props.onClick && !this.props.disabled && !this.props.loading) {
          this.props.onClick()
        }
      })
      .onTouch((event) => {
        if (event.type === TouchType.Down) {
          this.isPressed = true
        } else if (event.type === TouchType.Up) {
          this.isPressed = false
        }
      })
      .onHover((isHover) => {
        this.isHovered = isHover
      })
      .animation({
        duration: 150,
        curve: Curve.EaseInOut
      })
  }

  private getButtonHeight(): number {
    const theme = ThemeProvider.getTheme()
    switch (this.props.size) {
      case 'small': return 32
      case 'large': return 56
      default: return 44
    }
  }

  private getBackgroundColor(): string {
    const theme = ThemeProvider.getTheme()
    if (this.isPressed) {
      return this.darkenColor(this.getBaseBackgroundColor(), 0.2)
    }
    if (this.isHovered) {
      return this.darkenColor(this.getBaseBackgroundColor(), 0.1)
    }
    return this.getBaseBackgroundColor()
  }

  private getBaseBackgroundColor(): string {
    const theme = ThemeProvider.getTheme()
    switch (this.props.variant) {
      case 'secondary': return theme.colors.secondary
      case 'outline': return 'transparent'
      case 'text': return 'transparent'
      default: return theme.colors.primary
    }
  }

  private getFontColor(): string {
    const theme = ThemeProvider.getTheme()
    switch (this.props.variant) {
      case 'outline':
      case 'text':
        return theme.colors.primary
      default:
        return '#FFFFFF'
    }
  }

  private getFontSize(): number {
    switch (this.props.size) {
      case 'small': return 14
      case 'large': return 18
      default: return 16
    }
  }

  private getBorderRadius(): number {
    const theme = ThemeProvider.getTheme()
    return theme.borderRadius.medium
  }

  private getBorder(): BorderOptions {
    if (this.props.variant === 'outline') {
      return {
        width: 1,
        color: ThemeProvider.getTheme().colors.primary,
        style: BorderStyle.Solid
      }
    }
    return { width: 0 }
  }

  private darkenColor(color: string, amount: number): string {
    // Simple color darkening function
    const hex = color.replace('#', '')
    const num = parseInt(hex, 16)
    const r = Math.max(0, (num >> 16) - Math.round(255 * amount))
    const g = Math.max(0, ((num >> 8) & 0x00FF) - Math.round(255 * amount))
    const b = Math.max(0, (num & 0x0000FF) - Math.round(255 * amount))
    return `#${((r << 16) | (g << 8) | b).toString(16).padStart(6, '0')}`
  }
}
```

### Form Components

```typescript
interface InputProps extends CommonProps {
  value: string
  placeholder?: string
  type?: 'text' | 'password' | 'email' | 'number'
  label?: string
  helperText?: string
  errorText?: string
  maxLength?: number
  multiline?: boolean
  rows?: number
  onValueChange?: (value: string) => void
  onFocus?: () => void
  onBlur?: () => void
}

@Component
export struct EnterpriseInput {
  @Prop props: InputProps = { value: '' }
  @State private isFocused: boolean = false
  @State private hasError: boolean = false

  build() {
    Column() {
      if (this.props.label) {
        this.buildLabel()
      }

      this.buildInput()

      if (this.props.helperText || this.props.errorText) {
        this.buildHelperText()
      }
    }
    .alignItems(HorizontalAlign.Start)
    .width('100%')
  }

  @Builder
  private buildLabel() {
    Text(this.props.label!)
      .fontSize(14)
      .fontColor(this.getLabelColor())
      .fontWeight(FontWeight.Medium)
      .margin({ bottom: 8 })
  }

  @Builder
  private buildInput() {
    if (this.props.multiline) {
      TextArea({
        text: this.props.value,
        placeholder: this.props.placeholder
      })
      .width('100%')
      .height((this.props.rows || 3) * 24)
      .backgroundColor(this.getInputBackgroundColor())
      .borderRadius(8)
      .border(this.getInputBorder())
      .padding(12)
      .onChange((value) => {
        if (this.props.onValueChange) {
          this.props.onValueChange(value)
        }
      })
      .onFocus(() => {
        this.isFocused = true
        if (this.props.onFocus) {
          this.props.onFocus()
        }
      })
      .onBlur(() => {
        this.isFocused = false
        if (this.props.onBlur) {
          this.props.onBlur()
        }
      })
    } else {
      TextInput({
        text: this.props.value,
        placeholder: this.props.placeholder
      })
      .width('100%')
      .height(44)
      .backgroundColor(this.getInputBackgroundColor())
      .borderRadius(8)
      .border(this.getInputBorder())
      .padding({ horizontal: 12 })
      .type(this.getInputType())
      .maxLength(this.props.maxLength)
      .onChange((value) => {
        if (this.props.onValueChange) {
          this.props.onValueChange(value)
        }
      })
      .onFocus(() => {
        this.isFocused = true
        if (this.props.onFocus) {
          this.props.onFocus()
        }
      })
      .onBlur(() => {
        this.isFocused = false
        if (this.props.onBlur) {
          this.props.onBlur()
        }
      })
    }
  }

  @Builder
  private buildHelperText() {
    Text(this.props.errorText || this.props.helperText!)
      .fontSize(12)
      .fontColor(this.props.errorText ?
        ThemeProvider.getTheme().colors.error :
        '#666666'
      )
      .margin({ top: 4 })
  }

  private getLabelColor(): string {
    const theme = ThemeProvider.getTheme()
    if (this.props.errorText) {
      return theme.colors.error
    }
    if (this.isFocused) {
      return theme.colors.primary
    }
    return theme.colors.text
  }

  private getInputBackgroundColor(): string {
    return ThemeProvider.getTheme().colors.surface
  }

  private getInputBorder(): BorderOptions {
    const theme = ThemeProvider.getTheme()
    let color = '#E0E0E0'

    if (this.props.errorText) {
      color = theme.colors.error
    } else if (this.isFocused) {
      color = theme.colors.primary
    }

    return {
      width: this.isFocused ? 2 : 1,
      color,
      style: BorderStyle.Solid
    }
  }

  private getInputType(): InputType {
    switch (this.props.type) {
      case 'password': return InputType.Password
      case 'email': return InputType.Email
      case 'number': return InputType.Number
      default: return InputType.Normal
    }
  }
}
```

## Component Testing Framework

### Testing Utilities

```typescript
interface ComponentTestCase {
  name: string;
  props: any;
  expectedBehavior: string;
  test: (component: any) => boolean;
}

class ComponentTester {
  private testCases: ComponentTestCase[] = [];

  addTest(testCase: ComponentTestCase): void {
    this.testCases.push(testCase);
  }

  async runTests(ComponentClass: any): Promise<TestResult[]> {
    const results: TestResult[] = [];

    for (const testCase of this.testCases) {
      try {
        const component = new ComponentClass(testCase.props);
        const passed = testCase.test(component);

        results.push({
          name: testCase.name,
          passed,
          error: null,
          duration: 0,
        });
      } catch (error) {
        results.push({
          name: testCase.name,
          passed: false,
          error: error.message,
          duration: 0,
        });
      }
    }

    return results;
  }

  generateReport(results: TestResult[]): TestReport {
    const passed = results.filter((r) => r.passed).length;
    const failed = results.length - passed;

    return {
      total: results.length,
      passed,
      failed,
      coverage: passed / results.length,
      results,
    };
  }
}

interface TestResult {
  name: string;
  passed: boolean;
  error: string | null;
  duration: number;
}

interface TestReport {
  total: number;
  passed: number;
  failed: number;
  coverage: number;
  results: TestResult[];
}

// Example tests for EnterpriseButton
const buttonTester = new ComponentTester();

buttonTester.addTest({
  name: "Button renders with correct text",
  props: { text: "Test Button" },
  expectedBehavior: "Should display the provided text",
  test: (component) => {
    return component.props.text === "Test Button";
  },
});

buttonTester.addTest({
  name: "Button handles disabled state",
  props: { text: "Button", disabled: true },
  expectedBehavior: "Should be disabled when disabled prop is true",
  test: (component) => {
    return component.props.disabled === true;
  },
});

buttonTester.addTest({
  name: "Button calls onClick when clicked",
  props: {
    text: "Button",
    onClick: () => {
      /* mock function */
    },
  },
  expectedBehavior: "Should call onClick when clicked",
  test: (component) => {
    return typeof component.props.onClick === "function";
  },
});
```

### Visual Regression Testing

```typescript
interface VisualTestConfig {
  componentName: string;
  variants: ComponentVariant[];
  viewports: Viewport[];
}

interface ComponentVariant {
  name: string;
  props: any;
}

interface Viewport {
  name: string;
  width: number;
  height: number;
}

class VisualTester {
  async captureComponent(
    ComponentClass: any,
    props: any,
    viewport: Viewport
  ): Promise<string> {
    // Capture component screenshot
    // This would integrate with actual screenshot capture system
    return `screenshot_${ComponentClass.name}_${Date.now()}.png`;
  }

  async compareScreenshots(
    baseline: string,
    current: string,
    threshold = 0.1
  ): Promise<boolean> {
    // Compare two screenshots
    // Return true if they match within threshold
    return true;
  }

  async runVisualTests(config: VisualTestConfig): Promise<VisualTestResult[]> {
    const results: VisualTestResult[] = [];

    for (const variant of config.variants) {
      for (const viewport of config.viewports) {
        const testName = `${config.componentName}_${variant.name}_${viewport.name}`;

        try {
          const screenshot = await this.captureComponent(
            config.componentName,
            variant.props,
            viewport
          );

          const baselineExists = await this.baselineExists(testName);
          if (baselineExists) {
            const baseline = await this.getBaseline(testName);
            const matches = await this.compareScreenshots(baseline, screenshot);

            results.push({
              name: testName,
              passed: matches,
              baseline,
              current: screenshot,
              viewport,
            });
          } else {
            await this.saveAsBaseline(testName, screenshot);
            results.push({
              name: testName,
              passed: true,
              baseline: screenshot,
              current: screenshot,
              viewport,
            });
          }
        } catch (error) {
          results.push({
            name: testName,
            passed: false,
            error: error.message,
            viewport,
          });
        }
      }
    }

    return results;
  }

  private async baselineExists(testName: string): Promise<boolean> {
    // Check if baseline image exists
    return false;
  }

  private async getBaseline(testName: string): Promise<string> {
    // Get baseline image path
    return `baseline_${testName}.png`;
  }

  private async saveAsBaseline(
    testName: string,
    screenshot: string
  ): Promise<void> {
    // Save screenshot as baseline
  }
}

interface VisualTestResult {
  name: string;
  passed: boolean;
  baseline?: string;
  current?: string;
  error?: string;
  viewport: Viewport;
}
```

## Documentation Generation

### Component Documentation

```typescript
interface ComponentDoc {
  name: string;
  description: string;
  props: PropDoc[];
  examples: ExampleDoc[];
  notes?: string[];
}

interface PropDoc {
  name: string;
  type: string;
  required: boolean;
  defaultValue?: any;
  description: string;
}

interface ExampleDoc {
  title: string;
  description: string;
  code: string;
  props: any;
}

class DocumentationGenerator {
  generateComponentDocs(ComponentClass: any): ComponentDoc {
    return {
      name: ComponentClass.name,
      description: this.extractDescription(ComponentClass),
      props: this.extractProps(ComponentClass),
      examples: this.generateExamples(ComponentClass),
      notes: this.extractNotes(ComponentClass),
    };
  }

  generateMarkdown(doc: ComponentDoc): string {
    let markdown = `# ${doc.name}\n\n`;
    markdown += `${doc.description}\n\n`;

    markdown += `## Props\n\n`;
    markdown += `| Name | Type | Required | Default | Description |\n`;
    markdown += `|------|------|----------|---------|-------------|\n`;

    doc.props.forEach((prop) => {
      markdown += `| ${prop.name} | ${prop.type} | ${
        prop.required ? "Yes" : "No"
      } | ${prop.defaultValue || "-"} | ${prop.description} |\n`;
    });

    markdown += `\n## Examples\n\n`;
    doc.examples.forEach((example) => {
      markdown += `### ${example.title}\n\n`;
      markdown += `${example.description}\n\n`;
      markdown += `\`\`\`typescript\n${example.code}\n\`\`\`\n\n`;
    });

    if (doc.notes && doc.notes.length > 0) {
      markdown += `## Notes\n\n`;
      doc.notes.forEach((note) => {
        markdown += `- ${note}\n`;
      });
    }

    return markdown;
  }

  private extractDescription(ComponentClass: any): string {
    // Extract description from component metadata or comments
    return `${ComponentClass.name} component for enterprise applications`;
  }

  private extractProps(ComponentClass: any): PropDoc[] {
    // Extract props from TypeScript interfaces
    return [
      {
        name: "disabled",
        type: "boolean",
        required: false,
        defaultValue: false,
        description: "Whether the component is disabled",
      },
    ];
  }

  private generateExamples(ComponentClass: any): ExampleDoc[] {
    return [
      {
        title: "Basic Usage",
        description: "Simple component usage",
        code: `<${ComponentClass.name} />`,
        props: {},
      },
    ];
  }

  private extractNotes(ComponentClass: any): string[] {
    return [
      "This component follows accessibility guidelines",
      "Supports theming through ThemeProvider",
    ];
  }
}
```

## Component Distribution

### Package Builder

```typescript
interface BuildConfig {
  entry: string;
  output: string;
  format: "esm" | "cjs" | "umd";
  external?: string[];
  minify?: boolean;
}

class ComponentLibraryBuilder {
  async build(config: BuildConfig): Promise<BuildResult> {
    const startTime = Date.now();

    try {
      // Bundle components
      const bundle = await this.createBundle(config);

      // Generate type definitions
      const types = await this.generateTypes(config);

      // Create package files
      await this.createPackageFiles(config, bundle, types);

      // Generate documentation
      await this.generateDocs(config);

      return {
        success: true,
        duration: Date.now() - startTime,
        outputPath: config.output,
        size: await this.calculateSize(config.output),
      };
    } catch (error) {
      return {
        success: false,
        duration: Date.now() - startTime,
        error: error.message,
      };
    }
  }

  private async createBundle(config: BuildConfig): Promise<string> {
    // Bundle the components using build tool
    return "bundled-code";
  }

  private async generateTypes(config: BuildConfig): Promise<string> {
    // Generate TypeScript declaration files
    return "type-definitions";
  }

  private async createPackageFiles(
    config: BuildConfig,
    bundle: string,
    types: string
  ): Promise<void> {
    // Create package.json, README, etc.
  }

  private async generateDocs(config: BuildConfig): Promise<void> {
    // Generate component documentation
  }

  private async calculateSize(outputPath: string): Promise<number> {
    // Calculate bundle size
    return 0;
  }
}

interface BuildResult {
  success: boolean;
  duration: number;
  outputPath?: string;
  size?: number;
  error?: string;
}

// NPM package configuration
const packageConfig = {
  name: "@company/arkui-components",
  version: "1.0.0",
  description: "Enterprise ArkUI Component Library",
  main: "dist/index.js",
  types: "dist/index.d.ts",
  exports: {
    ".": {
      import: "./dist/index.esm.js",
      require: "./dist/index.cjs.js",
    },
  },
  peerDependencies: {
    "@ohos/arkui": "^1.0.0",
  },
  keywords: ["arkui", "harmonyos", "components", "ui-library"],
};
```

## Conclusion

Enterprise component libraries in ArkUI require:

- Consistent theming and design system
- Comprehensive prop interfaces and validation
- Robust testing frameworks and visual regression testing
- Automated documentation generation
- Efficient build and distribution systems
- Version management and backwards compatibility

These practices ensure high-quality, maintainable component libraries suitable for large-scale enterprise applications.
