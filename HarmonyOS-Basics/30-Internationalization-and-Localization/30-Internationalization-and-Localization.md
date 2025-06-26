# 30-Internationalization and Localization

## Introduction

Internationalization (i18n) and localization (l10n) are crucial for creating applications that serve global audiences. This article covers how to implement multi-language support, format locale-specific content, and handle different cultural conventions in HarmonyOS applications.

## Resource Management

### String Resources

```typescript
// resources/en_US/element/string.json
{
  "string": [
    {
      "name": "app_name",
      "value": "My Application"
    },
    {
      "name": "welcome_message",
      "value": "Welcome to our app!"
    },
    {
      "name": "user_greeting",
      "value": "Hello, %s!"
    },
    {
      "name": "item_count",
      "value": {
        "one": "You have %d item",
        "other": "You have %d items"
      }
    }
  ]
}

// resources/zh_CN/element/string.json
{
  "string": [
    {
      "name": "app_name",
      "value": "我的应用"
    },
    {
      "name": "welcome_message",
      "value": "欢迎使用我们的应用！"
    },
    {
      "name": "user_greeting",
      "value": "你好，%s！"
    },
    {
      "name": "item_count",
      "value": "您有 %d 个项目"
    }
  ]
}
```

### Using Localized Strings

```typescript
@Component
struct LocalizedComponent {
  @State private userName: string = 'John';
  @State private itemCount: number = 5;

  build() {
    Column({ space: 16 }) {
      Text($r('app.string.app_name'))
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text($r('app.string.welcome_message'))
        .fontSize(18)

      // String with parameters
      Text($r('app.string.user_greeting', this.userName))
        .fontSize(16)

      // Pluralized strings
      Text($r('app.string.item_count', this.itemCount))
        .fontSize(16)
    }
    .padding(20)
  }
}
```

## Locale-Aware Formatting

### Date and Time Formatting

```typescript
import { Locale } from "@ohos.intl";

export class DateTimeFormatter {
  private static getSystemLocale(): string {
    try {
      return Locale.getSystemLocale();
    } catch (error) {
      console.error("Failed to get system locale:", error);
      return "en-US";
    }
  }

  static formatDate(date: Date, options?: Intl.DateTimeFormatOptions): string {
    const locale = this.getSystemLocale();
    const defaultOptions: Intl.DateTimeFormatOptions = {
      year: "numeric",
      month: "long",
      day: "numeric",
    };

    try {
      return new Intl.DateTimeFormat(locale, {
        ...defaultOptions,
        ...options,
      }).format(date);
    } catch (error) {
      console.error("Date formatting failed:", error);
      return date.toLocaleDateString();
    }
  }

  static formatTime(date: Date, options?: Intl.DateTimeFormatOptions): string {
    const locale = this.getSystemLocale();
    const defaultOptions: Intl.DateTimeFormatOptions = {
      hour: "2-digit",
      minute: "2-digit",
    };

    try {
      return new Intl.DateTimeFormat(locale, {
        ...defaultOptions,
        ...options,
      }).format(date);
    } catch (error) {
      console.error("Time formatting failed:", error);
      return date.toLocaleTimeString();
    }
  }

  static formatRelativeTime(date: Date): string {
    const locale = this.getSystemLocale();
    const now = new Date();
    const diffInSeconds = Math.floor((date.getTime() - now.getTime()) / 1000);

    try {
      const rtf = new Intl.RelativeTimeFormat(locale, { numeric: "auto" });

      if (Math.abs(diffInSeconds) < 60) {
        return rtf.format(diffInSeconds, "second");
      } else if (Math.abs(diffInSeconds) < 3600) {
        return rtf.format(Math.floor(diffInSeconds / 60), "minute");
      } else if (Math.abs(diffInSeconds) < 86400) {
        return rtf.format(Math.floor(diffInSeconds / 3600), "hour");
      } else {
        return rtf.format(Math.floor(diffInSeconds / 86400), "day");
      }
    } catch (error) {
      console.error("Relative time formatting failed:", error);
      return date.toLocaleString();
    }
  }
}

// Number formatting
export class NumberFormatter {
  private static getSystemLocale(): string {
    try {
      return Locale.getSystemLocale();
    } catch (error) {
      console.error("Failed to get system locale:", error);
      return "en-US";
    }
  }

  static formatNumber(
    number: number,
    options?: Intl.NumberFormatOptions
  ): string {
    const locale = this.getSystemLocale();

    try {
      return new Intl.NumberFormat(locale, options).format(number);
    } catch (error) {
      console.error("Number formatting failed:", error);
      return number.toString();
    }
  }

  static formatCurrency(amount: number, currency: string = "USD"): string {
    const locale = this.getSystemLocale();

    try {
      return new Intl.NumberFormat(locale, {
        style: "currency",
        currency: currency,
      }).format(amount);
    } catch (error) {
      console.error("Currency formatting failed:", error);
      return `${currency} ${amount}`;
    }
  }

  static formatPercentage(value: number): string {
    const locale = this.getSystemLocale();

    try {
      return new Intl.NumberFormat(locale, {
        style: "percent",
        minimumFractionDigits: 1,
        maximumFractionDigits: 1,
      }).format(value);
    } catch (error) {
      console.error("Percentage formatting failed:", error);
      return `${(value * 100).toFixed(1)}%`;
    }
  }
}
```

