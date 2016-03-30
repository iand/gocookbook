# Trimming blanks from end of strings

## Problem

You are receiving strings from elsewhere that have varying amounts of whitespace appended and you want to clean the string by removing it.

## Solution

If you don't mind removing leading whitespace as well, use the `TrimSpace` function from the `strings` package:

```Go
package main

import (
    "fmt"
    "strings"
)

func main() {
    s := "   surrounded by whitespace    "
    fmt.Println(strings.TrimSpace(s))
}
```

If you only want to remove whitespace only from the end of the string, use the `TrimRightFunc` function with the `IsSpace` function from the `unicode` package:

```Go
package main

import (
    "fmt"
    "unicode"
    "strings"
)

func main() {
    s := "   surrounded by whitespace    "
    fmt.Println(strings.TrimRightFunc(s, unicode.IsSpace))
}
```

## Discussion

Go's `strings` package provides a suite of functions for easily removing characters from strings. They can be grouped into four types:

* `Trim`, `TrimLeft`, `TrimRight` - these functions take a `cutset` as the second argument and remove any characters present in the cutset from the target string. The Left and Right variants restrict the removal to the start or end of the string respectively, stopping as soon as they encounter a character not in the cutset.
* `TrimFunc`, `TrimLeftFunc`, `TrimRightFunc` - these take a test function as the second argument. Each character in the target string is passed to the test function and are removed if the function returns true. As above the Left and Right variants restrict their operation to the start and end of the target string, stopping as soon as the test function returns false.
* `TrimPrefix`, `TrimSuffix` - these take a second string as an argument and remove it from the target string only if it either appears as a prefix or a suffix.
* `TrimSpace` - this is a special case convenience function that covers the common case of removing leading and trailing whitespace from a string. It's identical to calling `TrimFunc` with the `unicode.IsSpace` function.

In this recipe we use the `TrimSpace` function to remove all leading and trailing whitespace. It's important to remember that the definition of whitespace here is anything that `unicode.IsSpace` returns true for. This includes spaces, tabs, newlines, vertical tabs, form feeds, carriage returns, next line (U+0085), no-break space (U+00A0) as well as a further 17 characters in the Space Separator category of Unicode such as paragraph separator (U+2029) and medium mathematical space (U+205F). The number of characters classed as whitespace could increase with future versions of Unicode.

This is normally what you want, but if you want to trim only space characters from the string then you need to use the `Trim` function to specify the exact characters to be removed. The following program compares the two:

```Go
package main

import (
    "fmt"
    "strings"
)

func main() {
    s := "\tsurrounded by whitespace  \n  "
    fmt.Printf("%q\n", strings.TrimSpace(s))
    fmt.Printf("%q\n", strings.Trim(s, " "))
}
```

Outputs:

```
"surrounded by whitespace"
"\tsurrounded by whitespace  \n"
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


