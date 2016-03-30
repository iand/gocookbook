# Reverse a Slice

## Problem
You have a slice of values but you want to reverse its order with minimal copying.

## Solution

Swap slice values in place using Go's tuple assignment.

```Go
v := []int{4, 6, 2, 7, 2, 8}

for i := 0; i < len(v)/2; i++ {
	v[i], v[len(v)-i-1] = v[len(v)-i-1],  v[i]
}
```

## Discussion

This situation can occur when you've been appending to a slice while processing but you need to return the values in the opposite order. Rather than prepend items to a slice it can be more efficient to append them and then reverse the slice before returning it.

The solution operates by looping through the slice and swapping values from the start of the slice with those at the end. The first item is swapped with the last, the second with the second to last, the third with the third from last and so on. It's only necessary to iterate through half the slice since each swap operation changes the position of two slice values.

The solution works for any number of slice items: when there are zero or one items the upper bound of the loop is zero so no iterations will be performed and the slice is unchanged.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

