# Pure functional core

 A function may be considered a *pure* function if both of the following statements about the function hold:

> 1. The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while program execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices.
2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices.
-- [Wikipedia.org](https://en.wikipedia.org/wiki/Pure_function)

## Why?

- A pure function is highly testable.
- You will focus on behaviour rather than state.
