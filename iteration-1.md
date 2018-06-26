### Simple sequential processing

First, we must make some assumption on the API:

```
interface EventStream {
     void consume(EventConsumer consumer);
 }
 
@FunctionalInterface
interface EventConsumer {
    Event consume(Event event);
}
 
class Event {
    private final Instant created;
    private final int clientId;
    private final UUID uuid;
}
```

A typical push-based API, similar to JMS. Note that `EventConsumer` is blocking, meaning new `Event` won't be delivered
until the previous one was consumed. The naive implementation is to attach simple consumer that takes about 10
milliseconds to complete.

In order to verify that requirements are met, we must plug in a few metrics. The most important one is latency of the
message consumption, measured as a time between message creation and start of processing.

Result:
- latency keeps growing infinetly
- `SimpleConsumer` can handle ~100 messages/second, we need ~1000