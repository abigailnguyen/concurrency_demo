### Obscure locking

Let's start from avoiding concurrent processing of events for the same `clientId`. If two events come very quickly
one after another, both related to the same `clientId`, `NaivePool` will pick both of them and start processing them
concurrently.
To prevent that let's have a `Lock` per `clientId`:
- option 1: FailOnConcurrentModification drops the message
- option 2: WaitOnConcurrentModification waits ~1 second to obtain the lock before dropping the message

Not only this code is really convoluted, but probably also broken in many subtle ways. For example what if two events
for the same clientId came almost exactly at the same time, but obviously one was first? Both events will ask for `Lock`
at the same time and we have no guarantee which event will obtain a non-fair Lock first, possibly consuming events
out of order. There must be a better way...