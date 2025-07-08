---
title: 'Linked Lists'
description: 'Explore how to implement and use linked lists in Go with code examples and best practices'
date: '2025-03-24'
category: 'Collections'
---

Linked lists are a fundamental data structure, consisting of nodes where each node has a value and a pointer to the next node. While Go does not have a built-in linked list implementation per se, the `container/list` package provides a doubly linked list structure.

## Basic Linked List Operations

Here's how you can use the `container/list` package to perform basic linked list operations:

```go
package main

import (
	"container/list"
	"fmt"
)

func main() {
	// Create a new linked list.
	l := list.New()

	// Add elements to the list.
	l.PushBack("golang")
	l.PushBack("docker")
	l.PushFront("kubernetes")

	// Iterate through the list and print its elements.
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Println(e.Value)
	}

	// Remove an element from the list.
	l.Remove(l.Front())

	// Iterate again.
	fmt.Println("After removing the first element:")
	for e := l.Front(); e != nil; e = e.Next() {
		fmt.Println(e.Value)
	}
}
```

## Custom Singly Linked List Implementation

For more control, you might want to implement your own singly linked list. Below is a simple example:

```go
package main

import "fmt"

// Node represents a node in the linked list.
type Node struct {
	value int
	next  *Node
}

// LinkedList represents the linked list.
type LinkedList struct {
	head *Node
}

// Insert adds a new node to the end of the list.
func (ll *LinkedList) Insert(value int) {
	newNode := &Node{value: value}
	if ll.head == nil {
		ll.head = newNode
	} else {
		current := ll.head
		for current.next != nil {
			current = current.next
		}
		current.next = newNode
	}
}

// Print outputs the elements of the list.
func (ll *LinkedList) Print() {
	current := ll.head
	for current != nil {
		fmt.Print(current.value, " ")
		current = current.next
	}
	fmt.Println()
}

func main() {
	ll := &LinkedList{}
	ll.Insert(42)
	ll.Insert(73)
	ll.Insert(91)

	ll.Print() // Outputs: 42 73 91
}
```

## Best Practices

- Always verify for nil pointers when navigating through custom linked list implementations to prevent runtime errors.

## Common Pitfalls

- Memory management: Ensure nodes are properly cleaned up when removed, particularly in circular linked lists.
- Incorrect pointer updates during insertions and deletions can disrupt the list structure.
- Failing to address edge cases such as empty lists, single-node lists, or operations on nil lists.
- Opting for linked lists when slices would suffice; Go's slices are typically more efficient for many scenarios.
- Traversal issues like infinite loops due to incorrect termination conditions or circular references.

## Performance Tips

- Linked lists can be more efficient than slices for frequent insertions and deletions at arbitrary positions.
- Opt for doubly linked lists if you require bidirectional traversal or frequent middle deletions.
- Utilize sync.Pool for node allocation in high-performance contexts to minimize garbage collection load.
- Analyze your application to assess if the pointer chasing overhead in linked lists justifies the flexibility over slice operations.
- Implement interfaces to generalize list operations, facilitating easier transitions between different implementations based on performance needs.