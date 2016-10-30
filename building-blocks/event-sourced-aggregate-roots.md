# Event-sourced aggregate roots

An aggregate root must conform to the following behaviour to implement the event-sourcing pattern.

- Each public function must accept a command, record any raised domain events, and return successfully or with an error.
- Its internal state may only be modified by applying a raised domain event to its current state.
- Its internal state can be rebuilt from an initial empty state by replaying all domain events in the order they were raised.

The command may be a single object or a list of arguments to the function. Current state of an aggregate root is a left fold of its raised domain events.

In Elixir, the aggregate root's state is required as an argument to each command function. The standard Elixir pattern is to indicate success or failure of a function by returning `:ok` or `:error` atoms. So an `{:ok, state}` tuple containing the modified state, including any raised domain events, is returned on success. An `{:error, reason}` tuple is returned on failure. Pattern matching can be used on the return value to determine whether the command succeeded.

## Building an aggregate root

We can build an Elixir module that implements event-sourcing using the requirements defined above. An aggregate root must track its internal state and any raised domain events. It will also require a unique identifier.

```elixir
defmodule AggregateRoot do
  defstruct uuid: nil, pending_events: [], state: nil

  def new(uuid) do
    %AggregateRoot{uuid: uuid, state: %{}}
  end

  defp update(%AggregateRoot{uuid: uuid, pending_events: pending_events, state: state} = aggregate, event) do
    state = AggregateRoot.apply(state, event)

    %AggregateRoot{aggregate |
      pending_events: pending_events ++ [event],
      state: state,
    }
  end
end
```

The `update/2` function is used to record a raised domain event, within the `pending_event` list, and call the aggregate root's `apply\2` function to modify internal state. Pattern matching on the event type is used to ensure the appropriate apply method is called.

## Example bank account aggregate root

Expanding the aggregate root module into a concrete bank account example. The internal state is defined as a struct within a `State` module.

Domain events are also structs defined within their own module. This allows simple pattern matching for event handling, such as in the aggregate's `apply/2` functions. It is worth remembering that domain events are the contracts that are recorded within the immutable event stream for the aggregate root. A recorded domain event cannot be changed; history cannot be altered.

This example provides a single public API function to open an account (`open_account/3`). A guard clause is used to protect the account from being opened with an invalid initial balance. This demonstrates how business rule violations can be handled with `ok` or `:error` denoted tuples.

```elixir
defmodule BankAccount do
  defstruct uuid: nil, pending_events: [], state: nil

  defmodule State do
    defstruct account_number: nil, balance: nil
  end

  defmodule Events do
    defmodule BankAccountOpened do
      defstruct account_number: nil, initial_balance: nil
    end
   end
 end

  def new(uuid) do
    %BankAccount{uuid: uuid, state: %BankAccount.State{}}
  end

  # public API

  def open_account(%BankAccount{} = account, account_number, initial_balance) when initial_balance > 0 do
    account =
      account
      |> update(%BankAccountOpened{account_number: account_number, initial_balance: initial_balance})

    {:ok, account}
  end

  def open_account(%BankAccount{} = account, account_number, initial_balance) when initial_balance <= 0 do
    {:error, :initial_balance_must_be_above_zero}
  end

  defp update(%BankAccount{uuid: uuid, pending_events: pending_events, state: state} = aggregate, event) do
    state = BankAccount.apply(state, event)

    %BankAccount{aggregate |
      pending_events: pending_events ++ [event],
      state: state,
    }
  end

  # state mutators

  def apply(%BankAccount.State{} = state, %BankAccountOpened{} = account_opened) do
    %BankAccount.State{state |
      account_number: account_opened.account_number,
      balance: account_opened.initial_balance,
    }
  end
end
```

### Using the aggregate root

```elixir
with account <- BankAccount.new("123"),
  {:ok, account} <- BankAccount.open_account(account, "ACC123", 100),
  {:ok, account} <- BankAccount.deposit(account, 50),
do: account
```

### Implementing an aggregate root with `eventsourced`

* [eventsourced](https://github.com/slashdotdash/eventsourced)
