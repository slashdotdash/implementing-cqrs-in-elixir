# Event-sourced aggregate roots

* [eventsourced](https://github.com/slashdotdash/eventsourced)

An aggregate root that implements the event-sourcing pattern requires a list of raised domain events and functions to rebuild its state from an initial empty state to its current from these events.

Current state is a left fold of its raised events.
