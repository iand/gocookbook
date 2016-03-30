# Log panics that occur in a program

## Problem

A long running program occasionally panics and stops running. You want it to log the error that caused the panic to a logfile so you can diagnose the problem.

## Solution

Use `defer` and `recover` to intercept panics and log them:

```Go
package main

import (
	"log"
	"runtime"
)

func main() {
	//Register a function to be called before main exits
	defer func() {
		if p := recover(); p != nil {
		    // Recovered a panic, so log it
			log.Printf("Panic: %v", p)
			buf := make([]byte, 4096)
			buf = buf[:runtime.Stack(buf, false)]
			log.Fatalf("%s", buf)
		}
	}()

	// Proceed with rest of function
	functionThatMayPanic(0)
}

func functionThatMayPanic(n int) {
	// Divide by something that might be zero
	println(46 / n)
}
```


## Discussion

Go encourages the prolific use of errors by functions to indicate failures of all kinds. These are usually returned as the sole or last return value from a function and it is customary to check the error after each function call. This flow is central to Go's philosophy of treating errors as part of the normal operation of a program.

However there are a class of errors that could arise in any situation or may be out of the programmer's control and should be signalled to the program outside of the normal error flow. Typically these are runtime errors such as dividing by zero, exceeding the bounds of a slice or invalid type assertions. In these cases the Go runtime will call `panic` which causes the program to stop executing and begin unwinding the call stack all the way to the main function which then exits the program.

In situations where a long running program is running under a supervisor that restarts it on failure, it can be troublesome to determine the cause of a panic. Logging the panic to the program's logfile can provide a persistent record of the problem for later analysis.

Go allows functions to be scheduled to execute just before a function exits, even when the stack is unwinding due to a panic. This is done using the keyword `defer` which is placed just before the call to the function that will be called on exit. For example:

```Go
func example() {
	println("entering function")
	defer println("exiting function")
	println("in the function")
}

```

When the `example` function is called it executes the first `println`, defers the second one, executes the third one and then executes the second one just before it exits the function. The output looks like this:

```
entering function
in the function
exiting function
```

In this recipe we use `defer` to defer a function that checks whether a panic occurred. This check is performed using the `recover` function. When used in a deferred function the `recover` function returns the error passed to the original call to `panic` or `nil` if there is no panic and the function is exiting normally. Importantly `recover` also halts the panic process and stops the call stack from being unwound any further.

In the recipe, this line checks whether we are currently panicking and begins our logging process if we are. If not, the deferred function simply continues and allows the outer function to exit cleanly.

```Go
if p := recover(); p != nil {
	// Recovered a panic, so log it
}
```

Once we have detected a panic we start our logging. We do this in two stages. First we log the panic message itself and then we log a portion of the stack trace that led to the panic using the function `runtime.Stack`. Stack traces in Go can be extremely long so we only log the first 4k and by passing false as the second argument to `runtime.Stack` we only log the stack from the current goroutine.

This is an example of the logged output:

```
2015/03/29 14:47:11 Panic: runtime error: integer divide by zero
2015/03/29 14:47:11 goroutine 1 [running]:
main.func·001()
	/home/iand/gorecipes/code/cmd/handlepanic.go:14 +0x144
main.main()
	/home/iand/gorecipes/code/cmd/handlepanic.go:20 +0x48
exit status 1
```

We log the stack trace using `log.Fatalf` which also exits the program with a return code of 1 which indicates to the shell that an error occurred.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


