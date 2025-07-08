---
title: 'Making HTTP Calls'
description: 'Learn how to make HTTP calls in Go using the net/http package and understand best practices and performance tips'
date: '2025-03-24'
category: 'HTTP'
---

Go's `net/http` package provides robust support for making HTTP calls. This snippet demonstrates how to perform HTTP requests using this package.

## Basic HTTP GET Request

Here's a simple example showing how to perform an HTTP GET request and read the response:

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	resp, err := http.Get("https://example.com")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(string(body))
}
```

## HTTP POST Request

Performing an HTTP POST request with a JSON payload:

```go
package main

import (
	"bytes"
	"fmt"
	"net/http"
	"log"
)

func main() {
	jsonData := []byte(`{"key": "value"}`)
	resp, err := http.Post("https://example.com/api", "application/json", bytes.NewBuffer(jsonData))
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	fmt.Println("Response status:", resp.Status)
}
```

## Customizing HTTP Requests

For more control over HTTP requests, you can create a `http.Client` and customize your requests:

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"time"
)

func main() {
	client := &http.Client{
		Timeout: time.Second * 10,
	}

	req, err := http.NewRequest("GET", "https://example.com", nil)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("User-Agent", "Go-HTTP-Client")

	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(body))
}
```

## Best Practices

- Always close response bodies using `defer` to prevent resource leaks.
- Use context with requests for better control over request lifecycles and cancellation.
- Set appropriate timeouts to avoid hanging connections.
- Consider using `http.Client` with connections re-use configurations for multiple requests.
- Handle HTTP status codes explicitly, especially when the response might not be as expected.

## Common Pitfalls

- Ignoring response status codes, which give crucial information about the request outcome.
- Forgetting to set proper content-type headers, especially in POST requests.
- Not setting timeouts, which can lead to hanging requests indefinitely.
- Not handling errors from reading the response body correctly.
- Re-creating `http.Client` for high concurrency; prefer having a single client per program.

## Performance Tips

- Use a single `http.Client` for the entire application, as it manages connection pooling efficiently.
- For heavy concurrent workloads, utilize Goroutines for managing multiple HTTP requests.
- Use the `http.Transport` settings to fine-tune for better connection handling and performance.
- Measure and profile request performance to identify bottlenecks in large-scale applications.
- Avoid large payloads in requests by ensuring efficient data serialization and compression, if applicable.