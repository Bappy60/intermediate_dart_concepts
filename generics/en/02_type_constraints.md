# Type Constraints: Setting Boundaries

As Ppt's application grew, he discovered a problem. Some operations only made sense for certain types. For example, calculating statistics on numeric data or formatting text data.

This is where type constraints come in—they let you restrict which types can be used with your generic class or method:

```dart
class NumericData<T extends num> {
  final T value;
  final DateTime timestamp;
  
  NumericData(this.value, this.timestamp);
  
  // This method is safe because we know T is a numeric type
  T multiplyBy(T factor) {
    return (value * factor) as T;
  }
  
  double get squared => value * value;
}
```

The `<T extends num>` means that `T` must be either `num` or a subclass of `num` (like `int` or `double`). Now Ppt can do numeric operations safely:

```dart
void main() {
  final intData = NumericData<int>(10, DateTime.now());
  final doubleData = NumericData<double>(5.5, DateTime.now());
  
  print(intData.multiplyBy(3)); // 30
  print(doubleData.squared); // 30.25
  
  // This would cause a compile-time error:
  // final stringData = NumericData<String>('error', DateTime.now());
}
```

The beauty here is that we get compile-time safety. If someone tries to use a non-numeric type, they'll get an error before the code even runs.

## Real-World Application(Example 1): API Response Parsing

After grasping the basics, Ppt faced a more complex challenge: parsing JSON responses from various API endpoints. Each endpoint returned different data structures, but the process of handling responses (checking for errors, extracting data) was similar across all of them.
"I bet I can use generics to solve this," Ppt thought. He sketched out some model classes first:

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

Then, he created a generic parser class that could work with any model type:

```dart
class ApiResponseParser<T> {
    final T Function(Map<String, dynamic> jsonData) fromJsonConverter;

    ApiResponseParser(this.fromJsonConverter);

    T parse(Map<String, dynamic> responseData) {
        // Common response checking logic
        if (responseData.containsKey('error')) {
            throw Exception('API Error: ${responseData['error']}');
        }

        // Extract the data payload
        final data = responseData['data'] as Map<String, dynamic>;

        // Use the provided converter to create an instance of T
        return fromJsonConverter(data);
    }
}
```

Using this approach, Ppt could now parse any API response with minimal code:

```dart
void main() {
    // Create a parser for User objects
    final userParser = ApiResponseParser<User>(User.fromJson);

    // Simulate an API response for a user
    final userApiResponse = {
        'status': 'success',
        'data': {'id': 'user123', 'name': 'Alice Wonderland'}
    };
    final User alice = userParser.parse(userApiResponse);
    print(alice); // Output: User(id: user123, name: Alice Wonderland)

    // Create a parser for Product objects
    final productParser = ApiResponseParser<Product>(Product.fromJson);

    // Simulate an API response for a product
    final productApiResponse = {
        'status': 'success',
        'data': {'sku': 'DARTBOOK01', 'price': 29.99}
    };
    final Product dartBook = productParser.parse(productApiResponse);
    print(dartBook); // Output: Product(sku: DARTBOOK01, price: 29.99)

    // Simulate an error response
    final errorResponse = {
        'status': 'error',
        'error': 'Invalid request'
    };
    try {
        userParser.parse(errorResponse);
    } catch (e) {
        print(e); // Output: Exception: API Error: Invalid request
    }
}
```

"This is amazing," Ppt realized. "With just a few lines of code, I can handle any API response type!"

## Real-World Application (Example 2): Processing Pipeline

Let's see how Ppt combined these concepts to build a processing pipeline for his lab data:

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
    // Simulate processing time
    await Future.delayed(Duration(milliseconds: 500));
    
    try {
      // Example: multiply by 2
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
    // Simulate processing time
    await Future.delayed(Duration(milliseconds: 300));
    
    try {
      // Example: convert to uppercase
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
        
        // Update data for the next processor in the pipeline
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

And here's how Ppt uses this pipeline:

```dart
void main() async {
  // Create temperature data
  final temperatures = [
    Data<double>(36.5, DateTime.now()),
    Data<double>(37.2, DateTime.now()),
    Data<double>(36.9, DateTime.now()),
  ];
  
  // Create notes data
  final notes = [
    Data<String>('Patient showing improvement', DateTime.now()),
    Data<String>('Prescribed medication X', DateTime.now()),
  ];
  
  // Set up processors
  final numericPipeline = ProcessingPipeline<double>([
    NumericProcessor<double>(),
    // Could add more processors here
  ]);
  
  final textPipeline = ProcessingPipeline<String>([
    TextProcessor(),
    // Could add more processors here
  ]);
  
  // Process data
  final processedTemps = await numericPipeline.runPipeline(temperatures);
  final processedNotes = await textPipeline.runPipeline(notes);
  
  // Print results
  for (var result in processedTemps) {
    print('Original: ${result.originalData.value} -> Processed: ${result.processedValue}');
  }
  
  for (var result in processedNotes) {
    print('Original: ${result.originalData.value} -> Processed: ${result.processedValue}');
  }
}
```

Output:

```bash
Original: 36.5 -> Processed: 73.0
Original: 37.2 -> Processed: 74.4
Original: 36.9 -> Processed: 73.8
Original: Patient showing improvement -> Processed: PATIENT SHOWING IMPROVEMENT
Original: Prescribed medication X -> Processed: PRESCRIBED MEDICATION X
```
