# Reformatting paragraphs to fixed column width

## Problem

## Solution

```Go
package main

import (
    "fmt"
    "io"
    "os"
    "unicode"
    "unicode/utf8"
)

type WrapWriter struct {
    writer      io.Writer
    size        int
    runesOnLine int
}

func NewWrapWriter(w io.Writer, s int) *WrapWriter {
    return &WrapWriter{
        writer: w,
        size:   s,
    }
}

func (ww *WrapWriter) Write(data []byte) (n int, err error) {
    i := 0
    for i < len(data) {
        runes, bytes, err := seekSpacePunct(data[i:])
        if err != nil {
            return n, err
        }

        if ww.runesOnLine+runes > ww.size {
            _, err := ww.writer.Write([]byte{'\n'})
            if err != nil {
                return n, err
            }

            ww.runesOnLine = 0
        }

        written, err := ww.writer.Write(data[i : i+bytes])
        n += written
        if err != nil {
            return n, err
        }

        ww.runesOnLine += runes
        i += bytes
    }

    if n < len(data) {
        return n, io.ErrShortWrite
    }

    return n, err
}

func seekSpacePunct(remainder []byte) (runes int, bytes int, err error) {
    var (
        width int
        c     rune
    )
    for i := 0; i < len(remainder); i += width {
        c, width = utf8.DecodeRune(remainder[i:])
        if c == utf8.RuneError {
            return 0, 0, fmt.Errorf("invalid rune: %#U", c)
        }
        runes++

        if unicode.IsSpace(c) || unicode.IsPunct(c) {
            return runes, i + width, nil
        }
    }

    return runes, len(remainder), nil
}

func main() {
    text := `It was so very clear for the first time, all of it... ` +
        `It was a village square in a green with trees and an old white-washed ` +
        `Spanish church with a cloister. Across the green, there was a big ` +
        `gray wooden house with a porch and shutters and a balcony above, ` +
        `a small garden, and next to it a livery stable with old carriages ` +
        `lined up inside... At the end of the green, there was a ` +
        `white-washed stone house with a lovely pepper tree at the corner...`

    ww := NewWrapWriter(os.Stdout, 40)
    fmt.Fprintf(ww, text)
}
```

This outputs:

```
It was so very clear for the first time,
 all of it... It was a village square
in a green with trees and an old white-
washed Spanish church with a cloister.
Across the green, there was a big gray
wooden house with a porch and shutters
and a balcony above, a small garden,
and next to it a livery stable with old
carriages lined up inside... At the end
of the green, there was a white-washed
stone house with a lovely pepper tree
at the corner...
```

## Discussion

Reflowing text could warrant a whole chapter by itself, one recipe won't do it full justice. However, here is a simple way to wrap text at a desired column width. The algorithm it uses is quite simple. The idea is to scan through the input looking for places to break the text. If we find a suitable place then check the number of runes that will be written plus the number of runes already written for the current line against the maximum line width. If we are under the maximum width then write the runes and advance the line's rune count, otherwise write a newline character and then the runes that have been gathered and reset the number of runes on the line.

Just like in the previous recipe we implement this as an `io.Writer` however this time we need to be aware of the actual encoded characters in the data. When we were prefixing lines we could afford to be byte-oriented because the only character we were checking was the newline `\n`, everything else could simply be copied to the output. Here it's different: we need to be very aware of the size of each character to ensure the writer will work for any Unicode script.

The type that does the work here is called `WrapWriter` which maintains three fields:

* `writer` - the writer that it will write wrapped text to
* `size` - the maximum number of runes to write on a line
* `runesOnLine` - the number of runes written to the current line

The `NewWrapWriter` constructor function creates a new WrapWriter, initializing its `writer` and `size` fields with the supplied arguments. Once again the majority of the work is done in the `Write` method which is designed to be compatible with `io.Writer` so we need to adhere to the semantics of that interface: return the number of bytes written and any errors that we encounter. Like the previous recipe we use named return values to pre-initialise the byte count and the error to their zero values (0 and nil respectively).

The `Write` method iterates through the bytes in `data` using the helper function `seekSpacePunct` to look for the next suitable place to break the text. This function, explained in more detail below, returns the number of bytes and the number of runes that make up the next chunk of text to write. It may also return an error which we return if it's non-nil, along with the number of bytes from `data` that we have written so far.

The number of runes in the chunk of text (`runes`) plus the number of runes written on the line so far (`runesOnLine`) is compared to the maximum number of runes to write (`size`). If the combination is greater then we end the current line by writing out a newline character and start a new one by resetting the number of runes on the current line to zero.

