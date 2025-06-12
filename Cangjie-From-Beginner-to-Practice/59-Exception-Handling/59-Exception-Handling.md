# 59-Exception Handling

Exceptions are a special class of errors that can be caught and handled by programmers. They are a general term for a series of abnormal behaviors that occur during program execution, such as array out-of-bounds, division by zero errors, calculation overflow, illegal input, etc. To ensure system correctness and robustness, many software systems contain a large amount of code for error detection and error handling.

Exceptions do not belong to the normal functionality of a program. Once an exception occurs, the program must handle it immediately, transferring program control from the normal functionality execution to the exception handling part. The Cangjie programming language provides an exception handling mechanism to handle various exceptional situations that may occur during program runtime.

In Cangjie, there are two exception classes: `Error` and `Exception`:

- The `Error` class describes internal errors and resource exhaustion errors in the Cangjie language runtime. Applications should not throw this type of error. If internal errors occur, they can only be notified to users and the program should be terminated as safely as possible.
- The `Exception` class describes exceptions caused by logical errors or IO errors during program runtime, such as array out-of-bounds or attempting to open a non-existent file. This type of exception needs to be caught and handled in the program.

Users cannot define custom exceptions by inheriting from the built-in Error or its subclasses in the Cangjie language, but they can inherit from the built-in Exception or its subclasses to define custom exceptions, for example:

```typescript
open class FatherException <: Exception {
    public init() {
        super("This is FatherException.")
    }
    public init(message: String) {
        super(message)
    }
    public open override func getClassName(): String {
        "FatherException"
    }
}

class ChildException <: FatherException {
    public init() {
        super("This is ChildException.")
    }
    public open override func getClassName(): String {
        "ChildException"
    }
}
```

The following list shows the main functions of `Exception` and their descriptions:

| Function Type | Function and Description                                     |
| :------------ | :----------------------------------------------------------- |
| Constructor   | `init()` Default constructor.                                |
| Constructor   | `init(message: String)` Constructor that can set exception message. |
| Member Property | `open prop message: String` Returns detailed information about the exception. This message is initialized in the exception class constructor, defaulting to an empty string. |
| Member Function | `open func toString(): String` Returns the exception type name and detailed exception information, where the detailed exception information defaults to calling message. |
| Member Function | `func getClassName(): String` Returns the user-defined class name. Subclasses need to override this method to return the subclass name. |
| Member Function | `func printStackTrace(): Unit` Prints stack information to standard error stream. |

The following list shows the main functions of `Error` and their descriptions:

| Function Type | Function and Description                                     |
| :------------ | :----------------------------------------------------------- |
| Member Property | `open prop message: String` Returns detailed information about the error. This message is internally initialized when the error occurs, defaulting to an empty string. |
| Member Function | `open func toString(): String` Returns the error type name and detailed error information, where the detailed error information defaults to calling message. |
| Member Function | `func printStackTrace(): Unit` Prints stack information to standard error stream. |