# Remove Duplicate Values from a Slice while Preserving Order

## Problem

You have a slice containing duplicate values that you need to eliminate but you need to preserve the order of the items.

## Solution

Use a map to keep track of unique items and copy unique items into the correct place in the slice.

```Go
v := []int{6, 3, 8, 1, 3, 2, 7, 9, 1, 3, 6}

seen := make(map[int]struct{}, len(v))

j := 0
for i := range v {
	if _, exists := seen[v[i]]; !exists {
		v[j] = v[i]
		seen[v[i]] = struct{}{}
		j++
	}
}
v = v[:j]
```

## Discussion

This recipe avoids sorting the input slice and instead uses a map to keep track of which items have already been seen. As it iterates through the slice it skips any items that are present in the map and copies new items into the next available position in the slice using the same technique as the previous recipe.

Go automatically expands the capacity of a map when you add new items into it. However each time it does so Go needs to allocate memory and possibly copy the contents of the map to the new memory. By creating the map with an initial size then we are pre-allocating the memory needed so no further allocations are needed during the loop. This makes the performance of the recipe much more predictable. We used the length of the input slice as the initial map size which caters for the worst case where every item is unique, but if you knew that there were a lot of duplicates then you could reduce the map size correspondingly.

Since we are simply using the map to maintain a unique set of items we only care about the keys in the map, the values aren't used. This means we can use the empty struct `struct{}` as the value type which also has the advantage of taking up zero storage in the map.

In the loop we test for the existence of the item in the map using the multiple return value version of the map lookup operator. Accessing the items in a map can be done using one or two return values, which Go chooses automatically for you. In the single return value form you either get the value stored against the supplied key or the zero value of the map's value type:

```Go
value := x[key]
```

However if you ask for a second return value then you receive a boolean that tells you whether the key existed in the map:

```Go
value, exists := x[key]
```

If you're only interested in whether the key exists or not you can use the underscore in place of the value to ignore that part of the return:

```Go
_, exists := x[key]
```

Because Go allows if statements to initialize a local variable before the test, we can ask the map to look up a key in the map and then test the result in a single line:

```Go
if _, exists := seen[v[i]]; !exists {
```

The recipe uses this if statement to decide whether to copy the slice item or not.

Finally, once we have iterated through all the slice items we use the same slice truncation idiom `v = v[:j]` as we did in the previous recipe to remove the remaining junk at the end of the slice. The same warning about pointers and garbage collection applies here too, so if your working with a slice of pointers then you should set all the remaining items to nil before truncating.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)


