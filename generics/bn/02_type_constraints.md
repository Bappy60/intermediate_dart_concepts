# Dart-এ Type Constraints: সীমারেখা নির্ধারণ

পিপিটির অ্যাপ্লিকেশন বড় হতে থাকলে তিনি বুঝতে পারলেন কিছু অপারেশন সব ধরনের টাইপের জন্য উপযুক্ত নয়। উদাহরণস্বরূপ, সংখ্যার উপর ভিত্তি করে গাণিতিক হিসাব বা স্ট্রিং ফরম্যাটিং শুধুমাত্র নির্দিষ্ট টাইপেই প্রাসঙ্গিক।

এখানেই **type constraints**-এর গুরুত্ব দেখা যায়—এগুলো ব্যবহার করে আমরা Generic ক্লাস বা মেথডে নির্দিষ্ট টাইপগুলোকেই অনুমোদন দিতে পারি:

```dart
class NumericData<T extends num> {
  final T value;
  final DateTime timestamp;
  
  NumericData(this.value, this.timestamp);
  
  // আমরা নিশ্চিত যে T একটি সংখ্যাসূচক টাইপ, তাই এই অপারেশন নিরাপদ
  T multiplyBy(T factor) {
    return (value * factor) as T;
  }
  
  double get squared => value * value;
}

`<T extends num>` মানে হলো `T` অবশ্যই `num` অথবা এর সাবক্লাস (int বা double) হতে হবে। এখন পিপিটি নিরাপদে গণনামূলক অপারেশন করতে পারেন:

```dart
void main() {
  final intData = NumericData<int>(10, DateTime.now());
  final doubleData = NumericData<double>(5.5, DateTime.now());
  
  print(intData.multiplyBy(2)); // Output: 20
  print(doubleData.multiplyBy(3.0)); // Output: 16.5


  // নিচের লাইনটি compile-time error দেবে:
  // final stringData = NumericData<String>('error', DateTime.now());
}
```

এই approach-এর সৌন্দর্য হলো compile-time safety—ভুল টাইপ ব্যবহার করলে রানটাইমে নয়, কোড কম্পাইল করার সময়ই এরর ধরা পড়ে।

## বাস্তবে Type Constraints কেন গুরুত্বপূর্ণ?

প্র্যাকটিক্যাল সফটওয়্যার ডেভেলপমেন্টে, আমরা প্রায়ই এমন ফাংশন বা ক্লাস তৈরি করি যা কেবল নির্দিষ্ট টাইপেই প্রাসঙ্গিক। উদাহরণস্বরূপ:

`num` টাইপ ছাড়া গাণিতিক অপারেশন করা সম্ভব নয়।

`String` ছাড়া `toUpperCase()` বা `length` ব্যবহার করা যায় না।

`DateTime` না হলে সময়সংক্রান্ত অপারেশন করা যুক্তিসঙ্গত নয়।

`Generics`-এর মাধ্যমে কোড পুনরায় ব্যবহারযোগ্য করা যায়, কিন্তু `Type Constraint` দিয়ে সেটাকে নিরাপদ ও প্রাসঙ্গিক রাখা যায়।

এটি আমাদের runtime crash থেকে রক্ষা করে এবং আমাদের কোড আরও বুঝতে সহজ ও maintainable করে তোলে।

## বাস্তব উদাহরণ ১: API Response Parsing

জেনেরিকস বোঝার পর, পিপিটি আরেকটি জটিল সমস্যার মুখোমুখি হন: বিভিন্ন API endpoint থেকে JSON response পার্স করা। প্রতিটি endpoint ভিন্ন data structure ফেরত দিচ্ছে, কিন্তু error-checking ও data extraction প্রক্রিয়া প্রায় এক।

পিপিটি ভাবলেন, “Generics দিয়ে তো এটা সহজে করা যায়!” এবং নিচের model ক্লাসগুলো তৈরি করলেন:

```dart
class User {
  final String id;
  final String name;
  User(this.id, this.name);

