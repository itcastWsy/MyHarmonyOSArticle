# HarmonyOS Comment Reply Dialog Best Practices

## Introduction

In mobile application development, comment and reply functionality is a common and important interaction scenario. This article will provide a detailed introduction on how to implement a comprehensive comment reply dialog in HarmonyOS, including dialog selection, rich text editing, soft keyboard adaptation, and other key technical points.

## Feature Overview

The comment reply dialog we want to implement includes the following features:

- Text input support
- Emoji selection support
- @friend functionality
- Seamless switching between soft keyboard and emoji panel
- Excellent user experience

## Technical Solution Analysis

### Dialog Component Selection

Before starting development, we need to choose the appropriate dialog implementation solution. HarmonyOS provides multiple dialog implementation methods. We compared three main solutions:

Through testing [CustomDialog custom dialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-methods-custom-dialog-box), [bindSheet semi-modal dialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-universal-attributes-sheet-transition#bindsheet), and [Navigation Dialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-navigation-navigation#%E9%A1%B5%E9%9D%A2%E6%98%BE%E7%A4%BA%E7%B1%BB%E5%9E%8B), we found that custom dialogs and semi-modal dialogs have certain specification limitations that would cause some unavoidable problems. We finally chose the [Navigation Dialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-navigation-navigation#%E9%A1%B5%E9%9D%A2%E6%98%BE%E7%A4%BA%E7%B1%BB%E5%9E%8B) solution to implement the comment module dialog. Below is a detailed explanation of the advantages and disadvantages of the three solutions.

#### Solution 1: CustomDialog Custom Dialog

[CustomDialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-methods-custom-dialog-box) is a standard dialog component provided by HarmonyOS.

**Advantages:**

- ✅ Ready to use out of the box, no need to implement dialog interaction logic
- ✅ Automatically avoids soft keyboard, simple to use
- ✅ System-level component with good stability

**Disadvantages:**

- ❌ Soft keyboard avoidance behavior cannot be customized
- ❌ Brief layout jumping when switching emoji panels
- ❌ Cannot obtain soft keyboard animation information, difficult to achieve smooth transitions

> **Problem Demonstration:** When users click the emoji button, the emoji panel will briefly appear in the wrong position during the soft keyboard retraction process, affecting user experience.

![CustomDialog Problem Demonstration](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113823.21128572248114659490332015929896:50001231000000:2800:520DD202B70134A32EFD837572631D8D5CD570EC988449ACB02C153105C878F8.gif)

> **Note:** [PromptAction.openCustomDialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-arkui-uicontext#promptaction) has the same effect as CustomDialog and has the same problems.

#### Solution 2: bindSheet Semi-modal Dialog

[bindSheet](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-universal-attributes-sheet-transition#bindsheet) is a semi-modal dialog component provided by HarmonyOS, commonly used for bottom-popup interaction scenarios.

**Advantages:**

- ✅ Ready to use out of the box, no need to implement dialog interaction logic
- ✅ Can solve the soft keyboard push-up problem in CustomDialog
- ✅ Supports gesture dragging with good interaction experience

**Disadvantages:**

- ❌ Internal scrolling behavior is difficult to control when height is adaptive
- ❌ Even when drag bar is disabled, the dialog can still be dragged
- ❌ The emoji panel may be exposed during dragging, affecting visual effects

> **Problem Demonstration:** After disabling the drag bar, users can still drag the dialog, which will expose the underlying emoji panel area during dragging.

![bindSheet Problem Demonstration](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113823.88787100697224156666358266633140:50001231000000:2800:DE9D0EB1CA70932656253517BD357AE13639560D02E047DDE00BD80B11921153.gif)

#### Solution 3: Navigation Dialog (Recommended)

[Navigation Dialog](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-navigation-navigation#%E9%A1%B5%E9%9D%A2%E6%98%BE%E7%A4%BA%E7%B1%BB%E5%9E%8B) is a dialog solution based on the Navigation routing system.

**Advantages:**

- ✅ Perfectly solves all problems of the first two solutions
- ✅ Based on routing stack management, dialog is completely decoupled from UI
- ✅ Can precisely control soft keyboard avoidance behavior
- ✅ Supports complex dialog hierarchy management

**Disadvantages:**

- ❌ Need to manually implement mask layer and click-to-close logic
- ❌ Relatively higher development complexity

> **Important Reminder:** Navigation Dialog has a lower z-axis level. If multiple dialog solutions are used simultaneously in the project, it is recommended to use Navigation Dialog uniformly to avoid level conflicts.

### Final Choice

After comprehensive comparison, we chose **Navigation Dialog** as the final solution, mainly for the following reasons:

1. **Perfect soft keyboard control**: Can precisely control soft keyboard avoidance behavior and achieve smooth switching animations
2. **Good architectural design**: Route-based design better aligns with modern application architecture concepts
3. **Strong scalability**: Convenient for future feature expansion and maintenance

Although the development complexity is slightly higher, the user experience improvement it brings is worthwhile.

### Edit Area Component Selection

Comment input boxes need to support multiple content types:

- 📝 **Text input**: Plain text content
- 😊 **Emoji symbols**: Image-form emojis
- 👥 **@friend functionality**: User tags with special styling

#### RichEditor Component Introduction

For this kind of mixed text and image layout requirement, the [RichEditor](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor) component provided by HarmonyOS is the best choice. It supports:

- **Rich text editing**: Mixed editing of text, images, and custom components
- **Flexible content management**: Managing content through different Span types
- **Rich interaction events**: Input, delete, select and other event listening

#### Content Types and Implementation Methods

RichEditor provides three main content addition methods:

| Content Type     | Implementation Method                                                                                                               | Purpose            |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| Text             | [addTextSpan](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#addtextspan)         | Plain text content |
| Image            | [addImageSpan](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#addimagespan)       | Emoji images       |
| Custom Component | [addBuilderSpan](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#addbuilderspan11) | @friend tags       |

> **Terminology Explanation:** For easy understanding, we refer to content added through these three methods as `textSpan`, `imageSpan`, and `builderSpan` respectively.

#### @Friend Functionality Implementation Solution Comparison

For @friend functionality, we have two implementation solutions to choose from:

##### Solution 1: Implementation using addTextSpan

Treat @friend as plain text.

**Problem Analysis:**

- ❌ **Text merging problem**: Text input before and after will automatically merge with @friend text, breaking tag independence
- ❌ **Complex interaction**: Need to manually handle cursor positioning and overall deletion logic
- ❌ **Difficult data association**: Can only get nickname text, cannot associate user's complete information (such as ID, avatar, etc.)

##### Solution 2: Implementation using addBuilderSpan (Recommended)

Treat @friend as a custom component.

**Advantage Analysis:**

- ✅ **Good independence**: Will not merge with surrounding text, maintaining tag integrity
- ✅ **Simple interaction**: System automatically handles cursor and deletion logic
- ✅ **Rich data**: Can maintain complete user information, convenient for subsequent processing

**Considerations:**

- Need to manually maintain builderSpan information, but this also brings greater flexibility

##### Final Choice

We chose the **addBuilderSpan** solution, mainly considering:

1. **Better user experience**: @friend tags as a whole, more natural interaction
2. **Stronger scalability**: Can easily add avatars, styles and other rich elements
3. **More reliable data management**: Complete user information convenient for business processing

## Core Feature Implementation

### 1. Dialog Display Implementation

#### Feature Flow

The comment dialog display flow is as follows:

1. User clicks the message button on the video page
2. Comment list page pops up
3. User clicks the write comment button
4. Comment input dialog pops up

![Dialog Display Flow](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113823.41492548141515634230758802114557:50001231000000:2800:EBF89F0BFC75642055069A6F1440ACE661042B11B7DFE7754C2939754FABE7CB.gif)

#### Technical Implementation Points

##### 1. Navigation Configuration

```typescript
// Main page Navigation configuration
Navigation() {
  // Page content
}
.mode(NavigationMode.Stack)  // Set to stack mode
.hideTitleBar(true)         // Hide title bar
```

##### 2. Dialog Component Structure

```typescript
// Dialog page component
@Component
struct CommentDialog {
  build() {
    NavDestination() {
      Stack() {
        // Mask layer
        Column()
          .width('100%')
          .height('100%')
          .backgroundColor('rgba(0,0,0,0.5)')
          .onClick(() => {
            // Click mask to close dialog
            router.back()
          })

        // Dialog content
        Column() {
          // Comment input component
        }
        .backgroundColor(Color.White)
        .borderRadius(12)
      }
    }
    .mode(NavDestinationMode.DIALOG)  // Set to dialog mode
    .expandSafeArea([SafeAreaType.KEYBOARD])  // Do not avoid soft keyboard
  }
}
```

##### 3. Key Configuration Explanation

| Configuration                             | Function                        | Importance |
| ----------------------------------------- | ------------------------------- | ---------- |
| `NavigationMode.Stack`                    | Enable routing stack management | ⭐⭐⭐     |
| `NavDestinationMode.DIALOG`               | Set as dialog type              | ⭐⭐⭐     |
| `expandSafeArea([SafeAreaType.KEYBOARD])` | Do not avoid soft keyboard      | ⭐⭐⭐     |
| Mask layer click event                    | Provide close interaction       | ⭐⭐       |

#### Dialog Management Strategy

- **Pop up**: Enter routing stack through `router.pushUrl()`
- **Close**: Exit routing stack through `router.back()`
- **Hierarchy**: The order of routing stack determines dialog hierarchy relationship

### 2. Soft Keyboard and Emoji Panel Switching Adaptation

#### Feature Requirements

In the comment dialog, users need to be able to seamlessly switch between the soft keyboard and emoji panel, providing a good input experience.

![Soft Keyboard Emoji Switching](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113823.89308693689825239388198025312122:50001231000000:2800:DB5CF03B542AEE021C84DCCBB26F1BC5723925B8A5B57C2E42202527F9C7B98A.gif)

#### Technical Implementation Solution

##### 1. Custom Keyboard Control

This article chooses custom keyboard to control the switching between soft keyboard and emoji panel:

- **Show emoji panel**: Set [RichEditor.customKeyboard](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#customkeyboard) to the emoji panel component's build function `EmojiKeyboard`
- **Show soft keyboard**: Set `customKeyboard` property to `undefined`
- **Focus management**: No need to manually handle RichEditor focus when switching this way

##### 2. Height Adaptation Strategy

To ensure the overall height of the comment module remains unchanged during the switching process, the following logic needs to be implemented:

**Soft keyboard height listening:**

```typescript
// Listen for soft keyboard height changes
window.on("keyboardHeightChange", (height: number) => {
  if (height > 0) {
    this.keyboardHeight = height;
  }
});
```

**Height calculation rules:**

- Emoji panel height = Common emoji list height + Soft keyboard height
- Placeholder element height = Current displayed component height (soft keyboard or emoji panel)

##### 3. Layout Adaptation Implementation

Since the dialog is set to not avoid the soft keyboard, layout needs to be controlled through placeholder elements:

```typescript
// Placeholder element height control
@State placeholderHeight: number = 0;

// When switching to soft keyboard
this.placeholderHeight = this.keyboardHeight;

// When switching to emoji panel
this.placeholderHeight = this.emojiPanelHeight + this.keyboardHeight;
```

#### Considerations

- ⚠️ **Memory management**: Must cancel keyboard height listening events before component destruction
- ⚠️ **Height changes**: Soft keyboard height may be manually adjusted by users, need real-time listening
- ⚠️ **Timing control**: Ensure correct timing of height settings during switching process

### 3. @Friend Functionality Implementation

#### Feature Overview

@Friend functionality allows users to mention other users in comments. The mentioned users will receive notifications, which is an important feature in social applications.

#### Trigger Methods

Users can trigger @friend functionality in two ways:

1. **Click @ button**: Directly click the @ button in the edit area
2. **Keyboard input**: Input @ symbol on the soft keyboard

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.25812803030167053648920547579046:50001231000000:2800:B789FEE11854FD2FD743AAD611B1179334B3545DCDA92A3661C802ECF58E5338.gif)

#### Implementation Flow

##### 1. Trigger @ Functionality

**When clicking @ button:**

```typescript
// Add @ symbol and show friend list
this.richEditorController.addTextSpan("@", {
  style: {
    fontColor: Color.Blue,
  },
});
this.showFriendList = true;
```

**Listen for keyboard input:**

```typescript
// Listen for input events, unified handling of @ symbol
.aboutToIMEInput((value: RichEditorInsertValue) => {
  if (value.insertValue === '@') {
    // Trigger @friend logic
    this.showFriendList = true;
    return true; // Prevent default input
  }
  return false;
})
```

Through [RichEditorController.addTextSpan](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#addtextspan) add @ symbol, and show friend list. Also listen for [RichEditor.aboutToIMEInput](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#abouttoimeinput) events to uniformly handle clicking @ button and keyboard input @ logic.

When clicking friend avatar in the friend list, through [RichEditorController.getSpans](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#getspans) you can get the content of the span before the cursor. If the span before the cursor is a textSpan with content @, delete it first, then through [RichEditorController.addBuilderSpan](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#addbuilderspan11) add "@[friend nickname]" as a whole with specified style to the edit area.

### 4. Content Deletion Functionality

#### Feature Requirements

When deleting @friend content, intelligent deletion needs to be implemented: the first time you click the delete key, the entire @friend component is selected, and the second time you click, it is deleted as a whole, rather than character by character deletion.

![Delete Functionality Demonstration](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.20287915377951093579309018074287:50001231000000:2800:A48EC5841269082BB512C5DF014428C6CC555CF1C28B46A31DBD61C36360A03B.gif)

#### Implementation Solution

```typescript
// Listen for delete events
.aboutToDelete((value: RichEditorDeleteValue) => {
  // Get span information to be deleted
  const spans = this.richEditorController.getSpans(value.offset, value.offset + value.length);

  if (spans.length > 0) {
    const span = spans[0];

    // If it's builderSpan (@friend) and not selected
    if (span.spanType === 'builderSpan' && !this.isSpanSelected(span)) {
      // First deletion: select entire @friend component
      this.richEditorController.setSelection(span.start, span.end);
      return false; // Prevent default deletion behavior
    }
  }

  return true; // Allow default deletion behavior
})
```

#### Technical Points

| Function          | API               | Description                                                |
| ----------------- | ----------------- | ---------------------------------------------------------- |
| Delete listening  | `aboutToDelete()` | Listen for delete operations, can prevent default behavior |
| Content selection | `setSelection()`  | Select content in specified range                          |
| Get content       | `getSpans()`      | Get span information at specified position                 |

Through [RichEditor.aboutToDelete](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#abouttodelete) event listening for delete operations, use [RichEditorController.setSelection](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#setselection11) to achieve overall selection and deletion of @friend components.

### 5. Content Retrieval and Display

#### Feature Overview

When users complete comment editing, all content (text, emojis, @friends) in the edit area needs to be retrieved for unified data processing and display.

![Content Display Effect](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.35426684108552004081985772349110:50001231000000:2800:7AE40EF9CA4BB477A35DF7FC5F15F2FDBCF0D161E0E2406524C9C2900F5C96CE.png)

#### Data Type Mapping

Through [RichEditorController.getSpans](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#getspans) get edit area content, return values include [RichEditorTextSpanResult](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#richeditortextspanresult) and [RichEditorImageSpanResult](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-richeditor#richeditorimagespanresult) two types.

Correspondence between different content types and data types:

textSpan can get text content through RichEditorTextSpanResult.value. imageSpan can get image resources through RichEditorImageSpanResult.valueResourceStr. However, builderSpan cannot get any related content information in RichEditorImageSpanResult, so when clicking friend avatar to add @friend content, these builderSpans need to be manually maintained.

In actual development, different types of content in the edit area often need a unified data structure to express, convenient for transmission and storage. This data structure needs to not only record edit area content, but also have the ability to carry some additional information, such as carrying @friend related user information. This article defines it as RichEditorSpan. (In actual development, the required attribute fields are flexibly adjusted according to requirements).

Use RichEditorSpan[] type array builderSpans to maintain builderSpan when @friend, note that the order of each builderSpan in the array must be consistent with the order that actually appears in the content. When adding builderSpan, by calculating the number of builderSpans before the current cursor position, determine the position to add to the builderSpans array, and put the friend information that needs to be carried into the data attribute.

When sending comments, use RichEditorSpan[] type array richEditorSpans to uniformly express the obtained content. Get all content through getSpans, if it is textSpan, get text content through value attribute, set RichEditorSpan.type to text, if it is imageSpan, get image resources through valueResourceStr attribute, set RichEditorSpan.type to image. If it is builderSpan, get from array builderSpans in order, and add them to richEditorSpans in order.

The final generated richEditorSpans data format is as follows:

When comment content needs to be displayed, just traverse richEditorSpans, according to the type attribute, handle the display logic of text, emojis, and @friends separately. The specific display form is determined by developers according to actual requirements.

- Select image

Click the image button to launch the system album and select local images for upload. This functionality is relatively independent in use scenarios, and this article does not introduce it in detail. If developers need to understand further details, they can refer to the following samples.

- [Select and view documents and media files](https://gitee.com/harmonyos_samples/picker)
- [File management](https://developer.huawei.com/consumer/cn/codelabsPortal/carddetails/tutorials_NEXT-FilesManger)
- [Publish image comments](https://gitee.com/harmonyos_samples/image-comment)
