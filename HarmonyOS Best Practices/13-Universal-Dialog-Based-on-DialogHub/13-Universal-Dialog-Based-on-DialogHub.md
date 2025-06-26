## Overview

In HarmonyOS development, dialogs are scenarios that every application will encounter, and their importance cannot be ignored. On one hand, dialogs can serve as an instant feedback mechanism to convey important information or prompts to users, such as login prompts, network request status, operation confirmations, etc. These dialogs usually have modal or semi-modal characteristics that can temporarily block other user operations, ensuring that users can notice and handle this information. On the other hand, dialogs can also be used to display advertisements, promotional content, or guide users to take the next step. For example, on the app's homepage or some key pages, displaying full-screen advertisements or guiding users to participate in certain activities through dialogs can effectively improve user engagement. To facilitate developers' efficient use of different dialog capabilities on HarmonyOS, the DialogHub solution was born.

DialogHub, as a solution for ArkUI dialog capabilities, provides the following functional features:

1.  **Page-level dialog capability**

    Ensures that dialogs are closely bound to the page lifecycle, automatically cleaning up dialog resources when the page is destroyed.

    Automatically checks and hides dialogs from old pages during page switching or navigation.

2.  **Dialog management capability**

    Provides dialog state management to distinguish whether dialogs are being displayed or have been closed.

    Provides listening mechanisms that allow developers to execute custom logic when dialog states change, including 4 states: pop-up, about to pop-up, close, and about to close.

3.  **Simplified dialog creation process**

    Streamlined chained API design ensures that common dialogs can be created through concise syntax.

    Provides default configurations to reduce unnecessary parameter settings and improve calling efficiency.

4.  **Custom dialog templates to improve usability**

    Allows developers to customize templates and save them to the template library for subsequent reuse.

5.  **Layer management, gesture passthrough and other custom configuration attributes**

    Provides more flexible layer management mechanisms, allowing developers to dynamically adjust the Z-axis order of dialogs.

    Provides resolution strategies for layer conflicts, such as resolution strategies for new and old top dialogs.

    Allows developers to customize gesture passthrough behavior, such as whether to allow gestures to penetrate dialogs and act on underlying pages.

    Provides more custom attributes such as dialog animation effects, background colors, corner radius, etc.

6.  **Dialog refresh mechanism**

    Provides dynamic update mechanisms for attribute values, allowing developers to modify attributes during dialog display.

This article mainly introduces the use of DialogHub with examples from various scenarios in actual development.

## Implementation Principles

- **Dialog capabilities**: Based on the OverlayManager and BindSheet capabilities in the ArkUI framework. [OverlayManager](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-arkui-uicontext#overlaymanager12) provides a display layer for dialogs that can overlay other UI elements, while [BindSheet](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-universal-attributes-sheet-transition#bindsheet) supports binding dialogs to specific pages or components for more fine-grained control.
- **Page-level dialog control**: Through [UIObserver](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-arkui-uicontext#uiobserver11), real-time monitoring of route changes within the application is achieved. When routes change, corresponding callbacks are triggered, allowing DialogHub to decide whether to show or hide dialogs based on the current page state.

## Development Process

**Prerequisites**

Developers refer to the [DialogHub Usage Instructions](https://gitee.com/hadss/DialogHub/blob/master/hadss_dialog/README.md) for installation and configuration.

Developers call the init() interface and pass in UIContext to initialize DialogHub.

### Dialog Capability Development Process

Create dialogs directly through DialogHub and then display or destroy them.

1.  **Get dialog constructor:**

    Call DialogHub's getToast() and other interfaces to get different types of dialog constructors DialogBuilder.

2.  **Configure dialog content:**

    Call DialogBuilder's setContent(), setAnimation() and other interfaces to configure the specific content, animation effects, styles, etc. of the dialog.

3.  **Create dialog instance:**

    Call DialogBuilder's build() interface to create the dialog instance InfDialog object.

4.  **Display and destroy dialogs:**

    Call the InfDialog object's show() method to display the dialog.

    Call the InfDialog object's dismiss() method to destroy the dialog.

### Template Reuse Capability Development Process

Developers customize templates and register them in the template library for subsequent reuse.

1.  **Create dialog template constructor:**

    Call DialogHub's createToastTemplate() and other interfaces to create template constructors DialogTemplate for different types of dialogs.

2.  **Configure template content:**

    Call DialogTemplate's setContent(), setAnimation() and other interfaces to configure the specific content, animation effects, styles, etc. of the template.

3.  **Register template:**

    Call DialogTemplate's register() interface to register and store the configured template.

4.  **Get and use dialog template:**

    Call DialogHub's getToastTemplate() and other interfaces to get the corresponding DialogBuilder based on the template name.

    Then follow steps 2-4 in the dialog capability development process to use DialogBuilder to configure and display dialogs.

    **Note**

    Properties configured after getting the template (such as animation, position, etc.) only take effect for the current dialog object and will not modify the template content.

5.  **(Optional) Update template:**

    Call DialogHub's updateToastTemplate(), updatePopupTemplate() and other interfaces to update the configuration of the corresponding template name and re-register.

6.  **(Optional) Delete template:**

    Call DialogHub's removeTemplate() interface to delete the dialog template with the corresponding template name.

7.  **(Optional) Check if template exists:**

    Call DialogHub's isTemplateExist() interface to determine whether a dialog template with the specified template name has been registered.

## Common Business Dialogs

### Plain Text Toast with Duration

A simple text Toast dialog that disappears after reaching the specified time. setDuration() sets the Toast duration.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.35889065581015384556981994691464:50001231000000:2800:A6489FC1C2DC21783AE67267AC5DA2D5D29B33ECF9351317E18910BA2A93DD53.png)

### Non-modal Dialog at Specified Position

A SnakeBar pops up at the bottom of the screen. This dialog can respond to user clicks to jump to pages or close the dialog.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.14588558373239365351766590061894:50001231000000:2800:C42EA86D7C4FD1D03CC0A92A42A5F342EDA098D33528DEB37E9DD05CB9728504.png)

### Timed Dialog with Pop-up Animation

Implement a timed dialog that automatically closes after 6 seconds.

- Set dialog pop-up animation through setAnimation().
- Dynamically refresh dialog content at timed intervals through the dialog instance's updateContent().

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.72278831174911229559848669146598:50001231000000:2800:FAFA49D268F95329B363AA754B692E79D5F30B50D471E27F35EDC28D83C1829A.png)

