# HarmonyOS Next Concurrent Programming: TaskPool and Worker

## Overview

![image-20241110140848578](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110140848578.png)

# Introduction

Concurrency refers to multiple code segments executing simultaneously at the same time. In ArkTS programming, concurrency is divided into asynchronous concurrency and multithreaded concurrency.

# Asynchronous Concurrency

Asynchronous concurrency is not true concurrency. For example, on a single-core device, executing multiple code segments simultaneously is actually achieved through rapid CPU scheduling. It's like a driver who can only drive one car at a time - they cannot actually drive two cars simultaneously. If we take an extreme example, this driver can appear to be driving two cars at the same time.

We divide driving into two steps:

1. Getting in the car
2. Starting to drive

If we reduce the time for the driver to get in the car to an extremely small amount, say 0.00001 seconds, then this driver can appear to be **driving two cars simultaneously**.

**Driving Xiaomi SU7**

1. Get in car (0.00001 seconds)
2. Drive

Immediately get out, run to the other car, then

**Driving Wenjie M7**

1. Get in car (0.00001 seconds)
2. Drive

---

As you can see, as long as the task switching speed is fast enough, it can be understood as driving two cars simultaneously. This is what our busy CPU needs to do.

We can consider setTimeout and Promise/Async as asynchronous concurrency technologies.

# Multithreaded Concurrency

Multithreaded concurrency models are commonly divided into concurrency models based on **memory sharing** and concurrency models based on **message communication**.

The Actor concurrency model, as a typical representative of message communication-based concurrency models, doesn't require developers to face the complex and occasional problems brought by locks, while maintaining relatively high concurrency, thus gaining widespread support and usage.

Currently, ArkTS provides **TaskPool** and **Worker** two concurrent capabilities. Both **TaskPool** and **Worker** are implemented based on the **Actor** concurrency model. **TaskPool** is actually a layer of encapsulation based on **Worker**.

# Comparison of TaskPool and Worker Implementation Features

> Mainly differences in usage methods

1. TaskPool can directly pass parameters, Worker needs to encapsulate parameters manually
2. TaskPool can directly receive return data, Worker receives through onmessage
3. TaskPool automatically manages the upper limit of task pool numbers, Worker has a maximum of 64 threads, depending on memory
4. TaskPool task execution time limit: synchronous code up to 3 minutes, asynchronous code unlimited. Worker unlimited
5. TaskPool can set task priority, Worker doesn't support
6. TaskPool supports task cancellation, Worker doesn't support

![image-20241110105619585](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110105619585.png)

# Comparison of TaskPool and Worker Application Scenarios

Both TaskPool and Worker support multithreaded concurrent capabilities. Since TaskPool's worker threads bind to system **scheduling priorities** and support load balancing (automatic scaling), while Worker requires developers to create them manually, which involves creation overhead and doesn't support setting scheduling priorities, **TaskPool outperforms Worker in terms of performance, so TaskPool is recommended for most scenarios**.

TaskPool focuses on independent task dimensions, where tasks execute in threads without needing to care about thread lifecycle. Super long tasks (longer than 3 minutes and not long-running tasks) will be automatically recycled by the system. Worker focuses on thread dimensions, supporting long-term thread occupation for execution, requiring active thread lifecycle management.

**Common development scenarios and specific applicable descriptions are as follows:**

- **Tasks running longer than 3 minutes** (not including time consumption of Promise and async/await asynchronous calls, such as network downloads, file read/write and other I/O task time consumption). For example, background prediction algorithm training for 1 hour and other CPU-intensive tasks, **need to use Worker**.

- Series of related synchronous tasks. For example, in scenarios that need to create and use handles, handle creation is different each time, and the handle needs to be permanently saved to ensure operations using that handle, need to use Worker.

- **Tasks requiring priority setting**. For example, in gallery histogram drawing scenarios, background calculated histogram data will be used for foreground interface display, affecting user experience, requiring high priority processing, **need to use TaskPool**.

