# Implementing HarmonyOS Next WeChat Chat 01

## Preface

The code will be unified on Gitee, and the complete static code will be placed at the end.

## Project Goals

> This is the actual WeChat chat interface functionality effect on Android phones

![image-20240902230122815](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902230122815.png)

## Actual Effect

![image-20240902231634937](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902231634937.png)

## Features

1. Immersive page layout
2. Chat content scrolling
3. Input box state switching
4. Chat message box width adaptation
5. Input method avoidance
6. Voice message width automatically adjusts based on duration
7. Canvas waveform - press and speak
8. Gesture coordinate detection to cancel sending - voice to text
9. Send text messages
10. Recording - send voice messages
11. Audio playback - voice messages
12. AI voice to text

## Create New Project

![image-20240902223823376](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902223823376.png)

## Modify Project Desktop Name and Icon

![image-20240902224250634](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902224250634.png)

1. `entry\src\main\resources\zh_CN\element\string.json`
   1. ```TypeScript
      {
        "string": [
          {
            "name": "module_desc",
            "value": "Module Description"
          },
          {
            "name": "EntryAbility_desc",
            "value": "description"
          },
          {
            "name": "EntryAbility_label",
            "value": "My Chat Project" // 😄
          }
        ]
      }
      ```
2. `\entry\src\main\module.json5`
   1. ```TypeScript
      ...
      "abilities": [
        {
          "name": "EntryAbility",
          "srcEntry": "./ets/entryability/EntryAbility.ets",
          "description": "$string:EntryAbility_desc",
          "icon": "$media:chat",😄
          ...
      ```

> $media:chat comes from the icon named chat under resource

![image-20240902224949027](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902224949027.png)

## Set Immersive Mode

![image-20240902225153206](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902225153206.png)

1. **Figure 1** shows the default page layout, where our page cannot reach the **top status bar** and **bottom navigation bar**
2. **Figure 2** shows the effect after setting immersive mode, where layout buttons can reach the top status bar
3. **Figure 3** shows dynamically obtaining the height of the top status bar, then adding corresponding padding to the container, pushing layout elements below the top status bar

### Set Immersive Mode and Get Top Status Bar Height

`\entry\src\main\ets\entryability\EntryAbility.ets`

```typescript
...
onWindowStageCreate(windowStage: window.WindowStage): void {
  // Main window is created, set main page for this ability
  hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onWindowStageCreate');

  windowStage.loadContent('pages/Index', (err) => {
    if (err.code) {
      hilog.error(0x0000, 'testTag', 'Failed to load the content. Cause: %{public}s', JSON.stringify(err) ?? '');
      return;
    }
    hilog.info(0x0000, 'testTag', 'Succeeded in loading the content.');
    // Set application to fullscreen
    let windowClass: window.Window = windowStage.getMainWindowSync(); // Get application main window

    windowClass.setWindowLayoutFullScreen(true)

    let type = window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR; // Navigation bar avoidance
    let avoidArea = windowClass.getWindowAvoidArea(type);

    let bottomRectHeight = avoidArea.bottomRect.height; // Get navigation bar area height

    const vpHeight = px2vp(bottomRectHeight) // Convert to vp unit value

    // Store navigation bar height data globally
    AppStorage.setOrCreate("vpHeight", vpHeight)
  });
}
...
```

### Page Uses Navigation Bar Height to Set Padding

```typescript
@Entry
@Component
struct Index {
  @StorageProp("vpHeight")
  topHeight: number = 0

  build() {
    Column() {
      Button("Button")
    }
    .width("100%")
    .height("100%")
    .backgroundColor(Color.Yellow)
    .padding({
      top: this.vpHeight,
    })
  }
}
```

## Build Basic Page Layout

![image-20240902232718492](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902232718492.png)

```typescript
@Entry
@Component
struct Index {
  // Status bar height
  @StorageProp("vpHeight")
  vpHeight: number = 0

  build() {
    Column() {
      // 1 Top title bar
      Row() {
        Image($r("app.media.left"))
          .width(25)
        Text("kto卋讓硪玩孫悟空")
        Image($r("app.media.more"))
          .width(25)
      }
      .width("100%")
      .justifyContent(FlexAlign.SpaceBetween)
      .border({
        width: {
          bottom: 1
        },
        color: "#ddd"
      })
      .padding(10)
      // 2 Chat scroll container
      // 3 Input panel
    }
    .height('100%')
    .width('100%')
    .backgroundColor("#EDEDED")
    .padding({
      top: this.vpHeight + 20
    })
  }
}
```

## Page Scrolling and Text Message Box

