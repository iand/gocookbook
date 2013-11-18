# Easy timing of a function

## Problem
You want a quick way to time a function call in Go with one line

## Solution
Create the following two helper functions in your Go app:

```Go
func trace(s string) (string, time.Time) {
    log.Println("START:", s)
    return s, time.Now()
}

func un(s string, startTime time.Time) {
    endTime := time.Now()
    log.Println("  END:", s, "ElapsedTime in seconds:", endTime.Sub(startTime))
}
```

Then, indentify a function you want to log the timing for and place this one liner at the top of the function:

```Go
func someFunctionToTime() {
    defer un(trace("SOME_ARBITRARY_STRING_SO_YOU_CAN_KEEP_TRACK"))

    //Your regular code goes here...
}
```

## Discussion

These handy functions do their magic by using Go's defer keyword.  Remember that in Go, when a function is called with the defer statement, Go will make sure to execute the function upon return of the surrounding function.  Also remember that when you defer a function, its arguments are evaluated not when it is called but rather when it is add to the defer list.

## See Also

[Go Language Specification: Defer, Panic and Recover](http://blog.golang.org/defer-panic-and-recover)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