- **Tasks requiring frequent cancellation**. For example, in gallery large image browsing scenarios, to improve experience, it will simultaneously cache 2 images on each side of the current image. When sliding to one side to jump to the next image, it needs to cancel a cache task on the other side, **need to use TaskPool**.

- **Large number or scattered scheduling point tasks**. For example, multiple modules of large applications contain multiple time-consuming tasks, inconvenient to use Worker for load management, recommended to use TaskPool.

# Inter-thread Communication Objects

Inter-thread communication refers to data exchange behavior between concurrent multithreads. Since the ArkTS language is compatible with TS/JS, its runtime implementation, like all other JS engines, provides concurrent capabilities based on the Actor memory isolation concurrency model.

## Common Objects

Common objects are passed across threads through **copying**.

## ArrayBuffer Objects

ArrayBuffer contains a block of Native memory internally. Its JS object shell, like common objects, needs to be copied and passed through serialization and deserialization, but Native memory has two transmission methods: copying and transfer.

## SharedArrayBuffer Objects

SharedArrayBuffer contains a block of Native memory internally, **supports sharing across concurrent instances**, but access and modification need to use the **Atomics** class to prevent data races.

## Transferable

Transferable objects (also called NativeBinding objects) refer to JS objects that bind to C++ objects, with main functionality provided by C++.

Can achieve **sharing and transfer modes**.

## Sendable Objects

On traditional JS engines, there's only one optimization method for object concurrent communication overhead: implementing it at the Native side, reducing concurrent communication overhead through transfer or sharing methods of [Transferable objects](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/transferabled-object-V5). However, developers still have many object concurrent communication demands, and this problem hasn't been solved in industry JS engine implementations.

ArkTS provides Sendable object types, supporting **reference passing** during concurrent communication to solve the above problems.

Sendable objects are shareable, pointing to the same JS object before and after cross-thread operations. If they contain JS or Native content, they can all be directly shared. If the underlying implementation is Native, thread safety needs to be considered.

# Multithreaded Concurrent Scenarios

For common business scenarios, they can mainly be categorized into three types of concurrent tasks: [time-consuming tasks](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/time-consuming-task-overview-V5), [long-running tasks](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/long-time-task-overview-V5), [resident tasks](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/resident-task-overview-V5).

## Time-consuming Task Concurrent Scenarios Introduction

Time-consuming tasks refer to tasks that need long execution time. If executed on the main thread, they may cause application lag, frame drops, slow response and other problems. Typical time-consuming tasks include CPU-intensive tasks, I/O-intensive tasks, and synchronous tasks.

For time-consuming tasks, common business scenario classifications are shown below:

| **Common Business Scenarios**     | **Specific Business Description**                                                                                                                  | **CPU-intensive** | **I/O-intensive** | **Synchronous Tasks** |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ----------------- | --------------------- |
| **Image/Video Encoding/Decoding** | **Encode/decode images or videos before display.**                                                                                                 | **√**             | **√**             | **×**                 |
| **Compression/Decompression**     | **Decompress local compressed packages or compress local files.**                                                                                  | **√**             | **√**             | **×**                 |
| **JSON Parsing**                  | **Serialization and deserialization operations on JSON strings.**                                                                                  | **√**             | **×**             | **×**                 |
| **Model Computing**               | **Model computing analysis on data.**                                                                                                              | **√**             | **×**             | **×**                 |
| **Network Downloads**             | Intensive network requests to download resources, images, files, etc.                                                                              | ×                 | √                 | ×                     |
| **Database Operations**           | Save chat records, page layout information, music list information to database, or read database to display related information when app restarts. | ×                 | √                 | ×                     |

## Long-running Task Concurrent Scenarios Introduction

In application business implementation, tasks that need to **run for long periods irregularly** are called long-running tasks. If long-running tasks are executed on the main thread, they will block the main thread's UI business, causing lag and frame drops that affect user experience. Therefore, these independent long-running tasks usually need to be placed in separate sub-threads for execution.

Typical long-running task scenarios are shown below:

