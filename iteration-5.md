### Deduplication and idempotency

In distributed environment it's quite common to receive duplicated events when your producer has at least once
guarantees. One way is to attach globally unique identifier (UUID) to every message and make sure on the consumer side
that messages with the same identifier aren't processed twice.

The most straightforward solution under our requirements is to simply store all seen UUIDs and verify on arrival that
received UUID was never seen before. Using `ConcurrentHashMap<UUID, UUID>` as-is will lead to memory leak, as we will
keep accumulating more and more IDs over time. That's why we only look for duplicates in the last 10 seconds.

Let's use Guava's Cache<UUID, UUID> with declarative eviction policy to do the work for us. 


Once again to be safe on production there are at least two metrics that might become useful:
- cache size 
- number of duplicates discovered 

Finally we have all the pieces to build our solution. The idea is to compose pipeline from `EventConsumer` instances
wrapping each other:
- apply `IgnoreDuplicates` to reject duplicates
- apply `SmartPool` that always pins given `clientId` to the same thread and executes next stage in that thread
- finally `SimpleConsumer` is invoked that does the real business logic

Optional: place `FailOnConcurrentModification` step between `SmartPool` and `SimpleConsumer` for extra safety
(concurrent modification shouldn't happen by design).