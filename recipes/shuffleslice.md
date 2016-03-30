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

Note that, due to Go's typing rules, shuffling other types of slices will require dedicated shuffle functions. Here's the int version of the shuffle function:

```Go
func shuffleInt(a []int) {
	for i := int32(len(a) - 1); i >= 0; i-- {
		j := rand.Int31n(i + 1)
		a[i], a[j] = a[j], a[i]
	}
}
```

However, looking more closely it's easy to see that the algorithm itself doesn't depend on the types of the slice members, just on two properties of the slice: its length and the ability to swap its members. This makes it possible to use an interface with two methods, Len and Swap, to write a more general version of the shuffle function. 


```Go
type Shuffleable interface {
	Len() int
	Swap(i, j int)
}

func shuffleAny(a Shuffleable) {
	for i := a.Len() - 1; i >= 0; i-- {
		j := int(rand.Int31n(int32(i) + 1))
		a.Swap(i, j)
	}
}
```

Note how the Shuffleable interface contains a subset of the methods defined by sort.Interface from the standard library. This means that any type that implements sort.Interface can also be used with the shuffleAny function:

```Go
func main() {
	as := []string{"red", "blue", "orange", "green"}
	shuffleAny(sort.StringSlice(as))
	fmt.Printf("%+v\n", as)
}
```

View this on Go Play: http://play.golang.org/p/ftVbRa0EHW


## See Also

[Wikipedia: Fisher–Yates shuffle](http://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle), [Go Blog: Arrays, slices (and strings)](http://blog.golang.org/slices), [Recipe: Exchanging Values Without Temporary Variables](exchangevariables.md)

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

