# Image Previewer

## Understanding Image Preview Components

### 🎯 What is an Image Previewer?

An image previewer is a specialized UI component that provides:

- **Full-screen image viewing**: Display images in maximum detail
- **Zoom and pan capabilities**: Allow users to examine images closely
- **Gallery navigation**: Browse through multiple images seamlessly
- **Rich interactions**: Pinch-to-zoom, double-tap, rotation support
- **Loading states**: Handle image loading and error scenarios

### 🎮 Common Use Cases

- **Photo galleries**: Social media and photo sharing apps
- **E-commerce**: Product detail views and image galleries
- **Document viewers**: PDF pages, scanned documents
- **Media browsers**: File managers and photo organizers
- **Social feeds**: Instagram-style image viewing
- **Avatar and profile picture selection**

---

## Basic Image Previewer Implementation

### 🎯 Simple Image Viewer

```typescript
@Component
struct BasicImagePreviewer {
  @State isVisible: boolean = false;
  @State currentImageUrl: string = "";
  @State imageScale: number = 1.0;
  @State imageOffsetX: number = 0;
  @State imageOffsetY: number = 0;

  build() {
    Stack() {
      // Main content
      Column({ space: 16 }) {
        Text("Image Gallery")
          .fontSize(24)
          .fontWeight(FontWeight.Bold);

        // Thumbnail grid
        Grid() {
          ForEach(this.imageUrls, (imageUrl: string) => {
            GridItem() {
              Image(imageUrl)
                .width("100%")
                .height(120)
                .borderRadius(8)
                .objectFit(ImageFit.Cover)
                .onClick(() => {
                  this.openImagePreviewer(imageUrl);
                });
            }
          }, (imageUrl: string) => imageUrl);
        }
        .columnsTemplate("1fr 1fr 1fr")
        .rowsGap(8)
        .columnsGap(8)
        .padding(16);
      }

      // Image previewer overlay
      if (this.isVisible) {
        this.buildImagePreviewerOverlay();
      }
    }
    .width("100%")
    .height("100%");
  }

  @Builder
  buildImagePreviewerOverlay() {
    Stack() {
      // Background overlay
      Column()
        .width("100%")
        .height("100%")
        .backgroundColor(Color.Black.opacity(0.9))
        .onClick(() => {
          this.closeImagePreviewer();
        });

      // Main image
      Image(this.currentImageUrl)
        .width("90%")
        .height("80%")
        .objectFit(ImageFit.Contain)
        .scale({ x: this.imageScale, y: this.imageScale })
        .translate({ x: this.imageOffsetX, y: this.imageOffsetY })
        .gesture(
          // Pinch gesture for zoom
          PinchGesture()
            .onActionStart(() => {
              console.log("Pinch start");
            })
            .onActionUpdate((event: GestureEvent) => {
              this.imageScale = Math.max(0.5, Math.min(3.0, event.scale));
            })
            .onActionEnd(() => {
              if (this.imageScale < 1.0) {
                this.resetImageTransform();
              }
            })
        )
        .gesture(
          // Pan gesture for moving
          PanGesture()
            .onActionUpdate((event: GestureEvent) => {
              if (this.imageScale > 1.0) {
                this.imageOffsetX += event.offsetX;
                this.imageOffsetY += event.offsetY;
              }
            })
        )
        .gesture(
          // Double tap to zoom
          TapGesture({ count: 2 })
            .onAction(() => {
              this.toggleZoom();
            })
        );

      // Close button
      Button("×")
        .fontSize(30)
        .fontColor(Color.White)
        .backgroundColor(Color.Transparent)
        .position({ x: "90%", y: "5%" })
        .onClick(() => {
          this.closeImagePreviewer();
        });
    }
    .width("100%")
    .height("100%")
    .zIndex(999);
  }

  private openImagePreviewer(imageUrl: string): void {
    this.currentImageUrl = imageUrl;
    this.isVisible = true;
    this.resetImageTransform();
  }

  private closeImagePreviewer(): void {
    this.isVisible = false;
    this.resetImageTransform();
  }

  private resetImageTransform(): void {
    animateTo({ duration: 300 }, () => {
      this.imageScale = 1.0;
      this.imageOffsetX = 0;
      this.imageOffsetY = 0;
    });
  }

  private toggleZoom(): void {
    animateTo({ duration: 300 }, () => {
      if (this.imageScale === 1.0) {
        this.imageScale = 2.0;
      } else {
        this.resetImageTransform();
      }
    });
  }
}
```

