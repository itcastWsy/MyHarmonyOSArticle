# Essential Guide for Beginners: HarmonyOS Next HMRouter Made Easy

## Preface

HMRouter serves as the page navigation solution for HarmonyOS, focusing on solving navigation logic for native pages within applications.

HMRouter encapsulates the system's Navigation at the bottom layer, integrating Navigation, NavDestination, and NavPathStack capabilities. It provides reusable route interception, page lifecycle management, custom transition animations, and extends system capabilities in terms of parameter passing, additional lifecycle features, and service-oriented routing.

The goal is to enable developers to focus on development without worrying about Navigation and NavDestination container component details and boilerplate code, shielding jump logic judgments, reducing complexity in implementing interceptors and custom transition animations, and enabling better decoupling between modules.

## Comparison

Currently, for HarmonyOS application development, the official provides two routing solutions: [Router](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-routing-V5) and [Navigation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5). Currently, Navigation is the officially recommended solution.

| Business Scenario                           | Navigation                                                 | Router                                                    |
| :------------------------------------------ | :--------------------------------------------------------- | :-------------------------------------------------------- |
| One-Many Capability                         | Supported, Auto mode adaptive single/double column display | Not supported                                             |
| Navigate to Specific Page                   | pushPath & pushDestination                                 | pushUrl & pushNameRoute                                   |
| Navigate to HSP Pages                       | Supported                                                  | Supported                                                 |
| Navigate to HAR Pages                       | Supported                                                  | Supported                                                 |
| Parameter Passing                           | Supported                                                  | Supported                                                 |
| Get Specific Page Parameters                | Supported                                                  | Not supported                                             |
| Parameter Types                             | Object-based parameters                                    | Object-based parameters, methods not supported in objects |
| Navigation Result Callback                  | Supported                                                  | Supported                                                 |
| Navigate to Singleton Page                  | Supported                                                  | Supported                                                 |
| Page Return                                 | Supported                                                  | Supported                                                 |
| Page Return with Parameters                 | Supported                                                  | Supported                                                 |
| Return to Specific Route                    | Supported                                                  | Supported                                                 |
| Page Return Dialog                          | Supported via route interception                           | showAlertBeforeBackPage                                   |
| Route Replacement                           | replacePath & replacePathByName                            | replaceUrl & replaceNameRoute                             |
| Route Stack Cleanup                         | clear                                                      | clear                                                     |
| Clear Specific Routes                       | removeByIndexes & removeByName                             | Not supported                                             |
| Transition Animations                       | Supported                                                  | Supported                                                 |
| Custom Transition Animations                | Supported                                                  | Supported, with limited animation types                   |
| Disable Transition Animations               | Supported globally and per-navigation                      | Supported by setting pageTransition method duration to 0  |
| geometryTransition Shared Element Animation | Supported (between NavDestinations)                        | Not supported                                             |
| Page Lifecycle Monitoring                   | UIObserver.on('navDestinationUpdate')                      | UIObserver.on('routerPageUpdate')                         |
| Get Page Stack Object                       | Supported                                                  | Not supported                                             |
| Route Interception                          | Supported via setInterception                              | Not supported                                             |
| Route Stack Information Query               | Supported                                                  | getState() & getLength()                                  |
| Route Stack Move Operations                 | moveToTop & moveIndexToTop                                 | Not supported                                             |
| Immersive Pages                             | Supported                                                  | Not supported, requires window configuration              |
| Set Page Title Bar and Toolbar              | Supported                                                  | Not supported                                             |
| Modal Nested Routing                        | Supported                                                  | Not supported                                             |

However, native Navigation lacks route interception, page lifecycle, and custom transition animations, and has limitations in parameter passing, additional lifecycle features, and service-oriented routing.

Therefore, HMRouter provides extensions and enhancements for these features.

## Learning Objectives

