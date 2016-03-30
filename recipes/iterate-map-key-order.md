# Iterate a map in key order

## Problem

You need to access the values of a map in alphabetical key order.

## Solution

Iterate over a sorted slice of all keys in the map.

```Go
import "sort"

func IterateSortedKeys(m map[string]int) {
	keys := make([]string, 0, len(m))

	for k := range m {
		keys = append(keys, k)
	}

	sort.Strings(keys)

	for _, k := range keys {
        println(m[k])
	}
}
```

## Discussion

You cannot rely on a consistent order for iteration of map keys in Go because the map type is free to re-order map keys whenever the map is updated. In fact between versions 1.2 and 1.3 the Go authors added deliberate randomisation of small maps to encourage programmers not to rely on map key order in their tests.

Thus, to iterate a map in any specific order you have to get a list of keys and put them in the order you need. In this recipe we simply sort them alphabetically but more complex orderings could be performed such as prioritising some keys before others.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

