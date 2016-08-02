# Commands and domain events

## Commands

Commands instruct an application to do something. They are named in the imperative: register account; transfer funds; mark fraudulent activity.

A command **must have** exactly one registered handler. It is an error to have a command without a corresponding handler, or with multiple handlers.

## Domain events

> A domain object that defines an event; something that has happened. A domain event is an event that a domain expert would care about.

Domain events indicate that something of importance has occurred,  within the context of an aggregate root. They are named in the past tense: account registered; funds transferred; fraudulent activity detected.

A domain event may have zero, one or more registered handlers. It is acceptable to have no handler for a domain, outside of the aggregate root that raised the event.
