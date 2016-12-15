# Exceptions

## Broken invariants

An aggregate root should protect itself against commands that would cause an invariant to be broken. Raising an exception would be an appropriate response.

## Invalid commands

Commands should be validated before being passed to the aggregate root.

### References

- [Exception or event?](http://thinkbeforecoding.github.io/FsUno.Prod/Exception%20or%20Event.html)
- [Business errors are just ordinary events](http://thinkbeforecoding.com/post/2009/12/10/Business-Errors-are-Just-Ordinary-Events).
