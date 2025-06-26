# ArkUI Internationalization Best Practices

## Introduction

Internationalization (i18n) is crucial for creating applications that reach global audiences. This guide covers advanced internationalization techniques, localization strategies, and cultural adaptation patterns in ArkUI applications.

## i18n Architecture

### Resource Management System

```typescript
interface LocaleResource {
  code: string;
  name: string;
  direction: "ltr" | "rtl";
  dateFormat: string;
  timeFormat: string;
  numberFormat: NumberFormatOptions;
  currencyFormat: CurrencyFormatOptions;
  translations: Record<string, string>;
}

class InternationalizationManager {
  private currentLocale: string = "en-US";
  private resources = new Map<string, LocaleResource>();
  private fallbackLocale = "en-US";
  private listeners: Set<(locale: string) => void> = new Set();

  registerLocale(resource: LocaleResource): void {
    this.resources.set(resource.code, resource);
  }

  setLocale(localeCode: string): void {
    if (this.resources.has(localeCode)) {
      this.currentLocale = localeCode;
      this.notifyListeners();
    }
  }

  getCurrentLocale(): string {
    return this.currentLocale;
  }

  translate(key: string, params?: Record<string, any>): string {
    const resource = this.getResource(this.currentLocale);
    let translation = resource?.translations[key];

    if (!translation) {
      // Fallback to default locale
      const fallbackResource = this.getResource(this.fallbackLocale);
      translation = fallbackResource?.translations[key] || key;
    }

    return this.interpolateParams(translation, params);
  }

  formatDate(date: Date, format?: string): string {
    const resource = this.getResource(this.currentLocale);
    const locale = this.currentLocale;
    const dateFormat = format || resource?.dateFormat || "short";

    return new Intl.DateTimeFormat(
      locale,
      this.getDateFormatOptions(dateFormat)
    ).format(date);
  }

  formatNumber(value: number, options?: Intl.NumberFormatOptions): string {
    const resource = this.getResource(this.currentLocale);
    const mergedOptions = { ...resource?.numberFormat, ...options };

    return new Intl.NumberFormat(this.currentLocale, mergedOptions).format(
      value
    );
  }

  formatCurrency(value: number, currency: string): string {
    const resource = this.getResource(this.currentLocale);
    const options: Intl.NumberFormatOptions = {
      style: "currency",
      currency,
      ...resource?.currencyFormat,
    };

    return new Intl.NumberFormat(this.currentLocale, options).format(value);
  }

  getTextDirection(): "ltr" | "rtl" {
    const resource = this.getResource(this.currentLocale);
    return resource?.direction || "ltr";
  }

  subscribe(listener: (locale: string) => void): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private getResource(localeCode: string): LocaleResource | undefined {
    return this.resources.get(localeCode);
  }

  private interpolateParams(
    text: string,
    params?: Record<string, any>
  ): string {
    if (!params) return text;

    return text.replace(/\{\{(\w+)\}\}/g, (match, key) => {
      return params[key]?.toString() || match;
    });
  }

  private notifyListeners(): void {
    this.listeners.forEach((listener) => listener(this.currentLocale));
  }
}

// Global i18n instance
const i18n = new InternationalizationManager();

// Register locales
i18n.registerLocale({
  code: "en-US",
  name: "English (US)",
  direction: "ltr",
  dateFormat: "MM/dd/yyyy",
  timeFormat: "h:mm a",
  numberFormat: { useGrouping: true },
  currencyFormat: { minimumFractionDigits: 2 },
  translations: {
    welcome: "Welcome",
    hello_user: "Hello, {{name}}!",
    product_count: "You have {{count}} products",
  },
});

i18n.registerLocale({
  code: "zh-CN",
  name: "中文 (简体)",
  direction: "ltr",
  dateFormat: "yyyy年MM月dd日",
  timeFormat: "HH:mm",
  numberFormat: { useGrouping: true },
  currencyFormat: { minimumFractionDigits: 2 },
  translations: {
    welcome: "欢迎",
    hello_user: "你好，{{name}}！",
    product_count: "您有 {{count}} 个产品",
  },
});
```

