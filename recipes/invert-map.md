# Invert a map

## Problem

You have a map that maps one value to another and you need to invert this so that you can find the original map's key by its value.

## Solution

Iterate over the map and use the value as the key in a new map.

```Go
func InvertMap(m map[string]int) map[int]string {
	inv := make(map[int]string, len(m))

	for k, v := range m {
		inv[v] = k
	}

	return inv
}
```

## Discussion

This situation can occur when you are using a map for lookups such as finding which database ID to use for a country. If you need to find the country given a database ID then you'd need to to invert the map.

The recipe shows an InvertMap function that accepts a map of a string to an int as an argument. The return value has to be the inverse: a map of an int to a string. Note that this implies that some maps cannot be inverted since Go restricts the types that can be used as keys of a map. Only "comparable" types can be used as a key which excludes slices, maps and function pointers. This means maps of maps, maps of slices and maps of functions are not invertible.

We know the size of the map needed so we can initialise the inverse with the correct length which improves efficiency of the function.

Once the inverse map is created we iterate over the input map using the two variable form of the `range` keyword. As discussed in the previous recipe, when `range` is used with a map argument it can either return each key or each key and value. We want both the key and the value and simply add them to the map, using the input map's value as the key for the inverse map and its key as the inverse map's value.

This recipe works well when the map only contains unique values, which is typical in a lookup. However many maps contain duplicate values. For example if you had a map of country to continents then a function like InvertMap would return a map with only one key and value per continent. The other countries' values would have been lost. To counter this, the more general version of the InvertMap function returns a map of slices. This allows each key to map to a list of possible values.

The basic operation of the function is the same as for the simple approach, but instead of setting the using the map's key as the inverse map's value, we append it to a slice:


```Go
func InvertMapGeneral(m map[string]int) map[int][]string {
	inv := make(map[int][]string, len(m))

	for k, v := range m {
		inv[v] = append(inv[v], k)
	}

	return inv
}
```

Note the use of `append` to add a value directly to the map. This works because `append` returns a new slice which can be used directly as the new map value, it doesn't modify the existing value held in the map.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


