# Pad a string to a fixed length

## Problem

You have a series of strings of different lengths and you want to pad them with spaces so they are all the same length.

## Solution

Prepate a buffer of padding space and append them using string slicing:

```Go
package main

import (
    "fmt"
    "strings"
    "unicode/utf8"
)

type Padder struct {
    padding string
}

func NewPadder(size int) Padder {
    return Padder{
        padding: strings.Repeat(" ", size),
    }
}

func (p Padder) Pad(s string) string {
    if rc := utf8.RuneCountInString(s); rc < len(p.padding) {
        return s + p.padding[rc:]
    }
    return s
}

func main() {
    input := []string{"hello", "¡hola", "sveiki", "здравствуйте", "你好"}
    padder := NewPadder(14)

    for _, s := range input {
        fmt.Println(padder.Pad(s), ":", len(s))
    }
}
```

## Discussion

This recipe could be tackled with a one liner but by wrapping it up as the Padder type it potentially becomes more reusable, plus can deal with Unicode scripts more accurately. The one liner is interesting though because it shows the general principle of the solution:

```Go
    s = s + "              "[len(s):]
```

The trick is to use a buffer of spaces and use it to create a variable amount of padding depending on the length of the original string. If the padding string consists of 14 spaces and s is three bytes long then [len(s):] becomes [3:] which means take the bytes from 3 to the end of the padding. This is 11 spaces, or just enough to pad the original string to a length of 14.

Where this falls down is when the string to pad contains non-latin characters. For example the Russian "здравствуйте" is twelve characters long, but takes up 24 bytes whereas the Chinese "你好" is two characters long but consists of six bytes. The naive approach would not pad the Russian version and under pad the Chinese text.

The problem is that `len` counts bytes, not runes. The `utf8` package in the Go standard library provides the `RuneCountInString` which does what we need and this is what the recipe uses. The `Padder` type is very simple: it's just a fixed buffer of padding characters plus a method that can apply the correct amount of padding to an input string. The `NewPadder` constructor function initializes a `Padder` by using the `Repeat` function from the `strings` package to set up a buffer of spaces of the correct length.

The meat of the recipe is in the `Pad` function which counts the number of runes in the input string. If the count is smaller than the padding then it returns the input string plus a subslice of the padding, just as in the one-liner version, except that this is based on the number of runes rather than the number of bytes. If the input string is larger then we just return it.

Note how the we use the two-clause form of the if statement here. Go allows you to initialise some variables before you perform the test. The initialized variables are in scope throughout the if statement and any related else clause. In the recipe we use this to avoid calling `RuneCountInString` twice, once in the if statement and once again in the slicing operation. Counting runes in a string is fast, but there's no need to do it more times than is necessary.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