### Localized Components

```typescript
// Translation hook for components
function useTranslation() {
  const [locale, setLocale] = useState(i18n.getCurrentLocale())

  useEffect(() => {
    const unsubscribe = i18n.subscribe(setLocale)
    return unsubscribe
  }, [])

  return {
    t: (key: string, params?: Record<string, any>) => i18n.translate(key, params),
    locale,
    setLocale: (newLocale: string) => i18n.setLocale(newLocale),
    formatDate: (date: Date, format?: string) => i18n.formatDate(date, format),
    formatNumber: (value: number, options?: Intl.NumberFormatOptions) => i18n.formatNumber(value, options),
    formatCurrency: (value: number, currency: string) => i18n.formatCurrency(value, currency),
    isRTL: () => i18n.getTextDirection() === 'rtl'
  }
}

@Component
struct LocalizedText {
  @Prop translationKey: string
  @Prop params?: Record<string, any>
  @Prop fallback?: string
  @State private translation: string = ''

  aboutToAppear() {
    this.updateTranslation()

    // Subscribe to locale changes
    i18n.subscribe(() => {
      this.updateTranslation()
    })
  }

  build() {
    Text(this.translation || this.fallback || this.translationKey)
      .textAlign(this.getTextAlign())
  }

  private updateTranslation(): void {
    this.translation = i18n.translate(this.translationKey, this.params)
  }

  private getTextAlign(): TextAlign {
    return i18n.getTextDirection() === 'rtl' ? TextAlign.Right : TextAlign.Left
  }
}

@Component
struct LocalizedButton {
  @Prop textKey: string
  @Prop params?: Record<string, any>
  @Prop onClick?: () => void

  build() {
    Button() {
      LocalizedText({
        translationKey: this.textKey,
        params: this.params
      })
    }
    .onClick(this.onClick)
    .layoutDirection(i18n.getTextDirection() === 'rtl' ? LayoutDirection.Rtl : LayoutDirection.Ltr)
  }
}
```

## Cultural Adaptation

### Layout Direction Support

```typescript
@Component
struct AdaptiveLayout {
  @State private isRTL: boolean = false

  aboutToAppear() {
    this.updateDirection()

    i18n.subscribe(() => {
      this.updateDirection()
    })
  }

  build() {
    Row() {
      if (this.isRTL) {
        this.buildRTLLayout()
      } else {
        this.buildLTRLayout()
      }
    }
    .layoutDirection(this.isRTL ? LayoutDirection.Rtl : LayoutDirection.Ltr)
  }

  @Builder
  buildLTRLayout() {
    Column() {
      // Left-to-right layout
      this.buildHeader()
      this.buildContent()
      this.buildFooter()
    }
  }

  @Builder
  buildRTLLayout() {
    Column() {
      // Right-to-left adapted layout
      this.buildHeader()
      this.buildContent()
      this.buildFooter()
    }
  }

  @Builder
  buildHeader() {
    Row() {
      if (this.isRTL) {
        this.buildMenuButton()
        Blank()
        this.buildLogo()
      } else {
        this.buildLogo()
        Blank()
        this.buildMenuButton()
      }
    }
    .width('100%')
    .padding(16)
  }

  private updateDirection(): void {
    this.isRTL = i18n.getTextDirection() === 'rtl'
  }
}

// Date and time formatting
@Component
struct LocalizedDateTime {
  @Prop date: Date
  @Prop format?: 'date' | 'time' | 'datetime'
  @State private formattedText: string = ''

  aboutToAppear() {
    this.updateFormat()

    i18n.subscribe(() => {
      this.updateFormat()
    })
  }

  build() {
    Text(this.formattedText)
  }

  private updateFormat(): void {
    switch (this.format) {
      case 'date':
        this.formattedText = i18n.formatDate(this.date)
        break
      case 'time':
        this.formattedText = i18n.formatDate(this.date, 'time')
        break
      case 'datetime':
      default:
        this.formattedText = i18n.formatDate(this.date, 'datetime')
    }
  }
}

// Number and currency formatting
@Component
struct LocalizedPrice {
  @Prop amount: number
  @Prop currency: string
  @State private formattedPrice: string = ''

  aboutToAppear() {
    this.updatePrice()

    i18n.subscribe(() => {
      this.updatePrice()
    })
  }

  build() {
    Text(this.formattedPrice)
      .fontWeight(FontWeight.Bold)
      .fontColor('#007AFF')
  }

  private updatePrice(): void {
    this.formattedPrice = i18n.formatCurrency(this.amount, this.currency)
  }
}
```

