### Naive thread pool

The most obvious solution is to invoke `EventConsumer` from multiple threads:
- instead of changing implementation of `SimpleConsumer`, use decorator pattern to wrap `SimpleConsumer.consume(...)`
with another implementation if `EventConsumer`, that adds concurrency.
- use `ExecutorService` (fixed thread pool = 10) to call `SimpleConsumer.consume(...)` asynchronously

Note that such design promotes:
- loose coupling: various `EventConsumer` don't know about each other and can be combined freely
- single responsibility: each does one job and delegates to the next component
- open/closed principle: we can change the behavior of the system without modifying existing implementations.

Result
- delay is still growing, though it is growing slower than previously
- ever-growing size of the queue holding pending tasks hurst the latency
- changing thread pool size from 10 to 20 finally yields decent results

 Now things look better, however we still didn't address duplicates and protecting from concurrent modification of
 events for the same clientId.