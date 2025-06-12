# AtomicService

In the era of Internet of Everything, the number of devices per person continues to rise, device types and usage scenarios are becoming more diverse, making application development and application entry points increasingly complex. Against this backdrop, application providers and users urgently need a new service delivery method that makes application development simpler and service acquisition and usage (such as listening to music, calling a taxi, etc.) more convenient. For this reason, HarmonyOS not only supports traditional applications that require installation (hereinafter referred to as traditional applications), but also supports more convenient and quick-to-use installation-free applications, namely AtomicServices.

AtomicService is a lightweight application form provided by HarmonyOS, featuring instant opening and direct access, pure and refreshing experience; accompanying services that are timely and appropriate; instant use and departure with account following; dual-natured integration with embedded operation; native intelligence with full-domain search; efficient development with inherent trustworthiness.

AtomicServices can be independently published, distributed, and operated, independently implementing business closed loops, and can significantly improve the efficiency of information and service acquisition.

# Differences Between AtomicServices and Applications

![image-20241124191141178](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191141178.png)

# AtomicService Development Journey

![image-20241124191156792](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191156792.png)

# Create a New Project on AGC Platform

[Link](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/harmonyOSDevPlatform/172249065903274453)

One project can contain multiple applications

![image-20241124191241104](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191241104.png)

# Create a New AtomicService Application in AGC

![image-20241124191300505](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191300505.png)

# Create a New Local AtomicService Project

![image-20241124191643071](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191643071.png)

---

If you have successfully created an AtomicService on the AGC platform, it will be automatically displayed here

![image-20241124191744693](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191744693.png)

# Modify AtomicService Name

![image-20241124191327227](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191327227.png)

# Modify AtomicService Icon

Important: app store review is very strict

![image-20241124191343134](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191343134.png)

1. First download any image yourself

2. Use drawing tools to modify image properties to 1024px

   ![PixPin_2024-11-24_19-15-16](HarmonyOS%20Next%20简单上手元服务开发.assets/PixPin_2024-11-24_19-15-16.gif)

3. Use Image Asset in the development tool to create images

![image-20241124191814129](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124191814129.png)

![image-20241124192132959](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124192132959.png)

# Add AtomicService Cards

> Windows emulator does not support adding AtomicService cards

## Create New Card

![image-20241124192208171](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124192208171.png)

![image-20241124192221915](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124192221915.png)

# Restrictions in AtomicService Development

## Each package size cannot exceed 2M

![image-20241124192250939](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124192250939.png)

## Total AtomicService project size is generally 10M, can apply for 20M in special cases

## Maximum 16 service cards

## Service cards cannot arbitrarily jump to other applications or AtomicServices through cards

## Service cards cannot use call events

# AtomicServiceTabs

