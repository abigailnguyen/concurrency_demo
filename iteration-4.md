### Dedicated threads / smart pool

Let's take a step back.
How do you ensure things aren't happening concurrently => well, just use one thread!
That's what we did in the very beginning but the throughput was unsatisfactory.
But we don't care about concurrency for different `clientId`s, we just have to make sure events with the same `clientId`
are always processed by the same thread!

A thread per `clientId`... possibly creates thousands of threads with, each idle most of the time... NOT GOOD.

Good compromise - fixed size pool of threads, each responsible for a well-known subset of `clientId`s. This way two
different clientIds may end up on the same thread but the same clientId will always be handled by the same thread.

Crucial part:
```
int threadIdx = event.getClientId() % threadPools.size();
ExecutorService executor = threadPools.get(threadIdx);
```

At this point no locking is necessary and sequential invocation is guaranteed because events for the same client are
always executed by the same thread.

To be extra safe let's plug in metrics for average queue size in each and every thread pool. 