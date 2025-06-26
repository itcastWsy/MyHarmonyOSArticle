# Text Expand and Collapse

## Understanding Text Expand/Collapse Components

### 🎯 What is Text Expand/Collapse?

Text expand/collapse is a UI pattern that allows:

- **Show limited text initially**: Display only a portion of long content
- **Progressive disclosure**: Let users choose to see more content
- **Space optimization**: Maintain clean layouts while providing full content
- **Improved readability**: Reduce cognitive load and visual clutter

### 🎮 Common Use Cases

- **Article previews**: Blog posts and news articles
- **Product descriptions**: E-commerce detailed descriptions
- **Comments and reviews**: User-generated content in social apps
- **Terms and conditions**: Legal text and policy documents
- **FAQ sections**: Question and answer interfaces
- **Bio and profile descriptions**: User profile information

---

## Basic Text Expand/Collapse Implementation

### 🎯 Simple Expand/Collapse Text

```typescript
@Component
struct BasicExpandableText {
  @State isExpanded: boolean = false;
  @State fullText: string = "This is a very long text that needs to be truncated when collapsed. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.";
  @State maxLines: number = 3;

  build() {
    Column({ space: 8 }) {
      // Main text content
      Text(this.fullText)
        .fontSize(16)
        .lineHeight(24)
        .fontColor(Color.Black)
        .maxLines(this.isExpanded ? undefined : this.maxLines)
        .textOverflow({ overflow: TextOverflow.Ellipsis })
        .animation({ duration: 300, curve: Curve.EaseInOut });

      // Expand/collapse button
      if (this.shouldShowToggleButton()) {
        Button(this.isExpanded ? "Show Less" : "Show More")
          .type(ButtonType.Capsule)
          .fontSize(14)
          .fontColor(Color.Blue)
          .backgroundColor(Color.Transparent)
          .padding({ left: 0, right: 0, top: 4, bottom: 4 })
          .onClick(() => {
            this.toggleExpansion();
          });
      }
    }
    .alignItems(HorizontalAlign.Start)
    .padding(16);
  }

  private shouldShowToggleButton(): boolean {
    // In a real implementation, you would measure text height
    // to determine if it exceeds the maxLines
    return this.fullText.length > 100; // Simplified check
  }

  private toggleExpansion(): void {
    animateTo({
      duration: 300,
      curve: Curve.EaseInOut
    }, () => {
      this.isExpanded = !this.isExpanded;
    });
  }
}
```

### 🎯 Advanced Expandable Text with Animation

```typescript
@Component
struct AnimatedExpandableText {
  @State isExpanded: boolean = false;
  @State textHeight: number = 0;
  @State collapsedHeight: number = 72; // Height for 3 lines
  @State fullHeight: number = 0;
  @State content: string = "";
  @State maxCollapsedLines: number = 3;

  build() {
    Column({ space: 12 }) {
      // Text container with animated height
      Column() {
        Text(this.content)
          .fontSize(16)
          .lineHeight(24)
          .fontColor(Color.Black)
          .width("100%")
          .onAreaChange((oldValue: Area, newValue: Area) => {
            if (!this.isExpanded) {
              this.fullHeight = Number(newValue.height);
            }
          });
      }
      .width("100%")
      .height(this.isExpanded ? this.fullHeight : this.collapsedHeight)
      .clip(true)
      .animation({
        duration: 400,
        curve: Curve.EaseInOut
      });

      // Gradient overlay when collapsed
      if (!this.isExpanded && this.needsExpansion()) {
        Stack() {
          Rectangle()
            .width("100%")
            .height(24)
            .linearGradient({
              colors: [
                [Color.White.opacity(0), 0.0],
                [Color.White, 1.0]
              ],
              direction: GradientDirection.TopToBottom
            })
            .position({ x: 0, y: this.collapsedHeight - 24 });
        }
        .width("100%")
        .height(24);
      }

      // Toggle button with icon
      if (this.needsExpansion()) {
        Row({ space: 8 }) {
          Text(this.isExpanded ? "Show Less" : "Show More")
            .fontSize(14)
            .fontColor(Color.Blue);

          Text(this.isExpanded ? "↑" : "↓")
            .fontSize(14)
            .fontColor(Color.Blue);
        }
        .onClick(() => {
          this.toggleExpansion();
        })
        .padding({ top: 8, bottom: 8 });
      }
    }
    .alignItems(HorizontalAlign.Start)
    .padding(16)
    .width("100%");
  }

  private needsExpansion(): boolean {
    return this.fullHeight > this.collapsedHeight;
  }

  private toggleExpansion(): void {
    animateTo({
      duration: 400,
      curve: Curve.EaseInOut
    }, () => {
      this.isExpanded = !this.isExpanded;
    });
  }
}
```

## Best Practices Summary

### 🎯 Implementation Guidelines

- [ ] Provide clear visual indicators for expandable content
- [ ] Use smooth animations for expand/collapse transitions
- [ ] Implement proper text measurement for truncation
- [ ] Add fade effects for better visual continuity
- [ ] Support keyboard navigation for accessibility
- [ ] Handle long content with lazy loading
- [ ] Provide "Show Less" option when expanded

### 🎯 User Experience Tips

- [ ] Keep preview text meaningful and engaging
- [ ] Use consistent animation timing (300-400ms)
- [ ] Provide estimated read times for long content
- [ ] Show progress indicators for lengthy articles
- [ ] Support both tap and swipe gestures
- [ ] Remember user preferences for expansion state
- [ ] Test with various content lengths

### 🎯 Performance Considerations

- [ ] Implement lazy loading for very long content
- [ ] Use text virtualization for extremely large texts
- [ ] Cache expanded content to avoid re-loading
- [ ] Optimize animations for smooth performance
- [ ] Consider memory usage with large text blocks
- [ ] Implement proper cleanup in `aboutToDisappear`

### 🎯 Accessibility Features

- [ ] Add proper ARIA labels and roles
- [ ] Support screen reader announcements
- [ ] Provide keyboard shortcuts for expand/collapse
- [ ] Ensure sufficient color contrast
- [ ] Add semantic markup for content structure
- [ ] Support voice navigation

With these text expand/collapse patterns, you can create intuitive and efficient content presentation in your HarmonyOS applications!