## Language Manager

### Dynamic Language Switching

```typescript
export class LanguageManager {
  private static readonly LANGUAGE_KEY = "selected_language";
  private static readonly SUPPORTED_LANGUAGES = [
    "en-US",
    "zh-CN",
    "ja-JP",
    "ko-KR",
  ];
  private static currentLanguage: string = "";
  private static listeners: ((language: string) => void)[] = [];

  static async initialize(): Promise<void> {
    try {
      // Load saved language preference
      const savedLanguage = await PreferencesManager.getString(
        this.LANGUAGE_KEY
      );

      if (savedLanguage && this.SUPPORTED_LANGUAGES.includes(savedLanguage)) {
        this.currentLanguage = savedLanguage;
      } else {
        // Use system locale as default
        const systemLocale = Locale.getSystemLocale();
        this.currentLanguage = this.findBestMatch(systemLocale);
      }

      await this.applyLanguage(this.currentLanguage);
    } catch (error) {
      console.error("Failed to initialize language manager:", error);
      this.currentLanguage = "en-US";
    }
  }

  static async changeLanguage(language: string): Promise<void> {
    if (!this.SUPPORTED_LANGUAGES.includes(language)) {
      throw new Error(`Unsupported language: ${language}`);
    }

    this.currentLanguage = language;

    try {
      await PreferencesManager.setString(this.LANGUAGE_KEY, language);
      await this.applyLanguage(language);
      this.notifyListeners(language);
    } catch (error) {
      console.error("Failed to change language:", error);
      throw error;
    }
  }

  static getCurrentLanguage(): string {
    return this.currentLanguage;
  }

  static getSupportedLanguages(): LanguageOption[] {
    return [
      { code: "en-US", name: "English", nativeName: "English" },
      { code: "zh-CN", name: "Chinese", nativeName: "中文" },
      { code: "ja-JP", name: "Japanese", nativeName: "日本語" },
      { code: "ko-KR", name: "Korean", nativeName: "한국어" },
    ];
  }

  static addLanguageChangeListener(listener: (language: string) => void): void {
    this.listeners.push(listener);
  }

  static removeLanguageChangeListener(
    listener: (language: string) => void
  ): void {
    const index = this.listeners.indexOf(listener);
    if (index > -1) {
      this.listeners.splice(index, 1);
    }
  }

  private static async applyLanguage(language: string): Promise<void> {
    try {
      // Apply language configuration
      // Note: Actual implementation would depend on HarmonyOS APIs
      console.log(`Applied language: ${language}`);
    } catch (error) {
      console.error("Failed to apply language:", error);
      throw error;
    }
  }

  private static findBestMatch(systemLocale: string): string {
    // Direct match
    if (this.SUPPORTED_LANGUAGES.includes(systemLocale)) {
      return systemLocale;
    }

    // Language match (ignore region)
    const languageCode = systemLocale.split("-")[0];
    const match = this.SUPPORTED_LANGUAGES.find((lang) =>
      lang.startsWith(languageCode)
    );

    return match || "en-US";
  }

  private static notifyListeners(language: string): void {
    this.listeners.forEach((listener) => {
      try {
        listener(language);
      } catch (error) {
        console.error("Language change listener error:", error);
      }
    });
  }
}

interface LanguageOption {
  code: string;
  name: string;
  nativeName: string;
}
```

