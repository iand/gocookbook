# Throttle Concurrent Goroutines

## Problem
You have a large number of tasks to perform but you want to throttle the number of goroutines that are running concurrently.


## Solution
Use a channel to distribute work to a limited number of goroutines. 


```Go

// Represents an item to be worked on, use a struct to hold more information
type workitem string


// Worker performs the actual work
func worker(wg *sync.WaitGroup, work chan string) {
	defer wg.Done()
	for item := range work {
		// ... do work ...
	}
}

func processItems(items []workitem) {
	// A channel containing the work items
	work := make(chan workitem)
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go worker(&wg, in)
	}

	// Feed the workers with work
	for _, item := range items {
	    work <- item
	}

	close(work)

	// Wait for all the goroutines to complete
	wg.Wait()
}
```

## Discussion
This is a situation that often occurs in applications that crawl web pages. The developer wants a concurrent approach but doesn't want to overload the sites being crawled. The solution presented in this recipe solves this by creating a limited pool of worker goroutines that listen on a channel for work items to process. In the web crawler scenario the work item could be a URL to crawl. 

As each goroutine worker starts it blocks waiting for items to be sent on the work channel. When items are added to the work channel a random goroutine will be unblocked and will receive the item for processing. If there are more items than workers then the channel will start accumulating work items for later processing. Once all the items have been sent on the channel, the `processItems` function closes the work channel. This indicates that no more items will be sent on the channel but does not destroy the channel or lose the items it contains. The `processItems` function then uses the WaitGroup to wait for all the goroutines to complete.

When the closed channel has been emptied by the workers, the `for...range` loop in each worker will terminate and the goroutine will terminate, calling the waitgroup's Done method as it does so. Once all goroutines have completed, the `processItems` function terminates.


This recipe uses the technique discussed in the [Wait For A Group of Goroutines To Complete](waitgroup.md) recipe to synchronise the completion of the worker goroutines.

## See Also

[Recipe: Wait For A Group of Goroutines To Complete](waitgroup.md), [Go Language Specification: Close](http://golang.org/ref/spec#Close)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

