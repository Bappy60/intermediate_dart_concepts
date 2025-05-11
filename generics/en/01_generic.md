# Mastering Generics in Dart: When One Size Doesn't Fit All

Ever found yourself copying and pasting the same code blocks over and over, changing just the data types? We've all been there. Meet Ppt, a developer who uses ChatGPT to help with his coding but still finds himself stuck in repetitive patterns. He's building a data processing application that needs to handle a wide variety of data: user profiles, product information, API responses, and cached data.

"This is getting ridiculous," Ppt muttered as he asked ChatGPT to help him create yet another nearly identical class. "There has to be a better way."

That better way? Generics. Let's explore how they transformed Ppt's code—and how they can transform yours too.

## The Problem: Type-Specific Code Duplication

Ppt initially faced a common problem: data caching. As his application grew, he needed to cache different types of data. Following traditional patterns, he started by creating specific cache interfaces for each type:

```dart
abstract class StringCache {
  String getByKey(String key);
  void setByKey(String key, String value);
}

abstract class NumberCache {
  num getByKey(String key);
  void setByKey(String key, num value);
}

abstract class UserCache {
  User getByKey(String key);
  void setByKey(String key, User value);
}

abstract class ProductCache {
  Product getByKey(String key);
  void setByKey(String key, Product value);
}
```

"Wait a minute," Ppt realized, looking at his editor full of nearly identical interfaces. "I'm going to end up with dozens of these cache interfaces for every type in my application!"

The same problem extended to his data processing classes:

```dart
class IntegerData {
  final int value;
  final DateTime timestamp;
  
  IntegerData(this.value, this.timestamp);
  
  void process() {
    print('Processing integer: $value from ${timestamp.toIso8601String()}');
    // Processing logic here
  }
}

class StringData {
  final String value;
  final DateTime timestamp;
  
  StringData(this.value, this.timestamp);
  
  void process() {
    print('Processing string: $value from ${timestamp.toIso8601String()}');
    // Processing logic here
  }
}

class UserData {
  final User value;
  final DateTime timestamp;
  
  UserData(this.value, this.timestamp);
  
  void process() {
    print('Processing user: ${value.name} from ${timestamp.toIso8601String()}');
    // Processing logic here
  }
}
```

See the pattern? Nearly identical interfaces and classes with only the type changing. And Ppt would need to create new ones for every new data type in his application.

## Generic Classes: One Class to Rule Them All

After pouring his frustrations out to Ragib, a seasoned developer with years of experience, Ppt finally found a glimmer of hope. Ragib leaned back in his chair, a knowing smile on his face, and said, "You know, Ppt, you're not the first developer to face this. There's a way to break free from this cycle of repetitive code."

Ppt's eyes lit up with curiosity. "What is it? Tell me!"

Ragib leaned forward, his voice steady and confident. "Generics. They’re not just a feature; they’re a paradigm shift. They’ll change the way you think about your code."

Intrigued, Ppt listened intently as Ragib explained the two transformative benefits of generics:

1. **Type Safety**: "Imagine catching type errors before your application even runs," Ragib said, his tone almost reverent. "Generics give you that power. They ensure your code is robust and reliable, saving you from those dreaded runtime crashes."

2. **Code Reusability**: "Why write the same code over and over when you can write it once and use it everywhere?" Ragib asked, his voice rising with excitement. "Generics let you parameterize types, turning your code into a flexible, reusable masterpiece."

Ppt felt a surge of inspiration. Could this be the breakthrough he had been searching for? He leaned closer, eager to learn more. "Show me how, Ragib. Show me how to wield this power."

And with that, Ppt's journey into the world of generics began. For his cache problem, Ppt created a single generic interface:

```dart
abstract class Cache<T> {
  T getByKey(String key);
  void setByKey(String key, T value);
}
```

With this single interface, he could now implement caches for any type:

```dart
// A memory-based implementation of the Cache interface
class MemoryCache<T> implements Cache<T> {
  final Map<String, T> _cache = {};
  
  @override
  T getByKey(String key) {
    return _cache[key] ?? (throw CacheException('Key not found: $key'));
  }
  
  @override
  void setByKey(String key, T value) {
    _cache[key] = value;
  }
}

// Usage:
final stringCache = MemoryCache<String>();
final numberCache = MemoryCache<num>();
final userCache = MemoryCache<User>();
```

Similarly, for his data processing classes, Ppt created a generic solution:

```dart
class Data<T> {
  final T value;
  final DateTime timestamp;
  
  Data(this.value, this.timestamp);
  
  void process() {
    print('Processing ${T.toString()}: $value from ${timestamp.toIso8601String()}');
    // Processing logic here
  }
}
```

Now Ppt can create data objects of any type:

```dart
void main() {
  final intData = Data<int>(42, DateTime.now());
  final stringData = Data<String>('Profile updated', DateTime.now());
  final userData = Data<User>(User('uid123', 'Alice'), DateTime.now());
  
  intData.process();
  stringData.process();
  userData.process();
}
```

Notice how we specify the type in angle brackets `<int>`, `<String>`, and `<User>`? That's us telling Dart what specific type the generic placeholder `T` should be for each instance.

Ppt observed an immediate benefit: his code was now much more concise while maintaining full type safety. When he tried to use the wrong type, the compiler caught it immediately:

```dart
var names = <String>[];
// names.add(42); // Error: The argument type 'int' can't be assigned to the parameter type 'String'.
```

Does this seem too simple? Don't worry—we'll dive deeper. But first, I want you to think about your own code. Where are you writing nearly identical classes or functions that only differ by type?

## Type Parameters: Not Just 'T'

While `T` is conventional, you can use any identifier for your type parameters. When working with multiple type parameters, descriptive names can make your code more readable:

```dart
class Pair<Key, Value> {
  final Key key;
  final Value value;
  
  Pair(this.key, this.value);
  
  @override
  String toString() => 'Pair<$Key, $Value>($key, $value)';
}

void main() {
  final coordinates = Pair<String, double>('latitude', 37.7749);
  final userPreference = Pair<String, bool>('darkMode', true);
  
  print(coordinates); // Outputs: Pair<String, double>(latitude, 37.7749)
  print(userPreference); // Outputs: Pair<String, bool>(darkMode, true)
}
```

## Generic Methods: When Classes Are Too Much

Ppt's application needed to save data to various storage backends. Instead of creating generic classes for this functionality, he used generic methods:

```dart
Future<bool> saveToDatabase<T>(List<Data<T>> dataList) async {
  try {
    print('Saving ${dataList.length} items of type ${T.toString()} to database');
    // Database saving logic here
    return true;
  } catch (e) {
    print('Error saving to database: $e');
    return false;
  }
}

Future<bool> exportToCSV<T>(List<Data<T>> dataList, String filename) async {
  try {
    print('Exporting ${dataList.length} items of type ${T.toString()} to $filename');
    // CSV export logic here
    return true;
  } catch (e) {
    print('Error exporting to CSV: $e');
    return false;
  }
}
```

Now Ppt can save any type of data with the same methods:

```dart
void main() async {
  final temperatures = [
    Data<double>(36.5, DateTime.now()),
    Data<double>(37.2, DateTime.now()),
    Data<double>(36.9, DateTime.now()),
  ];
  
  final patientNotes = [
    Data<String>('Patient showing improvement', DateTime.now()),
    Data<String>('Prescribed medication X', DateTime.now()),
  ];
  
  await saveToDatabase<double>(temperatures);
  await exportToCSV<String>(patientNotes, 'patient_notes.csv');
}
```

What's happening is that we're defining methods that can work with any type of data, but we specify the exact type at the call site.
