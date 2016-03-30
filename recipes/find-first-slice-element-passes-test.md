# Find first slice element that passes a test

## Problem

You need to find the first item in a slice that matches some specific criteria.

## Solution

Create a function that tests each item in a slice against a supplied match function.

```Go
import "fmt"

func FindFirstInSlice(values []string, matcher func(string) bool) (string, bool) {
	for _, s := range values {
		if matcher(s) {
			return s, true
		}
	}
	return "", false
}

func main() {
	values := []string{"orange", "pear", "apricot", "mango"}

	matcher := func(v string) bool { return len(v) > 6 }
	s, found := FindFirstInSlice(values, matcher)
	fmt.Println(s, found)

}
```

## Discussion

Unlike many other languages Go doesn't supply a standard way to find an item in a slice. It's quite trivial to loop through the slice's items and test each one though. This recipe extends that idea a little and shows a function that allows you to evaluate any matching criteria against a slice of strings.

The function FindFirstInSlice takes two arguments: the slice to be searched and a function to use to determine a match. It returns two values: the string that was found, if any plus a boolean that is true when there was a match and false otherwise. Using multiple return values to indicate "no match" is more idiomatic Go than returning a special string value.

The second argument is of type `func(string) bool` which allows any function that matches that signature to be passed, even methods on another type. In the `main()` function we simply pass in a function literal that returns true when the length of the string is greater than 6, but we could create a type with more complex behaviour and pass in one of its methods just as easily.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

