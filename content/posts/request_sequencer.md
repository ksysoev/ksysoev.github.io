+++
title = 'Simplifying Concurrent Code with Request Sequencing in Go'
date = 2025-05-29T22:43:23+08:00
draft = false
tags = ['GoLang', 'concurrency', 'middleware']
+++

In the world of concurrent programming, managing simultaneous requests for the same resource or user can quickly become a nightmare of race conditions, complex locking mechanisms, and hard-to-debug issues. What if there was a way to eliminate these concurrency concerns entirely for specific use cases? Enter request sequencing—a pattern that ensures requests for the same entity are processed one at a time, dramatically simplifying your codebase while maintaining system responsiveness.

This article explores how to implement a request sequencer in Go, transforming complex concurrent operations into straightforward sequential processing. By the end, you'll understand when and how to apply this pattern to build more maintainable and reliable applications.

## The Challenge of Concurrent Requests

Consider a typical chat bot application where multiple messages from the same user might arrive simultaneously. Without proper coordination, these messages could be processed concurrently, leading to several problems:

- **Race conditions** when accessing user state or session data
- **Complex synchronization logic** required throughout the codebase  
- **Difficult-to-reproduce bugs** that only surface under specific timing conditions
- **Inconsistent user experience** when messages are processed out of order

Traditional solutions involve adding mutexes, atomic operations, or complex state management throughout your application. However, these approaches increase code complexity and create opportunities for deadlocks and performance bottlenecks.

## Understanding Request Sequencing

Request sequencing offers an elegant alternative: instead of making your entire application thread-safe, you ensure that requests for the same entity (user, session, resource) are processed sequentially. This approach provides several key benefits:

- **Simplified application logic** - no need for complex synchronization
- **Easier testing and debugging** - predictable execution order
- **Reduced cognitive overhead** - developers don't need to think about concurrent access
- **Maintained system throughput** - different users can still be processed concurrently

The core idea is to create per-entity queues that serialize requests while allowing the system to handle multiple entities in parallel.

## Implementation Deep Dive

Let's examine a simplified implementation of a request sequencer:

```golang
func WithRequestSequencer() Middleware {
	userQueues := make(map[int64]chan struct{})
	mu := sync.Mutex{}

	return func(next Handler) Handler {
		return HandlerFunc(func(ctx context.Context, message *Message) error {
			userID := message.UserID

			// Get or create a queue for this user
			mu.Lock()
			queue, exists := userQueues[userID]
			if !exists {
				queue = make(chan struct{}, 1)
				queue <- struct{}{} // Ready to process
				userQueues[userID] = queue
			}
			mu.Unlock()

			// Wait for our turn or context cancellation
			select {
			case <-queue:
				// Release the queue when done
				defer func() {
					mu.Lock()
					select {
					case queue <- struct{}{}:
						// Next request can proceed
					default:
						// Clean up if no one is waiting
						delete(userQueues, userID)
					}
					mu.Unlock()
				}()

				// Process the request
				return next.Handle(ctx, message)
				
			case <-ctx.Done():
				return ctx.Err()
			}
		})
	}
}
```
```

### How It Works

The implementation uses Go's channels as semaphores to create per-user queues:

**1. Per-User Queues**: Each user gets a buffered channel (`chan struct{}`) with capacity 1, acting as a binary semaphore to ensure only one request per user is processed at a time.

**2. Queue Creation**: When a user's first request arrives, we create a new channel and immediately send a token into it, making it ready for processing.

**3. Request Serialization**: Subsequent requests for the same user must read from the channel first. This blocks until the previous request completes and releases its token.

**4. Token Release**: The deferred function ensures the token is returned to the channel when processing completes, allowing the next queued request to proceed.

**5. Cleanup**: If no other requests are waiting (the channel send fails), we remove the queue from the map to prevent memory leaks.

### When to Use Request Sequencing

Request sequencing is ideal when:
- Entity-specific operations need to maintain consistency
- The cost of complex synchronization outweighs the latency trade-off
- Operations for the same entity are relatively quick
- Different entities can be processed independently

Avoid this pattern when:
- High throughput for individual entities is critical
- Operations are long-running or involve significant I/O
- The system requires true parallel processing for performance

## Conclusion

Request sequencing offers a powerful way to simplify concurrent applications by trading some latency for significantly reduced complexity. For applications like chat bots, user session management, and transaction processing, this pattern can eliminate entire classes of concurrency bugs while making your codebase more maintainable.

The key insight is that often we don't need full concurrency—we just need the appearance of it. By processing different entities concurrently while serializing operations for the same entity, we can achieve both system responsiveness and code simplicity.

Consider implementing request sequencing in your next project where entity-specific operations are causing concurrency headaches. Your future self (and your team) will thank you for the cleaner, more predictable codebase.