## Dynamic Content Loading

### Async Resource Loading

```typescript
class ResourceLoader {
  private cache = new Map<string, LocaleResource>()
  private loadingPromises = new Map<string, Promise<LocaleResource>>()

  async loadLocale(localeCode: string): Promise<LocaleResource> {
    // Return cached resource if available
    if (this.cache.has(localeCode)) {
      return this.cache.get(localeCode)!
    }

    // Return ongoing promise if already loading
    if (this.loadingPromises.has(localeCode)) {
      return this.loadingPromises.get(localeCode)!
    }

    // Start loading the resource
    const loadingPromise = this.fetchLocaleResource(localeCode)
    this.loadingPromises.set(localeCode, loadingPromise)

    try {
      const resource = await loadingPromise
      this.cache.set(localeCode, resource)
      i18n.registerLocale(resource)
      return resource
    } finally {
      this.loadingPromises.delete(localeCode)
    }
  }

  private async fetchLocaleResource(localeCode: string): Promise<LocaleResource> {
    const response = await fetch(`/locales/${localeCode}.json`)

    if (!response.ok) {
      throw new Error(`Failed to load locale: ${localeCode}`)
    }

    return response.json()
  }

  preloadLocales(localeCodes: string[]): Promise<LocaleResource[]> {
    return Promise.all(localeCodes.map(code => this.loadLocale(code)))
  }
}

// Lazy loading component
@Component
struct LazyLocalizedApp {
  @State private loading: boolean = true
  @State private selectedLocale: string = 'en-US'
  private resourceLoader = new ResourceLoader()

  aboutToAppear() {
    this.initializeLocalization()
  }

  build() {
    Column() {
      if (this.loading) {
        this.buildLoadingScreen()
      } else {
        this.buildMainApp()
      }
    }
    .width('100%')
    .height('100%')
  }

  @Builder
  buildLoadingScreen() {
    Column() {
      LoadingProgress()
        .width(48)
        .height(48)
        .margin({ bottom: 16 })

      Text('Loading...')
        .fontSize(16)
    }
    .justifyContent(FlexAlign.Center)
    .width('100%')
    .height('100%')
  }

  @Builder
  buildMainApp() {
    Column() {
      this.buildLanguageSelector()
      this.buildContent()
    }
  }

  @Builder
  buildLanguageSelector() {
    Row() {
      Text('Language:')
        .margin({ right: 8 })

      Select([
        { value: 'en-US', text: 'English' },
        { value: 'zh-CN', text: '中文' },
        { value: 'es-ES', text: 'Español' },
        { value: 'fr-FR', text: 'Français' }
      ])
        .selected(this.getSelectedIndex())
        .onSelect(async (index: number) => {
          await this.changeLocale(this.getLocaleByIndex(index))
        })
    }
    .padding(16)
    .width('100%')
  }

  private async initializeLocalization(): Promise<void> {
    try {
      // Detect system locale
      const systemLocale = this.detectSystemLocale()

      // Load initial locale
      await this.resourceLoader.loadLocale(systemLocale)
      i18n.setLocale(systemLocale)
      this.selectedLocale = systemLocale

      // Preload common locales
      this.resourceLoader.preloadLocales(['en-US', 'zh-CN'])
    } catch (error) {
      console.error('Failed to initialize localization:', error)
      // Fallback to English
      i18n.setLocale('en-US')
    } finally {
      this.loading = false
    }
  }

  private async changeLocale(localeCode: string): Promise<void> {
    try {
      this.loading = true
      await this.resourceLoader.loadLocale(localeCode)
      i18n.setLocale(localeCode)
      this.selectedLocale = localeCode
    } catch (error) {
      console.error(`Failed to load locale ${localeCode}:`, error)
    } finally {
      this.loading = false
    }
  }

  private detectSystemLocale(): string {
    // Detect system locale (implementation depends on platform)
    return navigator.language || 'en-US'
  }
}
```

