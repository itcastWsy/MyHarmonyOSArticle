# ArkUI Advanced Accessibility Features

## Introduction

Advanced accessibility features ensure ArkUI applications are usable by people with disabilities. This guide covers screen reader integration, voice navigation, gesture accessibility, keyboard navigation, and adaptive interfaces.

## Accessibility Framework

### Core Accessibility Manager

```typescript
interface AccessibilityConfig {
  screenReaderEnabled: boolean;
  highContrastMode: boolean;
  largeTextMode: boolean;
  reducedMotion: boolean;
  voiceNavigationEnabled: boolean;
  keyboardNavigationEnabled: boolean;
  gestureSimplification: boolean;
}

interface AccessibilityEvent {
  type: "focus" | "selection" | "navigation" | "announcement";
  element: string;
  content: string;
  priority: "low" | "medium" | "high" | "urgent";
  timestamp: number;
}

class AccessibilityManager {
  private config: AccessibilityConfig;
  private announcements: string[] = [];
  private focusHistory: string[] = [];
  private eventListeners = new Set<(event: AccessibilityEvent) => void>();

  constructor() {
    this.config = this.detectAccessibilityPreferences();
    this.setupGlobalEventListeners();
  }

  announceToScreenReader(
    message: string,
    priority: "low" | "medium" | "high" | "urgent" = "medium"
  ): void {
    this.announcements.push(message);

    const event: AccessibilityEvent = {
      type: "announcement",
      element: "screen-reader",
      content: message,
      priority,
      timestamp: Date.now(),
    };

    this.notifyListeners(event);
    this.speakMessage(message, priority);
  }

  setFocus(elementId: string, announce: boolean = true): void {
    this.focusHistory.push(elementId);

    if (announce && this.config.screenReaderEnabled) {
      const element = document.getElementById(elementId);
      if (element) {
        const announcement = this.generateFocusAnnouncement(element);
        this.announceToScreenReader(announcement);
      }
    }

    const event: AccessibilityEvent = {
      type: "focus",
      element: elementId,
      content: "",
      priority: "medium",
      timestamp: Date.now(),
    };

    this.notifyListeners(event);
  }

  navigateToElement(direction: "next" | "previous" | "first" | "last"): void {
    const focusableElements = this.getFocusableElements();
    const currentIndex = this.getCurrentFocusIndex(focusableElements);

    let nextIndex: number;
    switch (direction) {
      case "next":
        nextIndex = (currentIndex + 1) % focusableElements.length;
        break;
      case "previous":
        nextIndex =
          currentIndex > 0 ? currentIndex - 1 : focusableElements.length - 1;
        break;
      case "first":
        nextIndex = 0;
        break;
      case "last":
        nextIndex = focusableElements.length - 1;
        break;
    }

    if (focusableElements[nextIndex]) {
      this.setFocus(focusableElements[nextIndex].id);
    }
  }

  updateConfig(updates: Partial<AccessibilityConfig>): void {
    this.config = { ...this.config, ...updates };
    this.applyAccessibilitySettings();
  }

  getConfig(): AccessibilityConfig {
    return { ...this.config };
  }

  onAccessibilityEvent(listener: (event: AccessibilityEvent) => void): void {
    this.eventListeners.add(listener);
  }

  private detectAccessibilityPreferences(): AccessibilityConfig {
    return {
      screenReaderEnabled: this.isScreenReaderActive(),
      highContrastMode: window.matchMedia("(prefers-contrast: high)").matches,
      largeTextMode: window.matchMedia("(prefers-text-size: large)").matches,
      reducedMotion: window.matchMedia("(prefers-reduced-motion: reduce)")
        .matches,
      voiceNavigationEnabled: false,
      keyboardNavigationEnabled: true,
      gestureSimplification: false,
    };
  }

  private isScreenReaderActive(): boolean {
    // Detect screen reader presence
    return (
      ("speechSynthesis" in window && navigator.userAgent.includes("NVDA")) ||
      navigator.userAgent.includes("JAWS") ||
      navigator.userAgent.includes("VoiceOver")
    );
  }

  private setupGlobalEventListeners(): void {
    // Keyboard navigation
    document.addEventListener("keydown", (event) => {
      if (this.config.keyboardNavigationEnabled) {
        this.handleKeyboardNavigation(event);
      }
    });

    // Focus management
    document.addEventListener("focusin", (event) => {
      if (event.target instanceof HTMLElement) {
        this.setFocus(event.target.id, false);
      }
    });
  }

  private handleKeyboardNavigation(event: KeyboardEvent): void {
    switch (event.key) {
      case "Tab":
        if (event.shiftKey) {
          this.navigateToElement("previous");
        } else {
          this.navigateToElement("next");
        }
        break;
      case "Home":
        this.navigateToElement("first");
        event.preventDefault();
        break;
      case "End":
        this.navigateToElement("last");
        event.preventDefault();
        break;
      case "ArrowDown":
        if (event.ctrlKey) {
          this.navigateToElement("next");
          event.preventDefault();
        }
        break;
      case "ArrowUp":
        if (event.ctrlKey) {
          this.navigateToElement("previous");
          event.preventDefault();
        }
        break;
    }
  }

  private getFocusableElements(): HTMLElement[] {
    const selector =
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])';
    return Array.from(document.querySelectorAll(selector)) as HTMLElement[];
  }

  private getCurrentFocusIndex(elements: HTMLElement[]): number {
    const activeElement = document.activeElement as HTMLElement;
    return elements.findIndex((el) => el === activeElement);
  }

  private generateFocusAnnouncement(element: HTMLElement): string {
    const role = element.getAttribute("role") || element.tagName.toLowerCase();
    const label =
      element.getAttribute("aria-label") ||
      element.getAttribute("title") ||
      element.textContent ||
      "unlabeled element";

    return `${role} ${label}`;
  }

  private speakMessage(message: string, priority: string): void {
    if ("speechSynthesis" in window && this.config.screenReaderEnabled) {
      const utterance = new SpeechSynthesisUtterance(message);
      utterance.rate = 1;
      utterance.pitch = 1;
      utterance.volume = priority === "urgent" ? 1 : 0.8;

      if (priority === "urgent") {
        speechSynthesis.cancel(); // Interrupt current speech
      }

      speechSynthesis.speak(utterance);
    }
  }

  private applyAccessibilitySettings(): void {
    document.documentElement.classList.toggle(
      "high-contrast",
      this.config.highContrastMode
    );
    document.documentElement.classList.toggle(
      "large-text",
      this.config.largeTextMode
    );
    document.documentElement.classList.toggle(
      "reduced-motion",
      this.config.reducedMotion
    );
  }

  private notifyListeners(event: AccessibilityEvent): void {
    this.eventListeners.forEach((listener) => listener(event));
  }
}
```

