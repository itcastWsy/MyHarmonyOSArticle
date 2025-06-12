# Practical Tips: DevEco Profiler Performance Optimization - Time

# Background

DevEco Studio development tool provides a Profiler panel that offers solutions for performance-related issues encountered during actual application development, such as slow response, animation lag, memory leaks, overheating, fast battery consumption, etc. The Profiler provides features for real-time monitoring and deep recording. From an analysis perspective, there are mainly the following dimensions for analysis:

| Scenario                        | Panel       |
| ------------------------------- | ----------- |
| Basic Time Consumption Analysis | Time        |
| Basic Memory Analysis           | Allocation  |
| Memory Leak Analysis            | Snapshot    |
| CPU Activity Analysis           | CPU         |
| Cold Start Analysis             | Launch      |
| Parallel Concurrency Analysis   | Concurrency |
| Load Frame Drop Analysis        | ArkWeb      |
| Network Diagnosis               | Network     |

![PixPin_2024-11-19_23-45-42](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/PixPin_2024-11-19_23-45-42.gif)

Note that the Profiler can only be used with real devices; simulators are temporarily not supported.

# Time

The Time panel's common requirement is to analyze function time consumption in applications. Functions serve as the basic building blocks of application development, and common time-consuming work is generally performed within functions.

Time can analyze both synchronous and asynchronous functions. It also provides a convenient one-click jump to source code feature, ensuring high efficiency for developers debugging code.

# Debug Materials

For convenient debugging, we provide the following materials:

![image-20241119235346173](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241119235346173.png)

```typescript
import { promptAction } from '@kit.ArkUI'

@Entry
@Component
struct Index {
  fn1() {
    this.fn2()
  }

  fn2() {
    this.fn3()
  }

  fn3() {
    this.fn4()
  }

  fn4() {
    promptAction.showToast({ message: `Quick task completed` })
  }

  build() {
    Column() {
      Button("Quick Task")
        .onClick(() => {
          this.fn1()
        })
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

# Set Function Names

Add the following configuration in the module's build-profile.json5, otherwise the collected functions might all be anonymous functions.

> In entry/build-profile.json5, set strip to false

```json
{
  "apiType": "stageMode",
  "buildOption": {
    "nativeLib": {
      "debugSymbol": {
        "strip": false
      }
    }
  },
```

# Select Application and Process

Click Profiler in the menu, then select the corresponding device and application. The process will be automatically selected.

![image-20241119235643530](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241119235643530.png)

Then select the Time menu:

![image-20241119235903928](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241119235903928.png)

# Start Tracking to Locate Time-Consuming Tasks

1. After everything is ready, click Create Session

2. Then click the button to start recording

   ![image-20241120000051504](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241120000051504.png)

3. During recording, start using your application to reproduce the problem. After completing operations, remember to click finish to complete this workflow recording.

   ![PixPin_2024-11-20_00-08-45](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/PixPin_2024-11-20_00-08-45.gif)

# Time Panel Introduction

After clicking to end recording, you can see this screen:

![image-20241120001739740](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241120001739740.png)

# ArkTS Callstack

Ark runtime function call swimlane displays CPU usage and virtual machine execution status based on timeline, as well as current call stack names and call types. Due to privacy and security policies, applications already published on the app market do not support recording this swimlane.

Call stacks are categorized from the language perspective into ArkTS, NAPI, and Native, and from the ownership perspective into developer code and system code. From these two aspects, call stack types can be categorized as follows:

- ArkTS: The program is executing ArkTS code
- NAPI: The program is running NAPI code
- Native: The program is executing Native code

The **bright** and **gray** colors of each type represent developer and system code respectively.

![image-20241120001849449](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241120001849449.png)

1. ArkTS Callstack contains code written by the developer. Click on it to display the details panel below

2. Weight represents the total time consumption of the function, Self represents the function's own time consumption.

   ![image-20241120002136473](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241120002136473.png)

3. **Heaviest Stack** represents the complete call stack with the longest time consumption where the selected node in the **Details** area is located

4. The green part represents code written by the developer, and you can see corresponding fn1, fn2, fn3, etc. on the right side. Through this function call stack, you can determine which function is more time-consuming

5. At this point, if you want to quickly locate the source code, double-click the function

   ![PixPin_2024-11-20_00-24-52](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/PixPin_2024-11-20_00-24-52.gif)

# User Trace

The Time panel is generally used directly to analyze synchronous code. If you want to analyze and locate asynchronous code, it's recommended to use hiTraceMeter interfaces with User Trace.

hiTraceMeter interfaces:

| Interface Name                                         | Description                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| hiTraceMeter.startTrace(name: string, taskId: number)  | Asynchronous time slice tracking interface that marks the start of a pre-tracked time-consuming task. taskId is the ID used in trace to represent association. If multiple tasks with the same name are executed in parallel, each call to startTrace has a different taskId; if tasks with the same name are executed serially, taskId can be the same. |
| hiTraceMeter.finishTrace(name: string, taskId: number) | Asynchronous time slice tracking interface. name and taskId must match the corresponding parameter values of hiTraceMeter.startTrace at the beginning of the process.                                                                                                                                                                                    |

# Debug Materials

```typescript
import { hiTraceMeter } from '@kit.PerformanceAnalysisKit'

@Entry
@Component
struct Index {
  sleep(n: number) {
    const promise = new Promise<void>(resolve => {
      setTimeout(() => {
        resolve()
      }, n * 1000)
    })
    return promise
  }

  async asyncCb1() {
    hiTraceMeter.startTrace("Async Long Task", 1)
    await this.sleep(1)
    hiTraceMeter.finishTrace("Async Long Task", 1)
    await this.asyncCb2()
  }

  async asyncCb2() {
    hiTraceMeter.startTrace("Async Long Task", 2)
    await this.sleep(2)
    hiTraceMeter.finishTrace("Async Long Task", 2)
    await this.asyncCb3()
  }

  async asyncCb3() {
    hiTraceMeter.startTrace("Async Long Task", 3)
    await this.sleep(3)
    hiTraceMeter.finishTrace("Async Long Task", 3)
  }

  build() {
    Column() {
      Button("Async Long Task")
        .onClick(async () => {
          await this.asyncCb1()
          AlertDialog.show({ message: JSON.stringify('Completed', null, 2) })
        })
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

---

![PixPin_2024-11-20_00-30-56](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/PixPin_2024-11-20_00-30-56.gif)

# Analysis

![image-20241120003154861](%E5%AE%9E%E6%88%98%E6%8A%80%E5%B7%A7%20DevEco%20Profiler%20%E6%80%A7%E8%83%BD%E8%B0%83%E4%BC%98%20Time.assets/image-20241120003154861.png)

1. Expand User Trace
2. You can see 3 pink asynchronous tasks on the User Trace swimlane
3. Click on any of the above, and you can see the specific time consumption of this function in the detail

# Summary

During application or service development, when encountering performance issues such as lag and loading time consumption, developers usually focus on the time consumption of related function execution. The Time scenario analysis task provided by DevEco Profiler can display call stack situations based on CPU and process time consumption analysis in hot areas while the application/service is running, and provide the ability to jump to related code, making it more convenient for developers to perform code optimization.
