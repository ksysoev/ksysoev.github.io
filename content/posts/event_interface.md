+++
title = 'Building a Event System in Go Without Union Types'
date = 2025-05-01T18:01:08+08:00
draft = false
tags = ['GoLang', 'Generics', 'Interface Design', 'Event Systems']
+++

One challenge in Go development is the absence of union types, which can make it tricky to design clean interfaces for event processing systems. When working with various event types with different payloads, we often resort to type assertions or empty interfaces, resulting in verbose and potentially error-prone code.

In this post, I'll demonstrate a pattern that creates a convenient, structured event interface without relying on union types.

## The Problem

When designing event-driven systems, we typically need to:
1. Identify the type of event
2. Parse its payload correctly
3. Route it to the appropriate handler

Without union types, achieving this cleanly can be challenging.

## A Structured Event Interface Solution

Let's start with a simple but effective approach:

```golang
type EventType uint8

const (
    EventTypeA EventType = iota
    EventTypeB
)

type Event interface {
    Type() EventType
    ParsePayload(dest any) error
}
```

Our `Event` interface provides two key methods:
- `Type()` - Returns an enum identifying the event type
- `ParsePayload()` - Handles the conversion of the event's data to the expected type

Let's implement this interface for a specific event type:

```golang
type EventA struct {
    PayloadA string
}

func (e EventA) Type() EventType {
    return EventTypeA
}

func (e EventA) ParsePayload(dest any) error {
    strPtr, ok := dest.(*string)
    if !ok {
        return fmt.Errorf("invalid payload receiver for EventA: expected *string")
    }
    
    *strPtr = e.PayloadA
    return nil
}

// Similarly for EventB
type EventB struct {
    PayloadB int
}

func (e EventB) Type() EventType {
    return EventTypeB
}

func (e EventB) ParsePayload(dest any) error {
    intPtr, ok := dest.(*int)
    if !ok {
        return fmt.Errorf("invalid payload receiver for EventB: expected *int")
    }
    
    *intPtr = e.PayloadB
    return nil
}
```

Now, we can process events without awkward type assertions in our business logic:

```golang
func ProcessEvent(event Event) error {
    switch event.Type() {
    case EventTypeA:
        var payload string
        if err := event.ParsePayload(&payload); err != nil {
            return fmt.Errorf("failed to parse payload: %w", err)
        }

        return handleEventA(payload)
    case EventTypeB:
        var payload int
        if err := event.ParsePayload(&payload); err != nil {
            return fmt.Errorf("failed to parse payload: %w", err)
        }

        return handleEventB(payload)
    default:
        return fmt.Errorf("unknown event type %v", event.Type())
    }
}
```

This pattern is particularly elegant because:

1. The type checking happens inside the event implementation, not in client code
2. The client code receives properly typed data without manual type assertions
3. Adding new event types doesn't require changing the interface, just implementing it for new types

## Why Enums Over Type Assertions?

You might wonder why we're using an enum approach (`EventType`) instead of Go's built-in type assertions. There are several compelling reasons:

1. **Explicit Error Handling**: With this approach, type mismatches result in explicit errors that can be handled gracefully at runtime, rather than panics.

2. **Explicit Contract**: The enum approach makes the set of possible event types explicit in your codebase. This creates clearer documentation and a more discoverable API compared to type assertions where supported types are only visible in implementation details.

3. **Serialization Friendly**: Enums can be easily serialized to/from various formats. When events cross system boundaries (databases, message queues, APIs), having a numeric or string identifier is significantly more practical than depending on Go's type system.

4. **More Flexible Type Hierarchies**: With enums, you can have multiple implementations of the same event type, or events that share payload structures but represent different business concepts. Type assertions limit you to a 1:1 mapping between Go types and event types.

5. **Testing Simplicity**: Mock implementations are easier to create when you can simply implement the interface and return the expected enum value, rather than needing to create entirely new types for testing.

## Benefits of This Approach

1. **Runtime Type Checking**: The `ParsePayload` method ensures appropriate type checking by returning an error if the wrong type is provided.
2. **Clean API**: Consumers of the event don't need to perform type assertions.
3. **Extensibility**: Adding new event types is straightforward—just implement the interface.
4. **Compatibility**: Works well with Go's type system without requiring union types.

## Practical Applications

This pattern is particularly useful for:
- Processing events from message queues
- Event-driven architectures
- Plugin systems
- Command pattern implementations
- Any situation where you need to process different types through a unified interface

## Conclusion

While Go doesn't provide union types, we can use interfaces and careful design to create elegant, structured event processing systems. This pattern leverages Go's strengths—interfaces and explicit error handling—to provide a clean solution that scales well as your event system grows.

By encapsulating the type checking within the event implementations, we keep our business logic clean and maintainable, focusing on what to do with events rather than how to safely extract their data.
