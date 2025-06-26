# Common Carousel Solutions

## Understanding Carousel Components

### 🎯 What is a Carousel?

A carousel is a UI component that displays multiple items in a rotating fashion, allowing users to:

- **Browse through content**: Images, cards, or any UI elements
- **Navigate** using swipe gestures, pagination dots, or arrows
- **Auto-play** content with timed transitions
- **Loop infinitely** or have bounded navigation

### 🎮 Common Use Cases

- **Image galleries**: Photo browsers and slideshows
- **Product showcases**: E-commerce product displays
- **Banner advertisements**: Promotional content rotation
- **Feature highlights**: App feature demonstrations
- **News feeds**: Article or story browsing
- **Onboarding flows**: App introduction screens

---

## Basic Carousel Implementation

### 🎯 Simple Swiper Carousel

```typescript
@Component
struct BasicCarousel {
  @State currentIndex: number = 0;
  @State images: string[] = [
    "https://example.com/image1.jpg",
    "https://example.com/image2.jpg",
    "https://example.com/image3.jpg",
    "https://example.com/image4.jpg"
  ];

  build() {
    Column({ space: 16 }) {
      // Main carousel
      Swiper() {
        ForEach(this.images, (imageUrl: string, index: number) => {
          Image(imageUrl)
            .width("100%")
            .height(200)
            .borderRadius(12)
            .objectFit(ImageFit.Cover);
        }, (imageUrl: string) => imageUrl);
      }
      .width("100%")
      .height(200)
      .autoPlay(true)
      .interval(3000)
      .indicator(true)
      .loop(true)
      .onChange((index: number) => {
        this.currentIndex = index;
      });

      // Page indicator
      Row({ space: 8 }) {
        ForEach(this.images, (_, index: number) => {
          Circle({ width: 8, height: 8 })
            .fill(this.currentIndex === index ? Color.Blue : Color.Gray)
            .animation({ duration: 200 });
        }, (_, index: number) => index.toString());
      }
      .justifyContent(FlexAlign.Center);

      // Current page info
      Text(`${this.currentIndex + 1} / ${this.images.length}`)
        .fontSize(14)
        .fontColor(Color.Gray);
    }
    .padding(16);
  }
}
```

### 🎯 Card-based Carousel

```typescript
interface CarouselItem {
  id: string;
  title: string;
  subtitle: string;
  imageUrl: string;
  backgroundColor: Color;
}

@Component
struct CardCarousel {
  @State currentIndex: number = 0;
  @State items: CarouselItem[] = [
    {
      id: "1",
      title: "Summer Sale",
      subtitle: "Up to 50% off on selected items",
      imageUrl: "https://example.com/summer.jpg",
      backgroundColor: Color.Orange
    },
    {
      id: "2",
      title: "New Arrivals",
      subtitle: "Fresh styles for the new season",
      imageUrl: "https://example.com/arrivals.jpg",
      backgroundColor: Color.Blue
    },
    {
      id: "3",
      title: "Premium Collection",
      subtitle: "Luxury items at great prices",
      imageUrl: "https://example.com/premium.jpg",
      backgroundColor: Color.Purple
    }
  ];

  build() {
    Column({ space: 20 }) {
      // Card carousel
      Swiper() {
        ForEach(this.items, (item: CarouselItem) => {
          this.buildCarouselCard(item);
        }, (item: CarouselItem) => item.id);
      }
      .width("100%")
      .height(250)
      .itemSpace(16)
      .displayCount(1.2) // Show partial next card
      .autoPlay(false)
      .loop(true)
      .onChange((index: number) => {
        this.currentIndex = index;
      });

      // Custom pagination
      this.buildPagination();
    }
    .padding(16);
  }

  @Builder
  buildCarouselCard(item: CarouselItem) {
    Stack({ alignContent: Alignment.BottomStart }) {
      // Background image
      Image(item.imageUrl)
        .width("100%")
        .height("100%")
        .borderRadius(16)
        .objectFit(ImageFit.Cover);

      // Gradient overlay
      Column()
        .width("100%")
        .height("100%")
        .borderRadius(16)
        .linearGradient({
          colors: [
            [Color.Transparent, 0.0],
            [Color.Black.opacity(0.7), 1.0]
          ],
          direction: GradientDirection.TopToBottom
        });

      // Card content
      Column({ space: 8 }) {
        Text(item.title)
          .fontSize(24)
          .fontWeight(FontWeight.Bold)
          .fontColor(Color.White)
          .maxLines(1);

        Text(item.subtitle)
          .fontSize(16)
          .fontColor(Color.White)
          .opacity(0.9)
          .maxLines(2);

        Button("Learn More")
          .type(ButtonType.Capsule)
          .backgroundColor(Color.White)
          .fontColor(item.backgroundColor)
          .fontSize(14)
          .padding({ left: 16, right: 16, top: 8, bottom: 8 });
      }
      .alignItems(HorizontalAlign.Start)
      .padding(20);
    }
    .width("100%")
    .height("100%")
    .backgroundColor(item.backgroundColor)
    .borderRadius(16)
    .shadow({ radius: 8, color: Color.Black.opacity(0.3), offsetY: 4 });
  }

  @Builder
  buildPagination() {
    Row({ space: 12 }) {
      ForEach(this.items, (_, index: number) => {
        if (index === this.currentIndex) {
          // Active indicator
          Capsule({ width: 24, height: 8 })
            .fill(Color.Blue);
        } else {
          // Inactive indicator
          Circle({ width: 8, height: 8 })
            .fill(Color.Gray);
        }
      }, (_, index: number) => index.toString());
    }
    .justifyContent(FlexAlign.Center)
    .animation({ duration: 300, curve: Curve.EaseInOut });
  }
}
```