| Common Business Scenarios         | Specific Business Description                                                                                                                      |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Regular Sensor Data Collection    | Periodically collect sensor information (such as location, speed sensors, etc.), running continuously for long periods during application runtime. |
| Listen to Socket Port Information | Long-term listening to Socket data, needing irregular response processing.                                                                         |

The above business scenarios are all independent long-running tasks with long execution cycles and simple external interactions. After being dispatched to background threads, they need irregular responses to get results. These types of tasks can use TaskPool to simplify development workload, avoid managing complex lifecycles, avoid thread proliferation. Developers only need to put the above independent long-running tasks into TaskPool queues and wait for results.

## Resident Task Concurrent Scenarios

In application business implementation, for some long time-consuming (longer than 3min) resident task scenarios with low concurrency, Worker is used to run these time-consuming logic in background threads, avoiding blocking the main thread and causing frame drops and lag that affect user experience.

Resident tasks refer to tasks that take longer compared to short-term tasks, possibly consistent with main thread lifecycle. Compared to long-running tasks, resident tasks are more inclined to be thread-bound tasks with longer single execution times (such as over 3 minutes).

For resident tasks, relatively common business scenarios are as follows:

| Common Business Scenarios          | Specific Business Description                                                                      |
| ---------------------------------- | -------------------------------------------------------------------------------------------------- |
| Game Backend Scenarios             | Start sub-thread as main logic thread for game business, UI thread only responsible for rendering. |
| Long Time-consuming Task Scenarios | Background long-term model prediction tasks, or hardware testing, etc.                             |

# TaskPool Usage

## TaskPool Basic Usage 1

The simplest usage of TaskPool is as follows.

Note that the function executing the task must be decorated with **@Concurrent**.

```typescript
taskpool.execute(function, parameters)
```

---

![image-20241110123246585](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110123246585.png)

```typescript
import { taskpool } from '@kit.ArkTS'

// Task specific logic
@Concurrent
function func1(n: number) {
  return 1 + n
}

@Entry
@Component
struct Index {
  @State
  num: number = 0
  fn1 = async () => {
    // Execute task
    const res = await taskpool.execute(func1, this.num)
    this.num = Number(res)
  }

  build() {
    Column() {
      Button("TaskPool Basic Usage " + this.num)
        .onClick(this.fn1)
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

## TaskPool Basic Usage 2

TaskPool basic usage can also specify task priority like:

```
taskpool.execute(task, priority)
```

![image-20241110124508588](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110124508588.png)

```typescript
import { taskpool } from '@kit.ArkTS'

// Task specific logic
@Concurrent
function func1(n: number) {
  console.log(`Task ${n}`)
}

@Entry
@Component
struct Index {
  @State
  num: number = 0
  fn1 = async () => {
    // Create and execute 10 tasks simultaneously, specify the 10th task as highest priority
    for (let index = 1; index <= 10; index++) {
      // Create task, pass in function to execute and its parameters
      const task = new taskpool.Task(func1, index)
      if (index !== 10) {
        // Execute task and specify priority taskpool.Priority.LOW
        taskpool.execute(task, taskpool.Priority.LOW)
      } else {
        taskpool.execute(task, taskpool.Priority.HIGH)
      }
    }
  }

