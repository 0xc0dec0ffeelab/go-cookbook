---
title: 'Wrapping Errors'
description: 'Learn how to effectively wrap errors in Go using error handling features'
date: '2025-03-24'
category: 'Error Handling'
---

Go's error handling typically involves returning errors from functions. Wrapping errors adds additional context, making debugging easier. This snippet demonstrates how to wrap errors using Go’s standard library features.

## Basic Error Wrapping

The `fmt` package provides basic capabilities to wrap errors with additional context:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := someFunction()
	if err != nil {
		fmt.Println(err)
	}
}

func anotherFunction() error {
	return errors.New("an error occurred")
}

func someFunction() error {
	err := anotherFunction()
	if err != nil {
		return fmt.Errorf("someFunction failed: %w", err)
	}
	return nil
}
```

## Using the `errors` Package for Wrapping

Go 1.13 introduced an enhancement to the `errors` package for error wrapping:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := doSomething()
	if err != nil {
		fmt.Println(err)
	}
}

func faultyFunction() error {
	return errors.New("a critical failure")
}

func doSomething() error {
	err := faultyFunction()
	if err != nil {
		return fmt.Errorf("doSomething encountered an error: %w", err)
	}
	return nil
}
```

## Unwrapping Errors

To extract the original error after wrapping, use `errors.Unwrap` or `errors.Is` and `errors.As`:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := performTask()
	if err != nil {
		if errors.Is(err, ErrSpecific) {
			fmt.Println("Found specific error:", ErrSpecific)
		} else {
			fmt.Println(err)
		}
	}
}

var ErrSpecific = errors.New("specific error")

func taskFunction() error {
	return ErrSpecific
}

func performTask() error {
	err := taskFunction()
	if err != nil {
		return fmt.Errorf("performTask failed: %w", err)
	}
	return nil
}
```

## Joining Multiple Errors

Go 1.20 introduced `errors.Join` to combine multiple errors into a single error value. This is useful when multiple operations fail simultaneously:

```go
package main

import (
	"errors"
	"fmt"
	"io"
	"os"
)

func main() {
	err := processFiles()
	if err != nil {
		fmt.Println("Error processing files:", err)
	}
}

func validateInput() error {
	return errors.New("invalid input format")
}

func checkPermissions() error {
	return os.ErrPermission
}

func readConfigFile() error {
	return io.EOF
}

func processFiles() error {
	var errs []error
	
	if err := validateInput(); err != nil {
		errs = append(errs, fmt.Errorf("validation failed: %w", err))
	}
	
	if err := checkPermissions(); err != nil {
		errs = append(errs, fmt.Errorf("permission check failed: %w", err))
	}
	
	if err := readConfigFile(); err != nil {
		errs = append(errs, fmt.Errorf("config read failed: %w", err))
	}
	
	if len(errs) > 0 {
		return errors.Join(errs...)
	}
	
	return nil
}
```

## Combining Joining with Wrapping

You can effectively combine `errors.Join` with traditional error wrapping for comprehensive error handling:

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := deployApplication()
	if err != nil {
		fmt.Println("Deployment failed:", err)
	}
}

func startDatabase() error {
	return errors.New("database connection timeout")
}

func loadConfiguration() error {
	return errors.New("missing required config key")
}

func initializeCache() error {
	return errors.New("cache server unreachable")
}

func deployApplication() error {
	var deploymentErrors []error
	
	if err := startDatabase(); err != nil {
		deploymentErrors = append(deploymentErrors, err)
	}
	
	if err := loadConfiguration(); err != nil {
		deploymentErrors = append(deploymentErrors, err)
	}
	
	if err := initializeCache(); err != nil {
		deploymentErrors = append(deploymentErrors, err)
	}
	
	if len(deploymentErrors) > 0 {
		joinedErr := errors.Join(deploymentErrors...)
		return fmt.Errorf("application deployment failed with %d errors: %w", 
			len(deploymentErrors), joinedErr)
	}
	
	return nil
}
```

## Best Practices

- **Use `%w` for Wrapping**: Wrap errors using `%w` in `fmt.Errorf` to allow unwrapping.
- **Add Context**: When wrapping, ensure additional context is meaningful to aid debugging.
- **Maintain Error Chains**: Keep original errors in the chain for complete traceability.

## Common Pitfalls

- **Overuse**: Avoid wrapping errors if it does not add significant value—overwrapping can clutter logs.
- **Mixing nil Values with Join**: Calling `errors.Join` with nil values is safe, but be mindful that it filters them out automatically.
- **Large Error Collections**: Joining too many errors can create unwieldy error messages that are difficult to parse.

## Performance Tips

- **Avoid Excessive Wrapping**: Multiple unnecessary wraps can add overhead without providing additional benefits.
- **Use Built-in Support**: Rely on Go's built-in error package's features for efficiency rather than implementing custom solutions.
- **Profile Critical Sections**: Ensure error handling doesn't become a bottleneck in performance-critical applications by profiling.
- **Preallocate Error Slices**: When collecting errors for joining, preallocate slices with known capacity to reduce memory allocations.
- **Consider Error Frequency**: Use `errors.Join` judiciously in high-frequency code paths as it creates new error values and can impact performance.