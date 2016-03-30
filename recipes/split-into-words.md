# Split a string into words

## Problem

You have a string and want a list of the words it contains.

## Solution

Use the `FieldsFunc` function from the `strings` package with a custom function that uses the `IsLetter` function from the `unicode` package:

```Go
package main

import (
    "fmt"
    "strings"
    "unicode"
)

func SplitWords(s string) []string {
    return strings.FieldsFunc(s, func(c rune) bool {
            return !unicode.IsLetter(c)
    })
}

func main() {
    text := `The trouble with having an open mind, of course, is that people will insist on coming along and trying to put things in it.`
    words := SplitWords(text)
    fmt.Printf("%q\n", words)
}
```

## Discussion

Splitting some text into its component words seems like a simple problem at first: just use a function like `strings.Split` and split on every space in the input. However this naive approach doesn't deal with repeated spaces, newlines, variable-width spaces or punctuation. A better approach is to use a whitelist approach, defining what you want to keep and use this to extract just the words.

The `FieldsFunc` function is a partner to the `Fields` function, both in the `strings` package. The `Fields` function simply splits a string up into whitespace-delimited fields, which is close to what we need but it treats punctuation as word characters. This is fine for texts such as computer programs but when dealing with prose text it's better to eliminate punctuation too.

`FieldsFunc` is similar to the `TrimFunc` function we used in the Trimming Blanks recipe in that it takes two arguments: the string to split and a function that tests each character in the string. Just like `TrimFunc` The test function for `FieldsFunc` has a signature of `func(rune) bool` which means it takes a single rune as an argument and reports `true` if the rune passes the test, `false` otherwise.

The test function is best thought of as a test for the field delimiters. `FieldsFunc` processes each rune in the string in sequence and as soon as the test function reports `true` it marks that point as the end of a field and continues processing runes. When the test function reports `false` again, it marks that as the start of the next field. Once it has processed all the runes in the string, it returns all the marked fields as a slice of strings.

In this recipe, the `SplitWords` function uses the `IsLetter` function from the `unicode` package to test for our fields. We return `true` when the rune being tested is not a letter and `false` when it is a letter. This makes all non-letter characters appear as delimiters to `FieldsFunc` leaving all the letter characters as the fields to return.

If you wanted to include numbers in the fields then you could modify `SplitWords` to also test using the `IsNumber` function:

```Go
func SplitWordsAndNumbers(s string) []string {
    return strings.FieldsFunc(s, func(c rune) bool {
            return !unicode.IsLetter(c) && !unicode.IsNumber(c)
    })
}
```

Alternatively, if you're just interested in extracting all the numbers from a string to use in later computation you could use:

```Go
func SplitDigits(s string) []string {
    return strings.FieldsFunc(s, func(c rune) bool {
            return !unicode.IsDigit(c)
    })
}
```

 This could be a quick way to parse a stream of delimited measurement data from a sensor if you know it only reports whole numbers. Note we use the `IsDigit` function rather than the `IsNumber` function because we only want the digits from 0-9. The `IsNumber` function is more comprehensive and includes numbers from international scripts such as MODI DIGIT NINE (U+11659), MYANMAR TAI LAING DIGIT ZERO (U+A9F0) and MATHEMATICAL SANS-SERIF BOLD DIGIT ZERO (U+1D7EC) which would are not easily parseable as numbers.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

