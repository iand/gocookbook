# Compare strings case insensitively

## Problem

You want to efficiently determine whether two strings are the same in a case-insensitive way.

## Solution

Use the `EqualFold` function from the `strings` package in the Go standard library.

```Go
package main

import (
    "strings"
    "fmt"
)

func main() {
    a := "wikipedia.org"
    b := "WIKIPEDIA.org"

    if strings.EqualFold(a, b) {
        fmt.Println("strings are the same")
    } else {
        fmt.Println("strings are different")
    }
}
```

## Discussion

Go provides a case-insensitive string comparison via the some-what obscurely named `EqualFold` function. In Unicode terminology a case fold is a language-neutral mapping of the lower-case, upper-case and title-case variants of a character. It's designed to assist in comparing strings that may contain mixed case characters.

It's tempting, when asked to compare strings case-intensively, to simply convert the case of the strings to upper or lower and compare the results. This approach, while simple, has two big drawbacks. Firstly it only works for scripts that have case, the vast majority of scripts have no case at all. Secondly, even when the strings are known to be using ASCII only, it's highly inefficient. Converting the strings to upper or lower-case involved making a copy of them, whereas comparing them by their fold simply looks up each character in a table and compares the results.

The following benchmarks demonstrate the difference:

```Go
var s1 = "Wax on, right hand. Wax off, left hand. Wax on, wax off."
var s2 = "Wax on, right hand. Wax off, left hand. Wax on, wax Off."
var eq bool

func BenchmarkStringEqualToUpper(b *testing.B) {
    for i := 0; i < b.N; i++ {
        eq = (strings.ToUpper(s1) == strings.ToUpper(s2))
    }
}

func BenchmarkStringEqualFold(b *testing.B) {
    for i := 0; i < b.N; i++ {
        eq = strings.EqualFold(s1, s2)
    }
}

func BenchmarkStringEqualOperator(b *testing.B) {
    for i := 0; i < b.N; i++ {
        eq = (s1 == s2)
    }
}
```

The results show that `EqualFold` is ten times faster than using `ToUpper` and, better still, makes no heap allocations which reduces pressure on the garbage collector. For comparison, the timings for a straight case-sensitive equality test are also shown.

```
BenchmarkStringEqualToUpper     2759 ns/op      256 B/op    4 allocs/op
BenchmarkStringEqualFold        267 ns/op       0 B/op  0 allocs/op
BenchmarkStringEqualOperator        10.9 ns/op      0 B/op  0 allocs/op
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