Next, this article will guide you through hands-on application of [HMRouter](https://gitee.com/hadss/hmrouter).

## Project Structure

After creating a new project, create a `Cart` dynamic shared package module:

1. Project directory name is study
2. Entry module is entry
3. cart is an hsp module

![image-20241224094003606](小白必看%20HarmonyOS%20Next%20HMRouter%20轻松上手秘籍.assets/image-20241224094003606.png)

## Environment Configuration

### Install Dependencies with ohpm

```
ohpm install @hadss/hmrouter
ohpm install @hadss/hmrouter-transitions
```

### Compilation Plugin Configuration

1. Modify the project's `hvigor/hvigor-config.json` file to include the routing compilation plugin

   ```json
   {
     "dependencies": {
       "@hadss/hmrouter-plugin": "^1.0.0-rc.10"
       // Use npm repository version number
     }
     // ...other configurations
   }
   ```

2. Import the routing compilation plugin in modules using HMRouter by modifying `hvigorfile.ts`

   > Our project modules are either Hap, Har, or Hsp. Use the corresponding syntax based on your current module type

   1. **Hap**

      ```javascript
      // entry/hvigorfile.ts  entry module hvigorfile.ts
      import { hapTasks } from "@ohos/hvigor-ohos-plugin";
      import { hapPlugin } from "@hadss/hmrouter-plugin";

      export default {
        system: hapTasks,
        plugins: [hapPlugin()], // All modules using HMRouter tags need configuration, consistent with module type
      };
      ```

   2. **Har**

      ```javascript
      import { harTasks } from "@ohos/hvigor-ohos-plugin";
      import { harPlugin } from "@hadss/hmrouter-plugin";

      export default {
        system: harTasks,
        plugins: [harPlugin()], // All modules using HMRouter tags need configuration, consistent with module type
      };
      ```

   3. **Hsp**

      ```javascript
      import { hspTasks } from "@ohos/hvigor-ohos-plugin";
      import { hspPlugin } from "@hadss/hmrouter-plugin";

      export default {
        system: hspTasks,
        plugins: [hspPlugin()], // All modules using HMRouter tags need configuration, consistent with module type
      };
      ```

### Initialize Routing Framework

> entry/src/main/ets/entryability/EntryAbility.ets

```javascript
export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    HMRouterMgr.init({
      context: this.context,
    });
  }
}
```

### Define Routing Entry Point

> entry/src/main/ets/pages/Index.ets

The current page serves as the root container for the entire routing system

```typescript
import { HMDefaultGlobalAnimator, HMNavigation } from "@hadss/hmrouter";
import { AttributeUpdater } from "@kit.ArkUI";

class MyNavModifier extends AttributeUpdater<NavigationAttribute> {
  initializeModifier(instance: NavigationAttribute): void {
    // instance.hideNavBar(true);  // Comment out for now, otherwise results won't be visible
  }
}

@Entry
@Component
export struct Index {
  modifier: MyNavModifier = new MyNavModifier();

  build() {
    // @Entry needs to be wrapped in another container component, Column or Stack
    Column() {
      // Use HMNavigation container
      HMNavigation({
        navigationId: 'mainNavigation', homePageUrl: 'MainPage',
        options: {
          standardAnimator: HMDefaultGlobalAnimator.STANDARD_ANIMATOR,
          dialogAnimator: HMDefaultGlobalAnimator.DIALOG_ANIMATOR,
          modifier: this.modifier
        }
      }) {
        Column({ space: 10 }) {
          Button("Navigate to Login Page")
        }
      }
    }
    .height('100%')
    .width('100%')
  }
}
```

## Intra-Module Navigation

![PixPin_2024-12-24_10-22-23](小白必看%20HarmonyOS%20Next%20HMRouter%20轻松上手秘籍.assets/PixPin_2024-12-24_10-22-23.gif)

Let's first demonstrate navigation to a page within the current module.

HMRouter defaults to specifying the page directory as `entry/src/main/ets/components`

Create a new component `entry/src/main/ets/components/LoginPage.ets`

```typescript
import { HMRouter } from "@hadss/hmrouter"

@HMRouter({
  pageUrl: 'LoginPage',
})
@Component
export struct LoginPage {
  build() {
    Column() {
      Button('Login Page')
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

Now, return to the homepage to implement click navigation to login

```typescript
Button("Navigate to Login Page").onClick(() => {
  HMRouterMgr.push({ pageUrl: "LoginPage" });
});
```

## Route Parameter Passing

### Passing Parameters

```javascript
HMRouterMgr.push({ pageUrl: "LoginPage", param: { data } });
```

### Receiving Parameters

```javascript
HMRouterMgr.getCurrentParam(HMParamType.all);
```

## Specify Compilation Directory

The login page we just created was stored in the components directory. In actual development, we might use **views** to store **pages**, so let's configure this.

Create a routing compilation plugin configuration file `study/hmrouter_config.json` in the project root directory (optional)

```json
{
  "scanDir": ["src/main/ets/views"]
}
```

Then rename the previous folder from `entry/src/main/ets/components` to `entry/src/main/ets/views`

Recompile and execute.

## Inter-Module Navigation

The previous demonstration was within the same module. Now let's demonstrate navigation between different modules.

The goal is to navigate from the entry module to the cart module.

### Configure Compilation Plugin for cart Module

> cart is hsp
>
> cart/hvigorfile.ts

```javascript
import { hspTasks } from "@ohos/hvigor-ohos-plugin";
import { hspPlugin } from "@hadss/hmrouter-plugin";

export default {
  system: hspTasks,
  plugins: [hspPlugin()], // All modules using HMRouter tags need configuration, consistent with module type
};
```

### Create Shopping Cart Detail Page

> cart/src/main/ets/views/CartDetail.ets

```typescript
import { HMRouter } from "@hadss/hmrouter"

@HMRouter({
  pageUrl: 'CartDetail',
})
@Component
export struct CartDetail {
  build() {
    Column() {
      Button('This is the Shopping Cart Detail Page')
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

### Import cart Module in entry Module

> entry/oh-package.json5

```json
  "dependencies": {
    "cart": "file:../cart"
  },
```

### Navigate from Homepage

> entry/src/main/ets/pages/Index.ets

```javascript
Button("Navigate to Shopping Cart Detail Page").onClick(() => {
  HMRouterMgr.push({ pageUrl: "CartDetail" });
});
```

### Result

![PixPin_2024-12-24_10-48-50](小白必看%20HarmonyOS%20Next%20HMRouter%20轻松上手秘籍.assets/PixPin_2024-12-24_10-48-50.gif)

## Navigation Animations

We can specify navigation animations when jumping between pages.

This involves two steps:

1. Define animation
2. Use animation

### Define Animation

Assuming A navigates to B, then B uses the animation. For convenience, you can define the animation in page B.

Let's continue with the above example.

index navigates to CarDetail, so we define the animation in CarDetail

> cart/src/main/ets/views/CartDetail.ets

```javascript
@HMAnimator({ animatorName: "liveCommentsAnimator" })
export class liveCommentsAnimator implements IHMAnimator {
  effect(enterHandle: HMAnimatorHandle, exitHandle: HMAnimatorHandle): void {
    // Enter animation
    enterHandle.start(
      (
        translateOption: TranslateOption,
        scaleOption: ScaleOption,
        opacityOption: OpacityOption
      ) => {
        translateOption.y = "100%";
      }
    );
    enterHandle.finish(
      (
        translateOption: TranslateOption,
        scaleOption: ScaleOption,
        opacityOption: OpacityOption
      ) => {
        translateOption.y = "0";
      }
    );
    enterHandle.duration = 500;

    // Exit animation
    exitHandle.start(
      (
        translateOption: TranslateOption,
        scaleOption: ScaleOption,
        opacityOption: OpacityOption
      ) => {
        translateOption.y = "0";
      }
    );
    exitHandle.finish(
      (
        translateOption: TranslateOption,
        scaleOption: ScaleOption,
        opacityOption: OpacityOption
      ) => {
        translateOption.y = "100%";
      }
    );
    exitHandle.duration = 500;
  }
}
```

### Use Animation

Use in HMRouter

```javascript
@HMRouter({
  pageUrl: 'CartDetail',
  // 2. Use animation
  animator: "liveCommentsAnimator"
})
@Component
export struct CartDetail {
  build() {
    Column() {
      Button('This is the Shopping Cart Detail Page')
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

### Result

![PixPin_2024-12-24_11-22-25](小白必看%20HarmonyOS%20Next%20HMRouter%20轻松上手秘籍.assets/PixPin_2024-12-24_11-22-25.gif)

## Interceptors

Interceptors can be divided into 2 types: local interceptors and global interceptors.

Marked on objects implementing `IHMInterceptor`, declaring this object as an interceptor.

- interceptorName: string, interceptor name, required
- priority: number, interceptor priority, higher numbers have higher priority, optional, default is 9
- global: boolean, whether it's a global interceptor. When set to true, all navigations pass through this interceptor; default is false. When false, it needs to be configured in @HMRouter's interceptors to take effect.

**Execution Timing:**

Called before route stack changes and before transition animations occur.

1. When push/replace routing occurs and pageUrl is empty, the interceptor won't execute; pageUrl path must be provided
2. When the target pageUrl page doesn't exist, global and source page interceptors execute. If no interceptor executes DO_REJECT, then the route's onLost callback is executed
3. When the target pageUrl page exists, global, source page, and target page interceptors execute

**Interceptor Execution Order:**

1. Execute in priority order, regardless of custom or global interceptors. When priorities are equal, custom interceptors defined in @HMRouter execute first
2. When priorities are equal, execution order is srcPage > targetPage > global

> srcPage represents the navigation source page.
>
> targetPage represents the page displayed at navigation end.

### Local Interceptors

Local interceptors only affect a single page. Let's test with the login page - when navigating from homepage to login page, the login page's interceptor will trigger.

> entry/src/main/ets/views/LoginPage.ets

**Define Interceptor**

```javascript
@HMInterceptor({ interceptorName: "Loginterceptor", global: false })
export class JumpInfoInterceptor implements IHMInterceptor {
  handle(info: HMInterceptorInfo): HMInterceptorAction {
    console.log("Interceptor", JSON.stringify(info));
    // DO_NEXT  Normal navigation
    // DO_REJECT Reject navigation
    return HMInterceptorAction.DO_NEXT;
  }
}
```

**Use Interceptor**

```typescript
// Use interceptor
@HMRouter({
  pageUrl: 'LoginPage',
  interceptors: ['Loginterceptor']
})
@Component
export struct LoginPage {
  build() {
    Column() {
      Button('Login Page')
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

**Output Result**

```json
{
  "srcName": "HM_NavBar",
  "targetName": "LoginPage",
  "type": "push",
  "routerPathInfo": {
    "pageUrl": "LoginPage"
  },
  "context": {
    "instanceId_": 100000
  },
  "isSrc": false
}
```

### Global Interceptors

Use directly on the index page

```javascript
  aboutToAppear(): void {
    // Register global interceptor
    HMRouterMgr.registerGlobalInterceptor({
      interceptorName: "Interceptor Name",
      // Priority
      priority: 1,
      // Interceptor
      interceptor: {
        // Handler function
        handle(info: HMInterceptorInfo) {
          return HMInterceptorAction.DO_NEXT
        }
      }
    })
  }
```

## Lifecycle

```
@HMLifecycle(lifecycleName, priority, global)
```

Marked on objects implementing IHMLifecycle, declaring this object as a custom lifecycle handler.

- lifecycleName: string, custom lifecycle handler name, required
- priority: number, lifecycle priority, higher numbers have higher priority, optional, default is 9
- global: boolean, whether it's a global lifecycle. When set to true, all page lifecycle events are forwarded to this object; default is false

**Lifecycle Trigger Order:**

Triggered in priority order, regardless of custom or global lifecycle. When priorities are equal, custom lifecycles defined in @HMRouter execute first.

We can continue testing the corresponding lifecycle on the login page.

> entry/src/main/ets/views/LoginPage.ets

### Define Lifecycle

```javascript
@HMLifecycle({ lifecycleName: 'LoginLifecycle' })
export class PageDurationLifecycle implements IHMLifecycle {
  private time: number = 0;

  onShown(ctx: HMLifecycleContext): void {
    this.time = new Date().getTime();
    console.log("Lifecycle", JSON.stringify(ctx))
  }

  onHidden(ctx: HMLifecycleContext): void {
    const duration = new Date().getTime() - this.time;
  }
}
```

### Use Lifecycle

```javascript
// Use interceptor
@HMRouter({
  pageUrl: 'LoginPage',
  interceptors: ['Loginterceptor'],
  lifecycle: "LoginLifecycle"
})
@Component
export struct LoginPage {
  build() {
    Column() {
      Button('Login Page')
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

### Complete Lifecycle

```javascript
export interface HMLifecycleContext {
    uiContext: UIContext;
    navContext?: NavDestinationContext;
}
export type HMLifecycleCallback = (ctx: HMLifecycleContext) => boolean | void;
export interface IHMLifecycle {
    onPrepare?(ctx: HMLifecycleContext): void;
    onAppear?(ctx: HMLifecycleContext): void;
    onDisAppear?(ctx: HMLifecycleContext): void;
    onShown?(ctx: HMLifecycleContext): void;
    onHidden?(ctx: HMLifecycleContext): void;
    onWillAppear?(ctx: HMLifecycleContext): void;
    onWillDisappear?(ctx: HMLifecycleContext): void;
    onWillShow?(ctx: HMLifecycleContext): void;
    onWillHide?(ctx: HMLifecycleContext): void;
    onReady?(ctx: HMLifecycleContext): void;
    onBackPressed?(ctx: HMLifecycleContext): boolean;
}
```

### Page Component and Lifecycle Data Interaction

Objects can be initialized in lifecycle instances and retrieved in UI components as state variables.

```javascript
import {
  HMInterceptor,
  HMInterceptorAction,
  HMInterceptorInfo,
  HMLifecycle,
  HMLifecycleContext,
  HMRouter,
  HMRouterMgr,
  IHMInterceptor,
  IHMLifecycle
} from '@hadss/hmrouter';

// Define interceptor
@HMInterceptor({ interceptorName: 'Loginterceptor', global: false })
export class JumpInfoInterceptor implements IHMInterceptor {
  handle(info: HMInterceptorInfo): HMInterceptorAction {
    console.log("Interceptor", JSON.stringify(info))
    return HMInterceptorAction.DO_NEXT;
  }
}

@Observed
export class ObservedModel {
  isLoad: boolean = false;
}

@HMLifecycle({ lifecycleName: 'LoginLifecycle' })
export class PageDurationLifecycle implements IHMLifecycle {
  model: ObservedModel = new ObservedModel()

  onAppear(ctx: HMLifecycleContext): void {
  }
}

// Use interceptor
@HMRouter({
  pageUrl: 'LoginPage',
  interceptors: ['Loginterceptor'],
  lifecycle: "LoginLifecycle"
})
@Component
export struct LoginPage {
  @State model: ObservedModel | null =
    (HMRouterMgr.getCurrentLifecycleOwner()?.getLifecycle() as PageDurationLifecycle).model;

  build() {
    Column() {
      Button('Login Page' + this.model?.isLoad)
        .onClick(() => {
          this.model!.isLoad = !this.model?.isLoad
        })
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

## Summary

The official documentation for hmrouter is quite scattered and needs to be studied in conjunction with supporting documentation.

### HMRouter Interface and Properties List

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/docs/Reference.md)

### HMRouterPlugin Compilation Plugin Usage Instructions

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/HMRouterPlugin/README.md)

### HMRouterTransitions Advanced Transition Animation Usage Instructions

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/HMRouterTransitions/README.md)

### Custom Template Usage Instructions

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/docs/CustomTemplate.md)

### Custom Transition Animation Usage Instructions

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/docs/CustomTransition.md)

### More Common Solutions

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/docs/Scene.md)

### SampleCode

[View Details](https://gitee.com/harmonyos_samples/HMRouter)

### More Examples

[View Details](https://gitee.com/hadss/hmrouter/tree/dev/HMRouterExamples)

### FAQ

[View Details](https://gitee.com/hadss/hmrouter/blob/dev/docs/FAQ.md)

### Principles Introduction

[View Details](https://gitee.com/link?target=https%3A%2F%2Fdeveloper.huawei.com%2Fconsumer%2Fcn%2Fforum%2Ftopic%2F0207153170697988820%3Ffid%3D0109140870620153026)
