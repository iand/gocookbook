# Get all the keys in a map

## Problem

You have a map and need a list of all its keys.

## Solution

Iterate over the map and add each key to a slice of the correct type.

```Go
func MapKeys(m map[string]int) []string {
	keys := make([]string, 0, len(m))

	for k := range m {
		keys = append(keys, k)
	}

	return keys
}
```
## Discussion

Go's map type has the benefit of being built into the language and performs well, however it doesn't provide any additional methods other than setting, getting and deleting map keys and values.

This recipe shows a straightforward way of copying the keys into a slice. First we initialise the slice we want to return. Here we use the three argument form of `make`: the first argument specifies the type, the second the size of slice we want and the third is the slice capacity. We pass zero for the size because we want the slice to start empty. By passing the length of the map we pre-allocate the memory needed to add all the map keys into the slice. Capacity is a soft limit and Go allows slices to expand beyond their default capacity. However when it needs to expand the slice's capacity it will need to allocate new memory and very often copy the underlying slice data to the new memory. Thus the function would still work if we didn't pass a capacity but the function would be slower since Go would have to allocate new memory and copy slice items multiple times as we add them to the `keys` slice.

Once we have initialised the slice, we simply iterate over all the map keys. The recipe uses the `range` keyword to perform the iteration. When used with a map, `range` returns one or two values depending on the number of variables to the left of the `range` keyword. If one variable is used then it will get the value of the current map key and if two variables are used then they get assigned the key and its value. In this recipe we don't need the value so we simply use the one variable form of the `range` keyword.

In the body of the loop we use the built in `append` function to add the current map key to the `keys` slice. Because we pre-allocated the capacity of the `keys` slice the `append` function will simply place the key at the next available memory location.

Lastly we return the `keys` slice which now contains all the keys of the supplied map `m`. Returning slices from functions is very efficient in Go since the slice is just a small reference to the underlying data. The data itself is not returned.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