### 🎯 Gallery Image Previewer with Navigation

```typescript
interface GalleryImage {
  id: string;
  url: string;
  title?: string;
  description?: string;
}

@Component
struct GalleryImagePreviewer {
  @State isVisible: boolean = false;
  @State currentIndex: number = 0;
  @State images: GalleryImage[] = [];
  @State isLoading: boolean = false;
  @State loadError: boolean = false;
  @State showInfo: boolean = false;

  // Transform states
  @State imageScale: number = 1.0;
  @State imageOffsetX: number = 0;
  @State imageOffsetY: number = 0;

  build() {
    Stack() {
      // Main gallery grid
      this.buildGalleryGrid();

      // Image previewer modal
      if (this.isVisible) {
        this.buildImagePreviewerModal();
      }
    }
  }

  @Builder
  buildGalleryGrid() {
    Column({ space: 16 }) {
      Text("Photo Gallery")
        .fontSize(24)
        .fontWeight(FontWeight.Bold);

      Grid() {
        ForEach(this.images, (image: GalleryImage, index: number) => {
          GridItem() {
            Stack() {
              Image(image.url)
                .width("100%")
                .height(120)
                .borderRadius(8)
                .objectFit(ImageFit.Cover);

              // Overlay with index
              Text(`${index + 1}`)
                .fontSize(12)
                .fontColor(Color.White)
                .backgroundColor(Color.Black.opacity(0.6))
                .padding(4)
                .borderRadius(4)
                .position({ x: "5%", y: "5%" });
            }
            .onClick(() => {
              this.openGallery(index);
            });
          }
        }, (image: GalleryImage) => image.id);
      }
      .columnsTemplate("1fr 1fr 1fr")
      .rowsGap(8)
      .columnsGap(8)
      .padding(16);
    }
  }

  @Builder
  buildImagePreviewerModal() {
    Stack() {
      // Background
      Column()
        .width("100%")
        .height("100%")
        .backgroundColor(Color.Black)
        .onClick(() => {
          if (this.imageScale === 1.0) {
            this.closeGallery();
          }
        });

      // Main image container
      Stack() {
        if (this.isLoading) {
          LoadingProgress()
            .width(50)
            .height(50)
            .color(Color.White);
        } else if (this.loadError) {
          Column({ space: 8 }) {
            Text("Failed to load image")
              .fontColor(Color.White)
              .fontSize(16);
            Button("Retry")
              .onClick(() => {
                this.retryLoadImage();
              });
          }
        } else {
          Image(this.getCurrentImage().url)
            .width("100%")
            .height("100%")
            .objectFit(ImageFit.Contain)
            .scale({ x: this.imageScale, y: this.imageScale })
            .translate({ x: this.imageOffsetX, y: this.imageOffsetY })
            .onComplete(() => {
              this.isLoading = false;
              this.loadError = false;
            })
            .onError(() => {
              this.isLoading = false;
              this.loadError = true;
            })
            .gesture(
              PinchGesture()
                .onActionUpdate((event: GestureEvent) => {
                  this.imageScale = Math.max(0.5, Math.min(5.0, event.scale));
                })
                .onActionEnd(() => {
                  if (this.imageScale < 1.0) {
                    this.resetTransform();
                  }
                })
            )
            .gesture(
              PanGesture()
                .onActionUpdate((event: GestureEvent) => {
                  if (this.imageScale > 1.0) {
                    this.imageOffsetX += event.offsetX;
                    this.imageOffsetY += event.offsetY;
                  } else {
                    // Handle swipe navigation when not zoomed
                    const threshold = 50;
                    if (Math.abs(event.offsetX) > threshold) {
                      if (event.offsetX > 0 && this.currentIndex > 0) {
                        this.navigateToPrevious();
                      } else if (event.offsetX < 0 && this.currentIndex < this.images.length - 1) {
                        this.navigateToNext();
                      }
                    }
                  }
                })
            )
            .gesture(
              TapGesture({ count: 2 })
                .onAction(() => {
                  this.toggleZoom();
                })
            );
        }
      }
      .width("90%")
      .height("80%");

      // Top controls
      Row() {
        Button("×")
          .fontSize(24)
          .fontColor(Color.White)
          .backgroundColor(Color.Transparent)
          .onClick(() => {
            this.closeGallery();
          });

        Blank();

        Text(`${this.currentIndex + 1} / ${this.images.length}`)
          .fontColor(Color.White)
          .fontSize(16);

        Blank();

        Button("Info")
          .fontSize(14)
          .fontColor(Color.White)
          .backgroundColor(Color.Transparent)
          .onClick(() => {
            this.showInfo = !this.showInfo;
          });
      }
      .width("100%")
      .padding(16)
      .position({ x: 0, y: 0 });

      // Navigation arrows
      if (this.currentIndex > 0) {
        Button("‹")
          .fontSize(30)
          .fontColor(Color.White)
          .backgroundColor(Color.Black.opacity(0.5))
          .borderRadius(20)
          .position({ x: "5%", y: "50%" })
          .onClick(() => {
            this.navigateToPrevious();
          });
      }

      if (this.currentIndex < this.images.length - 1) {
        Button("›")
          .fontSize(30)
          .fontColor(Color.White)
          .backgroundColor(Color.Black.opacity(0.5))
          .borderRadius(20)
          .position({ x: "90%", y: "50%" })
          .onClick(() => {
            this.navigateToNext();
          });
      }

      // Image info panel
      if (this.showInfo) {
        this.buildImageInfoPanel();
      }
    }
    .width("100%")
    .height("100%")
    .zIndex(999);
  }

  @Builder
  buildImageInfoPanel() {
    Column({ space: 8 }) {
      if (this.getCurrentImage().title) {
        Text(this.getCurrentImage().title!)
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .fontColor(Color.White);
      }

      if (this.getCurrentImage().description) {
        Text(this.getCurrentImage().description!)
          .fontSize(14)
          .fontColor(Color.White)
          .opacity(0.8);
      }

      Text(`Image ${this.currentIndex + 1} of ${this.images.length}`)
        .fontSize(12)
        .fontColor(Color.White)
        .opacity(0.6);
    }
    .padding(16)
    .backgroundColor(Color.Black.opacity(0.7))
    .borderRadius(8)
    .position({ x: "5%", y: "85%" })
    .width("90%");
  }

  private getCurrentImage(): GalleryImage {
    return this.images[this.currentIndex] || { id: "", url: "" };
  }

  private openGallery(index: number): void {
    this.currentIndex = index;
    this.isVisible = true;
    this.isLoading = true;
    this.resetTransform();
  }

  private closeGallery(): void {
    this.isVisible = false;
    this.showInfo = false;
    this.resetTransform();
  }

  private navigateToPrevious(): void {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      this.isLoading = true;
      this.resetTransform();
    }
  }

  private navigateToNext(): void {
    if (this.currentIndex < this.images.length - 1) {
      this.currentIndex++;
      this.isLoading = true;
      this.resetTransform();
    }
  }

  private resetTransform(): void {
    animateTo({ duration: 300 }, () => {
      this.imageScale = 1.0;
      this.imageOffsetX = 0;
      this.imageOffsetY = 0;
    });
  }

  private toggleZoom(): void {
    animateTo({ duration: 300 }, () => {
      if (this.imageScale === 1.0) {
        this.imageScale = 2.0;
      } else {
        this.resetTransform();
      }
    });
  }

  private retryLoadImage(): void {
    this.isLoading = true;
    this.loadError = false;
  }
}
```

