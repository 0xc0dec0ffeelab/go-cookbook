---
title: 'Working with JWT in Go'
description: 'Learn how to generate, parse, and validate JSON Web Tokens (JWTs) in Go using the popular go-jose.v2 library'
date: '2025-03-24'
category: 'Security'
---

JSON Web Tokens (JWTs) are used extensively for securing APIs and managing user sessions. In Go, the `go-jose.v2` library is a popular choice for handling JWTs due to its comprehensive support for JSON Web Encryption (JWE) and JSON Web Signature (JWS).

## Generating a JWT

Here's a basic example illustrating how to generate a JWT:

```go
package main

import (
	"fmt"
	"log"
	"time"

	jose "gopkg.in/square/go-jose.v2"
	jwt "gopkg.in/square/go-jose.v2/jwt"
)

func main() {
	// Define a shared key.
	key := []byte("my-secret-key")

	// Create a signer.
	signer, err := jose.NewSigner(jose.SigningKey{Algorithm: jose.HS256, Key: key}, nil)
	if err != nil {
		log.Fatalf("Failed to create signer: %v", err)
	}

	// Define claims.
	cl := jwt.Claims{
		Issuer:    "example.com",
		Audience:  jwt.Audience{"example-audience"},
		Expiry:    jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
		IssuedAt:  jwt.NewNumericDate(time.Now()),
	}

	// Create and sign the JWT.
	raw, err := jwt.Signed(signer).Claims(cl).CompactSerialize()
	if err != nil {
		log.Fatalf("Failed to create JWT: %v", err)
	}

	fmt.Printf("JWT: %s\n", raw)
}
```

## Parsing and Validating a JWT

Below is an example of how to parse and validate a JWT:

```go
package main

import (
	"fmt"
	"log"

	jose "gopkg.in/square/go-jose.v2"
	jwt "gopkg.in/square/go-jose.v2/jwt"
)

func main() {
	// Simulate a JWT string received from somewhere.
	rawJWT := "your.jwt.string.here"

	// The same shared key used for signing.
	key := []byte("my-secret-key")

	// Parse the JWT.
	token, err := jwt.ParseSigned(rawJWT)
	if err != nil {
		log.Fatalf("Invalid JWT: %v", err)
	}

	// Declare a variable to store claims.
	claims := jwt.Claims{}

	// Validate the JWT signature and extract claims.
	if err := token.Claims(key, &claims); err != nil {
		log.Fatalf("JWT claims validation failed: %v", err)
	}

	// Additional validations can be performed here.
	fmt.Printf("JWT Claims: %+v\n", claims)
}
```

## Best Practices

- Use strong and unique secret keys for signing tokens, update periodically to ensure security.
- Store tokens securely, preferably in secure cookies with flags like `HttpOnly` and `Secure`.
- Set appropriate expiry times for your JWTs. Shorter validity periods reduce the impact of a leaked token.
- Always validate the token claims, including audience and issuer, to ensure they conform to expected values.

## Common Pitfalls

- Failing to validate tokens properly can open up security vulnerabilities.
- Hardcoding secret keys directly in the code can lead to security issues if your repositories are compromised; consider using environment variables or a secrets management service.
- Not setting a short expiry for each token can increase the risk window in case of a token leak.
- Overlooking the use of HTTPS when transmitting JWTs could lead to token exposure.

## Performance Tips

- Consider caching validated tokens to avoid unnecessary recomputation of the same validation logic.
- Large payload data in tokens can increase the size of the token considerably, impacting performance, so keep tokens small.
- Use goroutines for parallel token processing in high-throughput applications to improve performance without blocking operations.