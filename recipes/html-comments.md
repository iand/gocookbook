# Comments in HTML template:

# Problem

You need to output comments from your templates but Go strips them automatically.

# Solution

Use a custom function that converts its argument into a `template.HTML` type which Go's template engine will emit without further escaping.

```Go
package main

import (
    "html/template"
    "log"
    "os"
)

var templateContent = `
{{rawhtml "<!--[if IE 9]>"}}
IE 9 stuff here
{{rawhtml "<![endif]-->"}}
`

func rawHTML(text string) template.HTML {
    return template.HTML(text)
}

func main() {
    tmpl := template.New("comments")

    tmpl.Funcs(map[string]interface{}{
        "rawhtml": rawHTML,
    })

    _, err := tmpl.Parse(templateContent)
    if err != nil {
        log.Fatalf("failed to parse template: %v", err)
    }

    err = tmpl.Execute(os.Stdout, nil)
    if err != nil {
        log.Fatalf("failed to execute template: %v", err)
    }
}
```

This outputs:

```
<!--[if IE 9]>
IE 9 stuff here
<![endif]-->
```

# Discussion

The authors of Go's html templating system had security in mind when they designed it and many potential avenues for exploitation are automatically guarded against. The security model they implemented trusts template authors, but treats any data used by templates as untrusted and a potential attack vector. The assumption is that manual sanitization of data is too complex for template authors and maintainers and so should be automated.

The HTML sanitization algorithm used is extremely accurate but sometimes you need to override the rules to achieve more control over the output or to relax the strictness of the checking. In particular, the algorithm takes a draconian approach to HTML comments: they are removed entirely on the basis that they don't contain parseable structure or human readable content. Unfortunately this approach isn't entirely accurate since comments have been used in the past to provide processing directives for versions of Internet Explorer up to version 9.

This recipe shows how we can use one of the template package's in-built types to override the sanitization rules. We create a custom function called rawHTML that accepts a string as an argument and converts it to a `template.HTML` type which the template system usees to output raw HTML. We register that function with our template so it's available to use as a directive. We can then use it whenever we want precise control over the HTML output, such as emitting conditional comments for older versions of Internet Explorer.

To understand why this works we have to delve deeper into the workings of the `html/template` package. This package works in a similar way to `text/template` but also automatically escapes data emitted by the template in a context-sensitive way to ensure that the right kind of escaping is performed. For example, it recognises the HTML script tag so that any template directives occurring within one have their output encoded using Javascript rules. This is a much richer solution than the naive approach of HTML-escaping every output which can easily miss important Javascript injection attacks. Consider the following example. Suppose you have this in your template:

```HTML
<script>
doAction('{{.Action}}');
</script>
```

If Action was filled with data supplied by an end user from a form or clicking on a link then without escaping it's possible that they could inject malicious Javascript. If Action contained `', alert('attack'), '` then the unescaped template would emit:

```HTML
<script>
doAction('', alert('attack'), '');
</script>
```

This would execute the code the attacker supplied in the Action variable. The naive approach of HTML-escaping all input would produce:

```HTML
<script>
doAction('&apos;, alert(&apos;attack&apos;), &apos;');
</script>
```

A web browser would see the `&apos;` HTML entities and treat them as single quotes. so this is actually completely identical to the unescaped version and the attackers code still executes.

The correct way to escape this kind of data is to recognise that it's using Javascript and apply Javascript rules for encoding quotes within strings. This is exactly what Go's html template system does, producing:

```HTML
<script>
doAction('\x27, alert(\x27attack\x27), \x27');
</script>
```

In almost every case this is exactly what you need to happen. But there are times when you want more control. Handily, `html/template` provides some support for relaxing the escaping performed in various contexts by defining named types for the various contexts. If the template system encounters data using one of these types it assumes that the template author has performed the necessary escaping beforehand. In essence it moves the data from the untrusted realm to the trusted one. It still performs the minimal escaping needed to ensure the data can be used cleanly in its context but skips the sanitization of the data.

The type we need is called `template.HTML` and is intended to represent a fragment of sanitized HTML. Data with this type won't be subject to any HTML escaping rules so when the template emits it no escaping is done and the raw HTML is output.

There are two approaches to using this with the template system. The recipe solution shows the most convenient way: register a function that converts its argument into the `template.HTML` type. However, if you're building a template system that other people will use, you might want a more controlled approach to avoid template authors abusing the function and undermining all the in-build security goodness of the `html/template` package.

The alternate approach is to use the `template.HTML` type directly in your data structs that you pass to the template when it is executed. For example, you might have a contact type like this:

```Go
type Contact struct {
    Name   string
    Email  string
    Status template.HTML
}
```

and a corresponding template:

```
<h1>Contact</h1>
{{.Name}} <{{.Email}}>
<div>{{.Status}}</div>
```

The `Name` and `Email` fields will be emitted with all the normal HTML sanitization, but the `Status` field will be emitted as raw, unescaped HTML. This is slightly safer than allowing raw HTML to be emitted for any data via a function, but it's still a way to circumvent some extremely secure templating rules so take care when using it.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

