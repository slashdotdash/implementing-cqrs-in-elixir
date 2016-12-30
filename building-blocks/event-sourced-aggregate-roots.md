# Event-sourced aggregate roots

An aggregate root must conform to the following behaviour to implement the event sourcing pattern.

- Each public function must accept a command and return any resultant domain events, or raise an error.
- Its internal state may only be modified by applying a domain event to its current state.
- Its internal state can be rebuilt from an initial empty state by replaying all domain events in the order they were raised.

The command may be a single object or a list of arguments to the function. Current state of an aggregate root is a left fold of its raised domain events applied to its empty state.

## Implementing an aggregate root in Elixir

We can build an Elixir module that implements event sourcing using the requirements defined above.

```elixir
defmodule AggregateRoot do
  # aggregate root state  
  defstruct [
    uuid: nil,
    name: nil,
  ]

  # domain event
  defmodule CreatedEvent do
    defstruct [
      uuid: nil,
      name: nil,
    ]
  end

  # public command API
  def create(%AggregateRoot{}, uuid, name) do
    %CreatedEvent{
      uuid: uuid,
      name: name,
    }
  end

  # state mutator
  def apply(%AggregateRoot{} = aggregate, %CreatedEvent{uuid: uuid, name: name}) do
    %AggregateRoot{aggregate |
      uuid: uuid,
      name: name,
    }
  end
end
```

Each command function requires the aggregate root's state and the command, or command arguments. The return value may be none, one, or many resultant domain events.

The standard Elixir pattern is to indicate success or failure of a function by returning `:ok` or `:error` atoms. So you can choose to return an `{:ok, events}` tuple with the resultant domain events on success and return an `{:error, reason}` tuple on failure.

## Example bank account aggregate root

Expanding the aggregate root module into a concrete bank account example. The internal state is defined as a struct containing the account number, balance, and account state.

Domain events are also defined as structs within their own modules. This allows simple pattern matching for state mutating in the aggregate's `apply/2` functions.

It is worth remembering that domain events are the contracts that are recorded within the immutable event stream for the aggregate root. A recorded domain event cannot be changed; history cannot be altered.

This example provides a three public API functions.

1. To open an account: `open_account/2`.
2. To deposit money: `deposit/2`.
3. To withdraw money: `withdraw/2`.

A guard clause is used to prevent the account from being opened with an invalid initial balance. This protects the aggregate from violating the business rule that an account must be opened with a positive balance.

```elixir
defmodule Commanded.ExampleDomain.BankAccount do
  defstruct [
    account_number: nil,
    balance: 0,
    state: nil,
  ]

  alias Commanded.ExampleDomain.BankAccount

  defmodule Commands do
    defmodule OpenAccount,        do: defstruct [:account_number, :initial_balance]
    defmodule DepositMoney,       do: defstruct [:account_number, :transfer_uuid, :amount]
    defmodule WithdrawMoney,      do: defstruct [:account_number, :transfer_uuid, :amount]
    defmodule CloseAccount,       do: defstruct [:account_number]
  end

  defmodule Events do
    defmodule BankAccountOpened,  do: defstruct [:account_number, :initial_balance]
    defmodule MoneyDeposited,     do: defstruct [:account_number, :transfer_uuid, :amount, :balance]
    defmodule MoneyWithdrawn,     do: defstruct [:account_number, :transfer_uuid, :amount, :balance]
    defmodule AccountOverdrawn,   do: defstruct [:account_number, :balance]
    defmodule BankAccountClosed,  do: defstruct [:account_number]
  end

  alias Commands.{OpenAccount,DepositMoney,WithdrawMoney,CloseAccount}
  alias Events.{BankAccountOpened,MoneyDeposited,MoneyWithdrawn,AccountOverdrawn,BankAccountClosed}

  def open_account(%BankAccount{state: nil}, %OpenAccount{account_number: account_number, initial_balance: initial_balance})
    when is_number(initial_balance) and initial_balance > 0
  do
    %BankAccountOpened{account_number: account_number, initial_balance: initial_balance}
  end

  def deposit(%BankAccount{state: :active, balance: balance}, %DepositMoney{account_number: account_number, transfer_uuid: transfer_uuid, amount: amount})
    when is_number(amount) and amount > 0
  do
    balance = balance + amount

    %MoneyDeposited{account_number: account_number, transfer_uuid: transfer_uuid, amount: amount, balance: balance}
  end

  def withdraw(%BankAccount{state: :active, balance: balance}, %WithdrawMoney{account_number: account_number, transfer_uuid: transfer_uuid, amount: amount})
    when is_number(amount) and amount > 0
  do
    case balance - amount do
      balance when balance < 0 ->
        [
          %MoneyWithdrawn{account_number: account_number, transfer_uuid: transfer_uuid, amount: amount, balance: balance},
          %AccountOverdrawn{account_number: account_number, balance: balance},
        ]
      balance ->
        %MoneyWithdrawn{account_number: account_number, transfer_uuid: transfer_uuid, amount: amount, balance: balance}
    end
  end

  def close_account(%BankAccount{state: :active}, %CloseAccount{account_number: account_number}) do
    %BankAccountClosed{account_number: account_number}
  end

  # state mutators

  def apply(%BankAccount{} = state, %BankAccountOpened{account_number: account_number, initial_balance: initial_balance}) do
    %BankAccount{state |
      account_number: account_number,
      balance: initial_balance,
      state: :active,
    }
  end

  def apply(%BankAccount{} = state, %MoneyDeposited{balance: balance}), do: %BankAccount{state | balance: balance}

  def apply(%BankAccount{} = state, %MoneyWithdrawn{balance: balance}), do: %BankAccount{state | balance: balance}

  def apply(%BankAccount{} = state, %AccountOverdrawn{}), do: state

  def apply(%BankAccount{} = state, %BankAccountClosed{}) do
    %BankAccount{state |
      state: :closed,
    }
  end
end
```

### Using the aggregate root

```elixir
# initial empty account state
account = %BankAccount{}

# opening the account returns an account opened event
account_opened = BankAccount.open_account(account, %OpenAccount{account_number: "ACC123", initial_balance: 100})

# mutate the bank account state by applying the opened event
account =  BankAccount.apply(account, account_opened)
```

An aggregate root command function may return none (`nil` or `[]`), one, or many domain events. To mutate the aggregate's state in a generic way we use `List.wrap/1` to wrap the events in a list. If the events are already a list, it returns the events. If the events are nil, it returns an empty list. The events list is reduced by calling the `apply/2` function on the aggregate root. Using its state as the accumulator and passing each event in turn.

```elixir
events = BankAccount.open_account(account, %OpenAccount{account_number: "ACC123", initial_balance: 100})

events
|> List.wrap
|> Enum.reduce(account, &BankAccount.apply(&2, &1))
```
