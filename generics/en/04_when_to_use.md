# When to Use Generics

After all this, you might be tempted to use generics everywhere. But like any tool, generics have their place. Here's when Ppt found them most useful:

1. **When you need the same functionality for different types**: Collections, data containers, result wrappers.
2. **When you want to preserve type information**: Instead of using `Object` and casting.
3. **When you want compile-time type safety**: Catching type errors before runtime.
4. **When you're building reusable components**: Libraries, frameworks, and utilities.

And here's when Ppt avoided them:

1. **When there's only one type you'll ever use**: No need for generics if you're only dealing with strings.
2. **When type erasure is a concern**: Remember that generic type information is erased at runtime.
3. **When it makes the code harder to understand**: Sometimes explicit types are clearer.

## Conclusion: Ppt's Transformation

By embracing generics, Ppt transformed his codebase:

- **Before**: 20+ similar classes for different data types
- **After**: ~5 generic classes that handled all data types

- **Before**: Code duplication everywhere
- **After**: Write once, use for any type

- **Before**: Runtime type errors when processing data
- **After**: Compile-time type checking

But most importantly, when a new data type was needed:

- **Before**: Write a new class, new processing logic, new storage handlers
- **After**: Just use the existing generic infrastructure

Now it's your turn. Look at your own codebase—where are you repeating yourself for different types? Where could generics make your code more concise, safer, and more flexible?

## Challenge Exercise

Before you go, I have a challenge for you. Try implementing a `Result<S, E>` class that represents either a success value of type `S` or an error value of type `E`. This pattern is common in languages like Rust and can greatly improve error handling in Dart. Here's a sketch to get you started:

```dart
class Result<S, E> {
  final S? _success;
  final E? _error;
  final bool isSuccess;
  
  Result.success(S value)
      : _success = value,
        _error = null,
        isSuccess = true;
  
  Result.error(E error)
      : _success = null,
        _error = error,
        isSuccess = false;
  
  // TODO: Implement methods like:
  // - getOrElse(S defaultValue)
  // - map<T>(T Function(S) mapper)
  // - mapError<F>(F Function(E) mapper)
  // - flatMap<T>(Result<T, E> Function(S) mapper)
}
```

See if you can complete this class and use it in a practical example. Good luck!
