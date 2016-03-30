# Concatenate Two Slices

## Problem

You have two slices of the same type that you want to concatenate

## Solution
Use the built in append function with the ... operator.

```Go
s1 := []string{"fee", "fi"}
s2 := []string{"fo", "fum"}

s1 = append(s1, s2...)
```

## Discussion

Go's append function is the most efficient way to add values to a slice. It is most commonly seen being used with two arguments so it's easy to forget that append is variadic and can accept any number of arguments to be appended to the slice.

In this solution the variadic operator ... is used to pass the contents of a slice to the append function. The variadic operator allows you to pass all the members of a slice as though they were separate arguments.

Be careful not to omit the ... by mistake. If you do you'll receive a compilation error since Go assumes you're trying to append the slice itself rather than its contents:

```
s1 = append(s1, s2)

> cannot use s2 (type []string) as type string in append
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

