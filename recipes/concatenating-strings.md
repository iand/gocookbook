# Efficiently concatenating strings

## Problem

You have a series of strings that you want to concatenate to build a single, longer string.

## Solution

Use the `Buffer` type from the `bytes` package to build up the string and then retrieve it with the `String` method.

```Go
package main

import (
    "bytes"
    "fmt"
)

var nominals = []string{"", "one", "two", "three", "four",
    "five", "six", "seven", "eight",
    "nine", "ten", "eleven", "twelve",
    "thirteen", "fourteen", "fifteen",
    "sixteen", "seventeen", "eighteen",
    "nineteen"}

var decades = []string{"twenty", "thirty", "forty", "fifty",
    "sixty", "seventy", "eighty", "ninety"}

func NumberToWords(n int) string {
    if n == 0 {
        return "zero"
    }

    buf := bytes.Buffer{}
    if n < 0 {
        buf.WriteString("minus")
        n = -n
    }

    if n > 99 {
        if buf.Len() > 0 {
            buf.WriteString(" ")
        }
        buf.WriteString(nominals[n/100])
        buf.WriteString(" hundred")
        n = n % 100
        if n > 0 {
            buf.WriteString(" and")
        }
    }

    if n > 19 {
        if buf.Len() > 0 {
            buf.WriteString(" ")
        }
        buf.WriteString(decades[n/10-2])
        n = n % 10
    }

    if n > 0 {
        if buf.Len() > 0 {
            buf.WriteString(" ")
        }
        buf.WriteString(nominals[n])
    }

    return buf.String()
}

func main() {
    fmt.Println(358, "is", NumberToWords(358))
    fmt.Println(-999, "is", NumberToWords(-999))
}
```

Outputs:

```
358 is three hundred and fifty eight
-999 is minus nine hundred and ninety nine
```

## Discussion

The primary concern when concatenating strings is memory usage and allocations. Go's strings are immutable byte slices so there is no way to extend a string in-place: every concatenation means allocating space for a new string, copying the old string into the new space followed by the concatenated string. Go's compiler does this for you automatically when you join strings using the `+` operator and this works well for concatenating two or three strings together occasionally. However when you have more complex requirements, such as in the `NumberToWords` function above or when you have a variable number of strings then you need something more versatile and efficient.

If your strings are already in a slice then it makes sense to simple use the `Join` function from the `strings` package with an empty string as the join text. This is very efficient and clear.

In this recipe we use a `Buffer` to concatenate the words and punctuation needed to describe a number in words. The `NumberToWords` function can describe numbers ranging from -999 to +999 (and could be extended to much larger range with a little effort).

Although, in principle, building a description of a number like this is simply a matter of concatenating the right parts, there are a number of conditional paths that deal with quirks of grammar, such as putting the word 'and' between the hundreds and the tens, but only if the number is not a whole multiple of 100. This is a great use case for a `Buffer`: we use it to collect and assemble the parts we need and then call its `String` method to access the final string.

Using a  `Buffer` also brings a number of other advantages. It supports the `io.Writer` and `io.Reader` interfaces so can be used efficiently in streaming situations. It also provides a `Reset` function that clears the buffer without shrinking it which can avoid memory allocations when being reused many times.

The `Buffer` type performs very well The following benchmarks demonstrate the difference between the naive approach of using the `+` operator, using `Join` and using a `Buffer`:

```Go
var parts = []string{"abc", "cde", "der", "gwa", "tsj", "ood", "hhg", "apo", "dtw", "ppw"}

var sout string

func BenchmarkStringConcatNaive(b *testing.B) {
    for i := 0; i < b.N; i++ {
        s := ""
        for _, part := range parts {
            s = s + part
        }
        sout = s
    }
}

func BenchmarkStringConcatJoin(b *testing.B) {
    for i := 0; i < b.N; i++ {
        pp := make([]string, len(parts))
        for i, part := range parts {
            pp[i] = part
        }
        sout = strings.Join(pp, "")
    }
}

func BenchmarkStringConcatBuffer(b *testing.B) {
    for i := 0; i < b.N; i++ {
        buf := bytes.Buffer{}
        for _, part := range parts {
            buf.WriteString(part)
        }
        sout = buf.String()
    }
}
```

The results show that the naive approach performed worst of all, being twice as slow as the Buffer version and making four and a half times as many allocations. With longer strings to concatenate the performance would be even worse. Using Join is more efficient but still uses the same amount of memory as the naive approach, even though it made fewer allocations. Join is smart enough to pre-calculate the amount of memory so it avoids the multiple allocations made by the naive method.


```
BenchmarkStringConcatNaive   747 ns/op       224 B/op          9 allocs/op
BenchmarkStringConcatJoin    403 ns/op       224 B/op          3 allocs/op
BenchmarkStringConcatBuffer  328 ns/op       144 B/op          2 allocs/op
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

