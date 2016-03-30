# Remove Duplicate Values from a Slice

## Problem

You have a slice containing duplicate values that you need to eliminate, leaving the slice containing only unique values.

## Solution

Sort the slice and copy only values into the right place in the slice.

```Go
import "sort"

func RemoveDuplicates() {
	v := []int{6, 3, 8, 1, 3, 2, 7, 9, 1, 3, 6}

	sort.Ints(v)
	j := 0
	for i := range v {
		if i == 0 || v[i] != v[i-1] {
			v[j] = v[i]
			j++
		}
	}
	v = v[:j]
}
```

## Discussion

This is similar to the previous recipe where we were finding unique values in a slice, except it alters the input slice rather than returning a new one. This means we can reuse the memory already allocated in the slice and avoid allocating new memory for the results.

The recipe starts by sorting the input slice in ascending order and then begins to iterate through each of the slice's items. Like the previous recipe we keep an item only if it is the first in the slice or if it is different from the previous item. However instead of appending the unique item to a new slice we copy it to a position in the input slice just after the last unique item we found. The variable `j` is used to keep track of where we need to copy the item to. Each time we make a copy we increment `j` to the next available position.

If there are no duplicate items in the input slice then `i` and `j` keep pace with one another and always have the same value. In this case each copy is just overwriting the value in the current position. A possible optimisation would be to skip the assignment `v[j] = v[i]` for cases where `i` equals `j`. For int slices it's probably not worth doing this, but for large items such as structs it may be preferable to avoid this copy.

When a duplicate is found no copy is made and `j` is not incremented. The loop continues leaving `j` pointing to the location just after the last unique item.

After the loop all the unique items will have been copied to the start of the slice but the remainder of the slice will contain junk that we don't need. Since `j` is incremented every time we find a unique item we know that once the loop completes `j` will equal the total number of unique items found. The recipe uses this information to truncate the input slice to the number of unique items. The input slice now only contains unique items in sorted order.

Normally after the truncation of the slice the lost items will be garbage collected. However if the items are pointers or structs containing pointers then references to them can potentially still be held by the array underlying the slice which would prevent them from being garbage collected. Setting these items to nil before truncating should avoid this:

```Go
// Use this when v is a slice of pointers
for i := j; i < len(v); i++ {
	v[i] = nil
}
v = v[:j]
```

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