## Accessible Components

### Enhanced Accessible Button

```typescript
@Component
struct AccessibleButton {
  @Prop text: string = ''
  @Prop icon?: string
  @Prop accessibilityLabel?: string
  @Prop accessibilityHint?: string
  @Prop disabled: boolean = false
  @Prop loading: boolean = false
  @State private isFocused: boolean = false
  @State private isPressed: boolean = false

  private onClick?: () => void
  private accessibilityManager = new AccessibilityManager()

  build() {
    Stack() {
      if (this.loading) {
        LoadingProgress()
          .width(20)
          .height(20)
          .color('#FFFFFF')
      } else {
        Row() {
          if (this.icon) {
            Image(this.icon)
              .width(16)
              .height(16)
              .margin({ right: this.text ? 8 : 0 })
          }

          if (this.text) {
            Text(this.text)
              .fontSize(16)
              .fontColor('#FFFFFF')
          }
        }
      }
    }
    .width('auto')
    .height(44)
    .padding({ horizontal: 16 })
    .backgroundColor(this.getBackgroundColor())
    .borderRadius(8)
    .border({
      width: this.isFocused ? 2 : 0,
      color: '#007AFF',
      style: BorderStyle.Solid
    })
    .opacity(this.disabled ? 0.5 : 1)
    .scale({ x: this.isPressed ? 0.95 : 1, y: this.isPressed ? 0.95 : 1 })
    .animation({
      duration: 150,
      curve: Curve.EaseOut
    })
    .focusable(true)
    .onFocus(() => this.handleFocus())
    .onBlur(() => this.handleBlur())
    .onKeyEvent((event) => this.handleKeyEvent(event))
    .onTouch((event) => this.handleTouch(event))
    .onClick(() => this.handleClick())
    .accessibilityGroup(true)
    .accessibilityText(this.getAccessibilityText())
    .accessibilityDescription(this.accessibilityHint || '')
    .accessibilityLevel('button')
  }

  private getBackgroundColor(): string {
    if (this.disabled) return '#8E8E93'
    if (this.isPressed) return '#005BBB'
    if (this.isFocused) return '#0066CC'
    return '#007AFF'
  }

  private getAccessibilityText(): string {
    if (this.accessibilityLabel) return this.accessibilityLabel
    if (this.loading) return `${this.text} loading`
    return this.text
  }

  private handleFocus(): void {
    this.isFocused = true
    this.accessibilityManager.announceToScreenReader(
      `Focused on ${this.getAccessibilityText()}`,
      'low'
    )
  }

  private handleBlur(): void {
    this.isFocused = false
  }

  private handleKeyEvent(event: KeyEvent): void {
    if (event.type === KeyType.Down) {
      if (event.keyCode === KeyCode.KEYCODE_ENTER || event.keyCode === KeyCode.KEYCODE_SPACE) {
        this.handleClick()
        event.stopPropagation()
      }
    }
  }

  private handleTouch(event: TouchEvent): void {
    switch (event.type) {
      case TouchType.Down:
        this.isPressed = true
        break
      case TouchType.Up:
      case TouchType.Cancel:
        this.isPressed = false
        break
    }
  }

  private handleClick(): void {
    if (this.disabled || this.loading) return

    this.accessibilityManager.announceToScreenReader(
      `${this.text} activated`,
      'medium'
    )

    this.onClick?.()
  }
}
```

