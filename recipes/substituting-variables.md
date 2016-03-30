# Substituting variables in strings

## Problem

You need to implement a simple templating language to substitute variables in multiple documents.

## Solution

Use the `Replacer` type from the `strings` package:

```Go
package main

import (
    "fmt"
    "strings"
)

func main() {
    macros := strings.NewReplacer(
        "$ABOUT", `<a href="/about.html">About Us</a>`,
        "$HOME", `<a href="/index.html">Home Page</a>`,
        "$SEARCH", `<a href="http://search.example.com/query.cgi">Search</a>`,
    )

    input := `
            <p>Navigation:
             <ul>
            <li>$HOME</li>
            <li>$SEARCH</li>
            <li>$ABOUT</li>
             </ul>
        </p>`

    fmt.Println(macros.Replace(input))
}
```


## Discussion

This recipe shows how a `Replacer` could be used to implement a simple macro-based system for a website. The macros are held in a new `Replacer` created with the `NewReplacer` function. This function expects an even number of string arguments. Each pair represents a string to look for and a string to replace it with.

Performing the search and replace is simply done by calling the `Replace` method on the `Replacer`, passing in the input string. It returns a copy of the string with all the search strings substituted by their replacements.

The `Replacer` type is heavily optimized for repeated replacement operations. For small numbers of replacements it performs a fast linear search and replace but for longer lists it builds a trie index of replacements. Because of this, it's important to reuse Replacers whenever possible. If you have replacements that change frequently or must be computed then `Replacer` is not the best solution and you'd be better off directly using the `Replace` function in the `strings` package:

```Go
package main

import (
    "fmt"
    "strings"
    "time"
)

func main() {
    input := `<p>Published on $DATE.</p>`

    fmt.Println(strings.Replace(input, "$DATE", time.Now().Format("Jan 2, 2006"), -1))
}
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

