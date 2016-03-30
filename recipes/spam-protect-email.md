# Spam-protecting email addresses in templates

## Problem

You want to ensure that any email addresses output in a template are obscured from spam harvesting bots.

## Solution

Use a custom function to transform the email address and use this in your template whenever you write an email address:

```Go
package main

import (
    "html/template"
    "log"
    "os"
    "strings"
)

type Contact struct {
    Name  string
    Email string
}

var contacts = []Contact{
    Contact{
        Name:  "Jo Kruschke",
        Email: "jk@example.com",
    },
    Contact{
        Name:  "Sue Sharman",
        Email: "sue.sharman@example.com",
    },
    Contact{
        Name:  "Dave Nicholls",
        Email: "dave@example.com",
    },
}

var contactTemplate = `
<h1>Contact Details</h1>
<ul>
    {{range .}}
        <li>{{.Name}} <{{obscureEmail .Email}}></li>
    {{end}}
</ul>
`

func obscureEmail(s string) string {
    at := strings.Index(s, "@")

    if at < 2 {
        return s
    }

    return s[:1+at/4] + "..." + s[at:]
}

func main() {
    tmpl := template.New("contacts")
    funcs := map[string]interface{}{
        "obscureEmail": obscureEmail,
    }

    tmpl.Funcs(funcs)

    _, err := tmpl.Parse(contactTemplate)
    if err != nil {
        log.Fatalf("failed to parse template: %v", err)
    }

    err = tmpl.Execute(os.Stdout, contacts)
    if err != nil {
        log.Fatalf("failed to execute template: %v", err)
    }
}
```

## Discussion

Go's template system provides a useful range of built in functions but when these don't cover the functionality you need then you can also supply your own custom functions. This recipe demonstrates how a custom function can be used to obscure an email address to hide it from spambots but still show some of the address for humans.

The template for this recipe is a simple html fragment showing a contact list. The data for the template is a slice of a custom Contact type that holds the person's name and email address. We range over each contact using the `{{range}}` directive using "dot" as the argument. Each contact gets passed to the next section of the template which emits the contact name and the email address. The directive for emitting the email address looks like this:

```
<{{obscureEmail .Email}}>
```

As you can see, we're using a custom directive name `obscureEmail` here. The argument, which is the value of `.Email`, is passed to the directive and whatever the function returns is emitted in place of the whole directive. But how does the template know what to do when it sees the directive?

The registering of the custom function for the directive is done in the `main` function of the recipe. First of all we create a new, empty template which we name "contacts". We then define a `FuncMap` which is simply a map of directive name to an `interface{}` value which must be a function that returns either a value or a value and an error. The registered function can take any number of arguments.

We pass the `FuncMap` to the template by calling the `Funcs` method on the template. Then we proceed to parse the template content and execute it as usual. It's important that the call to `Funcs` happens before any calls to `Parse` otherwise the parsing will fail to find the custom functions.

In the recipe, we simply map the directive name `obscureEmail` to a local function of the same name. The `obscureEmail` function itself is fairly straightforward. It expects one argument in which it searches for the first @ symbol. If it finds one that isn't at the very start of the string it returns the first character in the string plus a quarter of the characters between the start and the @ symbol, followed by three dots and then the @ symbol and the domain name. This produces an obscured version of the email, with just enough of a hint for a human to recognise an email address they may already know for a person. The output of the template looks something like:

```HTML
<h1>Contact Details</h1>
<ul>
    <li>Jo Kruschke &lt;j...@example.com></li>
    <li>Sue Sharman &lt;sue...@example.com></li>
    <li>Dave Nicholls &lt;da...@example.com></li>
</ul>
```

Note that although we mapped the `obscureEmail` function to a custom directive with the same name in the template, there's no rule that says we have to. We could just as easily have mapped it to simpler name such as `email` like this:

```
    funcs := map[string]interface{}{
        "email": obscureEmail,
    }

    tmpl.Funcs(funcs)
```

Then we would have used it in the template like this:

```
<{{email .Email}}>
```

As you can see, the template just uses the name you pass in with the function map so you have complete flexibility to align custom function names with your particular needs.

It's also worth pointing out that the recipe uses a very simple technique for obscuring email addresses. More sophisticated spambot counter measures could be implemented in the same way by extending the custom function without any changes needed in the template itself.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

