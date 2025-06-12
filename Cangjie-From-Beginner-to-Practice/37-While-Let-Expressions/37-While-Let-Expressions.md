# 37-while-let Expressions

The `while-let` expression first evaluates the expression on the right side of `<-` in the condition. If this value can match the pattern on the left side of `<-`, it executes the loop body, then repeats this process. If pattern matching fails, the loop ends and continues executing the code after the `while-let` expression. For example:

```javascript
import std.random.*

// This function simulates receiving data in communication, data acquisition may fail
func recv(): Option<UInt8> {
    let number = Random().nextUInt8()
    if (number < 128) {
        return Some(number)
    }
    return None
}

main() {
    // Simulate loop receiving communication data, end the loop if it fails
    while (let Some(data) <- recv()) {
        println(data)
    }
    println("receive failed")
}
```

Running the above program, possible output could be:

```javascript
73
94
receive failed
```