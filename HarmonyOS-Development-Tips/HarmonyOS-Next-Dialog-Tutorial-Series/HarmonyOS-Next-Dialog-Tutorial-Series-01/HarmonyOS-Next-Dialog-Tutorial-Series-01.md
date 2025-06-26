# HarmonyOS Next Dialog Series Tutorial (1)

## Introduction to Dialogs

### Dialog Overview

Dialogs generally refer to UI interfaces that automatically pop up when opening an application or pop up during user interaction operations, used to display information that users need to pay attention to or operations that need to be handled within a short period of time.

For example, the following effects:

![image-20241226195728205](HarmonyOS%20Next%20%E5%BC%B9%E7%AA%97%E7%B3%BB%E5%88%97%E6%95%99%E7%A8%8B%EF%BC%881%EF%BC%89.assets/image-20241226195728205.png)

### Types of Dialogs

According to user interaction operation scenarios, dialogs can be divided into **modal dialogs** and **non-modal dialogs**, with the difference being whether users must respond to them.

- **Modal Dialogs:** Strong interaction form that interrupts the user's current operation flow, requiring users to respond before continuing with other operations. Usually used for scenarios where important information needs to be conveyed to users.
- **Non-Modal Dialogs:** Weak interaction form that does not affect the user's current operation behavior. Users can choose not to respond to them. Usually have time limits and automatically disappear after appearing for a period of time. Generally used for scenarios where users need to be informed of information content and also need users to perform functional operations.

### Modal Dialogs

Strong interaction form that interrupts the user's current operation flow, requiring users to respond before continuing with other operations. Usually used for scenarios where important information needs to be conveyed to users.

![image-20241226195844696](HarmonyOS%20Next%20%E5%BC%B9%E7%AA%97%E7%B3%BB%E5%88%97%E6%95%99%E7%A8%8B%EF%BC%881%EF%BC%89.assets/image-20241226195844696.png)

### Non-Modal Dialogs

![PixPin_2024-12-26_20-02-31](HarmonyOS%20Next%20%E5%BC%B9%E7%AA%97%E7%B3%BB%E5%88%97%E6%95%99%E7%A8%8B%EF%BC%881%EF%BC%89.assets/PixPin_2024-12-26_20-02-31.gif)

### Dialog Classification

HarmonyOS Next categorizes dialogs as follows:

| Dialog Name                                                                                                                               | Application Scenario                                                                                                                                                                                             |
| :---------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Dialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-base-dialog-overview-V5)                                  | When you need to display information content or operations that users currently need or must pay attention to, such as secondary exit from applications, this dialog should be prioritized.                      |
| [Menu](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-popup-and-menu-components-menu-V5)                          | When you need to bind executable operations to specified components, such as long-pressing icons to display operation options, this dialog should be prioritized.                                                |
| [Popup](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-popup-and-menu-components-popup-V5)                        | When you need to provide tips for specified components, such as clicking a question mark icon to pop up a help tip, this dialog should be prioritized.                                                           |
| [Bind Modal Pages (bindContentCover/bindSheet)](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-modal-overview-V5) | When you need a new interface to overlay on top of an old interface without the old interface disappearing, such as clicking a thumbnail image to view a larger image, this dialog should be prioritized.        |
| [Instant Feedback (Toast)](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-create-toast-V5)                        | When you need to provide simple feedback about the user's current operation in a small window, such as prompting that a file has been saved successfully, this dialog should be prioritized.                     |
| [Overlay Manager](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-create-overlaymanager-V5)                        | When you need to completely customize content, behavior, and style, you can use overlays to display UI on top of pages, such as music/voice playback floating balls/capsules, this dialog should be prioritized. |

## Dialog Overview

Users need to complete related interactive tasks within modal dialogs before they can exit modal mode. Dialogs can be unbound to any component, and their content usually consists of multiple components such as text, lists, input boxes, images, etc., to implement layout. ArkUI currently provides two types of dialog components: **fixed style** and **custom**.

- **Custom Dialogs:** Developers need to pass in custom components to fill in the dialog according to usage scenarios to implement custom dialog content. Mainly includes basic custom dialogs (CustomDialog) and custom dialogs that don't depend on UI components (openCustomDialog).
- **Fixed Style Dialogs:** Developers can use fixed style dialogs, specify the text content and button operations to be displayed, and complete simple interactive effects. Mainly includes alert dialogs (AlertDialog), list selection dialogs (ActionSheet), picker dialogs (PickerDialog), dialogs (showDialog), and action menus (showActionMenu).

## openCustomDialog Usage Tutorial

We use `openCustomDialog` dialogs in 3 steps:

1. Create content to display
2. Declare UiContext
3. Declare promptAction
4. Create nodes to display in dialog
5. Show dialog

### Create Content to Display

Functions decorated with **@Builder** are not ordinary functions, but content construction functions responsible for rendering nodes.

```typescript
@Builder
function buildText() {
  Column() {
    Text("Custom Title")
      .fontSize(50)
      .fontWeight(FontWeight.Bold)
      .margin({ bottom: 36 })
  }.backgroundColor('#FFF0F0F0')
}
```

### Declare UiContext

Get UI context instance **UIContext** object through the **getUIContext()** interface

```typescript
let uiContext = this.getUIContext();
```

### Declare promptAction

**promptAction** is used to create and display text prompts, dialogs, and action menus.

```typescript
let promptAction = uiContext.getPromptAction();
```

### Create Nodes to Display in Dialog

**ComponentContent** represents the entity encapsulation of component content. Its objects support creation and passing in non-UI components, facilitating developers to decouple and encapsulate dialog-type components.

wrapBuilder is a global function that wraps global @Builder to facilitate @Builder passing and calling.

```typescript
let contentNode = new ComponentContent(uiContext, wrapBuilder(buildText));
```

### Show Dialog

Finally, call promptAction's openCustomDialog method and pass in contentNode to achieve the dialog effect.

```typescript
promptAction.openCustomDialog(contentNode);
```

![PixPin_2024-12-26_21-05-37](HarmonyOS%20Next%20%E5%BC%B9%E7%AA%97%E7%B3%BB%E5%88%97%E6%95%99%E7%A8%8B%EF%BC%881%EF%BC%89.assets/PixPin_2024-12-26_21-05-37.gif)

### Complete Code

```typescript
import { ComponentContent } from '@kit.ArkUI';

@Builder
function buildText() {
  Column() {
    Text("Custom Title")
      .fontSize(50)
      .fontWeight(FontWeight.Bold)
      .margin({ bottom: 36 })
  }.backgroundColor('#FFF0F0F0')
}

@Entry
@Component
struct Index {
  build() {
    Row() {
      Column() {
        Button("click me")
          .onClick(() => {
            let uiContext = this.getUIContext();
            let promptAction = uiContext.getPromptAction();
            let contentNode = new ComponentContent(uiContext, wrapBuilder(buildText));
            promptAction.openCustomDialog(contentNode);
          })
      }
      .width('100%')
      .height('100%')
    }
    .height('100%')
  }
}
```

## Reference Links

1. [wrapBuilder](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V13/arkts-wrapbuilder-V13)
2. [ComponentContent](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V13/js-apis-arkui-componentcontent-V13)
3. [getUIContext](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V13/ts-custom-component-api-V13#getuicontext)
4. [getPromptAction](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V13/js-apis-arkui-uicontext-V13#getpromptaction)