## Accessibility Integration

### Screen Reader Support

```typescript
@Component
struct AccessibleLocalizedText {
  @Prop translationKey: string
  @Prop accessibilityLabel?: string
  @Prop params?: Record<string, any>

  build() {
    Text(i18n.translate(this.translationKey, this.params))
      .accessibilityText(
        this.accessibilityLabel ||
        i18n.translate(this.translationKey + '_accessibility', this.params)
      )
      .accessibilityLevel('auto')
  }
}

// Voice-over friendly number formatting
@Component
struct AccessiblePrice {
  @Prop amount: number
  @Prop currency: string

  build() {
    Text(i18n.formatCurrency(this.amount, this.currency))
      .accessibilityText(this.getAccessiblePriceText())
  }

  private getAccessiblePriceText(): string {
    const currencyName = this.getCurrencyName(this.currency)
    const formattedAmount = i18n.formatNumber(this.amount)

    return i18n.translate('price_accessibility', {
      amount: formattedAmount,
      currency: currencyName
    })
  }

  private getCurrencyName(currency: string): string {
    const currencyNames: Record<string, string> = {
      'USD': i18n.translate('currency_usd'),
      'EUR': i18n.translate('currency_eur'),
      'CNY': i18n.translate('currency_cny')
    }

    return currencyNames[currency] || currency
  }
}
```

## Testing Internationalization

### i18n Testing Framework

```typescript
class I18nTester {
  async testAllLocales(
    component: any,
    testCases: TestCase[]
  ): Promise<TestResult[]> {
    const results: TestResult[] = [];
    const locales = ["en-US", "zh-CN", "ar-SA", "es-ES"]; // Include RTL locale

    for (const locale of locales) {
      i18n.setLocale(locale);

      for (const testCase of testCases) {
        const result = await this.runTest(component, testCase, locale);
        results.push(result);
      }
    }

    return results;
  }

  private async runTest(
    component: any,
    testCase: TestCase,
    locale: string
  ): Promise<TestResult> {
    try {
      const translation = i18n.translate(testCase.key, testCase.params);

      return {
        locale,
        key: testCase.key,
        expected: testCase.expected?.[locale],
        actual: translation,
        passed: this.validateTranslation(translation, testCase, locale),
      };
    } catch (error) {
      return {
        locale,
        key: testCase.key,
        expected: testCase.expected?.[locale],
        actual: "",
        passed: false,
        error: error.message,
      };
    }
  }

  validateTranslationCoverage(requiredKeys: string[]): CoverageReport {
    const locales = Array.from(i18n.resources.keys());
    const coverage: Record<string, number> = {};

    for (const locale of locales) {
      const resource = i18n.resources.get(locale);
      const availableKeys = Object.keys(resource?.translations || {});
      const missingKeys = requiredKeys.filter(
        (key) => !availableKeys.includes(key)
      );

      coverage[locale] =
        ((requiredKeys.length - missingKeys.length) / requiredKeys.length) *
        100;
    }

    return { coverage, total: this.calculateOverallCoverage(coverage) };
  }
}

interface TestCase {
  key: string;
  params?: Record<string, any>;
  expected?: Record<string, string>;
}

interface TestResult {
  locale: string;
  key: string;
  expected?: string;
  actual: string;
  passed: boolean;
  error?: string;
}
```

## Conclusion

Effective internationalization in ArkUI applications requires:

- Comprehensive resource management systems
- Dynamic locale loading and switching
- Cultural adaptation including RTL support
- Proper date, number, and currency formatting
- Accessibility integration for global users
- Thorough testing across all supported locales
- Performance optimization for resource loading

These practices ensure applications provide native-quality experiences for users worldwide while maintaining maintainable and scalable codebases.
