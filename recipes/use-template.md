# Producing templated output

## Problem

You have a standard text template that you need to fill with different values each time it is executed.

## Solution

Use the `text/template` package from the Go standard library. In this example, a template is used to produce a readme file in Markdown format for a software project.

```Go
package main

import (
    "log"
    "os"
    "text/template"
)

type Project struct {
    Name        string
    Author      *Person
    Overview    string
    Description string
    License     int
    Year        int
}

type Person struct {
    Name  string
    Email string
}

const (
    None = iota
    RightsReserved
    MIT
    PublicDomain
)

var proj = Project{
    Name: "jkQuickDraw",
    Author: &Person{
        Name:  "Jo Kruschke",
        Email: "jk@example.com",
    },
    Overview:    "Drawing made easy.",
    Description: "jkQuickDraw is a drawing program for people with no artistic skill.",
    License:     RightsReserved,
    Year:        2015,
}

var readmeTemplate = `
# {{.Name}}

{{.Overview}}

{{with .Author}}Created by {{.Name}} <{{.Email}}>{{end}}

## Description

{{.Description}}
{{with .License}}
## License
{{if eq 1 .}}Copyright {{$.Year}} {{$.Author.Name}}. All rights reserved.
{{else if eq 2 .}}Copyright {{$.Year}} {{$.Author.Name}}. This software is made available under the terms of the MIT License.
{{else if eq 3 .}}Copyright waived and dedicated to the public domain {{$.Year}}.
{{end}}
{{end}}`

func main() {

    tmpl, err := template.New("readme").Parse(readmeTemplate)
    if err != nil {
        log.Fatalf("failed to compile template: %v", err)
    }

    err = tmpl.Execute(os.Stdout, proj)
    if err != nil {
        log.Fatalf("failed to execute template: %v", err)
    }
}
```

Running this program produces the following output:

```
# jkQuickDraw

Drawing made easy.

Created by Jo Kruschke <jk@example.com>

## Description

jkQuickDraw is a drawing program for people with no artistic skill.

## License
Copyright 2015. All rights reserved.
```

## Discussion

In the earlier "Substituting variables in strings" recipe we had a number of fixed variables that needed to be substituted into any number of input strings. This simple substitution solves a wide range of problems but can become unwieldy when the variables change frequently or you need to vary the substitution based on the value of the variables. A better approach is to move to a template solution.

Go's template packages `text/template` and `html/template` are very powerful but can be daunting to use because of their complexity. In this recipe and the ones that follow we'll explore some of the features provided by these packages.

This recipe shows a simple scenario for templated output: a template is provided for creating a readme file for use with software projects. When a project needs a readme, data about it is passed to the template which formats the information into a markdown document. We define our Project type at the beginning of the recipe's code. It holds the name of the project, a short overview and a longer description as string fields. The author of the project is indicated by the `Author` field which holds a pointer to a type called `Person` which, in turn, defines string fields for the person's name and email address. Finally the `Project` type has a field for the type of source code license to be used and the year the project was published, which we'll use to write a copyright statement with the license information. The `License` field is an int and we define some constant values to represent different license options: None, RightsReserved, MIT and PublicDomain.

We next define a sample project called `proj` to test with. This just contains values for each field so we can see the results of the template's execution when we run the program.

The template is defined in the variable `readmeTemplate` for convenience but more normally it would be held in a file on disk. The template consists of the text content interspersed with template directives delimited by `{{` and `}}`. When a template is executed it is passed data that the directives use to control the template's operation.

The key to understanding Go's template language is the concept of the "dot". As the template is processed, the structure of the passed data is walked and "dot" represents the current position in the hierarchy of the template's data. All directives are performed in the context of where "dot" is currently. When the template is executed "dot" is set to the data that is passed in the Execute method so, in our case, "dot" initially refers to the instance of `Project` that we pass in.

The simplest form of directive is "dot" followed by a field name. In our template we have `{{.Name}}` on the first line. This kind of directive gets the value of the named field on the value "dot" points to and injects it into the template output. It doesn't change the position of "dot". This means we can use it again straight after to emit the value of the `Overview` field.

Another type of directive is the `with` action which tests the value of its argument and executes the next part of the template, up to the `{{end}}` directive only if the argument is non-empty. The argument is considered empty if it's value is:

 * false
 * zero
 * a nil pointer
 * a nil interface value
 * a zero length array, slice, or map
 * an empty string

If the the argument is non-empty then the `with` action moves "dot" to point to the value of the argument. In the recipe we use `with` in the following section of the template:

```
{{with .Author}}Created by {{.Name}} <{{.Email}}>{{end}}
```

This tests the `Author` field of the project and if it is not nil then moves "dot" to point to `Author`'s value and then continues processing the template. The next directive it meets `{{.Name}}` will now output the author's name, not the project name. When the `{{end}}` directive is encountered, the template moves "dot" back up to refer to the project again.

If our project did not have an author assigned then the `Author` field would be nil and the `with` directive would skip processing the associated subsection of the template. "dot" would be unchanged and the execution would pick up after the matching `{{end}}` directive.

We use the same `with` directive to test for the presence of a license. If the `License` field evaluates to zero, corresponding to the constant `None` then this section is skipped. If not, we immediately test the value of the `License` field using another directive: the `if` action. As you might expect, `if` tests its argument and executes the associated template section if the argument is not empty. It differs from `when` in a few ways though. Most importantly, it does not change the value of "dot". It can also be followed by an `else` directive which is executed if the argument is empty, or by `else if` directives which perform alternative tests. A series of `if` and `else if` directives can be used to emulate a `switch` statement, providing a list of alternatives. We use this in the recipe to choose the appropriate license text:

```
{{if eq 1 .}}Copyright {{$.Year}} {{$.Author.Name}}. All rights reserved.
{{else if eq 2 .}}Copyright {{$.Year}} {{$.Author.Name}}. This software is made available under the terms of the MIT License.
{{else if eq 3 .}}Copyright waived and dedicated to the public domain {{$.Year}}.
{{end}}
```

The argument we supply to the `if` and `else if` directives uses a function called `eq` which tests if the following two arguments are equal. Note how the second argument is "dot" on its own. The outer `{{with .License}}` directive has moved the "dot" to point at the value of the `License` field so the `eq` function is comparing a number with the value of the field. If the `eq` function returns true then we execute the template after the `if` directive up to the next '{{end}}' or `else if`.

Because `if` doesn't change "dot", during the execution of the next part of the template, "dot" is still pointing to the value of `License`. When we want to output the year of publication of the project we need to refer back up the data hierarchy to find the `Year` field. We do this by using the special variable `$` which always has the value of the data passed into the template by the Execute method. In this recipe, `$` will refer to the project passed in, so `{{$.Year}}` will emit the value of the project's `Year` field. We also use this to emit the name of the author again using `{{$.Author.Name}}`. Note how we can traverse the data hierarchy by using dots to separate fields. `{{$.Author.Name}}` means look for the `Author` field and then find that value's `Name` field.

This recipe has covered the essential operation of Go's template system. The processing model becomes very clear once you understand how "dot" traverses the data and provides a context for the directives. The `template` package provides many more directives and functions, some of which we'll use in the next few recipes.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)





