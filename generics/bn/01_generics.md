# ডার্টে জেনেরিকস-এ দক্ষতা অর্জন: যখন এক মাপ সবার জন্য যথেষ্ট নয়

তুমি কি কখনো নিজেকে একই কোড ব্লক বারবার কপি-পেস্ট করতে দেখেছ, শুধু ডেটার টাইপ পরিবর্তন করে? আমরা সবাই এমন পরিস্থিতির সম্মুখীন হয়েছি। পরিচিত হও পিপিটির সাথে—একজন ডেভেলপার, যিনি ChatGPT ব্যবহার করেন কোডিং-এ সাহায্যের জন্য, কিন্তু এখনো নিজেকে বারবার একই ধরনের কোড লেখার ফাঁদে আটকে পান। তিনি এমন একটি data processing অ্যাপ্লিকেশন তৈরি করছেন, যেটিকে বিভিন্ন ধরণের ডেটা—user profile, product information, API response এবং cached data—handle করতে হবে।

"এটা আসলেই এখন বিরক্তিকর হয়ে গেছে," পিপিটি গজগজ করতে করতে ChatGPT-কে আরেকটা প্রায় একই রকম class বানাতে বলেন। "এর চেয়ে ভালো কোনো উপায় নিশ্চয়ই আছে!"

হ্যাঁ, সত্যিই আছে। সেটি হলো জেনেরিকস। জেনেরিকস কীভাবে পিপিটির কোডকে রূপান্তর করেছে, আর কীভাবে তোমারটাকেও করতে পারে—চল দেখে নেওয়া যাক।

## সমস্যা: টাইপ-স্পেসিফিক কোড ডুপ্লিকেশন

পিপিটি প্রাথমিকভাবে একটি সাধারণ সমস্যার সম্মুখীন হয়েছিল: ডেটা ক্যাশিং। তার অ্যাপ্লিকেশনটি যখন বাড়তে শুরু করলো, তখন তাকে বিভিন্ন ধরণের ডেটা ক্যাশে করার প্রয়োজন হলো। ঐতিহ্যবাহী প্যাটার্ন অনুসরণ করে, সে প্রতিটি টাইপের জন্য নির্দিষ্ট ক্যাশ ইন্টারফেস তৈরি করা শুরু করলো:

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

"এক মিনিট," পিপিটি তার এডিটরে প্রায় একই রকম ইন্টারফেস ভর্তি দেখে বুঝতে পারলো। "আমার অ্যাপ্লিকেশনের প্রতিটি টাইপের জন্য আমাকে ডজন ডজন ক্যাশ ইন্টারফেস তৈরি করতে হবে\!"

একই সমস্যা তার ডেটা প্রসেসিং ক্লাসগুলোতেও দেখা গেল:

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

প্যাটার্নটা কি দেখতে পাচ্ছেন? প্রায় একই রকম ইন্টারফেস এবং ক্লাস, শুধু টাইপ পরিবর্তন হচ্ছে। আর পিপিটিকে তার অ্যাপ্লিকেশনের প্রতিটি নতুন ডেটা টাইপের জন্য নতুন করে এগুলো তৈরি করতে হতো।

## জেনেরিক Classes: একটি ক্লাসই যথেষ্ট

বহু বছরের অভিজ্ঞতাসম্পন্ন একজন অভিজ্ঞ ডেভেলপার রাগিবের কাছে নিজের হতাশার কথা বলার পর, পিপিটি অবশেষে আশার আলো দেখতে পেল। রাগিব চেয়ারে হেলান দিয়ে বসলেন, তার মুখে এক পরিচিত হাসি, এবং বললেন, "জানো তো পিপিটি, তুমিই প্রথম ডেভেলপার নও যে এই সমস্যার সম্মুখীন হচ্ছো। এই পুনরাবৃত্তিমূলক কোডের চক্র থেকে বেরিয়ে আসার একটা উপায় আছে।"

পিপিটির চোখ কৌতূহলে জ্বলে উঠলো। "সেটা কী? আমাকে বলো\!"

রাগিব ঝুঁকে এগিয়ে এলেন, তার কণ্ঠস্বর স্থির এবং আত্মবিশ্বাসী। "Generics। এগুলো শুধু একটা ফিচার নয়; এটা একটা প্যারাডাইম শিফট। এগুলো তোমার কোড সম্পর্কে চিন্তাভাবনা করার পদ্ধতি পরিবর্তন করে দেবে।"

আগ্রহী হয়ে পিপিটি মনোযোগ সহকারে শুনলো যখন রাগিব জেনেরিকসের দুটি transformational সুবিধা ব্যাখ্যা করলেন:

