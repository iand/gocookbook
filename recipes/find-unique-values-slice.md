# Find Unique Values in a Slice

## Problem

You have a slice containing items that may be duplicated but you want to produce a new slice containing only the unique items from the first slice

## Solution

Sort the slice and copy only unique values into a new slice

```Go
import "sort"

func FindUniques() {
	v := []int{6, 3, 8, 1, 3, 2, 7, 9, 1, 3, 6}

	uniques := make([]int, 0, len(v))

	sort.Ints(v)
	for i := range v {
		if i == 0 || v[i] != v[i-1] {
			uniques = append(uniques, v[i])
		}
	}

	fmt.Println(uniques)
}
```

## Discussion

The key to finding unique items in any list is being able to efficiently detect when two items are the same. The naive approach is to loop through each item in the list and check if each one appears elsewhere in the list. This approach results in poor performance especially in the case where there are no duplicates in the original list. A list of 100 items requires 101 iterations through the list and up to 10,000 comparisons;  a list of 5000 items requires 5001 iterations and up to 25 million comparisons.

This recipe uses the sort package from the Go standard library to order the items in the slice in ascending order. A single iteration through the slice is then performed and an item is added to the result slice only if it is the first item in the slice or if it is different from the previous item. This results in far fewer comparisons than the naive approach scales better for larger slices.

To reduce copying, when the recipe makes a new slice to hold the unique items it passes 0 and len(v) as the second and third argument. This tells Go to allocate a new slice with zero initial length but with a capacity of len(v). Although the subsequent calls to append will increase the slice capacity automatically, pre-specifying it like this will reduce the number of memory allocations needed and the amount of copying Go needs to do under the hood.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)