---

## Advanced Features

### 🎯 Image Previewer with Download and Share

```typescript
@Component
struct AdvancedImagePreviewer {
  @State showActionSheet: boolean = false;
  @State isDownloading: boolean = false;

  @Builder
  buildActionSheet() {
    Column({ space: 0 }) {
      Button("Download Image")
        .width("100%")
        .backgroundColor(Color.Transparent)
        .fontColor(Color.Blue)
        .onClick(() => {
          this.downloadImage();
          this.showActionSheet = false;
        });

      Divider();

      Button("Share Image")
        .width("100%")
        .backgroundColor(Color.Transparent)
        .fontColor(Color.Blue)
        .onClick(() => {
          this.shareImage();
          this.showActionSheet = false;
        });

      Divider();

      Button("Copy Link")
        .width("100%")
        .backgroundColor(Color.Transparent)
        .fontColor(Color.Blue)
        .onClick(() => {
          this.copyImageLink();
          this.showActionSheet = false;
        });

      Divider();

      Button("Cancel")
        .width("100%")
        .backgroundColor(Color.Transparent)
        .fontColor(Color.Red)
        .onClick(() => {
          this.showActionSheet = false;
        });
    }
    .backgroundColor(Color.White)
    .borderRadius(12)
    .padding(16)
    .margin(16);
  }

  private async downloadImage(): Promise<void> {
    this.isDownloading = true;

    try {
      const imageUrl = this.getCurrentImage().url;
      // Implement actual download logic here
      await this.downloadImageFromUrl(imageUrl);

      // Show success message
      this.showToast("Image downloaded successfully");
    } catch (error) {
      this.showToast("Failed to download image");
    } finally {
      this.isDownloading = false;
    }
  }

  private shareImage(): void {
    // Implement sharing functionality
    const imageUrl = this.getCurrentImage().url;
    // Use system sharing capabilities
  }

  private copyImageLink(): void {
    // Copy image URL to clipboard
    const imageUrl = this.getCurrentImage().url;
    // Implement clipboard copy
    this.showToast("Link copied to clipboard");
  }

  private showToast(message: string): void {
    // Implement toast notification
    console.log(message);
  }

  private async downloadImageFromUrl(url: string): Promise<void> {
    // Implement actual download logic
    return new Promise((resolve) => {
      setTimeout(resolve, 2000); // Simulate download
    });
  }
}
```

## Best Practices Summary

### 🎯 Performance Guidelines

- [ ] Implement lazy loading for large galleries
- [ ] Use image caching for better performance
- [ ] Optimize image sizes and formats
- [ ] Handle memory management properly
- [ ] Use progressive loading for large images
- [ ] Implement efficient gesture handling

### 🎯 User Experience Tips

- [ ] Provide smooth zoom and pan interactions
- [ ] Support common gestures (pinch, double-tap, swipe)
- [ ] Show loading states for slow networks
- [ ] Handle error states gracefully
- [ ] Add image information and metadata
- [ ] Support full-screen mode
- [ ] Implement keyboard navigation support

### 🎯 Accessibility Features

- [ ] Add proper alt text for images
- [ ] Support screen reader navigation
- [ ] Provide keyboard shortcuts
- [ ] Ensure sufficient contrast for controls
- [ ] Add haptic feedback for interactions
- [ ] Support voice-over descriptions

With these image previewer implementations, you can create rich and engaging image viewing experiences in your HarmonyOS applications!