![PixPin_2024-09-02_23-34-06](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/PixPin_2024-09-02_23-34-06.gif)

```typescript
build() {
  Column() {
    // 1 Top title bar
    ...
    // 2 Chat scroll container
    Scroll() {
      Column({ space: 10 }) {
        this.chatTextBuilder("Had dinner", `22:23`)
      }
      .width("100%")
      .padding(10)
      .justifyContent(FlexAlign.Start)
    }
    .layoutWeight(1)
    .align(Alignment.Top)
    .expandSafeArea([SafeAreaType.KEYBOARD], [SafeAreaEdge.BOTTOM])
  }
  .height('100%')
  .width('100%')
  .backgroundColor("#EDEDED")
  .backgroundImageSize(ImageSize.Cover)
  .padding({
    top: this.vpHeight + 20
  })
}

// Text message
@Builder
chatTextBuilder(text: string, time: string) {
  Column({ space: 5 }) {
    Text(time)
      .width("100%")
      .textAlign(TextAlign.Center)
      .fontColor("#666")
      .fontSize(14)
    Row() {
      Flex({ justifyContent: FlexAlign.End }) {
        Row() {
          Text(text)
            .padding(11);
          Text()
            .width(10)
            .height(10)
            .backgroundColor("#93EC6C")
            .position({
              right: 0,
              top: 15
            })
            .translate({
              x: 5,
            })
            .rotate({
              angle: 45
            });
        }
        .backgroundColor("#93EC6C")
        .margin({ right: 15 })
        .borderRadius(5);

        Image($r("app.media.avatar"))
          .width(40)
          .aspectRatio(1);
      }
      .width("100%");
    }
    .width("100%")
    .padding({
      left: 40
    })
    .justifyContent(FlexAlign.End)
  }
  .width("100%")
}
```

### Highlights

![image-20240902234252101](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902234252101.png)

The following code is the key to implementing adaptive width above:

1. When text is short, the green chat box width adapts automatically
2. When text is long, the green chat box width automatically expands but doesn't fill the entire line, just like WeChat's design

![image-20240902234447382](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240902234447382.png)

## Bottom Message Input Box

### Display Input Box or "Press and Hold to Speak"

![image-20240903000807542](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903000807542.png)

As you can see, the bottom message input box has at least three states:

1. Press and hold to speak
2. Text input box
3. Text input box - Send

In the program, enumerations are used to determine the two states: press and hold to speak - text input box

```typescript
/**
 * Current input state voice or text
 */
enum WXInputType {
  /**
   * Voice input
   */
  voice = 0,
  /**
   * Text input
   */
  text = 1,
}
```

---

![image-20240903001343178](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903001343178.png)

### Display "Send" Button

Additionally, by checking the length of the text input, it determines whether to display the green **Send** button

![image-20240903001128023](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903001128023.png)

---

![image-20240903001214873](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903001214873.png)

---

![image-20240903001146480](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903001146480.png)

### Text Input Box Automatically Gets Focus

![PixPin_2024-09-03_00-15-58](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/PixPin_2024-09-03_00-15-58.gif)

![image-20240903001515500](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903001515500.png)

## Set Keyboard Avoidance During Input

Without avoidance settings, you can see the bottom chat being pushed up by the popped-up keyboard.

![PixPin_2024-09-03_00-19-15](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/PixPin_2024-09-03_00-19-15.gif)

### Solution

Set the extended safe area to the soft keyboard area, and set the extended safe area direction to the bottom area

![image-20240903002417398](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/image-20240903002417398.png)

---

![PixPin_2024-09-03_00-26-12](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/PixPin_2024-09-03_00-26-12.gif)

## Send Text Messages

![PixPin_2024-09-03_21-55-36](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/PixPin_2024-09-03_21-55-36.gif)

### Define Message Type Enumeration

```typescript
enum MessageType {
  /**
   * Voice
   */
  voice = 0,
  /**
   * Text
   */
  text = 1,
}
```

### Define Message Class

Used to quickly generate message objects, can represent voice messages and text messages

```typescript
// Message
class ChatMessage {
  /**
   * Message type: [recording, text]
   */
  type: MessageType;
  /**
   * Content [recording-file path, text-content]
   */
  content: string;
  /**
   * Message time
   */
  time: string;
  /**
   * Sound duration in milliseconds
   */
  duration?: number;
  /**
   * Text converted from recording
   */
  translateText?: string;
  /**
   * Whether to show converted text
   */
  isShowTranslateText: boolean = false;

  constructor(
    type: MessageType,
    content: string,
    duration?: number,
    translateText?: string
  ) {
    this.type = type;
    this.content = content;
    const date = new Date();
    this.time = `${date.getHours().toString().padStart(2, "0")}:${date
      .getMinutes()
      .toString()
      .padStart(2, "0")}`;
    this.duration = duration;
    this.translateText = translateText;
  }
}
```