---

## Advanced Carousel Features

### 🎯 Auto-playing Carousel with Controls

```typescript
@Component
struct AutoPlayCarousel {
  @State currentIndex: number = 0;
  @State isAutoPlaying: boolean = true;
  @State autoPlayInterval: number = 4000;
  private swiperController: SwiperController = new SwiperController();
  private autoPlayTimer: number = -1;

  aboutToAppear() {
    this.startAutoPlay();
  }

  aboutToDisappear() {
    this.stopAutoPlay();
  }

  build() {
    Column({ space: 16 }) {
      // Control buttons
      Row({ space: 16 }) {
        Button(this.isAutoPlaying ? "Pause" : "Play")
          .type(ButtonType.Capsule)
          .onClick(() => {
            this.toggleAutoPlay();
          });

        Button("Previous")
          .type(ButtonType.Capsule)
          .onClick(() => {
            this.swiperController.showPrevious();
          });

        Button("Next")
          .type(ButtonType.Capsule)
          .onClick(() => {
            this.swiperController.showNext();
          });
      }
      .justifyContent(FlexAlign.Center);

      // Main carousel
      Swiper(this.swiperController) {
        ForEach(this.items, (item: CarouselItem) => {
          this.buildCarouselItem(item);
        }, (item: CarouselItem) => item.id);
      }
      .width("100%")
      .height(300)
      .autoPlay(false) // Control manually
      .loop(true)
      .onChange((index: number) => {
        this.currentIndex = index;
        this.onSlideChange();
      })
      .onTouch((event: TouchEvent) => {
        if (event.type === TouchType.Down) {
          this.pauseAutoPlayTemporarily();
        }
      });

      // Progress bar
      this.buildProgressBar();
    }
    .padding(16);
  }

  private toggleAutoPlay(): void {
    if (this.isAutoPlaying) {
      this.stopAutoPlay();
    } else {
      this.startAutoPlay();
    }
    this.isAutoPlaying = !this.isAutoPlaying;
  }

  private startAutoPlay(): void {
    if (this.autoPlayTimer !== -1) return;

    this.autoPlayTimer = setInterval(() => {
      this.swiperController.showNext();
    }, this.autoPlayInterval);
  }

  private stopAutoPlay(): void {
    if (this.autoPlayTimer !== -1) {
      clearInterval(this.autoPlayTimer);
      this.autoPlayTimer = -1;
    }
  }

  private pauseAutoPlayTemporarily(): void {
    this.stopAutoPlay();
    setTimeout(() => {
      if (this.isAutoPlaying) {
        this.startAutoPlay();
      }
    }, 3000); // Resume after 3 seconds
  }

  private onSlideChange(): void {
    // Reset auto-play timer on manual navigation
    if (this.isAutoPlaying) {
      this.stopAutoPlay();
      this.startAutoPlay();
    }
  }
}
```

## Best Practices Summary

### 🎯 Performance Guidelines

- [ ] Use lazy loading for large datasets
- [ ] Implement proper image caching
- [ ] Limit the number of simultaneously rendered items
- [ ] Use `@Reusable` components when appropriate
- [ ] Optimize animations and transitions
- [ ] Handle memory cleanup in `aboutToDisappear`

### 🎯 User Experience Tips

- [ ] Provide clear navigation indicators
- [ ] Support both touch and programmatic navigation
- [ ] Implement smooth animations (300ms recommended)
- [ ] Add haptic feedback for interactions
- [ ] Handle edge cases (empty states, single items)
- [ ] Support accessibility features
- [ ] Test on different screen sizes

### 🎯 Accessibility Considerations

- [ ] Add proper ARIA labels for screen readers
- [ ] Support keyboard navigation
- [ ] Provide alternative text for images
- [ ] Ensure sufficient color contrast
- [ ] Support voice-over navigation
- [ ] Implement focus management

With these carousel solutions, you can create engaging and performant content browsers for your HarmonyOS applications!
