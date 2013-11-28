# Shuffle a Slice

## Problem
You need to randomly shuffle a slice of strings.

## Solution

Use the Fisher–Yates shuffle algorithm.

```Go
func shuffle(a []string) {
	for i := int32(len(a) - 1); i >= 0; i-- {
		j := rand.Int31n(i + 1)
		a[i], a[j] = a[j], a[i]
	}
}
```

## Discussion

This recipe uses the Fisher–Yates shuffle algorithm to perform the randomisation of the slice. This algorithm is an in-place shuffle that simply swaps values in the slice without allocating new memory to hold the shuffled version. 

The shuffle function operates on the slice passed in as an argument. Newcomers to Go often think that they need to pass in a pointer to a slice but this is not necessary because a slice is simply a small struct that points to an underlying array. Operations on the slice affect the underlying array via the embedded pointer in the slice.


## See Also

[Wikipedia: Fisher–Yates shuffle](http://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle), [Go Blog: Arrays, slices (and strings)](http://blog.golang.org/slices), [Recipe: Exchanging Values Without Temporary Variables](exchangevariables.md)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

