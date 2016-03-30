# Prefix each line of a multi-line string

# Problem

You want to prefix all output from a process with a specified prefix string.

# Solution

Create a writer that scans its input for newlines and writes the prefix string just after each newline.

```Go
package main

import (
    "fmt"
    "io"
    "os"
)

type PrefixWriter struct {
    writer  io.Writer
    prefix  []byte
    newline bool
}

func NewPrefixWriter(w io.Writer, p string) *PrefixWriter {
    return &PrefixWriter{
        writer:  w,
        prefix:  []byte(p),
        newline: true,
    }
}

func (pw *PrefixWriter) Write(data []byte) (n int, err error) {
    for i, b := range data {
        if pw.newline {
            if _, err := pw.writer.Write(pw.prefix); err != nil {
                return n, err
            }
            pw.newline = false
        }
        if b == '\n' {
            pw.newline = true
        }

        written, err := pw.writer.Write(data[i : i+1])
        n += written
        if err != nil {
            return n, err
        }
    }

    if n < len(data) {
        return n, io.ErrShortWrite
    }
    return n, nil
}

var text = `It was so very clear for the first time, all of it...
It was a village square in a green with trees and an old white-washed
Spanish church with a cloister. Across the green, there was a big
gray wooden house with a porch and shutters and a balcony above,
a small garden, and next to it a livery stable with old carriages
lined up inside...
At the end of the green, there was a white-washed stone house with
a lovely pepper tree at the corner...`

func main() {
    pw := NewPrefixWriter(os.Stdout, ">>> ")
    fmt.Fprint(pw, text)
}

```

## Discussion

You might encounter this kind of problem across a diverse range of scenarios such as writing an email application that needs to quote text by prefixing it with a specific character or logging every line of output with the source of the log entry.

The algorithm to do this is quite straightforward: scan through the input and copy each byte to the output but when you encounter a new line, output it followed by the prefix.

To make this more reusable in this recipe we create a new type called `PrefixWriter` which implements `io.Writer` so it can be used with any stream-oriented functions. We also create a constructor function called `NewPrefixWriter` which accepts an `io.Writer` that will be wrapped by the `PrefixWriter` and the string that each line must be prefixed with. In the `main` function you can see how this is used to wrap Stdout so that any text written through the `PrefixWriter` will be output to the console, prefixed with ">>> ". Because `PrefixWriter` implements the `io.Writer` interface we can use it with the `fmt.Fprint` function to write our test string directly to it. The output of running the program looks like this:

```
>>> It was so very clear for the first time, all of it...
>>> It was a village square in a green with trees and an old white-washed
>>> Spanish church with a cloister. Across the green, there was a big
>>> gray wooden house with a porch and shutters and a balcony above,
>>> a small garden, and next to it a livery stable with old carriages
>>> lined up inside...
>>> At the end of the green, there was a white-washed stone house with
>>> a lovely pepper tree at the corner...
```

Let's delve into the implementation of `PrefixWriter`. The `PrefixWriter` type contains three fields:

* `writer` holds the wrapped writer that it will write its output to
* `prefix` holds the required string to prefix each line with. It's held as a byte slice to save converting it each time it is written.
* `newline` is a flag that indicates whether we have just written a new line. If we have then we'll want to write the prefix next time we encounter a byte.

The single function we provide is `Write` which has the required signature that makes it compatible with the `io.Writer` interface. It takes a single byte slice argument and returns the number of bytes written and any error encountered while writing. This satisfies the syntax requirements for implementing `io.Writer` but we mustn't forget to implement the semantics too. Two important details are that we must return the number of bytes written from the input data and we must not return a nil error if we haven't written all the data.

Note that we are using named return parameters `n` and `err`. This automatically makes those two variables available and initialized to their zero values for use in the function body. We don't need to declare them separately. We'll use `n` to keep a running count of bytes we have written from the input data.

The bulk of the `Write` function is taken up with a loop that traverses `data`, the supplied byte slice argument. For each iteration we check whether the `newline` flag is set. Until we encounter a new line in the data, this flag will be false. When it is true we write our prefix to the `writer` field and reset the `newline` flag. When we write the prefix we need to check for errors and return if we have a problem writing to the underling writer. Note though that we don't increase our bytes written count when we write the prefix. The semantics of `io.Writer` are clear that we must only return the count of bytes written from the `data` argument. Because the prefix is separate from `data`, we don't count its length in our running total.

After the newline check we examine the byte at the current position in our iteration over `data`. If it's a new line then we set our newline flag to true so we'll write the prefix on our next iteration.

Next we write the byte to the wrapped writer. We could wrap the byte in a new byte slice to pass to the writer but it's simpler just to take a subslice of `data` and pass that. Once we have written the byte we increment our byte counter `n` with the result of the call to `Write`. We do this before we check for errors because we always want to be sure that our byte count is accurate. It's possible that the call to `Write` returns 1 and an error, so we need to account for the byte being written even when there is a returned error. If there is an error, we return it with our count of bytes written so far.

Once we have completed the loop we could just return. However as a guard we check the number of bytes we have written. If it's less than the number of bytes in `data` then we must have dropped a byte or two somewhere. It's possible that a misbehaving wrapped writer returned a 0 and no error when we tried to write our byte to it. In this case we don't have many options, but the contract for `io.Writer` states that we must return an error if `n < len(data)`. Luckily the `io` package provides a pre-defined error for just this case, `io.ErrShortWrite`, so we return that. Otherwise we return the number of bytes written and nil for our error, indicating all went well.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