[Implement tab switching and title setting](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/ohos-atomicservice-atomicservicetabs-V5#示例)

![image-20241124192911325](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124192911325.png)

```typescript
// Index.ets
import { AtomicServiceTabs, TabBarOptions, TabBarPosition, OnContentWillChangeCallback } from '@kit.ArkUI';

@Entry
@Component
struct Index {
  @State message: string = 'Home';
  @State currentIndex: number = 0;

  @Builder
  tabContent1() {
    Column().width('100%').height('100%').alignItems(HorizontalAlign.Center).backgroundColor('#00CB87')
  }

  @Builder
  tabContent2() {
    Column().width('100%').height('100%').backgroundColor('#007DFF')
  }

  @Builder
  tabContent3() {
    Column().width('100%').height('100%').backgroundColor('#FFBF00')
  }

  build() {
    Stack() {
      AtomicServiceTabs({
        tabContents: [
          () => {
            this.tabContent1()
          },
          () => {
            this.tabContent2()
          },
          () => {
            this.tabContent3()
          }
        ],
        tabBarOptionsArray: [
          new TabBarOptions($r('sys.media.ohos_ic_public_phone'), 'Green', Color.Black, Color.Blue),
          new TabBarOptions($r('sys.media.ohos_ic_public_location'), 'Blue', Color.Black, Color.Blue),
          new TabBarOptions($r('sys.media.ohos_ic_public_more'), 'Yellow', Color.Black, Color.Blue)
        ],
        tabBarPosition: TabBarPosition.BOTTOM,
        barBackgroundColor: $r('sys.color.ohos_id_color_bottom_tab_bg'),

      })

    }.height('100%')
  }
}
```

# AtomicServiceNavigation

[Route management](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/ohos-atomicservice-atomicservicenavigation-V5#示例)

![PixPin_2024-11-24_19-32-52](HarmonyOS%20Next%20简单上手元服务开发.assets/PixPin_2024-11-24_19-32-52.gif)

## Index

```typescript
// Index.ets
import {
  AtomicServiceNavigation,
  NavDestinationBuilder,
  AtomicServiceTabs,
  TabBarOptions,
  TabBarPosition
} from '@kit.ArkUI';
import { HomeView } from '../views/HomeView';

export interface IParam {
  id: number
}

@Entry
@Component
struct Index {
  @StorageProp("safeTop")
  safeTop: number = 0
  @StorageProp("safeBottom")
  safeBottom: number = 0
  @State message: string = 'Theme';
  // Page navigation
  @Provide("pathStack")
  pathStack: NavPathStack = new NavPathStack();

  @Builder
  navigationContent() {
    Button("Content")
      .onClick(() => {
        const param: IParam = { id: 100 }
        this.pathStack.pushPathByName("HomeView", param)
      })
  }

  @Builder
  // Route page mapping - previously navNavigation required modifying configuration files!!!
  pageMap(name: string) {
    if (name === 'HomeView') {
      HomeView()
    }
  }

  build() {
    Row() {
      Column() {
        // Similar to navNavigation
        AtomicServiceNavigation({
          // Content displayed directly in container
          navigationContent: () => {
            this.navigationContent()
          },
          // Title!!!
          title: this.message,
          //
          titleOptions: {
            backgroundColor: '#fff',
            isBlurEnabled: false
          },
          // Route page mapping
          navDestinationBuilder: this.pageMap,
          navPathStack: this.pathStack,
          mode: NavigationMode.Stack
        })
      }
      .width('100%')
    }
    .height('100%')
    .padding({
      top: this.safeTop,
      bottom: this.safeBottom
    })
  }
}
```

## HomeView

```typescript
import { IParam } from "../pages/Index"
import { promptAction } from "@kit.ArkUI"

@Component
export struct HomeView {
  @Consume("pathStack")
  pathStack: NavPathStack

  aboutToAppear() {
    const param = this.pathStack.getParamByName("HomeView").pop() as IParam
    promptAction.showToast({ message: `${param.id}` })
  }

  build() {
    NavDestination() {
      Column() {
        Text("homeView")
          .fontSize(50)
      }
      .width("100%")
      .height("100%")
      .justifyContent(FlexAlign.Center)
    }
  }
}
```

# AtomicServiceWeb

Provides developers with Web advanced components that meet customization requirements, shields interfaces in native Web components that don't need attention, and provides JS extension capabilities.

AtomicServiceWeb will no longer support interfaces like [registerJavaScriptProxy](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V13/js-apis-webview-V13#registerjavascriptproxy) and [runJavaScript](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V13/js-apis-webview-V13#runjavascript-1). If you need to call AtomicService native page object methods from web pages loaded by Web components, you can obtain relevant data through interfaces provided by JS SDK. If JS SDK interfaces cannot meet requirements, you can also pass parameters so that AtomicService native pages execute object methods and then obtain results, passing the results to web pages loaded by Web components.

Within AtomicServices, only the AtomicServiceWeb component can be used to display Web pages, and business domain names need to be configured.

Please refer to: [AtomicServiceWeb Component Reference](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V5/ohos-atomicservice-atomicserviceweb-V5), [Configure Business Domain Names](https://developer.huawei.com/consumer/cn/doc/atomic-guides-V5/agc-help-harmonyos-business-domain-V5)

## Basic Usage

1. Created component **WebHome** to display **AtomicServiceWeb** container

   ```JavaScript
   import { AtomicServiceWeb, AtomicServiceWebController } from '@ohos.atomicservice.AtomicServiceWeb'

   @Entry
   @Component
   export struct WebHome {
     @State
     ascontroller: AtomicServiceWebController = new AtomicServiceWebController()

     build() {
       NavDestination() {
         Column() {
           AtomicServiceWeb({ src: $rawfile("index.html"), controller: this.ascontroller })
             .width("100%")
             .height("100%")
         }
         .width("100%")
         .height("100%")
         .justifyContent(FlexAlign.Center)
       }
     }
   }
   ```

2. Create H5 page index.html

   ![image-20241124193502932](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124193502932.png)

   ```html
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="UTF-8" />
       <meta name="viewport" content="width=device-width, initial-scale=1.0" />
       <title>Document</title>
       <style>
         body {
           background-color: red;
         }
       </style>
     </head>
     <body></body>
   </html>
   ```

3. Added navigation to WebHome in Index

   ![image-20241124193524156](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124193524156.png)

4. WebHome uses **AtomicServiceWeb container to display content**

   ![PixPin_2024-11-24_19-40-02](HarmonyOS%20Next%20简单上手元服务开发.assets/PixPin_2024-11-24_19-40-02.gif)

## Passing Data from HarmonyOS Pages to H5 Pages

Pass through src attribute

![image-20241124194101296](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124194101296.png)

### H5 Page Receiving and Processing

![image-20241124194121169](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124194121169.png)

## H5 Page Calling AtomicService Methods

In HTML, if you need to call AtomicService APIs, you can integrate AtomicService JS SDK to call relevant APIs for access

**Installation**

```
npm install @atomicservice/asweb-sdk
```

## Usage Methods

### es6

```typescript
import has from "@atomicservice/asweb-sdk";
has.asWeb.getEnv({ callback: (err, res) => {} });
```

### umd

```xml
<script src="../dist/asweb-sdk.umd.js"></script>
<script>
  has.asWeb.getEnv({
    callback: (err, res) => {
    }
  });
</script>
```

### More Methods

![image-20241124194443319](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124194443319.png)

# AtomicService Network Requests

1. Apply for permissions on AGC platform by adding request domain names to whitelist - **Must be done for production**

   ![image-20241124194529204](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124194529204.png)

2. On phone - Settings - System - Apps - AtomicService Exemption Management **Enable** - Can send requests during development

   > Emulator is not restricted

## Summary

![image-20241124194617678](HarmonyOS%20Next%20简单上手元服务开发.assets/image-20241124194617678.png)