### Define Message Array

```typescript
// Messages
@State
chatList: ChatMessage[] = []
```

### Define Send Text Message Method

```typescript
// Send text message
sendTextMessage = () => {
  if (!this.textValue.trim()) {
    return;
  }
  const chat = new ChatMessage(MessageType.text, this.textValue.trim());
  this.chatList.push(chat);
  this.textValue = "";
};
```

### Register Send Text Message Event

```typescript
Button("Send")
  .backgroundColor("#08C060")
  .type(ButtonType.Normal)
  .fontColor("#fff")
  .borderRadius(5)
  .onClick(this.sendTextMessage);
```

### Iterate Through Message Array

```typescript
// 2 Chat scroll container
Scroll() {
  Column({ space: 10 }) {
    ForEach(this.chatList, (item: ChatMessage, index: number) => {
      if (item.type === MessageType.text) {
        this.chatTextBuilder(item.content, item.time)
      }
    })
  }.width("100%")
  .padding(10)
  .justifyContent(FlexAlign.Start)
}
.layoutWeight(1)
.align(Alignment.Top)
.expandSafeArea([SafeAreaType.KEYBOARD], [SafeAreaEdge.BOTTOM])
```

## Press and Hold to Speak

![PixPin_2024-09-03_22-11-52](%E5%B8%A6%E4%BD%A0%E5%AE%9E%E7%8E%B0HarmonyOS%20Next%20%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A901.assets/PixPin_2024-09-03_22-11-52.gif)

### Define Variable for Whether Currently Speaking

```typescript
// Press and hold to speak recording modal
@State
showTalkContainer: boolean = false
```

### Register Touch Event

Triggered when pressing and holding **Press and Hold to Speak**. Touch events are continuously triggered, determine touch state by checking event.type:

1. down - pressed
2. move - moving
3. up - released

```typescript
Button("Press and Hold to Speak")
  .layoutWeight(1)
  .type(ButtonType.Normal)
  .borderRadius(5)
  .backgroundColor("#fff")
  .fontColor("#000")
  .onTouch(this.onPressTalk);
```

---

### Define this.onPressTalk

```typescript
// Press and hold to speak continuously triggered
onPressTalk = async (event: TouchEvent) => {
  if (event.type === TouchType.Down) {
    // Pressed down
    this.showTalkContainer = true;
  } else if (event.type === TouchType.Up) {
    // Released
    this.showTalkContainer = false;
  }
};
```

### Implement Full Screen Overlay Effect

This effect uses **full modal** in HarmonyOS applications [**bindContentCover**](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/ts-universal-attributes-modal-transition-V5#bindcontentcover)

Bind a full-screen modal page to a component, display the modal page after clicking. Modal page content is customizable, display mode can be set to no animation transition, up-down switching transition, and transparent gradient transition.

> this.talkContainerBuilder is the corresponding content layout when the full modal appears, it's a custom builder function

```typescript
Button("Press and Hold to Speak")
  .layoutWeight(1)
  .type(ButtonType.Normal)
  .borderRadius(5)
  .backgroundColor("#fff")
  .fontColor("#000")
  .bindContentCover($$this.showTalkContainer, this.talkContainerBuilder, {
    modalTransition: ModalTransition.NONE,
  })
  .onTouch(this.onPressTalk);
```

---

### Define this.talkContainerBuilder

```typescript
// Speaking page layout
@Builder
talkContainerBuilder() {
  Column() {
    // 1 Center prompt
    Row() {
      Text()
        .width(10)
        .height(10)
        .backgroundColor("#95EC6A")
        .position({
          bottom: -5,
          left: "50%"
        })
        .translate({
          x: "-50%"
        })
        .rotate({
          angle: 45
        })
    }
    .width("50%")
    .height(80)
    .backgroundColor("#95EC6A")
    .position({
      top: "40%",
      left: "50%"
    })
    .translate({
      x: "-50%"
    })
    .borderRadius(10)
    .justifyContent(FlexAlign.Center)
    .alignItems(VerticalAlign.Center)

    // 2 Cancel and convert to text
    Row() {
      Row() {
        Text("X")
          .fontSize(20)
          .width(60)
          .height(60)
          .borderRadius(30)
          .fontColor("#000")
          .backgroundColor("#fff")
          .textAlign(TextAlign.Center)
      }
    }
  }
}
```
