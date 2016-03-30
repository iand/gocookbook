# Modularising a large template

# Problem

You have a set of large, complex templates that contain a lot of duplication between them.

# Solution

Extract the duplication into named modular parts and include them by reference in your main templates. Split the template parts into separate files and load them all at once at execution time using the template package's `ParseFiles` or `ParseGlob` functions. For small snippets that don't warrant their own file, define named templates within a helper template file and invoke them directly in the primary template by using the `{{template}}` directive.

# Discussion

Over time templates tend to accrete markup and become hard to manage. This becomes more acute when there are multiple templates duplicating the same functionality over and over. A typical example is a large website with templates for product pages, articles, company information and bios. They all include the same boilerplate markup for headers, footers, navigation and branding and only differ in their handling of the main content. It's likely that there are small inconsistencies in the templates that could more easily ironed out if they shared markup. Extracting this duplication into reusable parts is the best approach here and there are a couple of ways Go's template system can help.

The first way is to split a single large template into separate smaller ones, each in their own files. To illustrate this, we'll use the following markup. Hopefully in real life your templates are going to be a lot more complex and you'll have a much more interesting website than this one:

```HTML
<!doctype html>
<html><head>
<title>{{.Title}}</title>
</head>
<body>
    <div class="sidebar">
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/products">Products</a></li>
            <li><a href="/offices">Offices</a></li>
        </ul>
    </div>

    <div class="content container">
       {{.Content}}
    </div>
    <div class="footer">
    Find out more <a href="/about">about us</a>. We're also <a href="/jobs">hiring</a>.
    </div>
</body>
```

To begin with, identify a section of the template that could be reused, such as the header, and copy the markup into a new file. Then right at the top of the new file add a `{{define}}` directive to name the template and at the end of the file add a corresponding `{{end}}` directive to signify the end of the definition. It should look something like this:

```
{{define "header"}}<!doctype html>
<html><head>
<title>{{.Title}}</title>
</head>
{{end}}
```

Save this in file called `header.tmpl`. The actual name does not matter, but for simplicity make sure all of the extracted templates have the same file extension. We're using `.tmpl` here to signal that it's a template but you could use `.template`, `.txt` or even `.html` - it makes no difference to Go's template system.

Next, replace the header markup in the original template with a `{{template}}` directive. This needs two arguments: the name of the template to invoke and the data it needs to operate on. Use the name you specified in the `{{define}}` directive, not the filename, and pass in "dot" as the argument for the moment:

```
{{template "header" .}}
<body>
    <div class="sidebar">
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/products">Products</a></li>
            <li><a href="/offices">Offices</a></li>
        </ul>
    </div>

    <div class="content container">
       {{.Content}}
    </div>
    <div class="footer">
    Find out more <a href="/about">about us</a>. We're also <a href="/jobs">hiring</a>.
    </div>
</body>
</html>
```

Repeat this for the footer and save to `footer.tmpl`:

```
{{define "footer"}}<div class="footer">
Find out more <a href="/about">about us</a>. We're also <a href="/jobs">hiring</a>.
</div>
{{end}}
```

And the sidebar ans save it to `sidebar.html`:

```
{{define "sidebar"}}
    <div class="sidebar">
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/products">Products</a></li>
            <li><a href="/offices">Offices</a></li>
        </ul>
    </div>
{{end}}
```

The page template now looks like this:

```
{{template "header" .}}
<body>
    {{template "sidebar" .}}
    <div class="content container">
       {{.Content}}
    </div>
    {{template "footer" .}}
</body>
</html>
```

It's already much more manageable, but there's a final step. You need to top and tail this template with a `{{define}}` directive too, so the final page template becomes:

```
{{define "page"}}
{{template "header" .}}
<body>
    {{template "sidebar" .}}
    <div class="content container">
       {{.Content}}
    </div>
    {{template "footer" .}}
</body>
</html>
{{end}}
```
Save this to `page.tmpl` in the same directory as the other templates. Now you should have a directory with four template files that work together to produce your final template. To execute the page template you need to load and parse all four files as a group. You can do this in a few ways. One is to use the `ParseFiles` method and supply all four filenames:

```Go
templates, err := template.ParseFiles("page.tmpl", "header.tmpl", "sidebar.tmpl", "footer.tmpl")
```

This gives you precise control over which templates are going to be used. However, when you have a lot of templates it can be more convenient to use the `ParseGlob` function which takes a "glob" as an argument. This is basically a pattern for locating files and supports `*` as a wildcard (see the documentation for `filepath.Match` in the Go standard library for the full syntax). This allows you to load all templates with a common file extension very simply:

```Go
templates, err := template.ParseGlob("*.tmpl")
```

Once you have your templates loaded and parsed you can execute the page template like this:

```Go
err := templates.ExecuteTemplate(os.Stdout, "page", data)
```

You can replace the `"page"` argument with the name of any template that has been defined in one of the files. Any data you supply as the third argument will be passed to the template as normal.

The second way Go's template system can help is by grouping related snippets of template together. You don't have to create a new file for every template definition, you can define as many as you like in a single file and they will all be available when you load them with `ParseFiles` or `ParseGlob`. You can use this to build up useful snippets of template markup that you want to reuse across many pages or even projects:

```
{{define "tags"}}
{{ if .Tags }}
    <div class="info">
      Other posts tagged as
      {{ range $i, $t := .Tags }}{{if $i}},{{end}}
      <a href="/tags/{{ $t }}/">{{ $t }}</a>{{ end }}
    </div>
{{ end }}
{{end}}

{{define "copyright"}}
Copyright {{.Publish.Year}} {{.Author}}. All rights reserved.
{{end}}
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