Then we write the chunk of text we just found, keeping track of the number of bytes that we managed to successfully write, which we add to the running total of bytes kept in `n` before we check the error. This ensures that even if there is an error at this point that we will return the actual number of bytes from `data` that were written. Assuming there were no errors, we advance `runesOnLine` by the number of runes we just found and advance our loop index `i`  by the number of bytes found. Then we continue with the next iteration and look for the next chunk of text to write.

Once we have completed our traversal of the bytes in `data` we check that the number of bytes written is not less than the number of bytes in `data`. If we wrote fewer than expected then we are required to return an error to the caller and in this case the most suitable one is the pre-defined `io.ErrShortWrite` indicating that we weren't able to write the entire byte slice.

Finally we return the number of bytes written and nil to indicate no error or, in other words, success!

All of that is a straightforward implementation of the algorithm outlined at the start of this discussion. The interesting part is in the `seekSpacePunct` function. This function takes slice of bytes and returns the number of runes and corresponding number of bytes until the next whitespace or punctuation character. When this function is called from `Write` we pass in the remainder of the `data` slice which is a fast, efficient operation since a slice is simply a small reference to the underlying array of data. This simplifies the implementation of `seekSpacePunct` since we just start iterating from the start of the `remainder` argument without needing to worry about byte offsets into the original `data` slice.

In `seekSpacePunct` we loop through the bytes using `i` as a loop index. This starts at 0 as normal but we increment it by the width of each character in bytes as we decode them inside the loop. This is actually equivalent to what the `range` operator does when applied to a string, but since we are using byte slices here we have to do it ourselves.

Inside the loop we use the `DecodeRune` function from the `unicode/utf8` package. This takes a slice of bytes as an argument and returns a rune and its width in bytes. Unusually for Go, this function returns a special value to indicate an error. We have to check the returned rune to see if it equals `utf8.RuneError`. If it does then we bail out of the function returning an error stating that we found an invalid rune. The most likely cause here would be operating against a byte slice that didn't contain text encoded as UTF-8.

We increment our rune count and check the decoded rune to see if it is a space or a punctuation character. To do this we use a couple of the helper functions from the `unicode` package: `IsSpace` and `IsPunct`. As the package name suggests these are unicode aware and can recognise a wide range of whitespace and punctuation characters. Potentially this could be extended to look for other types of characters or to avoid splitting on hyphens.

If the decoded rune matches either of the test functions then we return the number of runes that have been read and the number of bytes read so far. Otherwise we continue with the loop. Finally, if we haven't encountered any spaces or punctuation, we return the total number and the length of the supplied byte slice, indicating that the entire slice should be treated as the next chunk of text to write.

This approach works for simple English text but also works just as well for multi-byte encoded characters such as Chinese. This example uses a `WrapWriter` to wrap some Chinese text at 12 characters:

```Go
func main() {
    text := `經常學習，不也喜悅嗎？遠方來了朋友，不也快樂嗎？得不到理解而不怨恨，不也是君子嗎？`

    ww = NewWrapWriter(os.Stdout, 12)
    fmt.Fprintf(ww, text)
}
```

The output looks like this:

```
經常學習，不也喜悅嗎？
遠方來了朋友，
不也快樂嗎？
得不到理解而不怨恨，
不也是君子嗎？
```

As an added bonus, because `WrapWriter` implements `io.Writer` it can be used in conjunction with `PrefixWriter` from the previous recipe. For example, here's how you could prefix the wrapped text with a `>` character for quoting in an email:

```Go
    pr := NewPrefixWriter(os.Stdout, "> ")
    ww := NewWrapWriter(pr, 40)
    fmt.Fprintf(ww, text)
```

Here we create a `PrefixWriter` that wraps Stdout. Any lines written to Stdout via the `PrefixWriter` gets prefixed with `> `. We wrap the `PrefixWriter` with a `WrapWriter` that converts long lines of text into individual lines that it writes to the `PrefixWriter` which prefixes them before writing to Stdout.

With the text from the recipe above, we should expect this output:

```
> It was so very clear for the first time,
>  all of it... It was a village square
> in a green with trees and an old white-
> washed Spanish church with a cloister.
> Across the green, there was a big gray
> wooden house with a porch and shutters
> and a balcony above, a small garden,
> and next to it a livery stable with old
> carriages lined up inside... At the end
> of the green, there was a white-washed
> stone house with a lovely pepper tree
> at the corner...
```

Be sure to get the writers the correct way round, otherwise you'll prefix one line with a `>` and then wrap the result, which is much less effective!

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)



