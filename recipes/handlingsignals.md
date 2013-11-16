# Handling Signals

## Problem
You want your program to respond to operating system interrupt signals

## Solution
Use the Notify function from the os/signal package.

```Go
go func() {
	ch := make(chan os.Signal, 1)
	signal.Notify(ch, syscall.SIGHUP, syscall.SIGTERM, syscall.SIGUSR2)

	for {
		sig := <-ch

		switch sig {
		case syscall.SIGHUP:
			// ... handle ...
		case syscall.SIGTERM:
			// ... handle ...
		case syscall.SIGUSR2:
			// ... handle ...
		}
	}
}()
```
## Discussion
The Notify() function takes a channel of type os.Signal and a list of signals to be handled. When a signal of the appropriate type is received it is passed to the channel. If no signals are specified then the handler will catch any signal sent by the operating environment.

In this recipe the signal handling code is executed concurrently in a goroutine. The argument-free `for` statement sets up an infinite loop waiting for and processing signals. In the loop the goroutines blocks waiting for a signal to be received on the channel. When a signal arrives the goroutine continues via the switch statement to deal with the specific type of signal received.

Note that it is important to use a buffered channel to catch the signal. The channel should have a large enough capacity to deal with the expected rate of signals otherwise it may be possible to miss a signal.

## See Also
[signal.Notify](http://golang.org/pkg/os/signal/#Notify)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

