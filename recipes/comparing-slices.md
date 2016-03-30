# Test If Two Slices Are Equal

## Problem

You have two slices that you want to test for equality but Go won't let you compare them directly with the `==` operator.

## Solution

Use a built-in or custom type-specific comparison function. If you have slices of byte then you can use the `Equal` function from the `bytes` package in the Go standard library:

```Go
import "bytes"
import "fmt"

func main() {
	x := []byte{1, 2, 4}
	y := []byte{1, 2, 4}
	z := []byte{2, 4}

	fmt.Println("x == y?", bytes.Equal(x, y))
	fmt.Println("x == z?", bytes.Equal(x, z))
}
```

Otherwise you can write a function to compare two slices of the same type:

```Go
import "fmt"

func EqualSlices(x, y []int) bool {
	if len(x) != len(y) {
		return false
	}

	for i := range x {
		if x[i] != y[i] {
			return false
		}
	}
	return true
}

func main() {
	x := []int{10, 9, 12}
	y := []int{10, 9, 12}
	z := []int{10, 9}

	fmt.Println("x == y?", EqualSlices(x, y))
	fmt.Println("x == z?", EqualSlices(x, z))
}
```

If you don't know the types of the slices you are working with, you can use the `DeepEqual` function from the `reflect` package in the Go standard library:

```Go
import (
	"fmt"
	"reflect"
)


func main() {
	x := []int{10, 9, 12}
	y := []int{10, 9, 12}
	z := []int{10, 9}

	fmt.Println("x == y?", reflect.DeepEqual(x, y))
	fmt.Println("x == z?", reflect.DeepEqual(x, z))
}
```

## Discussion

Go only allows you to use the `==` and `=!` operators on values that are defined to be comparable in the specification. This covers boolean, integer, floating point and complex values as well as pointers, channels, structs and arrays. Even strings, which are essentially wrappers around byte slices, are comparable. However slices, maps and function pointers cannot be compared to anything except `nil`.

Go provides the `Equal` function for byte slices since it's a very common operation in other areas of the standard library. However as the second part of the solution shows it's only a few lines of code to write a slice equality function.

The `EqualSlices` function above shows a couple of techniques to speed up execution. First the lengths of the slices are compared and the function returns immediately if they are different since they can't possibly be equal. This is extremely fast since the length of a slice is stored as an `int` in the slice's header.

Go encourages the idiom of returning early from functions as a way to clarify logic and avoid deeply nested if statements. Once we get past the length check we know that the slices are the same length so we can simply loop over one slice and check the corresponding value in the second slice, again returning early when we find a value that is different.

The final alternative, using `DeepEqual`, is the most generic and will work with any kind of slice. However that generality comes at a price because it's also very much slower than the simple `EqualSlices` function for the same type of data.

Benchmarking both functions comparing slices of 4000 integers that differ only in the last element show that `DeepEqual` is almost 200 times slower than `EqualSlices` taking nearly a millisecond to execute compared to the 5 microseconds taken by `EqualSlices`. The main reason is that `DeepEqual` needs to allocate memory to convert the input slices and their contents to generic `interface{}` so it can recursively call `DeepEqual` on each pair of elements.

Function      Time per Operation    Bytes per Operation    Allocations per Operation
--------    --------------------  ---------------------  ---------------------------
EqualSlices           5085 ns/op                 0 B/op                  0 allocs/op
DeepEqual           976713 ns/op            128064 B/op               8002 allocs/op


Slices are not marked as comparable because they are references to an underlying array. This array may be shared with other slices which means that the values a slice contains can be modified via operations on another slice. This doesn't play well with one of the big uses of comparable types: map keys. When an element is added to a map a copy of the key is stored in the map itself. If this copy were a reference type like a slice then changing the underlying data of the slice would not affect the map key copy. This would lead to bizarre results such as storing data under a key and never being able to retrieve it again with the same key because the underling array was modified elsewhere in the program. Rather than cause confusion the Go language designers decided to not allow comparison of slices at all.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


