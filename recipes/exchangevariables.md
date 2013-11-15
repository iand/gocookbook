# Exchanging Values Without Temporary Variables

## Problem
You want to exchange two variables without creating an intermediate value.

## Solution
Use tuple assignment

```Go
a := 5
b := 12

a, b = b, a
```

## Discussion

Go supports assigning to a list of variables. When assigning a list of variables to another list the process is split into two steps. In the first step any expressions such as slice indexing or pointer indirections are evaluated in the usual order of precedence. Then the assignments are carried out in left to right order.

This two step process means that the same technique can be used to exchange values in a slice, for example:

```Go
s := []int{5,12}
s[0], s[1] = s[1], s[0]
```

Or the value of two pointers:

```Go
var a = new(int)
var b = new(int)

*a = 5
*b = 12

*b, *a = *a, *b
```

## See Also

[Go Language Specification: Assignments](http://golang.org/ref/spec#Assignments)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

