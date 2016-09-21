# Implementing CQRS in Elixir

An introduction to implementing the Command Query Responsibility Segregation (CQRS) architecture in Elixir applications.

This guide describes how the following libraries have been designed. Allowing you to build your own applications using these dependencies.

- Building functional, event-sourced domain models using [eventsourced](https://github.com/slashdotdash/eventsourced).
- Using the [eventstore](https://github.com/slashdotdash/eventstore) to persist events to a PostgreSQL database.
- Command registration and dispatch; delegation to aggregate roots; event handling; and long running process managers using [commanded](https://github.com/slashdotdash/commanded).

The reader should be familiar with the [Elixir programming language](http://elixir-lang.org/) and the basic principles of [domain-driven design](https://en.wikipedia.org/wiki/Domain-driven_design).
