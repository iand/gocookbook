# Initialise a struct with default values

## Problem

You have a struct type that includes fields that need to be initialised to default values before use.

## Solution

Write a constructor function for the type.

## Discussion

Go doesn't provide a way to specify default values for fields in a struct. This can be a problem when you have maps or channels as field types since they will be nil when the type is created. The simplest approach is to provide a constructor function for the type. A constructor function is simply a function that returns a new, initialised version of the type, ready to be used. There are no special naming or behaviours required: constructor functions are just a convention.

Suppose you have a type that contains an embedded HTTP client for use when making calls to an image processing web service:

```Go
import (
	"bytes"
	"net/http"
)

type APICaller struct {
	Address string
	Client http.Client
}

func (a *APICaller) SendImage(pixels []byte) error {
	buf := bytes.NewBuffer(pixels)
	resp, err := a.Client.Post(a.Address, "image/json", buf)
	if err != nil {
		return err
	}
	resp.Body.Close()
	return nil
}
```

This code is prone to errors because the caller must remember to set the HTTP client whenever they create an `APICaller`. If they forget then the program will panic when the `SendImage` method tries to call `Post` on the `Client` field, which will be at its default `nil` value. To avoid this, provide a constructor function:

```Go
func NewAPICaller(address string) *APICaller {
	return &APICaller{
		Address: address,
		Client: &http.Client{},
	}
}
```

Any user of the code can then call `NewAPICaller` to get a ready-to-use instance of the `APICaller` type. If they have a specific need to customise the HTTP client used by the `APICaller`, e.g. to specify timeouts or proxy information then they can either create the type as before and set the Client to one that meets their need or use the `NewAPICaller` and override the `Client` field.

Constructor functions don't have to start with the word "New" or even contain the name of the type they are constructing. They should be named according to their function. A database connection type, for example, might have a constructor function called `Open` since that clearly conveys the intended behaviour.

Constructor functions are especially useful when the type has a lot of fields to initialise or if the initialisation is more complex than usual. An example in the Go standard library is the `Regexp` type in the `regexp` package which represents a regular expression used for string matching. The package provides several constructor functions such as `Compile` and `CompilePOSIX` that initialise a `Regexp` instance with various default configurations and set up the state machine
that is used to execute the regular expression.

----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

