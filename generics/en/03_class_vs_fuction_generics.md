# Class-level vs. Function-level Generics

The distinction between when to use class-level generics and function-level generics
hinges on the scope of the type parameter's relevance.

**Class-level generics** (e.g., `Cache<T>`, `ApiResponseParser<T>`) are used when the
type parameter is fundamental to the entire class and its instances. An instance
of `Cache<String>` is inherently a cache of strings; its methods and internal
state are all tied to `String`. The type is specified when the class instance is
created.

For example, Ppt's `Cache<T>` is a class-level generic because it is designed to
store any type of data. To use it, Ppt creates an instance of `Cache<String>` or
`Cache<User>`, depending on the type of data he wants to store.

**Function-level generics** (e.g., `first<T>(List<T> ts)`) are used when the type
parameter is specific to that single method's operation and not necessarily tied
to the state of an encompassing class (if any). The `first` function's ability to
handle different list types doesn't depend on any class instance; its genericity
is self-contained. The type is often inferred from the arguments at the call
site or can be explicitly provided.

For example, Ppt's `first<T>(List<T> ts)` function is a function-level generic
because it can be used to retrieve the first element from any type of list. To
use it, Ppt simply calls `first<String>(['a', 'b', 'c'])` or
`first<User>([user1, user2, user3])`, and the type is inferred from the
arguments.

A class can also have generic methods even if the class itself is not generic, or
if its generic methods use different type parameters than the class.
