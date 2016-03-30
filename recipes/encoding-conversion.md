# Converting text to and from UTF-8

## Problem

You want to work with text from an external source that isn't encoded as UTF-8.

## Solution

Use the `transform` and `encoding` packages from Go's extended library to convert the text input to UTF-8. This program reads the lines from a file and decodes them from the Windows-1252 character encoding to UTF-8:

```Go
package main

import (
    "bufio"
    "fmt"
    "os"

    "golang.org/x/text/encoding/charmap"
    "golang.org/x/text/transform"
)

func DecodeLines(filename string) ([]string, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer file.Close()

    decodingReader := transform.NewReader(file, charmap.Windows1252.NewDecoder())

    lines := []string{}

    scanner := bufio.NewScanner(decodingReader)
    for scanner.Scan() {
        lines = append(lines, scanner.Text())
    }
    return lines, scanner.Err()
}

func main() {
    lines, err := DecodeLines("input.txt")
    if err != nil {
        log.Fatalf("problem reading from file: %v", err)
    }

    for _, line := range lines {
        fmt.Println(line)
    }
}
```


## Discussion

As discussed in the previous recipe, strings in Go are just read-only byte slices with no built-in encoding. However if the string happens to be encoded as UTF-8 then a whole range of powerful operators and functions can be used. For example, Go's `range` operator automatically understands and decodes UTF-8 from strings so you can iterate over each character in a string very simply. This works extremely well with the requirement that all Go source code must be encoded as UTF-8. The consequence is that any string literal that you use in your program will already be in UTF-8 and processable by the `range` operator.

This pairing of source code encoding with `range`'s behaviour makes it easy to forget that Go's strings aren't always UTF-8. This can bite when you read input from an external source that might be in a different encoding. In these cases you need to explicitly convert the encoding to UTF-8 before you can use `range` and other UTF-8 aware operations and functions.

The Go team provide a comprehensive range of packages that deal with character encodings but they are not part of the the standard library. Instead they resided in the `/x/` package namespace, a special area for "extended" packages that aren't subject to the strict compatibility and stability constraints of the standard library. This doesn't mean they aren't as good, just that the Go team aren't prepared to provide the same level of support for them.

The key package here is `golang.org/x/text/transform` which provides various functions for transforming the bytes that represent text. As is common in Go packages there are stream-based functions as well as simpler string and byte slice oriented ones. The two stream functions that are of interest are `NewReader` and `NewWriter` which wrap an existing `io.Reader` or `io.Writer` and apply a transform to any bytes read or written with them. The transforms can be any types that satisfy the `Transformer` interface.

The `golang.org/x/text/encoding` package and its subpackages provide encoders and decoders to and from a huge range of character encodings and UTF-8. Each encoder or decoder implements the `Transformer` interface so can be used directly with the transform package. A decoder transforms its input from a character encoding to UTF-8, an encoder transforms from UTF-8 to the required character encoding. There are 6 subpackages:

* `golang.org/x/text/encoding/charmap` - contains common encodings that cover proprietary IBM, ISO, Windows and Macintosh code pages.
* `golang.org/x/text/encoding/japanese` - provides support for the most common Japanese character encodings EUC-JP, ISO-2022-JP and Shift JIS
* `golang.org/x/text/encoding/korean` - contains the EUC-KR Korean encoding.
* `golang.org/x/text/encoding/simplifiedchinese` - provides encodings for Simplified Chinese such as GB18030, GBK and HZ-GB2312
* `golang.org/x/text/encoding/traditionalchinese` - contains the Big5 encoding for Traditional Chinese
* `golang.org/x/text/encoding/unicode` - provides support for UTF-16 with and without BOMs (byte-order marks)

In this recipe's code, we're working with text in the Windows-1252 encoding which is the default encoding on many Windows systems. The `DecodeLines` function does all the work. First of all we open the file specified by the `filename` argument. Files in Go support the `io.Reader` interface so they can be used as streams with many of the other functions in the standard library. In this recipe we want to wrap the file with a transformer that can decode text encoded using Windows-1252 into UTF-8.

To do this we use the `transform.NewReader` function to create a `transform.Reader` that wraps the file. We also pass in the result of calling `charmap.Windows1252.NewDecoder()` which creates a transformer that can decode bytes in Windows-1252 to the UTF-8 encoding.

Because `transform.Reader` also implements `io.Reader` we can pass it directly to a `bufio.Scanner` which can read the file line by line. This chaining of readers is a fantastic way to compose functionality from small but specialized components. `bufio.Scanner` doesn't know anything about files or character encodings, it just knows how to scan a stream of bytes for newlines and return the chunks of bytes they delimit. We simply read all the lines from the file into a slice of strings and return it. The encoding to UTF-8 is done on the fly with minimal memory used.

If we wanted to quickly convert a string containing bytes encoded in Windows-1252 we could use one of the non-streaming convenience methods provided by the transform package. For example we could quickly convert a string from Windows-1252 to UTF-8 by using:

```Go
res, n, err := transform.String(charmap.Windows1252.NewEncoder(), s)
```

`transform.String` returns three arguments: the string resulting from the transformation, the number of bytes used from the input string and any error that may have occurred. The number of bytes will be less than the length of the input string if there are truncated characters at the end of it.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)