### Accessible Form Input

```typescript
@Component
struct AccessibleTextInput {
  @Prop label: string = ''
  @Prop placeholder: string = ''
  @Prop value: string = ''
  @Prop errorMessage?: string
  @Prop required: boolean = false
  @Prop type: 'text' | 'password' | 'email' | 'number' = 'text'
  @State private isFocused: boolean = false
  @State private isValid: boolean = true

  private onValueChange?: (value: string) => void
  private accessibilityManager = new AccessibilityManager()

  build() {
    Column() {
      if (this.label) {
        Text(this.label + (this.required ? ' *' : ''))
          .fontSize(14)
          .fontWeight(FontWeight.Medium)
          .margin({ bottom: 8 })
          .accessibilityLevel('label')
      }

      TextInput({
        text: this.value,
        placeholder: this.placeholder
      })
        .type(this.getInputType())
        .fontSize(16)
        .backgroundColor('#FFFFFF')
        .border({
          width: this.getBorderWidth(),
          color: this.getBorderColor(),
          style: BorderStyle.Solid
        })
        .borderRadius(8)
        .padding(12)
        .onChange((value) => this.handleValueChange(value))
        .onFocus(() => this.handleFocus())
        .onBlur(() => this.handleBlur())
        .accessibilityGroup(true)
        .accessibilityText(this.getAccessibilityText())
        .accessibilityDescription(this.getAccessibilityDescription())

      if (this.errorMessage && !this.isValid) {
        Text(this.errorMessage)
          .fontSize(12)
          .fontColor('#FF3B30')
          .margin({ top: 4 })
          .accessibilityLevel('alert')
      }
    }
    .alignItems(HorizontalAlign.Start)
    .width('100%')
  }

  private getInputType(): InputType {
    switch (this.type) {
      case 'password': return InputType.Password
      case 'email': return InputType.Email
      case 'number': return InputType.Number
      default: return InputType.Normal
    }
  }

  private getBorderWidth(): number {
    if (!this.isValid) return 2
    if (this.isFocused) return 2
    return 1
  }

  private getBorderColor(): string {
    if (!this.isValid) return '#FF3B30'
    if (this.isFocused) return '#007AFF'
    return '#E0E0E0'
  }

  private getAccessibilityText(): string {
    let text = this.label
    if (this.required) text += ', required'
    if (this.value) text += `, current value: ${this.value}`
    return text
  }

  private getAccessibilityDescription(): string {
    let description = this.placeholder
    if (this.errorMessage && !this.isValid) {
      description += `. Error: ${this.errorMessage}`
    }
    return description
  }

  private handleValueChange(value: string): void {
    this.isValid = this.validateInput(value)
    this.onValueChange?.(value)
  }

  private handleFocus(): void {
    this.isFocused = true
    this.accessibilityManager.announceToScreenReader(
      `Editing ${this.label}${this.required ? ', required field' : ''}`,
      'low'
    )
  }

  private handleBlur(): void {
    this.isFocused = false
    if (!this.isValid && this.errorMessage) {
      this.accessibilityManager.announceToScreenReader(
        `Validation error: ${this.errorMessage}`,
        'high'
      )
    }
  }

  private validateInput(value: string): boolean {
    if (this.required && !value.trim()) return false
    if (this.type === 'email' && value && !this.isValidEmail(value)) return false
    return true
  }

  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }
}
```

