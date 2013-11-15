# Wait For A Group of Goroutines To Complete

## Problem
You have a number of tasks running concurrently in goroutines and you need to wait for them all to complete before proceeding.

## Solution
Use a WaitGroup to block your program until your goroutines have completed. When you start each of your goroutines, increment the WaitGroup by calling the WaitGroup's Add() method. Pass a pointer to the WaitGroup to each of your goroutines and have them call Done() on the WaitGroup when they are complete. After starting your goroutines, block the current thread of execution by calling the WaitGroup's Wait() method.

```Go
var wg sync.WaitGroup

wg.Add(1)
go firstLongRunningFunction(&wg)

wg.Add(1)
go secondLongRunningFunction(&wg)

wg.Wait()
```

Your long running functions will look like this:

```Go
func longFirstRunningFunction(wg *sync.WaitGroup) {
    defer wg.Done()

    // ... do some work ...
    
    return
}
```

## Discussion
A WaitGroup is designed to synchronise a group of goroutines. It maintains a counter that is added to by the Add() method (which can also subtract by supplying a negative number). The Done() method decrements the counter by one. When the counter falls to zero then all goroutines blocked on calls to Wait() are released. In this recipe only the main goroutine is blocking waiting for its workers to finish but the same WaitGroup could be used in multiple goroutines to synchronise their execution, each calling Wait() at the appropriate time.

In the recipe, the call to Done() is placed as a defer statement to ensure that it is called as soon as the goroutine completes.

You should call Add() before you start your goroutine otherwise there is a chance that the call to Wait() will block on too few goroutines.

Make sure you pass a pointer to the WaitGroup and not the value, otherwise a copy will be used and your call to Done() will release the wrong WaitGroup causing your program to block forever.

## See Also

[sync.WaitGroup](http://golang.org/pkg/sync/#WaitGroup)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

