# English Translation Required

This file is marked for translation from: HarmonyNext 如何优雅的编写注释.md

Original Chinese file path: 鸿蒙开发技巧\HarmonyOS Next 如何优雅的编写注释\HarmonyNext 如何优雅的编写注释.md

Please translate the content from the original Chinese file to English.
The translation should maintain:

- Technical accuracy
- Code examples (translate comments but keep code structure)
- Image references
- Link references
- Formatting (headers, lists, etc.)

---

[Automatic placeholder - Replace with actual translation]

# HarmonyOS Next: How to Write Comments Elegantly

# Programmer's Axiom

**I hate two kinds of people in the world most:**

1. **The first kind: people who don't write comments**
2. **The second kind: people who make me write comments**

# Preface

With the accelerated development of HarmonyOS NEXT, many companies have gradually increased their resources to develop software projects. As projects develop, project teams also need to write project comments or code documentation according to certain standards.

I believe that the minimum cost for writing project comments or code documentation is to implement code usage documentation directly through **writing comments**.

Currently, mainstream IDEs support **jsDoc** or **TypeDoc**. By writing code comments according to specified formats, we can get the following benefits:

![image-20240929004710826](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929004710826.png)

When we want to call the global function **px2vp**, the prompt tool will clearly show us the relevant usage instructions. Additionally, if you're writing a utility library, you can also generate beautiful documentation based on related tools.

> Person.ets

```typescript
/**
 * A utility person class
 *
 * @since 11
 */
export class Person {
  /**
   * Age
   */
  age: number = 18;

  /**
   *
   * Calculate the sum of two ages
   * @param {number} n1 Age 1
   * @param {number} n2 Age 2
   * @returns {number} Total age
   */
  calcAge(n1: number, n2: number) {
    return n1 + n2;
  }
}
```

![image-20240929010044597](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929010044597.png)

# Built-in Syntax Hints in DevEco Studio

jsDoc provides modifiers for constants, classes, functions, interfaces, enums, etc. Generally, you don't need to add them manually because **DevEco Studio** can automatically recognize them.

> @constant @class @function @interface @enum etc.

**Class**

![image-20240929011503966](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929011503966.png)

**Enum**

![image-20240929011553049](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929011553049.png)

Moreover, when you import code hints, you can also intuitively observe here to determine what type it is.

![image-20240929011750736](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929011750736.png)

---

![image-20240929012128049](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929012128049.png)

# Common Code Hint Modifiers

![image-20240929012352082](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929012352082.png)

As shown in the image, if we want to implement some syntax hint functionality like **the right side of the image above**, then we need to write appropriate code hint modifiers ourselves.

Let's demonstrate by writing a class. First, we provide the following basic structure:

```typescript
export class Person {
  age: number = 18;

  protected static async calcAge4(n1: number, n2: number) {
    return n1 + n2;
  }

  calcAge1(n1: number, n2: number) {
    return n1 + n2;
  }

  async calcAge2(n1: number, n2: number) {
    return n1 + n2;
  }

  protected async calcAge3(n1: number, n2: number) {
    return n1 + n2;
  }
}
```

## Quick Generation of Specific Comments

If we want to add some remarks to Person, we cannot use the following types of comments:

```typescript
// This single-line comment won't work

/* This regular multi-line comment won't work either */
```

We can only use this type:

```
/**
*  This is OK
*/
```

You can quickly generate by typing `/** + tab` above the code you want to modify.

![PixPin_2024-09-29_01-31-34](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/PixPin_2024-09-29_01-31-34.gif)

Above functions with parameters, it will automatically add parameter modifiers, including return values.

![PixPin_2024-09-29_01-33-28](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/PixPin_2024-09-29_01-33-28.gif)

## @param and @returns

> @param modifies function parameters
>
> @returns modifies return values

![image-20240929013703645](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929013703645.png)

## @async

> @async modifies asynchronous functions

![image-20240929013924434](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929013924434.png)

## @public

> @public public
>
> @protected protected
>
> @private private

![image-20240929014127121](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929014127121.png)

## @static

![image-20240929014309520](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929014309520.png)

# Overview of Other jsDoc Standard Modifiers

