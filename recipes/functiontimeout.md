# Timeout a Function

## Problem
You want to limit the time you wait for a function to finish.

## Solution
Wrap the function in a goroutine and use select with a timer to wait for the response.

```Go
r := make(chan int, 1)
go func() { r <- longRunningFunction() }()
select {
case response := <-r:
    // The function completed in time
case <-time.After(5 * time.Second):
    // The function took too long
}
```

## Discussion

This recipe consists of two parts. First the long runnning function is wrapped in a goroutine so it can be executed concurrently with the timer. A channel of the same type as the return type of the function is used to capture the function response. When the function completes the return value is sent to the channel and the goroutine terminates. Note that the channel must have a capacity of at least 1 so that in the case a timeout occurs the function does not become blocked indefinitely on the channel send.

The select block waits for either a value to become available on the channel or for the timer to tick. If the function takes too long then the timer will trigger its clause in the select block and the program will continue from that point. 

## See Also

[Go Wiki: Timeouts](http://code.google.com/p/go-wiki/wiki/Timeouts)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

