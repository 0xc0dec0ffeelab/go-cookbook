---
title: 'HTTPS and TLS Configuration'
description: 'Learn how to securely configure HTTPS and TLS in Go applications using the crypto/tls package.'
date: '2025-03-24'
category: 'Security'
---

In Go, configuring HTTPS and TLS for securing communications in applications can be efficiently handled using the `crypto/tls` package. This snippet demonstrates how to set up HTTPS servers with TLS.

## Basic HTTPS Server

Here's how you can set up a simple HTTPS server in Go:

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, HTTPS!")
}

func main() {
	http.HandleFunc("/", handler)
	err := http.ListenAndServeTLS(":443", "server.crt", "server.key", nil)
	if err != nil {
		panic(err)
	}
}
```

In this example, ensure you have a valid `server.crt` and `server.key` file for your server.

## Configuring TLS with Custom Settings

For more advanced configurations, you might want to customize the TLS settings:

```go
package main

import (
	"crypto/tls"
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, secure world!")
}

func main() {
	tlsConfig := &tls.Config{
		MinVersion: tls.VersionTLS12,
	}

	server := &http.Server{
		Addr:      ":443",
		Handler:   http.HandlerFunc(handler),
		TLSConfig: tlsConfig,
	}

	err := server.ListenAndServeTLS("server.crt", "server.key")
	if err != nil {
		panic(err)
	}
}
```

This example sets the minimum TLS version to TLS 1.2, enhancing security by avoiding older, less secure versions.

## HTTPS Client with Custom TLS Config

You can also configure a custom HTTPS client with specific TLS settings:

```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io"
	"net/http"
	"os"
)

func main() {
	// Load client cert.
	cert, err := tls.LoadX509KeyPair("client.crt", "client.key")
	if err != nil {
		panic(err)
	}

	// Load CA cert.
	caCert, err := os.ReadFile("ca.crt")
	if err != nil {
		panic(err)
	}

	caCertPool := x509.NewCertPool()
	caCertPool.AppendCertsFromPEM(caCert)

	// TLS Config.
	tlsConfig := &tls.Config{
		Certificates: []tls.Certificate{cert},
		RootCAs:      caCertPool,
		MinVersion:   tls.VersionTLS12,
	}
	tlsConfig.BuildNameToCertificate()

	// Create HTTPS client.
	client := &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: tlsConfig,
		},
	}

	resp, err := client.Get("https://myserver.com")
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(body))
}
```

This example demonstrates how to configure an HTTPS client to include client certificates and a custom CA pool.

## Best Practices

- Always use the latest version of the TLS protocol that your platform supports.
- Use `tls.Config` to specify allowed cipher suites and enforce strong security policies.
- Regularly update your certificates and keys to prevent them from expiring.

## Common Pitfalls

- Forgetting to properly implement error checking, especially when dealing with network code.
- Not loading certificates correctly, leading to runtime errors or security vulnerabilities.
- Leaving out `tls.Config` leads to default settings, which may not be secure enough for your needs.

## Performance Tips

- Reuse HTTP/TLS connections to minimize handshake overhead using the `http.Client` with keep-alive turned on.
- Set appropriate timeouts for your server and TLS connections to avoid resource exhaustion.
- Profile your TLS configuration on load to ensure its performance meets your application needs.