| Modifier         | Meaning                                                            |
| ---------------- | ------------------------------------------------------------------ |
| @abstract        | Indicates an abstract member that cannot be directly instantiated. |
| @access          | Used to specify the access level of a member.                      |
| @alias           | Defines an alias.                                                  |
| @async           | Indicates an asynchronous function.                                |
| @augments        | Indicates that a class inherits from another class.                |
| @author          | Author information.                                                |
| @borrows         | Indicates a function or property borrowed from another module.     |
| @callback        | Indicates a callback function.                                     |
| @class           | Used to define a class.                                            |
| @classdesc       | Class description.                                                 |
| @constant        | Indicates a constant.                                              |
| @constructs      | Indicates that a function is a constructor.                        |
| @copyright       | Copyright information.                                             |
| @default         | Default value.                                                     |
| @deprecated      | Indicates a deprecated member.                                     |
| @description     | Description information.                                           |
| @enum            | Defines an enumeration.                                            |
| @event           | Indicates an event.                                                |
| @example         | Example code.                                                      |
| @exports         | Used to specify members to be exported.                            |
| @external        | Indicates members of external modules.                             |
| @file            | File information.                                                  |
| @fires           | Indicates triggered events.                                        |
| @function        | Defines a function.                                                |
| @generator       | Indicates a generator function.                                    |
| @global          | Indicates global members.                                          |
| @hideconstructor | Hide constructor.                                                  |
| @ignore          | Indicates ignored parts.                                           |
| @implements      | Indicates implemented interfaces.                                  |
| @inheritdoc      | Inherit documentation.                                             |
| @inner           | Inner members.                                                     |
| @instance        | Instance members.                                                  |
| @interface       | Defines an interface.                                              |
| @kind            | Type kind.                                                         |
| @lends           | Lends properties to another object.                                |
| @license         | License information.                                               |
| @listens         | Indicates listened events.                                         |
| @member          | Member.                                                            |
| @memberof        | Members belonging to an object.                                    |
| @mixes           | Mixes characteristics of multiple classes.                         |
| @mixin           | Defines a mixin.                                                   |
| @module          | Defines a module.                                                  |
| @name            | Name.                                                              |
| @namespace       | Namespace.                                                         |
| @override        | Indicates overridden members.                                      |
| @package         | Package information.                                               |
| @param           | Function parameter description.                                    |
| @private         | Private members.                                                   |
| @property        | Property.                                                          |
| @protected       | Protected members.                                                 |
| @public          | Public members.                                                    |
| @readonly        | Read-only property.                                                |
| @requires        | Indicates dependent modules.                                       |
| @returns         | Function return value description.                                 |
| @see             | Reference information.                                             |
| @since           | Starting from a certain version.                                   |
| @static          | Static members.                                                    |
| @summary         | Summary information.                                               |
| @this            | Current object.                                                    |
| @throws          | Description of thrown exceptions.                                  |
| @todo            | To-do items.                                                       |
| @tutorial        | Tutorial information.                                              |
| @type            | Type description.                                                  |
| @typedef         | Type definition.                                                   |
| @variation       | Variation conditions.                                              |
| @version         | Version information.                                               |
| @yields          | Description of generated values.                                   |

# DevEco Studio Supports Custom Modifiers

**DevEco Studio** supports custom modifiers, for example:

![image-20240929010933164](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929010933164.png)

Although you can set them arbitrarily, it's still recommended to follow certain standards for consistent team development, such as following some of the jsDoc standards mentioned above.

# DevEco Studio Quick Documentation Generation

**DevEco Studio NEXT Beta1(5.0.3.814)**

- Currently supports generating ArkTSDoc documentation for .ets/.ts/.js/.md format files in projects or directories.
- Exported variables, methods, interfaces, classes, etc. in files will generate corresponding ArkTSDoc documentation. Objects that are not exported do not support generation.
- If you choose to export ArkTSDoc documentation for the entire project/directory, the generated ArkTSDoc documentation directory structure will be consistent with the original directory structure.

![image-20240929014948209](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929014948209.png)

---

![image-20240929015033130](HarmonyNext%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E7%BC%96%E5%86%99%E6%B3%A8%E9%87%8A.assets/image-20240929015033130.png)

# Reference Links

1. [@use JSDoc](https://jsdoc.app/)
2. [Generate ArkTSDoc Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-arktsdoc-V5)