## Voice Navigation System

### Voice Command Handler

```typescript
interface VoiceNavigationConfig {
  enabled: boolean;
  language: string;
  commands: VoiceNavigationCommand[];
  sensitivity: number;
}

interface VoiceNavigationCommand {
  phrase: string[];
  action: string;
  parameters?: string[];
  description: string;
}

class VoiceNavigationManager {
  private config: VoiceNavigationConfig;
  private recognition: SpeechRecognition | null = null;
  private isListening = false;
  private accessibilityManager: AccessibilityManager;

  constructor(accessibilityManager: AccessibilityManager) {
    this.accessibilityManager = accessibilityManager;
    this.config = {
      enabled: false,
      language: "en-US",
      sensitivity: 0.8,
      commands: this.getDefaultCommands(),
    };
  }

  startListening(): void {
    if (!this.config.enabled || this.isListening) return;

    this.recognition = new (window.SpeechRecognition ||
      window.webkitSpeechRecognition)();
    this.recognition.continuous = true;
    this.recognition.interimResults = false;
    this.recognition.lang = this.config.language;

    this.recognition.onresult = (event) => {
      const transcript =
        event.results[event.results.length - 1][0].transcript.toLowerCase();
      this.processVoiceCommand(transcript);
    };

    this.recognition.onerror = (event) => {
      console.error("Voice recognition error:", event.error);
      this.accessibilityManager.announceToScreenReader(
        "Voice navigation error occurred",
        "medium"
      );
    };

    this.recognition.onend = () => {
      if (this.config.enabled) {
        this.recognition?.start(); // Restart listening
      }
    };

    this.recognition.start();
    this.isListening = true;

    this.accessibilityManager.announceToScreenReader(
      'Voice navigation activated. Say "help" for available commands.',
      "medium"
    );
  }

  stopListening(): void {
    if (this.recognition) {
      this.recognition.stop();
      this.recognition = null;
    }
    this.isListening = false;

    this.accessibilityManager.announceToScreenReader(
      "Voice navigation deactivated",
      "medium"
    );
  }

  enable(): void {
    this.config.enabled = true;
    this.startListening();
  }

  disable(): void {
    this.config.enabled = false;
    this.stopListening();
  }

  private processVoiceCommand(transcript: string): void {
    const command = this.matchCommand(transcript);

    if (command) {
      this.executeCommand(command, transcript);
    } else {
      this.accessibilityManager.announceToScreenReader(
        'Command not recognized. Say "help" for available commands.',
        "medium"
      );
    }
  }

  private matchCommand(transcript: string): VoiceNavigationCommand | null {
    for (const command of this.config.commands) {
      for (const phrase of command.phrase) {
        if (transcript.includes(phrase.toLowerCase())) {
          return command;
        }
      }
    }
    return null;
  }

  private executeCommand(
    command: VoiceNavigationCommand,
    transcript: string
  ): void {
    switch (command.action) {
      case "navigate_next":
        this.accessibilityManager.navigateToElement("next");
        this.accessibilityManager.announceToScreenReader(
          "Moved to next element",
          "medium"
        );
        break;

      case "navigate_previous":
        this.accessibilityManager.navigateToElement("previous");
        this.accessibilityManager.announceToScreenReader(
          "Moved to previous element",
          "medium"
        );
        break;

      case "activate":
        this.activateCurrentElement();
        break;

      case "go_home":
        this.accessibilityManager.navigateToElement("first");
        this.accessibilityManager.announceToScreenReader(
          "Moved to first element",
          "medium"
        );
        break;

      case "help":
        this.announceHelp();
        break;

      case "read_page":
        this.readPageContent();
        break;

      case "find_text":
        const searchText = this.extractSearchText(transcript);
        if (searchText) {
          this.findTextOnPage(searchText);
        }
        break;
    }
  }

  private activateCurrentElement(): void {
    const activeElement = document.activeElement as HTMLElement;
    if (activeElement) {
      if (activeElement.tagName === "BUTTON" || activeElement.tagName === "A") {
        activeElement.click();
        this.accessibilityManager.announceToScreenReader(
          "Element activated",
          "medium"
        );
      } else {
        this.accessibilityManager.announceToScreenReader(
          "Element cannot be activated",
          "medium"
        );
      }
    }
  }

  private announceHelp(): void {
    const helpText = this.config.commands
      .map((cmd) => `Say "${cmd.phrase[0]}" to ${cmd.description}`)
      .join(". ");

    this.accessibilityManager.announceToScreenReader(
      `Available commands: ${helpText}`,
      "medium"
    );
  }

  private readPageContent(): void {
    const mainContent = document.querySelector("main") || document.body;
    const textContent = mainContent.textContent?.trim() || "No content found";

    this.accessibilityManager.announceToScreenReader(textContent, "low");
  }

  private extractSearchText(transcript: string): string | null {
    const findMatch =
      transcript.match(/find (.+)/) || transcript.match(/search for (.+)/);
    return findMatch ? findMatch[1] : null;
  }

  private findTextOnPage(searchText: string): void {
    const walker = document.createTreeWalker(
      document.body,
      NodeFilter.SHOW_TEXT,
      null,
      false
    );

    let node;
    while ((node = walker.nextNode())) {
      if (node.textContent?.toLowerCase().includes(searchText.toLowerCase())) {
        const element = node.parentElement;
        if (element) {
          element.scrollIntoView({ behavior: "smooth", block: "center" });
          element.focus();
          this.accessibilityManager.announceToScreenReader(
            `Found "${searchText}" in ${element.tagName.toLowerCase()}`,
            "medium"
          );
          return;
        }
      }
    }

    this.accessibilityManager.announceToScreenReader(
      `Text "${searchText}" not found on page`,
      "medium"
    );
  }

  private getDefaultCommands(): VoiceNavigationCommand[] {
    return [
      {
        phrase: ["next", "move next", "go forward"],
        action: "navigate_next",
        description: "move to next element",
      },
      {
        phrase: ["previous", "move previous", "go back"],
        action: "navigate_previous",
        description: "move to previous element",
      },
      {
        phrase: ["activate", "click", "select", "choose"],
        action: "activate",
        description: "activate current element",
      },
      {
        phrase: ["home", "go home", "start", "beginning"],
        action: "go_home",
        description: "go to first element",
      },
      {
        phrase: ["help", "commands", "what can I say"],
        action: "help",
        description: "hear available commands",
      },
      {
        phrase: ["read page", "read content", "read all"],
        action: "read_page",
        description: "read page content",
      },
      {
        phrase: ["find", "search for"],
        action: "find_text",
        description: "find text on page",
      },
    ];
  }
}
```

## Conclusion

Advanced accessibility features in ArkUI applications provide:

- Comprehensive screen reader support with intelligent announcements
- Voice navigation capabilities for hands-free interaction
- Enhanced keyboard navigation with focus management
- Accessible component design patterns and best practices
- Adaptive interfaces that respond to user accessibility preferences
- Real-time accessibility event monitoring and feedback

These features ensure that ArkUI applications are inclusive and usable by all users, regardless of their abilities or assistive technology requirements.
