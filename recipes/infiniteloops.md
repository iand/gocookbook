# Infinite Loops

## Problem
You need to loop forever.

## Solution

Use the short form of the for statement.

```Go
for {
	// wait for some work to arrive
	x := <- work
	doWork(x)
}
```

## Discussion

Infinite loops occur quite frequently in Go due to its emphasis on concurrent programming. It's usual for a goroutine to start and block in an infinite loop waiting to receive or send on its channels. This idiom is so common that Go provides very simple syntax for it, simply a for statement with no further arguments.

Like any other for loop a `break` statement will break out and the `continue` statement will skip any following instructions and continue from the start of the loop.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

