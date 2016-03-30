# Return multiple errors

## Problem

You have a function that returns an error and you want to change it so it can return multiple errors at once without breaking any existing users of the function.

## Solution

Implement the error interface on a slice of errors.

Suppose you start with a simple Validate function that validates a map, returning an error as soon as it finds one:

```Go
	import "fmt"

	// Validate is a simple function to validate a map
	func Validate(m map[string]string) error {
		if _, exists m["field1"]; !exists {
			return fmt.Errorf("field1 must be supplied")
		}

		if _, exists m["field2"]; !exists {
			return fmt.Errorf("field2 must be supplied")
		}

		// No errors
		return nil
	}
```

To make this more efficient for callers you want to return all the errors in one go without breaking compatibility. You implement the error interface on a slice of errors and return that instead:

```Go
	import "fmt"

	type MultiError []error

	func (m MultiError) Error() string {
		switch len(m) {
		case 0:
			return ""
		case 1:
			return m[0].Error()
		case 2:
			return m[0].Error() + " and 1 other"
		default:
			return fmt.Sprintf("%s and %d others", m[0].Error(), len(m)-1)
		}
	}

	// Validate is a simple function to validate a map
	func Validate(m map[string]string) error {
		me := make(MultiError)

		if _, exists m["field1"]; !exists {
			me = append(me, fmt.Errorf("field1 must be supplied"))
		}

		if _, exists m["field2"]; !exists {
			me = append(me, fmt.Errorf("field2 must be supplied"))
		}

		// Check if any errors occurred and return them.
		if len(me) > 0 {
			return me
		}

		// No errors
		return nil
	}
```


## Discussion

A typical scenario for returning multiple errors is web form validation. In this situation, rather than force the user to fix an error and resubmit the form you'd prefer to return all the validation errors to give the user a chance of fixing them all at once.

In this recipe we take advantage of Go's ability to implement an interface using any type, in this case a slice. All we need to do to attach methods to a slice is create a named type. In our case we create one called MultiError as a slice of errors:

```Go
type MultiError []error
```

Now we have our slice type we can define methods for it, specifically the one that turns it into something other Go code will recognise as an error. The error interface is a built-in type and is defined as:

```Go
type error interface {
    Error() string
}
```

This means that any type that has a method called Error with no arguments and returns a string can be used as an error.

Here's our implementation of the Error interface:

```Go
	func (m MultiError) Error() string {
		switch len(m) {
		case 0:
			return ""
		case 1:
			return m[0].Error()
		case 2:
			return m[0].Error() + " and 1 other"
		default:
			return fmt.Sprintf("%s and %d others", m[0].Error(), len(m)-1)
		}
	}
```

We could have written a simpler error message that just returns the number of errors, but to make it nicer to use we return a different message depending on the size of the `MultiError` slice, including the text of the first error if we can.

Note that the receiver of our Error method is a value `m MultiError` and not the more commonly seen pointer. In this case we don't want a pointer because we're not changing `m`, just reading from it. In fact even if we were changing values in the slice we'd still use a value as the receiver since slices are already references. The only time we'd need a pointer to a slice in a receiver is if we were changing what the slice refers to, such as when we append to it.

Next we modify our original `Validate` function to use `MultiError`.

```Go
	// Validate is a simple function to validate a map
	func Validate(m map[string]string) error {
		me := make(MultiError)

		if _, exists m["field1"]; !exists {
			me = append(me, fmt.Errorf("field1 must be supplied"))
		}

		if _, exists m["field2"]; !exists {
			me = append(me, fmt.Errorf("field2 must be supplied"))
		}

		// Check if any errors occurred and return them.
		if len(me) > 0 {
			return me
		}

		// No errors
		return nil
	}
```

Whenever we have a validation failure, instead of returning the error immediately we add it to the `me` variable. We can use the `append` built-in function with our custom type just as easily as with a standard slice.

Just before we exit the `Validate` function we check the length of `me` and return it if it contains any errors. Otherwise we carry on and return nil, indicating that no error occurred.

Note that we didn't change our function signature: it still returns an `error` and any client code using the function will continue to work as before. However new code could take advantage of the fact `Validate` returns multiple errors and display them all to the user at once. To do that, they would need to perform a type assertion on the error returned from `Validate`. For example this code would work with both the old, single error, version and the new, multiple error, version of `Validate`:

```Go

// CheckInput checks the input map and displays any validation errors:
func CheckInput(m map[string]string) {

	err := Validate(m)
	if err == nil {
		// No validation errors
		return
	}

	// Check if we got a MultiError and if so, loop through all the
	// returned errors, printing them out
	if merr, ok := err.(MultiError); ok {
		for _, err := range merr {
			println(merr.Error())
		}
		return
	}

	// Otherwise we simply print the error that was returned.
	println(err.Error())
}


As an interesting aside, because we defined MultiError as a slice of errors we can use it to hold any type of error including other MultiErrors. That lets us return lists of lists of errors as easily as returning one. This could get complicated fast so it's best to use it sparingly.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