### Keyboard-avoiding Dialog

The avoidance mode can be configured through setConfig()'s keyboardAvoidMode. CustomKeyboardAvoidMode.CONTENT_AVOID is for dialog content avoidance.

requestFocusWhenShow is configured as true, when the dialog is displayed, the dialog automatically gains focus.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.45205622914669888677318622997228:50001231000000:2800:D0F78EAFB24715D04D50B3569493EDD6ADC87B9CE21875C580908637B50D22A9.png)![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113824.79491945946522684146895505528908:50001231000000:2800:19A05E8898BD15B5470EAB9D7ED835DBCFFB3A88CB025B38ACEF053B5C4F8D5A.png)

### Arrow Dialog Pointing to Selected Component

Construct a Popup dialog instance through getPopup(). enableArrow, arrowOffset, arrowWidth, arrowHeight in setStyle() can configure arrow properties.

preferPlacement in setConfig() can configure arrow preference.

**Note**

Binding components need to call setComponentTargetId(targetCompId). The targetCompId component id identifier must be unique, otherwise errors will occur and dialog position will be abnormal.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.07766498360154146187974185541962:50001231000000:2800:FF826DCB0C45226175DF418F153A89EE4AD0BDD0C67B0CD92BC9BE8F8587370A.png)

### Auto-close Dialog When Clicking Mask

To pop up this type of dialog, you need to turn on the isModal mask switch and set autoDismiss to true.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.56631643786969583261216656028341:50001231000000:2800:57F8F53CBA408C5C6E3EF56EF6E3B7C3C49B0C32530C7E3F5C34CDA5A2428770.png)

### Manually Closable Dialog

Can close the dialog by clicking dialog buttons. When setting dialog Content, call setOperableContent() and pass DialogHub's Dismiss event as a parameter to the Builder.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.80633210226318817985927591942313:50001231000000:2800:094477F1EFA2BCC17625151CF88BBCF57549906EFF5D9A09D9B6F1DF00D12130.png)

### Bottom Dialog with Dynamic Height Adjustment

Implement dynamic adjustment of dialog height, displaying different dialog content at different heights.

- Get DialogHub's Sheet type dialog instance
- Add Sheet height listener onHeightDidChange() to dialog instance. When height changes to a certain extent, updateContent() refreshes dialog content

**Note**

Sheet type dialogs must call setComponentTargetId(targetCompId) to implement page-level dialogs, and ensure that the bound component id exists.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.50284409663840205425318337189475:50001231000000:2800:35E4B321A0DCCB499637B3EA561D32C918D5F90011CBBEAE264AF6DA2240D916.png)![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.73468242706187798300739583262415:50001231000000:2800:560795A40C4D140DD0A322E0477C138C45A6F1A0DE1F818E815E913A3AB98A2C.png)

### Application Awareness of Dialog Opening and Closing

- Add lifecycle to dialog instances to intercept dialog display and destruction.
- Directly get dialog status

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.77777049893213827165313624319196:50001231000000:2800:C3E88EE2408915BD62B0631E70D469127FE474EF1FC7022D63F76ABA6862FA76.png)![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.90414307777072218287475868269848:50001231000000:2800:2A8EB302CC18FF9E5B953A393DE7FA6E0D173BDE85078C6D3705284334C88BC7.png)

## Dialog Interaction with Surroundings

### How to Define Whether Back Gesture Exits Page or Closes Dialog When Dialog Exists

Configure state variable backCloseDialog. Setting true means back gesture acts on dialog, false means it acts on page.

Intercept gestures in onBackPressed() and choose whether to exit page or close the top dialog

### User Can Operate Page Through Dialog Content

