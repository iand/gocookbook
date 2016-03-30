# Use an array with functions that expect a slice

## Problem

You have an array of values and you want to use it in functions that need a slice as an argument.

## Solution

Use the `[:]` operator on the array to initialise a new slice backed by the array and then pass the slice to the function. For example to sort an array you could use the following:

```Go
import (
	"fmt"
	"sort"
)

func SortArray() {
	x := [6]int{3, 1, 4, 1, 5, 9}

	sl := x[:]
	sort.Ints(sl)
	fmt.Println(x)
}
```

## Discussion

Most functions in Go's standard library operate on slices not arrays. However since a slice is simply a reference to an underlying array anything that operates on the slice will also operate on the underlying array's data.

The `[:]` syntax may look slightly odd but it's really just the standard specification for a slice. The expression x[start:end] creates a slice that includes the elements of x beginning at index `start` up to (but not including) index `end`.

```Go
x := [6]int{3, 1, 4, 1, 5, 9}

sl := x[2:5]
// sl is 4, 1, 5
```

Both `start` and `end` are optional. Not supplying start means the slice will begin at index 0:

```Go
x := [6]int{3, 1, 4, 1, 5, 9}

sl := x[:5]
// sl is 3, 1, 4, 1, 5
```

Missing out `end` means the slice will include all the array's elements from the start to the last element:

```Go
x := [6]int{3, 1, 4, 1, 5, 9}

sl := x[2:]
// sl is 4, 1, 5, 9
```

Missing out both `start` and `end` uses defaults for both so the slice will contain all the elements from index 0 to the last in the array:

```Go
x := [6]int{3, 1, 4, 1, 5, 9}

sl := x[:]
// sl is 3, 1, 4, 1, 5, 9
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

