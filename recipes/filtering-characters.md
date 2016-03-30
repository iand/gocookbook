# Stripping unwanted characters from strings

## Problem

You receive string input from an untrusted source and want to remove any characters that are not in a specified whitelist.

## Solution

Use the `Map` function from the `strings` package with a custom whitelist function.

```Go
package main

import (
    "strings"
    "fmt"
)


func InWhitelist(w string) func(rune) rune {
    return func(r rune) rune {
        if strings.ContainsRune(w, r) {
            return r
        }
        return -1
    }
}


func main() {
    input := "#1-800    555 1111"
    whitelist := "0123456789"

    output := strings.Map(InWhitelist(whitelist), input)

    fmt.Println("input:", input)
    fmt.Println("output:", output)
}
```

## Discussion

Cleaning up input from web forms or other sources is a common problem that is best tackled with a whitelist approach. Define the characters that are allowed and remove or replace all others. In the example used in this recipe, the whitelist is needed to normalize phone numbers entered into a form to ensure they only contain numbers.

The two key functions of this recipe are the `Map` function and the `InWhitelist` function. The `Map` function is contained in the `strings` package from Go's standard library. It takes a function as an argument and a string and works by iterating over each rune in the string, calling the function for each one. If the function returns a positive value then it is treated as a replacement and the string is updated with the new rune. If the function returns a negative number then the `Map` function interprets that as meaning the rune being tested should be removed from the string. We use this in the recipe to remove runes that don't match those in our whitelist.

The function passed to `Map` can be any function with the following signature:

```Go
func(rune) rune
```

In other words, any function that takes a rune as its only argument and returns a rune can be passed to `Map`. This could be a pre-existing named function, a closure or a method on a type. We can't pass any other argument to the `Map` function so we need a way to create a function that understands the whitelist we want to use. In this recipe we define a function called InWhitelist that returns a new closure function of the correct type. The InWhitelist function has the following declaration:

```Go
func InWhitelist(w string) func(rune) rune {
```

This may look strange because there are two `func` keywords in play at the same time. The key to understanding it is to remember that Go declares types reading from left to right. The first `func` keyword indicates that we are defining a function. Go then expects everything up to the first parenthesis `(` to be the name of the function, everything up to the closing  parenthesis `)` to be its arguments and everything up to the first curly brace `{` to be its return value. So, reading the above using these rules we can see that the recipe declares a function called `InWhitelist`, with one string argument and a return type of `func(r rune) rune`, i.e. it returns a function that takes a rune as an argument and returns a rune. This means that `InWhitelist` expects a string and returns a function that can be passed to the `Map` function later on.

The body of `InWhitelist` returns a function closure that simply tests to see if the rune passed to the closure appears in the supplied whitelist. If it is then the closure returns the rune, otherwise it returns -1 which will signal `Map` to remove the rune from the string. To perform the filtering, we simply pass the whitelist to the `InWhitelist` function and pass the result to `Map` along with our input string.

With a small modification, this technique could be used to substitute a safe character in place of unsafe ones, such as when asking for a valid filename. To do this, instead of returning -1 for runes not in the whitelist, return a substitute run instead, such as an underscore:

```Go
func InWhitelist(w string) func(rune) rune {
    return func(r rune) rune {
        if strings.ContainsRune(w, r) {
            return r
        }
        return '_'
    }
}
```

Alternatively, the `unicode` package in the Go standard library contains a suite of useful functions for tesing Unicode runes. These can easily be wrapped in a test function that can be passed to `Map`. For example, to filter a string so it only contains printable Unicode characters, wrap the `unicode.IsPrint` function like this:

```Go
func IsPrintable() func(rune) rune {
    return func(r rune) rune {
        if unicode.IsPrint(r) {
            return r
        }
        return -1
    }
}
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


