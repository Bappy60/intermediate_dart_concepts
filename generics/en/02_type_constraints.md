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

## Real-World Application: Processing Pipeline

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

## Advanced Generic Patterns

Now that you've seen the basics in action, let's look at some more advanced patterns that Ppt implemented:

### 1. Generic Extension Methods

Ppt wanted to add functionality to his `Data<T>` class without modifying it:

```dart
extension NumericDataExtension<T extends num> on Data<T> {
  Data<T> applyFactor(T factor) {
    return Data<T>((value * factor) as T, DateTime.now());
  }
  
  Data<double> average(Data<T> other) {
    return Data<double>((value + other.value) / 2, DateTime.now());
  }
}

extension StringDataExtension on Data<String> {
  Data<String> appendText(String text) {
    return Data<String>(value + text, DateTime.now());
  }
  
  Data<int> get wordCount {
    return Data<int>(value.split(' ').length, DateTime.now());
  }
}
```

Usage example:

```dart
void main() {
  final temp1 = Data<double>(36.5, DateTime.now());
  final temp2 = Data<double>(37.5, DateTime.now());
  
  final scaledTemp = temp1.applyFactor(2.0);
  final avgTemp = temp1.average(temp2);
  
  print(scaledTemp.value); // 73.0
  print(avgTemp.value); // 37.0
  
  final note = Data<String>('Patient recovering well', DateTime.now());
  final updatedNote = note.appendText(' - continues medication');
  final words = note.wordCount;
  
  print(updatedNote.value); // Patient recovering well - continues medication
  print(words.value); // 3
}
```