## UI Components for Localization

### Language Selector Component

```typescript
@Component
struct LanguageSelector {
  @State private selectedLanguage: string = '';
  @State private languages: LanguageOption[] = [];
  @State private showDropdown: boolean = false;
  private onLanguageChange?: (language: string) => void;

  aboutToAppear(): void {
    this.languages = LanguageManager.getSupportedLanguages();
    this.selectedLanguage = LanguageManager.getCurrentLanguage();
  }

  build() {
    Column() {
      // Language selector button
      Button() {
        Row() {
          Text(this.getSelectedLanguageName())
            .fontSize(16)
            .layoutWeight(1)

          Image($r('app.media.arrow_down'))
            .width(16)
            .height(16)
        }
        .width('100%')
        .padding({ horizontal: 12, vertical: 8 })
      }
      .backgroundColor(Color.White)
      .border({ width: 1, color: Color.Gray })
      .borderRadius(8)
      .onClick(() => {
        this.showDropdown = !this.showDropdown;
      })

      // Dropdown list
      if (this.showDropdown) {
        Column() {
          ForEach(this.languages, (language: LanguageOption) => {
            Button() {
              Row() {
                Text(language.nativeName)
                  .fontSize(14)
                  .layoutWeight(1)
                  .textAlign(TextAlign.Start)

                if (language.code === this.selectedLanguage) {
                  Image($r('app.media.check'))
                    .width(16)
                    .height(16)
                }
              }
              .width('100%')
              .padding({ horizontal: 12, vertical: 8 })
            }
            .backgroundColor(language.code === this.selectedLanguage ? '#E3F2FD' : Color.White)
            .width('100%')
            .onClick(async () => {
              await this.selectLanguage(language.code);
            })
          })
        }
        .backgroundColor(Color.White)
        .border({ width: 1, color: Color.Gray })
        .borderRadius(8)
        .margin({ top: 4 })
      }
    }
    .width('100%')
  }

  private async selectLanguage(languageCode: string): Promise<void> {
    try {
      await LanguageManager.changeLanguage(languageCode);
      this.selectedLanguage = languageCode;
      this.showDropdown = false;

      if (this.onLanguageChange) {
        this.onLanguageChange(languageCode);
      }
    } catch (error) {
      console.error('Failed to select language:', error);
    }
  }

  private getSelectedLanguageName(): string {
    const language = this.languages.find(lang => lang.code === this.selectedLanguage);
    return language ? language.nativeName : 'Unknown';
  }

  setLanguageChangeHandler(handler: (language: string) => void): LanguageSelector {
    this.onLanguageChange = handler;
    return this;
  }
}
```

### Localized Form Component

```typescript
@Component
struct LocalizedForm {
  @State private formData: FormData = new FormData();
  @State private currentLanguage: string = '';

  aboutToAppear(): void {
    this.currentLanguage = LanguageManager.getCurrentLanguage();

    LanguageManager.addLanguageChangeListener((language: string) => {
      this.currentLanguage = language;
    });
  }

  build() {
    Column({ space: 16 }) {
      Text($r('app.string.form_title'))
        .fontSize(20)
        .fontWeight(FontWeight.Bold)

      // Name input
      Column({ space: 8 }) {
        Text($r('app.string.name_label'))
          .fontSize(14)
          .alignSelf(ItemAlign.Start)

        TextInput({ placeholder: $r('app.string.name_placeholder') })
          .onChange((value: string) => {
            this.formData.name = value;
          })
      }
      .width('100%')
      .alignItems(HorizontalAlign.Start)

      // Email input
      Column({ space: 8 }) {
        Text($r('app.string.email_label'))
          .fontSize(14)
          .alignSelf(ItemAlign.Start)

        TextInput({ placeholder: $r('app.string.email_placeholder') })
          .type(InputType.Email)
          .onChange((value: string) => {
            this.formData.email = value;
          })
      }
      .width('100%')
      .alignItems(HorizontalAlign.Start)

      // Date picker
      Column({ space: 8 }) {
        Text($r('app.string.birthdate_label'))
          .fontSize(14)
          .alignSelf(ItemAlign.Start)

        Button(this.formatSelectedDate())
          .onClick(() => {
            this.showDatePicker();
          })
      }
      .width('100%')
      .alignItems(HorizontalAlign.Start)

      // Submit button
      Button($r('app.string.submit_button'))
        .width('100%')
        .margin({ top: 20 })
        .onClick(() => {
          this.submitForm();
        })
    }
    .padding(20)
  }

  private formatSelectedDate(): string {
    if (this.formData.birthDate) {
      return DateTimeFormatter.formatDate(this.formData.birthDate);
    }
    return $r('app.string.select_date');
  }

  private showDatePicker(): void {
    // Implement date picker
  }

  private submitForm(): void {
    // Implement form submission
  }
}

class FormData {
  name: string = '';
  email: string = '';
  birthDate: Date | null = null;
}
```