  factory User.fromJson(Map<String, dynamic> json) {
    return User(json['id'] as String, json['name'] as String);
  }

  @override
  String toString() => 'User(id: $id, name: $name)';
}

class Product {
  final String sku;
  final double price;
  Product(this.sku, this.price);

  factory Product.fromJson(Map<String, dynamic> json) {
    return Product(json['sku'] as String, (json['price'] as num).toDouble());
  }

  @override
  String toString() => 'Product(sku: $sku, price: $price)';
}
```

তারপর, তিনি একটি generic পার্সার ক্লাস তৈরি করলেন যা যেকোনো মডেল টাইপের সাথে কাজ করতে পারে:

```dart

class ApiResponseParser<T> {
    final T Function(Map<String, dynamic> jsonData) fromJsonConverter;

    ApiResponseParser(this.fromJsonConverter);

    T parse(Map<String, dynamic> responseData) {
        // সাধারণ রেসপন্স চেকিং লজিক
        if (responseData.containsKey('error')) {
            throw Exception('API Error: ${responseData['error']}');
        }

        // ডাটা পেলোড এক্সট্র্যাক্ট করুন
        final data = responseData['data'] as Map<String, dynamic>;

        // T এর ইনস্ট্যান্স তৈরি করতে প্রদত্ত কনভার্টার ব্যবহার করুন
        return fromJsonConverter(data);
    }
}
```

ব্যবহার:

```dart

void main() {
    // User অবজেক্টের জন্য একটি পার্সার তৈরি করুন
    final userParser = ApiResponseParser<User>(User.fromJson);

    // একজন ব্যবহারকারীর জন্য একটি API রেসপন্স সিমুলেট করুন
    final userApiResponse = {
        'status': 'success',
        'data': {'id': 'user123', 'name': 'Alice Wonderland'}
    };
    final User alice = userParser.parse(userApiResponse);
    print(alice); // আউটপুট: User(id: user123, name: Alice Wonderland)

    // Product অবজেক্টের জন্য একটি পার্সার তৈরি করুন
    final productParser = ApiResponseParser<Product>(Product.fromJson);

    // একটি পণ্যের জন্য একটি API রেসপন্স সিমুলেট করুন
    final productApiResponse = {
        'status': 'success',
        'data': {'sku': 'DARTBOOK01', 'price': 29.99}
    };
    final Product dartBook = productParser.parse(productApiResponse);
    print(dartBook); // আউটপুট: Product(sku: DARTBOOK01, price: 29.99)

    // একটি এরর রেসপন্স সিমুলেট করুন
    final errorResponse = {
        'status': 'error',
        'error': 'Invalid request'
    };
    try {
        userParser.parse(errorResponse);
    } catch (e) {
        print(e); // আউটপুট: Exception: API Error: Invalid request
    }
}
```

পিপিটি বললেন, "দারুণ! এত কম লাইনে যেকোনো API response পার্স করা যাচ্ছে।"

## বাস্তব উদাহরণ ২: Processing Pipeline

পিপিটি এর ল্যাব ডেটা প্রসেসিং করার জন্য নিচের মতো একটি pipeline তৈরি করলেন:

```dart
abstract class Processor<T> {
  Future<ProcessResult<T>> process(Data<T> input);
}

class ProcessResult<T> {
  final Data<T> originalData;
  final T processedValue;
  final bool success;
  final String? error;
  
  ProcessResult({
    required this.originalData,
    required this.processedValue,
    this.success = true,
    this.error,
  });
}

class NumericProcessor<T extends num> implements Processor<T> {
  @override
  Future<ProcessResult<T>> process(Data<T> input) async {
    // প্রসেসিং সময় অনুকরণ করুন
    await Future.delayed(Duration(milliseconds: 500));
    
    try {
      // উদাহরণ: 2 দ্বারা গুণ করুন
      final result = (input.value * 2) as T;
      return ProcessResult(
        originalData: input,
        processedValue: result,
        success: true,
      );
    } catch (e) {
      return ProcessResult(
        originalData: input,
        processedValue: input.value,
        success: false,
        error: e.toString(),
      );
    }
  }
}

