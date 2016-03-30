# Processing a String One Character at a Time

## Problem

You need to access specific characters in a string that might contain multi-byte encodings of characters.

## Solution

If you are iterating over all the characters in a string, use the `range` operator:

```Go
package main

import "fmt"

func main() {
    s := "Ræven løb under træet"
    for _, c := range s {
        fmt.Printf("%c\n", c)
    }
}
```

If you need to find the character at a specific position, convert the string to a rune slice and use the index operator:

```Go
package main

import "fmt"

func main() {
    s := "Ræven løb under træet"

    r := []rune(s)
    fmt.Printf("%c\n", r[5])
}
```

## Discussion

In Go a string is simply a read-only slice of bytes. These can actually be any bytes at all and don't have to make sense as text in any particular encoding. The `len` function always returns the number of bytes in a string and when you index into it with `[x]` you always access the byte at that position. However, when you use the `range` operator, Go will attempt to interpret the string as a set of UTF-8 encoded bytes. Each iteration of the `range` loop will return two values: the byte index of the start of the character and the `rune` that represents the character. A `rune` is a 32-bit integer that contains a Unicode code point. UTF-8 is a variable-width encoding so each iteration of the loop may increase the byte index by more than one to produce a single rune.

For example, this function prints the byte index with the character when ranging over the Danish string above:

```Go
func rangeString() {
    s := "Ræven løb under træet"
    character := 0
    for i, c := range s {
        character++
        fmt.Printf("char %d; byte %d; rune %U (%c)\n", character, i, c, c)
    }
}
```

This outputs:

```
char 1; byte 0; rune U+0052 (R)
char 2; byte 1; rune U+00E6 (æ)
char 3; byte 3; rune U+0076 (v)
char 4; byte 4; rune U+0065 (e)
char 5; byte 5; rune U+006E (n)
char 6; byte 6; rune U+0020 ( )
char 7; byte 7; rune U+006C (l)
char 8; byte 8; rune U+00F8 (ø)
char 9; byte 10; rune U+0062 (b)
char 10; byte 11; rune U+0020 ( )
char 11; byte 12; rune U+0075 (u)
char 12; byte 13; rune U+006E (n)
char 13; byte 14; rune U+0064 (d)
char 14; byte 15; rune U+0065 (e)
char 15; byte 16; rune U+0072 (r)
char 16; byte 17; rune U+0020 ( )
char 17; byte 18; rune U+0074 (t)
char 18; byte 19; rune U+0072 (r)
char 19; byte 20; rune U+00E6 (æ)
char 20; byte 22; rune U+0065 (e)
char 21; byte 23; rune U+0074 (t)
```

You can see that there are 21 characters encoded as 24 bytes and that occasionally the byte index will jump by two. These are characters that in UTF-8 are encoded as two bytes. For example, the second character is æ which in Unicode is code point U+00E6. When encoded as UTF-8 this takes two bytes: C3 A6.

Because strings are really just byte slices, indexing into the string doesn't work well for multi-byte characters. For example, this loop works fine for ASCII but for the multi-byte Danish string above it produces what looks like garbage:

```Go
    for i := 0; i < 6; i++ {
        fmt.Printf("%d: %c\n", i, s[i])
    }
```

Outputs:

```
0: R
1: Ã
2: ¦
3: v
4: e
5: n
```

You also need to be careful when creating a subslice of a string since the slicing operator also works on bytes not characters. For example:

```
s := "Ræven løb under træet"
fmt.Printf("%s\n", s[0:2])
```

Outputs an 'R' followed by an invalid character because the two bytes of 'æ' have been truncated.

If you want to index into or subslice multi-byte character strings safely, the simplest approach is to convert the string to a slice of runes. Go provides a convenient automatic conversion just for this purpose `[]rune(s)` will convert the string `s` to a slice of runes, interpreting each sequence of bytes as the corresponding UTF-8 encoding of a rune. This program demonstrates how you can use it to support string indexing and subslicing:

```Go
package main

import "fmt"

func main() {
    s := "Ræven løb under træet"
    r := []rune(s)

    for i := 0; i < 6; i++ {
        fmt.Printf("%d: %c\n", i, r[i])
    }

    fmt.Printf("Subslice: %s\n", string(r[0:2]))
}
```

Now we get the correct output:

```
0: R
1: æ
2: v
3: e
4: n
5:
Subslice: Ræ
```

It's important to remember that, like the `range` operator, this automatic conversion from a string to a rune slice only works if the string contains characters encoded as UTF-8 bytes. If not, you need to explicitly convert the string's encoding to UTF-8 first.


----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

