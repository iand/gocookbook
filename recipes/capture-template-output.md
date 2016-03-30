# 2.15 Capture the output of a template

## Problem

You need to execute a template and capture its output for use elsewhere in your program.

## Solution

Execute the template into a `Buffer` from the `bytes` package:

```Go
package main

import (
    "bytes"
    "fmt"
    "log"
    "text/template"
)

var templateText = `
Dear Sam,

Thanks so much for inviting me to the donut competition. I'm
looking forward to taking part and I've already been out
sourcing some amazing new toppings.

All the best,

Hamish`


func CaptureOutput(t *template.Template, data interface{}) (string, error) {
    b := &bytes.Buffer{}
    if err := t.Execute(b, data); err != nil {
        return "", err
    }
    return b.String(), nil
}

func main() {

    tmpl, err := template.New("letter").Parse(templateText)
    if err != nil {
        log.Fatalf("failed to compile template: %v", err)
    }


    output, err := CaptureOutput(tmpl, nil)
    if err != nil {
        log.Fatalf("failed to execute template: %v", err)
    }

    fmt.Println(output)

}
```

## Discussion

One of the most common scenarios for using templates is in web services that are generating HTML or other responses to clients. In these cases it's important to be able to handle large responses being sent to the client and so the `text/template` package takes a streaming approach. The `Execute` method requires an `io.Writer` as its first argument and all template output is written as it is produced, keeping the applicaion's memory footprint low.

However at times you might want to capture the output of a template for additional processing or you may be using a template to generate strings that are going to be used elsewhere in the program. To achieve this you need an implementation of the `io.Writer` interface that simply captures the output. Handily, Go provides one in the standard library: the `Buffer` type in the `bytes` package. This type has a Write method with the following signature:

```
func (b *Buffer) Write(p []byte) (n int, err error)
```

This matches the signature needed to be used as an `io.Writer` so a pointer to a `Buffer` can be used wherever a Writer is required:

```
type Writer interface {
    Write(p []byte) (n int, err error)
}
```


The steps to capture the template's output are shown in the recipe's `CaptureOutput` function. First we create a pointer to a new instance of the `Buffer` type and then we pass it as the first argument of the template's `Execute` method. To obtain the string version of the output, we simply call the buffer's `String` method and return it to the caller.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