class TextProcessor implements Processor<String> {
  @override
  Future<ProcessResult<String>> process(Data<String> input) async {
    // প্রসেসিং সময় অনুকরণ করুন
    await Future.delayed(Duration(milliseconds: 300));
    
    try {
      // উদাহরণ: বড় অক্ষরে রূপান্তর করুন
      final result = input.value.toUpperCase();
      return ProcessResult(
        originalData: input,
        processedValue: result,
        success: true,
      );
    } catch (e) {
      return ProcessResult(
        originalData: input,
        processedValue: input.value,
        success: false,
        error: e.toString(),
      );
    }
  }
}

class ProcessingPipeline<T> {
  final List<Processor<T>> processors;
  
  ProcessingPipeline(this.processors);
  
  Future<List<ProcessResult<T>>> runPipeline(List<Data<T>> dataList) async {
    final results = <ProcessResult<T>>[];
    
    for (var data in dataList) {
      ProcessResult<T> currentResult;
      var currentData = data;
      
      for (var processor in processors) {
        currentResult = await processor.process(currentData);
        
        if (!currentResult.success) {
          results.add(currentResult);
          break;
        }
        
        // পাইপলাইনের পরবর্তী প্রসেসরের জন্য ডেটা আপডেট করুন
        currentData = Data<T>(currentResult.processedValue, DateTime.now());
      }
      
      if (results.isEmpty || results.last.originalData != data) {
        results.add(ProcessResult(
          originalData: data,
          processedValue: currentData.value,
          success: true,
        ));
      }
    }
    
    return results;
  }
}
```

এবং এখানে পিপিটি কীভাবে এই পাইপলাইন ব্যবহার করে:

```dart
void main() async {
  // তাপমাত্রার ডেটা তৈরি করুন
  final temperatures = [
    Data<double>(36.5, DateTime.now()),
    Data<double>(37.2, DateTime.now()),
    Data<double>(36.9, DateTime.now()),
  ];
  
  // নোটের ডেটা তৈরি করুন
  final notes = [
    Data<String>('Patient showing improvement', DateTime.now()),
    Data<String>('Prescribed medication X', DateTime.now()),
  ];
  
  // প্রসেসর সেটআপ করুন
  final numericPipeline = ProcessingPipeline<double>([
    NumericProcessor<double>(),
    // এখানে আরও প্রসেসর যোগ করা যেতে পারে
  ]);
  
  final textPipeline = ProcessingPipeline<String>([
    TextProcessor(),
    // এখানে আরও প্রসেসর যোগ করা যেতে পারে
  ]);
  
  // ডেটা প্রসেস করুন
  final processedTemps = await numericPipeline.runPipeline(temperatures);
  final processedNotes = await textPipeline.runPipeline(notes);
  
  // ফলাফল প্রিন্ট করুন
  for (var result in processedTemps) {
    print('Original: ${result.originalData.value} -> Processed: ${result.processedValue}');
  }
  
  for (var result in processedNotes) {
    print('Original: ${result.originalData.value} -> Processed: ${result.processedValue}');
  }
}
```

আউটপুট:

```bash
Original: 36.5 -> Processed: 73.0
Original: 37.2 -> Processed: 74.4
Original: 36.9 -> Processed: 73.8
Original: Patient showing improvement -> Processed: PATIENT SHOWING IMPROVEMENT
Original: Prescribed medication X -> Processed: PRESCRIBED MEDICATION X
```

### `Type Constraint` অত্যন্ত গুরুত্বপূর্ণ ভূমিকা পালন করে কারণ

১. **টাইপ সেফটি**: `Type Constraint` আপনার generic ক্লাস বা মেথডের সাথে ব্যবহার করা যেতে পারে এমন টাইপগুলিকে সীমাবদ্ধ করে। এটি কম্পাইল-টাইমে ত্রুটি ধরা পড়ে, যা রানটাইমে অনাকাঙ্ক্ষিত ব্যবহার থেকে আপনার কোডকে রক্ষা করে। উদাহরণস্বরূপ, `NumericData<T extends num>` ক্লাসে, আমরা নিশ্চিত করতে পারি যে শুধুমাত্র সংখ্যাসূচক টাইপগুলি (যেমন int বা double) ব্যবহার করা যাবে।

২. **ইন্টেলিসেন্স এবং অটোকমপ্লিট**: `Type Constraint` আপনার IDE-কে সঠিক মেথড এবং প্রোপার্টি সম্পর্কে জানতে দেয়। যখন আপনি extends কীওয়ার্ড ব্যবহার করেন, তখন IDE জানে যে T এর সমস্ত মেথড এবং প্রোপার্টি উপলব্ধ থাকবে, যা আপনাকে যথাযথ অটোকমপ্লিট সাজেশন দেয়।

৩. কোড সিমেন্টিক্স: `Type Constraint` আপনার কোডের ইচ্ছা স্পষ্টভাবে প্রকাশ করে। `NumericProcessor<T extends num>` লিখে, আপনি অন্যান্য ডেভেলপারদের স্পষ্টভাবে জানাচ্ছেন যে এই প্রসেসর শুধুমাত্র সংখ্যাসূচক ডাটার জন্য ডিজাইন করা হয়েছে।

৪. **সঠিক অপারেশন নিশ্চিতকরণ**: কিছু অপারেশন শুধুমাত্র নির্দিষ্ট টাইপের জন্য অর্থপূর্ণ। উদাহরণস্বরূপ, আমরা শুধুমাত্র সংখ্যাসূচক ডাটাতে গাণিতিক অপারেশন করতে পারি বা শুধুমাত্র টেক্সট ডাটায় স্ট্রিং ম্যানিপুলেশন করতে পারি। `Type Constraint` ব্যবহার করে, আমরা নিশ্চিত করতে পারি যে এই অপারেশনগুলি কেবল সঠিক ধরণের ডাটার উপরই প্রয়োগ করা হয়।

৫. **API ডিজাইন উন্নতি**: `Type Constraint` API ডিজাইনকে আরও স্পষ্ট করে। যখন আপনি একটি লাইব্রেরি বা API তৈরি করেন, তখন `Type Constraint` আপনার লাইব্রেরির ব্যবহারকারীদের জানাতে সাহায্য করে যে কিভাবে এটি ব্যবহার করা উচিত।

৬. **টাইপ ক্যাস্টিং এড়ান**: সঠিক `Type Constraint` ব্যবহার করে, আপনি ম্যানুয়াল টাইপ ক্যাস্টিং হ্রাস করতে পারেন, যা সাধারণত রানটাইম এরর হওয়ার সম্ভাবনা বাড়ায়।

৭. **কম্পাইলার অপ্টিমাইজেশন**: কিছু ক্ষেত্রে, কম্পাইলার `Type Constraint` ব্যবহার করে কোড অপ্টিমাইজ করতে পারে, কারণ এটি আগে থেকেই জানে যে টাইপ সমর্থিত অপারেশনই ব্যবহার করা হবে।

পিপিটির উদাহরণে, `Type Constraint` তাকে শক্তিশালী এবং নমনীয় কোড তৈরি করতে সাহায্য করেছে যা বিভিন্ন ধরণের ডাটা নিয়ে কাজ করতে পারে, কিন্তু সেই সাথে বিশেষ প্রয়োজনীয়তার জন্য টাইপ-নির্দিষ্ট লজিক প্রয়োগ করতে পারে। এটি জেনেরিকস-এর সেরা উভয় দিক অর্জন করে - কোড রিইউজাবিলিটি এবং টাইপ সেফটি।