  build() {
    Column() {
      Button("TaskPool Basic Usage " + this.num)
        .onClick(this.fn1)
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

## TaskPool Basic Usage 3

TaskPool can also support passing in task groups, similar to Promise.all, can wait for all time-consuming tasks to finish.

```typescript
import { taskpool } from '@kit.ArkTS'

// Task specific logic
@Concurrent
function func1(n: number) {
  const promise = new Promise<number>(resolve => {
    resolve(n)
    setTimeout(() => {
    }, 1000 + n * 100)
  })
  return promise
}

@Entry
@Component
struct Index {
  fn1 = async () => {
    // Create task group
    const taskGroup = new taskpool.TaskGroup()
    for (let index = 1; index <= 10; index++) {
      // Create task, pass in function to execute and its parameters
      const task = new taskpool.Task(func1, index)
      // Add to task group
      taskGroup.addTask(task)
    }
    // Execute task group
    const res = await taskpool.execute(taskGroup)
    console.log("res", JSON.stringify(res))
  }

  build() {
    Column() {
      Button("TaskPool Basic Usage Execute Task Group")
        .onClick(this.fn1)
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

## TaskPool Communication with Main Thread

TaskPool can directly pass data back through return results. But if it's a continuous monitoring task, then you can use `sendData` and `onReceiveData` to communicate with the main thread.

![image-20241110131624375](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110131624375.png)

```typescript
import { taskpool } from '@kit.ArkTS'

// Task specific logic
@Concurrent
function func1(n: number) {
  // Assume this is continuously sent data
  taskpool.Task.sendData((new Date).toLocaleString())
  return 100
}

@Entry
@Component
struct Index {
  fn1 = async () => {
    const task = new taskpool.Task(func1, 100)

    // Data monitored by main thread
    task.onReceiveData((result: string) => {
      console.log("Data monitored by main thread", result)
    })

    // Directly get func1 result
    const res = await taskpool.execute(task)
    console.log("Directly returned result", res)
  }

  build() {
    Column() {
      Button("TaskPool Basic Usage Communication with Main Thread")
        .onClick(this.fn1)
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

## TaskPool Usage Restrictions

During TaskPool usage, you cannot directly use variables, functions, or classes in the same file as TaskPool, otherwise it will report an error.

![image-20241110132305457](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110132305457.png)

```
Only imported variables and local variables can be used in @Concurrent decorated functions. <ArkTSCheck>
```

### Solution: Export and Use

![image-20241110132421889](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110132421889.png)

# Worker Usage

Worker's main function is to provide a multithreaded runtime environment for applications, meeting the needs of applications to separate from the main thread during execution, running scripts in background threads for time-consuming operations, greatly avoiding tasks like compute-intensive or high-latency tasks from blocking main thread execution.

Worker mainly works through:

1. onmessage - listen for data
2. postMessage - send data

![image-20241110135833698](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110135833698.png)

## 1. Create Worker File

Right-click on your module and create a new worker module.

![image-20241110135437606](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110135437606.png)

> entry/src/main/ets/workers/Worker.ets

```typescript
import {
  ErrorEvent,
  MessageEvents,
  ThreadWorkerGlobalScope,
  worker,
} from "@kit.ArkTS";

const workerPort: ThreadWorkerGlobalScope = worker.workerPort;

workerPort.onmessage = (e: MessageEvents) => {
  // Receive data
  const data = e.data as Record<string, number>;
  // Send to main thread
  console.log(
    "Sub-thread monitored information and sent information to main thread"
  );
  setTimeout(() => {
    workerPort.postMessage(data.a + data.b);
  }, 1000);
};

workerPort.onmessageerror = (e: MessageEvents) => {};

workerPort.onerror = (e: ErrorEvent) => {};
```

## 2. Main Thread Using Worker for Communication

```typescript
import { MessageEvents, worker } from '@kit.ArkTS';

// 1. Declare worker
let workerStage1: worker.ThreadWorker | null

@Entry
@Component
struct Index {
  build() {
    Column() {
      Button("Create")
        .onClick(() => {
          // 2. Create worker
          workerStage1 = new worker.ThreadWorker('entry/ets/workers/Worker.ets');
        })
      Button("Listen for Data")
        .onClick(() => {
          // 3. Listen for information received by worker
          workerStage1!.onmessage = (e: MessageEvents) => {
            console.log("Sum", e.data)
          }
        })
      Button("Send Data")
        .onClick(() => {
          // 4. Send information to worker
          workerStage1!.postMessage({ a: 10, b: 20 })
        })
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

# Summary

![image-20241110140840938](HarmonyOS%20Next%20%E5%B9%B6%E5%8F%91%20taskpool%20%E5%92%8C%20worker.assets/image-20241110140840938.png)
