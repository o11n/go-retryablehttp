go-retryablehttp
================

[![Build Status](http://img.shields.io/travis/hashicorp/go-retryablehttp.svg?style=flat-square)][travis]
[![Go Documentation](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)][godocs]

[travis]: http://travis-ci.org/hashicorp/go-retryablehttp
[godocs]: http://godoc.org/github.com/hashicorp/go-retryablehttp

The `retryablehttp` package provides a familiar HTTP client interface with
automatic retries and exponential backoff. It is a thin wrapper over the
standard `net/http` client library and exposes nearly the same public API. This
makes `retryablehttp` very easy to drop into existing programs.

`retryablehttp` performs automatic retries under certain conditions. Mainly, if
an error is returned by the client (connection errors, etc.), or if a 500-range
response code is received (except 501), then a retry is invoked after a wait
period.  Otherwise, the response is returned and left to the caller to
interpret.

The main difference from `net/http` is that requests which take a request body
(POST/PUT et. al) can have the body provided in a number of ways (some more or
less efficient) that allow "rewinding" the request body if the initial request
fails so that the full request can be attempted again. See the
[godoc](http://godoc.org/github.com/hashicorp/go-retryablehttp) for more
details.

Version 0.6.0 and before are compatible with Go prior to 1.12. From 0.6.1 onward, Go 1.12+ is required.
From 0.6.7 onward, Go 1.13+ is required.

Example Use
===========

Using this library should look almost identical to what you would do with
`net/http`. The most simple example of a GET request is shown below:

```go
resp, err := retryablehttp.Get("/foo")
if err != nil {
    panic(err)
}
```

The returned response object is an `*http.Response`, the same thing you would
usually get from `net/http`. Had the request failed one or more times, the above
call would block and retry with exponential backoff.

## Retrying cases that fail after a seeming success

It's possible for a request to succeed, but then for later processing to fail, and sometimes this requires retrying the full request. E.g. a GET that returns a lot of data may "succeed" but the connection may drop while reading all of the data.

In such a case, you can use `DoWithResponseHandler` (or `GetWithResponseHandler`) rather than `Do` (or `Get`). A toy example (which will retry the full request and succeed on the second attempt) is shown below:

```go
c := retryablehttp.NewClient()
handlerShouldRetry := false
c.GetWithResponseHandler("/foo", func(*http.Response) bool {
    handlerShouldRetry = !handlerShouldRetry
    return handlerShouldRetry
})
```

## Getting a stdlib `*http.Client` with retries

It's possible to convert a `*retryablehttp.Client` directly to a `*http.Client`.
This makes use of retryablehttp broadly applicable with minimal effort. Simply
configure a `*retryablehttp.Client` as you wish, and then call `StandardClient()`:

```go
retryClient := retryablehttp.NewClient()
retryClient.RetryMax = 10

standardClient := retryClient.StandardClient() // *http.Client
```

For more usage and examples see the
[godoc](http://godoc.org/github.com/hashicorp/go-retryablehttp).
