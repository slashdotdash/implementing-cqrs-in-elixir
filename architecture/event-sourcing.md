# Event-sourcing

## Benefits

- Describe system activity over time using a rich, domain-specific language.
- Events are immutable.
- Events and their schema provide the ideal integration point for other systems.
- Provide a perfect audit log.
- Allow optimised read-only and highly specialised query models to be built.
- Support temporal queries.
- Allow migration of read-only data between persistence technologies by replaying and projecting all events.

## Costs

- Events are immutable.
- Events provide a history of your poor design decisions.
- Event versioning.
- Demands a richer understanding of the domain being modelled over basic CRUD activities.
- Slows iteration of the domain model as production events must be supported forever.
- Eventual consistency.

## Concept

```
f(state, event) => state
```
