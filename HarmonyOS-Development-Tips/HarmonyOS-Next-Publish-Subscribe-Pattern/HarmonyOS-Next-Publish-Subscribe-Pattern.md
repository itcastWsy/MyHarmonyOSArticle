# English Translation Required

This file is marked for translation from: HarmonyOS Next 发布-订阅模式.md

Original Chinese file path: 鸿蒙开发技巧\HarmonyOS Next 发布-订阅模式\HarmonyOS Next 发布-订阅模式.md

Please translate the content from the original Chinese file to English.
The translation should maintain:

- Technical accuracy
- Code examples (translate comments but keep code structure)
- Image references
- Link references
- Formatting (headers, lists, etc.)

---

# HarmonyOS Next Overview of the Publisher-Subscriber Pattern

# Preface

In current HarmonyOS application development, or in the era of front-end development with frameworks like Vue, React, mini-programs, etc., ordinary users increasingly encounter fewer scenarios where mastering design patterns is mandatory. In plain language, this means that some frameworks are so well-encapsulated that users can simply work within their ecosystems without needing to understand design patterns, and this won't hinder normal business development. However, when it comes to encapsulating tools or game development, the importance of design patterns becomes apparent. This is because when doing encapsulation work, if you don't use some design patterns, the encapsulated code is basically unusable. Fellow developers who share this experience are welcome to speak up. 😄

![image-20241120214237877](HarmonyOS%20Next%20%E5%8F%91%E5%B8%83-%E8%AE%A2%E9%98%85%E6%A8%A1%E5%BC%8F.assets/image-20241120214237877.png)

# Objective

In ArkTS, there exists an Emitter object that has the capability to continuously subscribe to events, subscribe to one-time events, cancel event subscriptions, and trigger events. We can use it as a reference for encapsulation to implement a similar wrapper ourselves.

The use of Emitter is a typical implementation of the publisher-subscriber design pattern, which can also be understood as the producer-consumer design pattern.

1. Subscription can be understood as subscribing to newspapers and magazines from the post office
2. Publishing can be understood as when newspapers and magazines are published, we naturally receive the corresponding new publications
3. For subscribers:
   1. We can subscribe to publications for unlimited duration (continuous subscription)
   2. We can subscribe to publications only once (one-time subscription)
   3. We can cancel subscriptions to publications
4. For publishers:
   1. They are responsible for publishing only

# Interface Design

| Method | Description             |
| ------ | ----------------------- |
| on     | Continuous subscription |
| once   | One-time subscription   |
| off    | Cancel subscription     |
| emit   | Publish                 |

# Implementation Details

## Defining Types

1. **eventType** defines a union type for event types, which can be **normal** or **once**
2. **IEventItem** defines an interface for event items, including properties such as event ID, type, callback function array, and specific event type

```typescript
// Define a union type for event types, which can be "normal" or "once"
type eventType = "normal" | "once";

// Define an interface for event items, including event ID, type, callback function array, and specific event type properties
interface IEventItem {
  eventId: number;
  type: string;
  cbs: Function[];
  eventType: eventType;
}
```

## Defining the Basic Class Structure

1. **MyEmitter** is the name of the custom class that wraps the Emitter
2. **listeners** is a private static array that stores all event listeners, initially empty
3. **\_eventId** is a private static variable used to generate unique event IDs, with an initial value of 0
4. **\_on** is a private static method used to add event listeners, accepting event type, event name, and callback function as parameters
5. **on** is a static method used to add normal type event listeners
6. **once** is a static method used to add event listeners that trigger only once
7. **emit** is a static method used to trigger events of a specified type, which traverses and executes all callback functions corresponding to that event type
8. **off** is a static method used to remove event listeners with a specified event ID, accepting event ID as a required parameter and optionally a callback function. If only the event ID is passed, the entire event item corresponding to that ID will be removed; if both event ID and callback function are passed, only the corresponding callback function in that event item will be removed

```typescript
class MyEmitter {
  // Private static array storing all event listeners, initially empty
  private static listeners: IEventItem[] = [];
  // Private static variable used to generate unique event IDs, initial value is 0
  private static _eventId: number = 0;

  // Getter method for private static property, returns incremented _eventId value each time called
  // Used to get the next available event ID
  private static get eventId() {
  }

  // Private static method used to add event listeners
  // Accepts event type, event name, and callback function as parameters
  private static _on(eventType: eventType, type: string, cb: Function) {

  }

  // Static method used to add normal type event listeners
  // Accepts event name and callback function as parameters
  // Internally calls private static method _on and passes "normal" event type
  static on(type: string, cb: Function) {

  }

  // Static method used to add event listeners that trigger only once
  // Accepts event name and callback function as parameters
  // Internally calls private static method _on and passes "once" event type
  static once(type: string, cb: Function) {

  }

  // Static method used to trigger events of a specified type
  // Accepts event name as required parameter, optionally accepts a data parameter
  // Traverses and executes all callback functions corresponding to that event type
  static emit<T = undefined>(type: string, data?: T) {

  }

  // Static method used to remove event listeners with specified event ID
  // Accepts event ID as required parameter, optionally accepts a callback function as parameter
  // If only event ID is passed, removes the entire event item corresponding to that ID; if both event ID and callback function are passed, removes only the corresponding callback function in that event item
  static off(eventId: number, cb?: Function) {

}

```

## Usage Example

```typescript
@Entry
@Component
struct Index {
  tid: number = -1

  build() {
    Column({ space: 10 }) {
      Button("1 Register normal event")
        .onClick(() => {
          this.tid = MyEmitter.on("login", (res: object) => {
            console.log(JSON.stringify(res))
          })
        })
      Button("1 Cancel normal event")
        .onClick(() => {
          MyEmitter.off(this.tid)
        })
      Button("1 Trigger normal event")
        .onClick(() => {
          MyEmitter.emit("login", 100)
        })
      Button("2 Register one-time event")
        .onClick(() => {
          this.tid = MyEmitter.once("login2", (res: object) => {
            console.log(JSON.stringify(res))
          })
        })
      Button("2 Cancel one-time event")
        .onClick(() => {
          MyEmitter.off(this.tid)
        })
      Button("2 Trigger one-time event")
        .onClick(() => {
          MyEmitter.emit("login2", 100)
        })
      Button("3 Register named event")
        .onClick(() => {
          this.tid = MyEmitter.on("login1", this.fn1)
        })
      Button("3 Cancel named event")
        .onClick(() => {
          MyEmitter.off(this.tid, this.fn1)
        })
      Button("3 Trigger named event")
        .onClick(() => {
          MyEmitter.emit("login1", 300)
        })

    }
    .height('100%')
    .width('100%')
  }

  fn1(n: number) {
    console.log("Named event", n)
  }
}
```

# Effect Screenshot

![image-20241120220915524](HarmonyOS%20Next%20%E5%8F%91%E5%B8%83-%E8%AE%A2%E9%98%85%E6%A8%A1%E5%BC%8F.assets/image-20241120220915524.png)

# Summary

The publisher-subscriber pattern is a very useful software design pattern that can achieve system decoupling, scalability, and flexibility. In practical applications, it's necessary to choose the appropriate implementation method based on specific requirements and scenarios.
