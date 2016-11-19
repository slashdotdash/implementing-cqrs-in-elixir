# Implementing CQRS in Elixir

An introduction to implementing the Command Query Responsibility Segregation (CQRS) architecture in Elixir applications.

This guide will help you to build your own Elixir applications following CQRS/ES principles.

- Building *pure* functional, event-sourced domain models.
- Using [eventstore](https://github.com/slashdotdash/eventstore) to persist event streams to a PostgreSQL database.
- Command registration and dispatch; delegation to aggregate roots; event handling; and long running process managers using [commanded](https://github.com/slashdotdash/commanded).

The reader should be familiar with the [Elixir programming language](http://elixir-lang.org/) and the basic principles of [domain-driven design](https://en.wikipedia.org/wiki/Domain-driven_design).
