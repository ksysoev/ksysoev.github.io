+++
title = 'Building a Type-Safe Event System in Go Without Union Types'
date = 2025-05-01T18:01:08+08:00
draft = true
tags = ['GoLang', 'Generics', 'Interface Design', 'Event Systems']
+++

One challenge in Go development is the absence of union types, which can make it tricky to design clean interfaces for event processing systems. When working with various event types with different payloads, we often resort to type assertions or empty interfaces, resulting in verbose and potentially unsafe code.

In this post, I'll demonstrate a pattern that creates a convenient, type-safe event interface without relying on union types.

## The Problem

When designing event-driven systems, we typically need to:
1. Identify the type of event
2. Parse its payload correctly
3. Route it to the appropriate handler

Without union types, achieving this cleanly can be challenging.

## A Type-Safe Event Interface Solution

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
- `ParsePayload()` - Handles the type-safe conversion of the event's data to the expected type

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

## Benefits of This Approach

1. **Type Safety**: The `ParsePayload` method ensures type safety by returning an error if the wrong type is provided.
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

While Go doesn't provide union types, we can use interfaces and careful design to create elegant, type-safe event processing systems. This pattern leverages Go's strengths—interfaces and type safety—to provide a clean solution that scales well as your event system grows.

By encapsulating the type assertions within the event implementations, we keep our business logic clean and maintainable, focusing on what to do with events rather than how to safely extract their data.
