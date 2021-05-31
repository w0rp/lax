# lax

Lax - A library for building simple REST APIs in Go.

```
     ^ ^
 ("\(-_-)/")
 )(       )(
((...) (...))
```

A library by w0rp for implementing HTTP request views for REST APIs, with
hopefully very few lines of code. The library doesn't do all that much for you,
it just makes it easier to write endpoints for a REST API, without wrapping
the entire standard library in slower abstractions.

## Example Usage

This library is very experimental. Here is some example usage as of the time
this README was written.

```go
package main

import (
    "log"
    "net/http"
    "github.com/w0rp/lax"
)

// Declare a type to marhsal/unmarshal
type Comment struct {
    Text string `json:"text"`
}

// Just store an Array of comments in memory, as an example.
var commentList []Comment = make([]Comment, 0, 20)

var commentListHandler http.HandlerFunc = lax.Wrap(lax.View{
    // A simple GET implementation that returns the data.
    Get: func(request *lax.Request) interface{} {
        return commentList
    },
    // A simple POST implementation that lets a user submit a new Comment.
    Post: func(request *lax.Request) interface{} {
        comment := Comment{}

        if err := request.JSON(&comment); err != nil {
            // Return a 400 error response with the error message.
            return lax.MakeBadRequestResponse(err)
        }

        if (len(comment.Text) == 0 || len(comment.Text) > 500) {
            // Return a 400 error response pointing out a problem.
            //
            // Comes out as [
            //   {
            //     "path": "text",
            //     "problem": "missing or invalid text"
            //   }
            // ]
            return lax.MakeErrorListResponse(
                lax.Issue("text", "missing or invalid text"),
            )
        }

        commentList = append(commentList, comment)

        return &comment
    },
})

func main() {
    // Serve the endpoint like any other with the standard library.
    //
    // You can mix and match the probably slower Lax views with more efficient
    // request handlers for very hot endpoints.
    http.HandleFunc("/comment", commentListHandler)

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```
