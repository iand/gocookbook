# Infinite Loops

## Problem
You need to loop forever, without doing any work.

## Solution

Use an empty select block.

    select{}

## Discussion

You may be tempted to use an infinite for loop such as:

    for {}

but this is inefficient since it wastes cpu time and cannot be pre-empted by the garbage collector. The select statement blocks efficiently.


## See Also

[Go Language Specification: Select statements](http://golang.org/ref/spec#Select_statements)