Pop up Toast type dialog, or actively call setConfig() to set passThroughGesture to true, to achieve dialog content gesture passthrough.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.32086059379781171334292193632155:50001231000000:2800:A8B7C793FB956C943A8891435CFF13E3591D44068A576FCD722D2E6194FCCDA6.png)

### Dialog That Needs to Return Data to Page

Pass a callback function that modifies page data to the Builder parameter and call it within the Builder.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.49547511750732945538097729282161:50001231000000:2800:54718755997F1743D9A693A63BC4270E1538F9D33F66CF4EF60782099CD57322.png)![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113825.67802751660170496582244560020445:50001231000000:2800:5729DBFDB6AA4AB54DB47BE8236FEFAC75750A7EE592BDAFEF66424BE2B9E294.png)

### Parent Page Refreshes Currently Displayed Dialog Content

Modify Builder parameter content, then call updateContent() to make changes.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113826.26223974618699349188114439853335:50001231000000:2800:FCDC2573EDEC270719E95409D2236D72304CA331517A291C3627FCA6752E563D.png)

### Page Needs to Sense Whether Current Page Has Dialogs

DialogHub registers a page dialog count listener, which triggers when the number of dialogs on the current page changes.

### Dialog with Jump Links

Click specific content on the dialog to jump to other pages.

router: Jump through router template within the dialog Builder.

Navigation: Pass pageInfos to the dialog Builder, then push pages within the dialog.

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113826.08726641569936163233559712538261:50001231000000:2800:DC36B7C95084D98DB43E22F90F7FFE54CBC4B28CD017EE1DB44184A95575FE0B.png)

### Dialogs at Different Positions in Foldable Screen Expanded State

Dialogs are centered on screen by default; dialogs can be positioned at different locations by setting dialog offset.

Dialog on left half screen:

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113826.60072460631041731880493484233028:50001231000000:2800:15301581D6C824CEED978B675D519E6276ABFB163E68B5EB7CFEE284BD07B60F.png)

Dialog on right half screen:

![](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20250616113826.92833164566547215497420032901048:50001231000000:2800:000EF8A1DAE5BE482FF787C98398B7DC26464ADB368437F961D8370F2FCCEA01.png)

## Dialog Content Reuse Scenarios

### Creating Dialogs Through Custom Dialog Templates

- Create dialog template
- Directly pop up template
- Delete dialog template
- Randomly modify dialog template background color
- (Optional) Define pop-up animation for this pop-up through dialog template

### Define a Reusable Dialog

Record the dialog instance object for reuse in the next dialog.

## Multiple Dialog Coexistence Scenarios

### New Dialog Suppressed by Existing Dialog

- **Dialog A suppresses dialog B pop-up when popping up**

  You can get the status of dialog A through dialog A object's getStatus() method to determine whether to allow dialog B to pop up.

- **Suppress dialog C pop-up when current page has dialogs**

  Get the number of dialogs on the current page by calling DialogHub's getCurrentPageDialogs() method, determine if the number is 0, and control dialog C pop-up accordingly.

### Developers Can Control Dialog Layers to Achieve Dialog Mutual Coverage

- Set dialog layer setLayerIndex()

- Set top dialog OLD_FIRST (old top dialog priority, new top dialog cannot pop up)

- Set top dialog NEW_FIRST (new dialog priority, new top dialog pops up, old top dialog is covered)

## Common Issues

### How to Handle Dialog Focus Issues

- For Sheet category dialogs, the focus behavior after dialog pop-up is consistent with the system BindSheet;
- Other category dialogs provided by DialogHub, such as CustomDialog, do not transfer focus from the parent page to the dialog by default when the dialog pops up.

  Developers can configure the dialog's requestFocusWhenShow property to achieve: when the dialog pops up, transfer the page focus to the dialog. This achieves the effect of [keyboard-avoiding dialogs](https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-hadss_dialoghub#section395448164119).

### Popup Binding Component, ID Error

[Component identifier id](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-universal-attributes-component-id#id) needs to be ensured unique by developers. After setComponentTargetId() sets the bound component id, if there are problems with the id, it will cause errors during show and abnormal dialog position.

- id does not exist: No node with this id exists, check if the bound component has set this id attribute. Error code 70000001.

### Cannot Continue Adding Properties After Calling build() and show() Interfaces

After calling the build() interface, what is returned is a Dialog instance, which only provides interfaces for updating configuration.

### Dialog Instances Created Using Templates Can Continue to Display After removeTemplate()

Deleting templates does not affect the display and related calls of dialogs previously created through templates.

### Calling isTemplateExist() Determines Template Exists, getxxxTemplate Template Reports Error 50000003

When creating and getting templates, ensure dialog types are consistent, otherwise templates cannot be obtained and error occurs, error code 50000003.

You can call queryTemplate() before getting template to query the dialog type of the template.

### Toast Dialog Top Strategy

Toast dialogs are top dialogs by default, and the top conflict strategy is TopDialogPriority.NEW_FIRST.

### Keyboard Avoidance Mode Changes

After using DialogHub for dialogs, the page keyboard avoidance mode will be changed to RESIZE. When the page has no dialogs or when page jumps, the avoidance mode is restored.