## RTL Support

### RTL Layout Handling

```typescript
export class RTLHelper {
  static isRTLLanguage(language: string): boolean {
    const rtlLanguages = ['ar', 'he', 'fa', 'ur'];
    const languageCode = language.split('-')[0];
    return rtlLanguages.includes(languageCode);
  }

  static getLayoutDirection(language: string): 'ltr' | 'rtl' {
    return this.isRTLLanguage(language) ? 'rtl' : 'ltr';
  }

  static getTextAlign(language: string): TextAlign {
    return this.isRTLLanguage(language) ? TextAlign.End : TextAlign.Start;
  }
}

@Component
struct RTLAwareComponent {
  @State private currentLanguage: string = '';
  @State private layoutDirection: 'ltr' | 'rtl' = 'ltr';

  aboutToAppear(): void {
    this.currentLanguage = LanguageManager.getCurrentLanguage();
    this.layoutDirection = RTLHelper.getLayoutDirection(this.currentLanguage);

    LanguageManager.addLanguageChangeListener((language: string) => {
      this.currentLanguage = language;
      this.layoutDirection = RTLHelper.getLayoutDirection(language);
    });
  }

  build() {
    Row() {
      if (this.layoutDirection === 'ltr') {
        this.buildLTRLayout();
      } else {
        this.buildRTLLayout();
      }
    }
    .width('100%')
    .direction(this.layoutDirection === 'rtl' ? Direction.Rtl : Direction.Ltr)
  }

  @Builder
  buildLTRLayout() {
    Image($r('app.media.user_avatar'))
      .width(50)
      .height(50)
      .margin({ right: 12 })

    Column() {
      Text($r('app.string.user_name'))
        .textAlign(TextAlign.Start)
        .width('100%')
      Text($r('app.string.user_status'))
        .textAlign(TextAlign.Start)
        .width('100%')
        .fontSize(12)
        .fontColor(Color.Gray)
    }
    .layoutWeight(1)
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  buildRTLLayout() {
    Column() {
      Text($r('app.string.user_name'))
        .textAlign(TextAlign.End)
        .width('100%')
      Text($r('app.string.user_status'))
        .textAlign(TextAlign.End)
        .width('100%')
        .fontSize(12)
        .fontColor(Color.Gray)
    }
    .layoutWeight(1)
    .alignItems(HorizontalAlign.End)

    Image($r('app.media.user_avatar'))
      .width(50)
      .height(50)
      .margin({ left: 12 })
  }
}
```

## Best Practices

### 1. Resource Organization

- Organize resources by locale and region
- Use consistent naming conventions
- Provide fallback resources for missing translations
- Test with pseudo-locales during development

### 2. Text Handling

- Avoid hardcoded strings in code
- Use placeholders for dynamic content
- Consider text expansion in different languages
- Handle pluralization correctly

### 3. Cultural Considerations

- Respect local date, time, and number formats
- Consider color and symbol meanings
- Handle different text directions (LTR/RTL)
- Adapt icons and images for local markets

### 4. Testing Strategy

- Test with different locales and languages
- Verify layout with longer/shorter text
- Test RTL languages if supported
- Use accessibility features for testing

## Conclusion

Implementing proper internationalization and localization ensures your HarmonyOS application can serve global audiences effectively. By following these practices and using the provided frameworks, you can create applications that feel native to users regardless of their language or cultural background.

Key takeaways:

1. **Plan Early**: Design your application architecture with i18n in mind from the start
2. **Use Resources**: Leverage HarmonyOS resource management for localized content
3. **Format Appropriately**: Use locale-aware formatting for dates, numbers, and currency
4. **Test Thoroughly**: Test with multiple locales and consider cultural differences
5. **Consider RTL**: Plan for right-to-left languages if targeting those markets
