# Returning the source of an error

## Problem

You want errors returned by your code to include information about the source of the error.

## Solution

Create a type that combines the original error and information about the error source. Implement the error interface on the new type so it can be used as an error itself.

```Go

type WrappedError struct {
	Source string
	Err error
}

func (w WrappedError) Error() string {
	return w.Err.Error()
}
```

## Discussion

Often it is useful to know more context about the source or cause of an error. Errors are often propogated upwards towards the start of the program and it's often hard to diagnose where errors originated.

Since error is an interface it's quite simple to create custom error types that provide more information about errors you return. By implementing Go's built-in error interface, your custom errors can be handled by any existing Go code while allowing context-aware code to access the extra information you are providing.

In this recipe we create a new type called `WrappedError` that consists of two fields: Source and Err. To make it usable as an error we need to implement Go's built-in error interface which is defined as:

```Go
type error interface {
    Error() string
}
```

In our implementation we simply delegate to the wrapped error's Error method:

```Go
func (w WrappedError) Error() string {
	return w.Err.Error()
}
```

Whenever we encounter an error we'll return an instance of our `WrappedError` type instead of the raw error like this:

```Go
func DoSomething() error {

	// try to open a file
	file, err := os.Open("myfile.txt")
	if err != nil {
		// error occurred, so return it with some context
		return WrappedError{
			Source: "os.Open",
			Err: err,
		}
	}

	// continue with function
}
```

Now, when we call the `DoSomething` function we can test the type of error returned and show the source of the error as well as the error itself:

```Go
import "fmt"

func main() {
	err := DoSomething()

	if err != nil {
		switch werr := err.(type) {
		case WrappedError:
			fmt.Printf("got error %s from %s", werr.Error(), werr.Source)
		default:
			fmt.Printf("got error %s", err.Error())
		}
	}
}
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)



