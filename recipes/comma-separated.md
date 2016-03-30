# Print a comma separated slice of values

## Problem

You want to output a list of values nicely formatted with commas.

## Solution

If you have a slice of strings then use the `Join` function from the `strings` package:

```Go
package main

import "strings"

func main() {
    values := []string{"apple", "orange", "pear"}
    println(strings.Join(values, ", "))
}
```

However if you have a slice of arbitrary things then convert each element to a string and write it to a byte buffer:

```Go
package main

import (
    "bytes"
    "fmt"
)

func main() {
    values := []interface{}{"apple", 89.1, complex(2, -2), true}

    buf := bytes.Buffer{}
    buf.WriteString(fmt.Sprint(values[0]))
    for i := 1; i < len(values); i++ {
        buf.WriteString(", ")
        buf.WriteString(fmt.Sprint(values[i]))
    }

    println(buf.String())
}
```

## Discussion

Go's `string` package is very versatile and provides a number of useful convenience functions that are common in text processing. The `Join` function is one that is often found in other languages as a method on a string type. The separator argument is not restricted to single characters and can be any string. The function operates very efficiently by measuring the length of all the strings involved and pre-allocating a string of the correct length before copying in all the slice items and separators.

However, Go doesn't support automatic conversion of slice types so if you have a slice of a different type you need to convert each value to a string yourself. You could create a new string slice and then convert each member of your slice to a string and add it to the new string slice. Then you could pass that to the `Join` function. This would work well but could be inefficient for large slices because you'd need to allocate memory to store the intermediate string slice, plus the memory used by `Join` when it makes room for the concatenated version. It would also create more objects that would need to be garbage collected.

A slightly more efficient way of joining these values is to use a `Buffer` from the `bytes` package. This is simply a wrapper around a byte slice with some convenience methods for writing to the slice and growing its capacity automatically.

In the second part of the recipe we have a slice of `interface{}` which is Go's catch-all type, capable of holding any kind of value. We create an empty `bytes.Buffer` and write the first value in the slice to the buffer. To convert the value to a string we use the `Sprint` function from the `fmt` package. This is capable of converting just about any Go type to a decent string version. If we knew the type of the items in the slice ahead of time it would be more efficient to use a specific converter such as `strconv.Atoi`.

Once we have written the first value to the buffer, we loop through the remaining values, writing a comma and a space before each one. The net result is the buffer contains all of our slice's items delimited by commas. To access the string version of the buffer we then use its `String` method which creates a new string pointing at the bytes in the buffer.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

