# Logging HTTP Requests

## Problem
You want to log every request to your HTTP server.

## Solution

Wrap your handler functions in another function that intercepts requests to the handler and logs details about the request.

```Go
func writeLog(handler http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Printf("%s %s %s", r.RemoteAddr, r.Method, r.URL)
		handler.ServeHTTP(w, r)
	})
}

func fooHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, foo!")
}

func barHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, bar!")
}

func main() {
    http.HandleFunc("/foo", writeLog(fooHandler))
    http.HandleFunc("/bar", writeLog(barHandler))
    http.ListenAndServe(":8080", nil)
}
```

## Discussion
The `main` function sets up the HTTP server with two functions that handle requests to /foo and /bar. These handlers do the actual work of responding to HTTP requests. In this recipe the normal handler functions are wrapped by a function called `writeLog` that intercepts and logs information about requests before forwarding them onto the wrapped handler.

The `writeLog` function takes an `http.Handler` as an argument and also returns one which allows it to be inserted in a chain of handlers. Note that `http.Handler` is an interface that defines the `ServeHTTP` function that is called by the HTTP server when it receives a request. `writeLog` creates a function that performs the logging and passes the request onto the wrapped handler:

```Go
func(w http.ResponseWriter, r *http.Request) {
		log.Printf("%s %s %s", r.RemoteAddr, r.Method, r.URL)
		handler.ServeHTTP(w, r)
	}
```

This function is then converted to a `http.HandlerFunc` which provides the necessary implementation of `ServeHTTP` so the function satisfies the `http.Handler` interface. 

An advantage of this approach is that it separates the concerns of logging from handling requests. The handler functions are unchanged by changes in the logging functionality.

## See Also
[http.HandlerFunc](http://golang.org/pkg/net/http/#HandlerFunc), [http.Handler](http://golang.org/pkg/net/http/#Handler), [log.Printf](http://golang.org/pkg/log/#Printf)



----
[no rights reserved](http://creativecommons.org/publicdomain/zero/1.0/)