১. **Type Safety (টাইপ নিরাপত্তা)**: "ভাবো তোমার অ্যাপ্লিকেশন রান করার আগেই টাইপ এরর ধরা পড়ছে," রাগিব বললেন। "জেনেরিকস তোমাকে সেই ক্ষমতা দেয়। এগুলো নিশ্চিত করে যে তোমার কোড শক্তিশালী এবং নির্ভরযোগ্য, তোমাকে সেই ভয়াবহ রানটাইম ক্র্যাশ থেকে বাঁচায়।"

২. **Code Reusability (কোড পুনঃব্যবহারযোগ্যতা)**: "একই কোড বারবার লেখার দরকার কী যখন তুমি একবার লিখে সব জায়গায় ব্যবহার করতে পারো?" রাগিব জিজ্ঞাসা করলেন। "জেনেরিকস তোমাকে টাইপ প্যারামিটারাইজ করতে দেয়, তোমার কোডকে একটি নমনীয়, পুনঃব্যবহারযোগ্য মাস্টারপিসে পরিণত করে।"

পিপিটি অনুপ্রেরণার ঢেউ অনুভব করলো। এটাই কি সেই breakthrough যা সে খুঁজছিল? সে আরও কাছে ঝুঁকে এলো, আরও জানার জন্য উৎসুক। "আমাকে দেখাও, রাগিব। আমাকে দেখাও কীভাবে এই ক্ষমতা ব্যবহার করতে হয়।"

এবং এর সাথেই, পিপিটির জেনেরিকসের জগতে যাত্রা শুরু হলো। তার ডাটা ক্যাশিং সমস্যার জন্য, পিপিটি একটি একক জেনেরিক ইন্টারফেস তৈরি করলো:

```dart
abstract class Cache<T> {
  T getByKey(String key);
  void setByKey(String key, T value);
}
```

এই একটি ইন্টারফেসের মাধ্যমে, সে এখন যেকোনো টাইপের জন্য ক্যাশিং বাস্তবায়ন করতে পারবে:

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

একইভাবে, তার ডেটা প্রসেসিং ক্লাসগুলোর জন্য, পিপিটি একটি জেনেরিক সমাধান তৈরি করলো:

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

এখন পিপিটি যেকোনো টাইপের ডেটা অবজেক্ট তৈরি করতে পারে:

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

লক্ষ্য করুন আমরা কীভাবে অ্যাঙ্গেল ব্র্যাকেটের `<int>`, `<String>` এবং `<User>` মধ্যে টাইপ নির্দিষ্ট করছি? এর মাধ্যমে আমরা ডার্টকে বলছি যে প্রতিটি ইনস্ট্যান্সের জন্য জেনেরিক প্লেসহোল্ডার `T`-এর নির্দিষ্ট টাইপ কী হওয়া উচিত।

পিপিটি একটি তাৎক্ষণিক সুবিধা লক্ষ্য করলো: তার কোড এখন অনেক সংক্ষিপ্ত অথচ সম্পূর্ণ টাইপ নিরাপদ। যখন সে ভুল টাইপ ব্যবহার করার চেষ্টা করলো, কম্পাইলার তাৎক্ষণিকভাবে তা ধরে ফেললো:

```dart
var names = <String>[];
// names.add(42); // Error: The argument type 'int' can't be assigned to the parameter type 'String'.
```

এটা কি খুব সহজ মনে হচ্ছে? চিন্তা করবেন না—আমরা আরও গভীরে যাবো। তবে প্রথমে, আমি চাই আপনি আপনার নিজের কোড সম্পর্কে ভাবুন। কোথায় আপনি প্রায় একই রকম ক্লাস বা ফাংশন লিখছেন যা কেবল টাইপের দিক থেকে ভিন্ন?

## Type Parameters: শুধু 'T' নয়

যদিও `T` একটি প্রথাগত নাম, আপনি আপনার টাইপ প্যারামিটারের জন্য যেকোনো আইডেন্টিফায়ার ব্যবহার করতে পারেন। একাধিক টাইপ প্যারামিটার নিয়ে কাজ করার সময়, বর্ণনামূলক নাম আপনার কোডকে আরও পঠনযোগ্য করে তুলতে পারে:

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

## জেনেরিক Methods: জেনেরিক মেথড

পিপিটির অ্যাপ্লিকেশনকে বিভিন্ন storage backend-এ data সেভ করতে হতো। এই কার্যকারিতার জন্য জেনেরিক ক্লাস তৈরি করার পরিবর্তে, সে জেনেরিক মেথড ব্যবহার করলো:

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

এখন পিপিটি একই মেথড ব্যবহার করে যেকোনো টাইপের ডেটা সংরক্ষণ করতে পারে:

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

এখানে যা ঘটছে তা হলো আমরা এমন মেথড সংজ্ঞায়িত করছি যা যেকোনো ধরণের ডেটা নিয়ে কাজ করতে পারে, তবে আমরা কল করার সময় সঠিক টাইপটি নির্দিষ্ট করছি।
