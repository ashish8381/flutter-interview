# **Flutter & Dart Basics**

## **1. Difference between StatelessWidget and StatefulWidget**
**StatelessWidget** is immutable - once created, its properties can't change. Used for static content that doesn't need to change over time.

**StatefulWidget** is mutable - it has a separate State object that can change during the widget's lifetime, triggering rebuilds when `setState()` is called.

## **2. Widgets used for layouts in Flutter**
**Row** - arranges children horizontally  
**Column** - arranges children vertically  
**Stack** - layers children on top of each other  
*All three are layout widgets, each serving different arrangement purposes.*

## **3. What does setState() do internally?**
`setState()` marks the widget as "dirty" and schedules a rebuild. It:
1. Updates the State object's internal state
2. Marks the element as needing rebuild
3. Schedules the framework to call `build()` method on the next frame
4. Only updates the subtree, not the entire widget tree

## **4. Difference between late and nullable (?) variables**
**`late`** - Declaration that a variable will be initialized before use (non-nullable but initialized later). Causes runtime error if accessed before initialization.

**`?` (nullable)** - Variable can hold `null` value. Requires null checks when accessing.

```dart
late String name; // Must be initialized before use
String? address; // Can be null
```

## **5. Flutter rendering pipeline**
Three-phase process:
1. **Animation** (beginning of frame) - Updates animations
2. **Build** - Creates/updates widget tree via `build()` methods
3. **Layout & Paint** - Calculates sizes/positions, paints to screen
4. **Compositing** - Combines layers for GPU rendering

## **6. What is BuildContext and why is it important?**
`BuildContext` is the location of a widget in the widget tree. It:
- Provides access to theme, media query, navigation
- Enables widget-to-widget communication
- Identifies a widget's position in the hierarchy
- Essential for showing dialogs, navigating, theming

## **7. Purpose of const constructor**
Optimizes performance by:
- Creating compile-time constants
- Enabling widget caching (same instance reused)
- Reducing garbage collection
- Allowing Flutter to skip rebuilding identical widgets

---

# **Dart Language**

## **1. Difference between final, const, and var**
**`var`** - Type inference, value can be reassigned (same type)  
**`final`** - Runtime constant, assigned once, can't be reassigned  
**`const`** - Compile-time constant, deeply immutable

```dart
var x = 5; x = 10; // OK
final y = DateTime.now(); // Runtime value
const z = 5; // Must be known at compile time
```

## **2. What are mixins in Dart?**
Mixins enable code reuse across class hierarchies without inheritance:
- Use `with` keyword to add functionality
- Can't be instantiated directly
- Alternative to multiple inheritance
- Great for adding reusable behaviors

```dart
mixin Logging {
  void log(String msg) => print(msg);
}
class MyClass with Logging {}
```

## **3. What is an isolate?**
Dart's concurrency model - isolates are:
- Separate memory heaps (no shared memory)
- Communicate via message passing
- Run in parallel (true parallelism)
- Each has its own event loop
*Alternative to threads but safer (no shared state)*

## **4. Difference between async, await, and Future**
**`Future`** - Represents a potential value/error that will be available later  
**`async`** - Marks a function as asynchronous (returns a Future)  
**`await`** - Pauses execution until Future completes (non-blocking)

```dart
Future<String> fetchData() async {
  var data = await http.get(url);
  return data;
}
```

## **5. What is null safety and why is it important?**
**Null safety** - Prevents null reference errors at compile time:
- Variables are non-nullable by default
- Must explicitly declare nullable types with `?`
- Eliminates runtime null exceptions
- Improves code reliability and maintainability
- Enables better performance optimizations

```dart
String name = 'John'; // Can't be null
String? address = null; // Explicitly nullable
```

## **1. Dart Fundamentals**

### **Variables & Data Types**
```dart
void main() {
  // 1. VAR - Type inferred, can be reassigned
  var name = 'John'; // Dart infers String type
  name = 'Jane'; // ✅ Can reassign
  // name = 123; // ❌ Error: Can't change type
  
  // 2. FINAL - Runtime constant, assigned once
  final age = 25;
  final currentTime = DateTime.now(); // Determined at runtime
  // age = 26; // ❌ Error: Can't reassign final
  
  // 3. CONST - Compile-time constant
  const pi = 3.14;
  const hoursInDay = 24;
  // const currentTime = DateTime.now(); // ❌ Error: Not compile-time
  
  // 4. Explicit types
  String city = 'New York';
  int population = 8000000;
  double temperature = 72.5;
  bool isSunny = true;
  
  // 5. Dynamic type (use sparingly)
  dynamic anything = 'Hello';
  anything = 100; // ✅ Can change type
  anything = true; // ✅ Can change type
  
  print('Name: $name, Age: $age, City: $city');
}
```

### **Null Safety**
```dart
void main() {
  // NON-NULLABLE (default - can't be null)
  String name = 'Alice';
  // name = null; // ❌ Compile-time error
  
  // NULLABLE (explicit with ?)
  String? nickname = null; // ✅ Can be null
  nickname = 'Ally'; // ✅ Can assign value
  
  // NULL AWARE OPERATORS
  String? middleName;
  
  // ?? (If null operator)
  String displayName = middleName ?? 'No middle name';
  print(displayName); // Output: No middle name
  
  // ??= (Null-aware assignment)
  String? address;
  address ??= 'Unknown address';
  print(address); // Output: Unknown address
  
  // ?. (Safe navigation)
  String? text;
  print(text?.length); // Output: null (not error)
  print(text?.toUpperCase()); // Output: null
  
  // ! (Bang operator - use cautiously)
  String? nullableString;
  // String requiredString = nullableString!; // ❌ Runtime error if null
  
  // LATE VARIABLES
  late String userName;
  // print(userName); // ❌ Runtime error: not initialized
  userName = 'Bob'; // Must initialize before use
  print(userName); // ✅ Output: Bob
}
```

### **Functions**
```dart
void main() {
  // 1. BASIC FUNCTION
  void greet() {
    print('Hello!');
  }
  greet();
  
  // 2. FUNCTION WITH PARAMETERS
  void introduce(String name, int age) {
    print('I am $name, $age years old');
  }
  introduce('John', 30);
  
  // 3. NAMED PARAMETERS (use {})
  void printDetails({String? name, int? age}) {
    print('Name: $name, Age: $age');
  }
  printDetails(name: 'Alice', age: 25);
  printDetails(age: 25); // name is optional
  
  // 4. REQUIRED NAMED PARAMETERS
  void requiredDetails({required String name, required int age}) {
    print('Required: $name, $age');
  }
  requiredDetails(name: 'Bob', age: 40);
  
  // 5. DEFAULT PARAMETERS
  void sayHello([String name = 'Guest']) {
    print('Hello, $name!');
  }
  sayHello(); // Output: Hello, Guest!
  sayHello('John'); // Output: Hello, John!
  
  // 6. RETURN VALUES
  int add(int a, int b) {
    return a + b;
  }
  int sum = add(5, 3);
  print('Sum: $sum'); // Output: Sum: 8
  
  // 7. ARROW FUNCTION (for single expression)
  int multiply(int a, int b) => a * b;
  print('Product: ${multiply(4, 5)}'); // Output: Product: 20
  
  // 8. ANONYMOUS FUNCTION
  List<int> numbers = [1, 2, 3, 4];
  numbers.forEach((number) {
    print('Number: $number');
  });
  
  // 9. HIGHER-ORDER FUNCTIONS
  void processNumbers(List<int> nums, Function processor) {
    for (var num in nums) {
      processor(num);
    }
  }
  processNumbers([1, 2, 3], (n) => print('Processing $n'));
}
```

### **Control Flow**
```dart
void main() {
  // 1. IF-ELSE
  int age = 18;
  if (age >= 18) {
    print('Adult');
  } else if (age >= 13) {
    print('Teenager');
  } else {
    print('Child');
  }
  
  // 2. TERNARY OPERATOR
  String status = age >= 18 ? 'Adult' : 'Minor';
  print('Status: $status');
  
  // 3. SWITCH STATEMENT
  String grade = 'B';
  switch (grade) {
    case 'A':
      print('Excellent');
      break;
    case 'B':
      print('Good');
      break;
    case 'C':
      print('Fair');
      break;
    default:
      print('Needs improvement');
  }
  
  // 4. FOR LOOPS
  // Traditional for loop
  for (int i = 0; i < 5; i++) {
    print('Count: $i');
  }
  
  // For-in loop
  List<String> fruits = ['Apple', 'Banana', 'Orange'];
  for (String fruit in fruits) {
    print('Fruit: $fruit');
  }
  
  // For-each with index
  for (var i = 0; i < fruits.length; i++) {
    print('${i + 1}. ${fruits[i]}');
  }
  
  // 5. WHILE LOOP
  int counter = 0;
  while (counter < 3) {
    print('While counter: $counter');
    counter++;
  }
  
  // 6. DO-WHILE LOOP
  int count = 0;
  do {
    print('Do-while count: $count');
    count++;
  } while (count < 3);
  
  // 7. BREAK AND CONTINUE
  for (int i = 0; i < 10; i++) {
    if (i == 2) continue; // Skip iteration
    if (i == 5) break; // Exit loop
    print('Value: $i');
  }
}
```

### **Collections (List, Set, Map)**
```dart
void main() {
  // 1. LISTS (Ordered, indexed, allows duplicates)
  List<String> colors = ['Red', 'Green', 'Blue'];
  
  // Adding elements
  colors.add('Yellow');
  colors.addAll(['Purple', 'Orange']);
  
  // Accessing elements
  print(colors[0]); // Output: Red
  print(colors.first); // Output: Red
  print(colors.last); // Output: Orange
  print(colors.length); // Output: 6
  
  // List methods
  colors.remove('Green');
  colors.removeAt(0);
  colors.insert(1, 'Pink');
  
  // Iterating
  for (String color in colors) {
    print('Color: $color');
  }
  
  colors.forEach((color) => print('ForEach: $color'));
  
  // List operations
  List<int> numbers = [1, 2, 3, 4, 5];
  List<int> doubled = numbers.map((n) => n * 2).toList();
  List<int> even = numbers.where((n) => n % 2 == 0).toList();
  int sum = numbers.reduce((value, element) => value + element);
  
  print('Doubled: $doubled'); // [2, 4, 6, 8, 10]
  print('Even: $even'); // [2, 4]
  print('Sum: $sum'); // 15
  
  // 2. SETS (Unique, unordered)
  Set<String> uniqueFruits = {'Apple', 'Banana', 'Orange', 'Apple'};
  print(uniqueFruits); // Output: {Apple, Banana, Orange}
  
  // Set operations
  Set<int> setA = {1, 2, 3, 4};
  Set<int> setB = {3, 4, 5, 6};
  
  print(setA.union(setB)); // {1, 2, 3, 4, 5, 6}
  print(setA.intersection(setB)); // {3, 4}
  print(setA.difference(setB)); // {1, 2}
  
  // 3. MAPS (Key-value pairs)
  Map<String, dynamic> person = {
    'name': 'John',
    'age': 30,
    'isStudent': false,
  };
  
  // Accessing values
  print(person['name']); // Output: John
  print(person['age']); // Output: 30
  
  // Adding/updating
  person['city'] = 'New York';
  person['age'] = 31;
  
  // Checking
  print(person.containsKey('name')); // true
  print(person.containsValue(31)); // true
  
  // Iterating
  person.forEach((key, value) {
    print('$key: $value');
  });
  
  // Map operations
  Map<String, int> prices = {'Apple': 2, 'Banana': 1, 'Orange': 3};
  Map<String, int> doubledPrices = Map.fromEntries(
    prices.entries.map((entry) => 
      MapEntry(entry.key, entry.value * 2))
  );
  print(doubledPrices); // {Apple: 4, Banana: 2, Orange: 6}
}
```

---

## **2. Object-Oriented Programming in Dart**

### **Classes & Objects**
```dart
void main() {
  // Creating objects
  Person person1 = Person('John', 30);
  person1.introduce();
  
  Person person2 = Person('Alice', 25);
  person2.introduce();
  
  // Using named constructor
  Person guest = Person.guest();
  guest.introduce();
  
  // Using getters and setters
  Employee emp = Employee('Bob', 50000);
  print('Salary: \$${emp.salary}');
  emp.salary = 55000; // Using setter
  print('New Salary: \$${emp.salary}');
}

// BASIC CLASS
class Person {
  // Properties (Fields)
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Named constructor
  Person.guest() 
    : name = 'Guest',
      age = 18;
  
  // Method
  void introduce() {
    print('Hello, I am $name and I am $age years old');
  }
}

// CLASS WITH GETTERS & SETTERS
class Employee {
  String name;
  double _salary; // Private variable (convention)
  
  Employee(this.name, this._salary);
  
  // Getter
  double get salary => _salary;
  
  // Setter with validation
  set salary(double value) {
    if (value > 0) {
      _salary = value;
    } else {
      print('Salary must be positive');
    }
  }
  
  // Method
  void work() {
    print('$name is working');
  }
}

// INHERITANCE
class Student extends Person {
  String studentId;
  
  Student(String name, int age, this.studentId) : super(name, age);
  
  @override
  void introduce() {
    print('I am student $name, ID: $studentId');
  }
  
  void study() {
    print('$name is studying');
  }
}

// ABSTRACT CLASS
abstract class Animal {
  String name;
  
  Animal(this.name);
  
  // Abstract method (must be implemented by subclasses)
  void makeSound();
  
  // Concrete method
  void sleep() {
    print('$name is sleeping');
  }
}

class Dog extends Animal {
  Dog(String name) : super(name);
  
  @override
  void makeSound() {
    print('$name says: Woof!');
  }
}

class Cat extends Animal {
  Cat(String name) : super(name);
  
  @override
  void makeSound() {
    print('$name says: Meow!');
  }
}
```

### **Mixins**
```dart
void main() {
  Developer dev = Developer();
  dev.code(); // From Developer class
  dev.walk(); // From Person mixin
  dev.swim(); // From Swimmer mixin
}

// MIXINS (Add functionality without inheritance)
mixin Walker {
  void walk() {
    print('Walking...');
  }
}

mixin Swimmer {
  void swim() {
    print('Swimming...');
  }
}

mixin Coder {
  void code() {
    print('Writing code...');
  }
}

// CLASS USING MIXINS
class Developer with Walker, Swimmer, Coder {
  // Can use walk(), swim(), and code() methods
}

// MIXINS WITH ON KEYWORD (Restrict to specific types)
mixin Jumping on Animal {
  void jump() {
    print('Jumping high!');
  }
}

class Kangaroo extends Animal with Jumping {
  Kangaroo(String name) : super(name);
}
```

### **Interfaces & Abstract Classes**
```dart
void main() {
  Rectangle rect = Rectangle(10, 5);
  print('Area: ${rect.area()}'); // Output: Area: 50
  print('Perimeter: ${rect.perimeter()}'); // Output: Perimeter: 30
  
  Circle circle = Circle(7);
  print('Circle Area: ${circle.area()}');
  print('Circle Perimeter: ${circle.perimeter()}');
}

// INTERFACE (Implicit in Dart - every class is an interface)
abstract class Shape {
  double area();
  double perimeter();
}

// IMPLEMENTING INTERFACE
class Rectangle implements Shape {
  double width;
  double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double area() => width * height;
  
  @override
  double perimeter() => 2 * (width + height);
}

class Circle implements Shape {
  double radius;
  
  Circle(this.radius);
  
  @override
  double area() => 3.14 * radius * radius;
  
  @override
  double perimeter() => 2 * 3.14 * radius;
}

// ABSTRACT CLASS VS INTERFACE
abstract class Vehicle {
  String name;
  
  Vehicle(this.name);
  
  // Can have concrete methods
  void start() {
    print('$name starting...');
  }
  
  // Abstract method
  void move();
}

class Car extends Vehicle {
  Car(String name) : super(name);
  
  @override
  void move() {
    print('$name moving on road');
  }
}

class Boat extends Vehicle {
  Boat(String name) : super(name);
  
  @override
  void move() {
    print('$name sailing on water');
  }
}
```

### **Static Members & Factory Constructors**
```dart
void main() {
  // Static members
  print('Max connections: ${Database.maxConnections}');
  Database.connect();
  
  // Factory constructor
  Logger logger1 = Logger('App');
  Logger logger2 = Logger('App'); // Returns same instance
  print(logger1 == logger2); // Output: true
  
  // Singleton
  AppConfig config1 = AppConfig.instance;
  AppConfig config2 = AppConfig.instance;
  print(config1 == config2); // Output: true
}

// STATIC MEMBERS
class Database {
  static const int maxConnections = 5;
  static int currentConnections = 0;
  
  static void connect() {
    if (currentConnections < maxConnections) {
      currentConnections++;
      print('Connected. Total connections: $currentConnections');
    } else {
      print('Max connections reached');
    }
  }
  
  static void disconnect() {
    currentConnections--;
    print('Disconnected. Total connections: $currentConnections');
  }
}

// FACTORY CONSTRUCTOR
class Logger {
  final String name;
  static final Map<String, Logger> _cache = <String, Logger>{};
  
  // Private constructor
  Logger._internal(this.name);
  
  // Factory constructor
  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }
  
  void log(String message) {
    print('[$name] $message');
  }
}

// SINGLETON PATTERN
class AppConfig {
  static final AppConfig _instance = AppConfig._internal();
  
  String apiUrl = 'https://api.example.com';
  bool isDebug = true;
  
  // Private constructor
  AppConfig._internal();
  
  // Factory constructor returns the singleton instance
  factory AppConfig() {
    return _instance;
  }
  
  // Getter for instance
  static AppConfig get instance => _instance;
}
```

---

## **3. Asynchronous Programming**

### **Future & Async/Await**
```dart
import 'dart:async';

void main() async {
  print('Program started');
  
  // 1. BASIC FUTURE
  Future.delayed(Duration(seconds: 2), () {
    print('After 2 seconds');
  });
  
  // 2. ASYNC/AWAIT
  await fetchUserData();
  
  // 3. HANDLING MULTIPLE FUTURES
  await multipleFutures();
  
  print('Program ended');
}

Future<void> fetchUserData() async {
  print('Fetching user data...');
  
  try {
    // Simulate network delay
    String user = await Future.delayed(
      Duration(seconds: 1),
      () => 'John Doe',
    );
    
    print('User: $user');
    
    // Chain futures
    int age = await getUserAge(user);
    print('Age: $age');
    
  } catch (error) {
    print('Error: $error');
  }
}

Future<int> getUserAge(String name) async {
  await Future.delayed(Duration(seconds: 1));
  return 30;
}

Future<void> multipleFutures() async {
  // Wait for all futures to complete
  List<String> results = await Future.wait([
    fetchData('User'),
    fetchData('Product'),
    fetchData('Order'),
  ]);
  
  print('All data fetched: $results');
  
  // Get first completed future
  String firstResult = await Future.any([
    Future.delayed(Duration(seconds: 3), () => 'Slow'),
    Future.delayed(Duration(seconds: 1), () => 'Fast'),
  ]);
  
  print('First completed: $firstResult');
}

Future<String> fetchData(String type) async {
  await Future.delayed(Duration(seconds: 1));
  return '$type Data';
}

// ERROR HANDLING WITH FUTURES
Future<void> handleErrors() async {
  // Using try-catch
  try {
    String data = await fetchWithError();
    print('Success: $data');
  } catch (e) {
    print('Caught error: $e');
  }
  
  // Using then() and catchError()
  fetchWithError()
    .then((data) => print('Data: $data'))
    .catchError((error) => print('Error: $error'))
    .whenComplete(() => print('Operation complete'));
}

Future<String> fetchWithError() async {
  await Future.delayed(Duration(seconds: 1));
  throw Exception('Network error');
}

// FUTURE BUILDER (Common in Flutter)
Future<String> getMessage() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Hello from Future!';
}

// In Flutter widget:
// FutureBuilder<String>(
//   future: getMessage(),
//   builder: (context, snapshot) {
//     if (snapshot.connectionState == ConnectionState.waiting) {
//       return CircularProgressIndicator();
//     } else if (snapshot.hasError) {
//       return Text('Error: ${snapshot.error}');
//     } else {
//       return Text('Message: ${snapshot.data}');
//     }
//   },
// )
```

### **Streams**
```dart
import 'dart:async';

void main() async {
  print('Stream Demo');
  
  // 1. CREATING STREAMS
  Stream<int> numberStream = countNumbers(5);
  
  // 2. LISTENING TO STREAMS
  StreamSubscription<int> subscription = numberStream.listen(
    (number) {
      print('Received: $number');
    },
    onError: (error) {
      print('Error: $error');
    },
    onDone: () {
      print('Stream completed');
    },
    cancelOnError: false,
  );
  
  // 3. STREAM CONTROLLER (Manual stream)
  StreamController<String> controller = StreamController<String>();
  
  controller.stream.listen((data) {
    print('Controller data: $data');
  });
  
  controller.sink.add('Hello');
  controller.sink.add('World');
  
  // Close when done
  await Future.delayed(Duration(seconds: 1));
  controller.close();
  
  // 4. STREAM TRANSFORMATIONS
  await transformStream();
  
  // 5. BROADCAST STREAM (Multiple listeners)
  await broadcastStream();
}

// GENERATOR FUNCTION (async*)
Stream<int> countNumbers(int max) async* {
  for (int i = 1; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emit value
  }
}

// STREAM TRANSFORMATIONS
Future<void> transformStream() async {
  Stream<int> numbers = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  // Map transformation
  Stream<String> mapped = numbers.map((n) => 'Number: $n');
  await for (String item in mapped) {
    print('Mapped: $item');
  }
  
  // Where filter
  Stream<int> evens = numbers.where((n) => n % 2 == 0);
  print('Even numbers:');
  await for (int even in evens) {
    print(even);
  }
  
  // Take first N items
  Stream<int> firstThree = numbers.take(3);
  print('First three:');
  await for (int num in firstThree) {
    print(num);
  }
  
  // Reduce
  Stream<int> numberStream = Stream.fromIterable([1, 2, 3, 4]);
  int sum = await numberStream.reduce((a, b) => a + b);
  print('Sum: $sum');
}

// BROADCAST STREAM
Future<void> broadcastStream() async {
  StreamController<int> controller = StreamController<int>.broadcast();
  
  // First listener
  controller.stream.listen((data) {
    print('Listener 1: $data');
  });
  
  // Second listener
  controller.stream.listen((data) {
    print('Listener 2: $data');
  });
  
  controller.sink.add(100);
  controller.sink.add(200);
  
  await Future.delayed(Duration(seconds: 1));
  controller.close();
}

// STREAM BUILDER (Common in Flutter)
Stream<int> getCounterStream() async* {
  for (int i = 1; i <= 10; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

// In Flutter widget:
// StreamBuilder<int>(
//   stream: getCounterStream(),
//   builder: (context, snapshot) {
//     if (snapshot.hasData) {
//       return Text('Count: ${snapshot.data}');
//     } else if (snapshot.hasError) {
//       return Text('Error: ${snapshot.error}');
//     } else {
//       return CircularProgressIndicator();
//     }
//   },
// )
```

### **Isolates (Concurrency)**
```dart
import 'dart:isolate';

void main() async {
  print('Main isolate started');
  
  // 1. SPAWNING AN ISOLATE
  ReceivePort receivePort = ReceivePort();
  
  await Isolate.spawn(
    heavyComputation, // Function to run in isolate
    receivePort.sendPort, // SendPort to communicate back
  );
  
  // Listen for messages from isolate
  receivePort.listen((message) {
    print('Received from isolate: $message');
    receivePort.close();
  });
  
  // 2. COMPUTE FUNCTION (Simpler API)
  int result = await compute(calculateFactorial, 10);
  print('Factorial computed: $result');
  
  print('Main isolate continuing...');
}

// Function that runs in isolate
void heavyComputation(SendPort sendPort) {
  print('Isolate started computation');
  
  // Simulate heavy computation
  int sum = 0;
  for (int i = 0; i < 1000000000; i++) {
    sum += i;
  }
  
  // Send result back to main isolate
  sendPort.send('Computation complete. Sum: $sum');
}

// Function for compute()
int calculateFactorial(int n) {
  int result = 1;
  for (int i = 1; i <= n; i++) {
    result *= i;
  }
  return result;
}

// Simpler way using compute()
Future<int> compute<T, R>(FutureOr<R> Function(T message) function, T message) async {
  final receivePort = ReceivePort();
  
  await Isolate.spawn(
    _spawn,
    _IsolateData(function, message, receivePort.sendPort),
  );
  
  return await receivePort.first as R;
}

class _IsolateData<T, R> {
  final FutureOr<R> Function(T) function;
  final T message;
  final SendPort sendPort;
  
  _IsolateData(this.function, this.message, this.sendPort);
}

void _spawn<T, R>(_IsolateData<T, R> data) async {
  final result = await data.function(data.message);
  data.sendPort.send(result);
}
```

---

## **4. Flutter Widgets & UI Basics**

### **StatelessWidget**
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  final String title = 'Home Page';
  final int count = 10;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'Welcome to Flutter!',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 20),
            Text('Count: $count'),
            SizedBox(height: 20),
            CustomButton(text: 'Click Me'),
          ],
        ),
      ),
    );
  }
}

// Reusable Stateless Widget
class CustomButton extends StatelessWidget {
  final String text;
  final Color color;
  
  const CustomButton({
    required this.text,
    this.color = Colors.blue,
    Key? key,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Button clicked!')),
        );
      },
      style: ElevatedButton.styleFrom(
        primary: color,
        padding: EdgeInsets.symmetric(horizontal: 30, vertical: 15),
      ),
      child: Text(
        text,
        style: TextStyle(fontSize: 16),
      ),
    );
  }
}
```

### **StatefulWidget**
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(CounterApp());
}

class CounterApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter App',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: CounterPage(),
    );
  }
}

class CounterPage extends StatefulWidget {
  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  // STATE VARIABLES
  int _counter = 0;
  String _message = 'Click the buttons!';
  
  // METHODS THAT UPDATE STATE
  void _incrementCounter() {
    setState(() {
      _counter++;
      _message = 'Increased to $_counter';
    });
  }
  
  void _decrementCounter() {
    setState(() {
      _counter--;
      _message = 'Decreased to $_counter';
    });
  }
  
  void _resetCounter() {
    setState(() {
      _counter = 0;
      _message = 'Reset to 0';
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Counter App'),
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: _resetCounter,
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'Current Count:',
              style: TextStyle(fontSize: 20),
            ),
            SizedBox(height: 10),
            Text(
              '$_counter',
              style: TextStyle(fontSize: 60, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 20),
            Text(
              _message,
              style: TextStyle(fontSize: 16, color: Colors.grey),
            ),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _decrementCounter,
                  child: Icon(Icons.remove),
                  style: ElevatedButton.styleFrom(
                    shape: CircleBorder(),
                    padding: EdgeInsets.all(20),
                  ),
                ),
                SizedBox(width: 20),
                ElevatedButton(
                  onPressed: _incrementCounter,
                  child: Icon(Icons.add),
                  style: ElevatedButton.styleFrom(
                    shape: CircleBorder(),
                    padding: EdgeInsets.all(20),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### **Common Widgets Examples**
```dart
import 'package:flutter/material.dart';

class WidgetsDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Widgets Demo')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. TEXT WIDGETS
            Text(
              'Text Widgets',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Text(
              'Regular text',
              style: TextStyle(fontSize: 16),
            ),
            Text(
              'Bold text',
              style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
            ),
            Text(
              'Colored text',
              style: TextStyle(fontSize: 16, color: Colors.blue),
            ),
            RichText(
              text: TextSpan(
                text: 'RichText with ',
                style: TextStyle(color: Colors.black, fontSize: 16),
                children: [
                  TextSpan(
                    text: 'different',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  TextSpan(text: ' styles'),
                ],
              ),
            ),
            
            SizedBox(height: 30),
            
            // 2. BUTTON WIDGETS
            Text(
              'Button Widgets',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            ElevatedButton(
              onPressed: () {},
              child: Text('Elevated Button'),
            ),
            SizedBox(height: 10),
            OutlinedButton(
              onPressed: () {},
              child: Text('Outlined Button'),
            ),
            SizedBox(height: 10),
            TextButton(
              onPressed: () {},
              child: Text('Text Button'),
            ),
            SizedBox(height: 10),
            IconButton(
              onPressed: () {},
              icon: Icon(Icons.favorite),
              color: Colors.red,
            ),
            
            SizedBox(height: 30),
            
            // 3. INPUT WIDGETS
            Text(
              'Input Widgets',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            TextField(
              decoration: InputDecoration(
                labelText: 'Username',
                hintText: 'Enter your username',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.person),
              ),
            ),
            SizedBox(height: 10),
            TextField(
              decoration: InputDecoration(
                labelText: 'Password',
                hintText: 'Enter your password',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.lock),
                suffixIcon: Icon(Icons.visibility),
              ),
              obscureText: true,
            ),
            
            SizedBox(height: 30),
            
            // 4. CONTAINER & BOX DECORATION
            Text(
              'Containers & BoxDecoration',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Container(
              padding: EdgeInsets.all(20),
              margin: EdgeInsets.symmetric(vertical: 10),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(10),
                border: Border.all(color: Colors.blue, width: 2),
                boxShadow: [
                  BoxShadow(
                    color: Colors.grey.withOpacity(0.5),
                    spreadRadius: 2,
                    blurRadius: 5,
                    offset: Offset(0, 3),
                  ),
                ],
              ),
              child: Text('Styled Container'),
            ),
            
            SizedBox(height: 30),
            
            // 5. ROW & COLUMN
            Text(
              'Row & Column Layouts',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Container(width: 50, height: 50, color: Colors.red),
                Container(width: 50, height: 50, color: Colors.green),
                Container(width: 50, height: 50, color: Colors.blue),
              ],
            ),
            SizedBox(height: 20),
            Column(
              children: [
                Container(height: 50, color: Colors.red, width: double.infinity),
                Container(height: 50, color: Colors.green, width: double.infinity),
                Container(height: 50, color: Colors.blue, width: double.infinity),
              ],
            ),
            
            SizedBox(height: 30),
            
            // 6. STACK & POSITIONED
            Text(
              'Stack & Positioned',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Container(
              height: 150,
              child: Stack(
                children: [
                  Container(
                    width: double.infinity,
                    height: 150,
                    color: Colors.blue[100],
                  ),
                  Positioned(
                    left: 20,
                    top: 20,
                    child: Container(
                      width: 50,
                      height: 50,
                      color: Colors.red,
                    ),
                  ),
                  Positioned(
                    right: 20,
                    bottom: 20,
                    child: Container(
                      width: 50,
                      height: 50,
                      color: Colors.green,
                    ),
                  ),
                  Center(
                    child: Container(
                      width: 50,
                      height: 50,
                      color: Colors.yellow,
                    ),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 30),
            
            // 7. LISTVIEW
            Text(
              'ListView',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Container(
              height: 150,
              child: ListView(
                children: [
                  ListTile(
                    leading: Icon(Icons.star),
                    title: Text('First Item'),
                    subtitle: Text('Subtitle'),
                    trailing: Icon(Icons.arrow_forward),
                    onTap: () {},
                  ),
                  Divider(),
                  ListTile(
                    leading: Icon(Icons.star),
                    title: Text('Second Item'),
                    trailing: Icon(Icons.arrow_forward),
                  ),
                  Divider(),
                  ListTile(
                    leading: Icon(Icons.star),
                    title: Text('Third Item'),
                    trailing: Icon(Icons.arrow_forward),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 30),
            
            // 8. GRIDVIEW
            Text(
              'GridView',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            GridView.count(
              shrinkWrap: true,
              physics: NeverScrollableScrollPhysics(),
              crossAxisCount: 3,
              crossAxisSpacing: 10,
              mainAxisSpacing: 10,
              children: List.generate(9, (index) {
                return Container(
                  color: Colors.blue[(index + 1) * 100],
                  child: Center(
                    child: Text('Item $index'),
                  ),
                );
              }),
            ),
            
            SizedBox(height: 30),
          ],
        ),
      ),
    );
  }
}
```

### **Layout Widgets Deep Dive**
```dart
import 'package:flutter/material.dart';

class LayoutDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Layout Demo')),
      body: ListView(
        children: [
          // 1. EXPANDED & FLEXIBLE
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text('Expanded & Flexible', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 10),
                Container(
                  height: 100,
                  child: Row(
                    children: [
                      Container(width: 50, color: Colors.red, child: Center(child: Text('Fixed'))),
                      Expanded(
                        flex: 2,
                        child: Container(color: Colors.green, child: Center(child: Text('Expanded (flex:2)'))),
                      ),
                      Flexible(
                        flex: 1,
                        fit: FlexFit.loose,
                        child: Container(color: Colors.blue, child: Center(child: Text('Flexible (flex:1)'))),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
          
          Divider(),
          
          // 2. SIZEDBOX & SPACER
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text('SizedBox & Spacer', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 10),
                Row(
                  children: [
                    Container(width: 50, height: 50, color: Colors.red),
                    SizedBox(width: 20), // Fixed space
                    Container(width: 50, height: 50, color: Colors.green),
                    Spacer(), // Takes all available space
                    Container(width: 50, height: 50, color: Colors.blue),
                  ],
                ),
              ],
            ),
          ),
          
          Divider(),
          
          // 3. WRAP WIDGET
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text('Wrap Widget', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 10),
                Wrap(
                  spacing: 8, // Horizontal space between items
                  runSpacing: 8, // Vertical space between lines
                  children: List.generate(10, (index) {
                    return Chip(
                      label: Text('Tag $index'),
                      backgroundColor: Colors.blue[100],
                    );
                  }),
                ),
              ],
            ),
          ),
          
          Divider(),
          
          // 4. MEDIAQUERY
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text('MediaQuery', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 10),
                Container(
                  padding: EdgeInsets.all(16),
                  color: Colors.grey[200],
                  child: Builder(
                    builder: (context) {
                      final size = MediaQuery.of(context).size;
                      final orientation = MediaQuery.of(context).orientation;
                      final padding = MediaQuery.of(context).padding;
                      
                      return Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text('Screen Width: ${size.width.toStringAsFixed(1)}'),
                          Text('Screen Height: ${size.height.toStringAsFixed(1)}'),
                          Text('Orientation: $orientation'),
                          Text('Top Padding (Safe Area): ${padding.top}'),
                        ],
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
          
          Divider(),
          
          // 5. LAYOUTBUILDER
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text('LayoutBuilder', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 10),
                Container(
                  height: 200,
                  color: Colors.grey[200],
                  child: LayoutBuilder(
                    builder: (context, constraints) {
                      return Container(
                        width: constraints.maxWidth,
                        height: constraints.maxHeight,
                        color: Colors.blue[100],
                        child: Center(
                          child: Text(
                            'Max Width: ${constraints.maxWidth}\nMax Height: ${constraints.maxHeight}',
                            textAlign: TextAlign.center,
                          ),
                        ),
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
          
          Divider(),
          
          // 6. INTRINSIC HEIGHT/WIDTH
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text('IntrinsicHeight/Width', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                SizedBox(height: 10),
                IntrinsicHeight(
                  child: Row(
                    crossAxisAlignment: CrossAxisAlignment.stretch,
                    children: [
                      Container(
                        width: 50,
                        color: Colors.red,
                        child: Center(child: Text('Short')),
                      ),
                      Container(
                        width: 50,
                        color: Colors.green,
                        height: 100,
                        child: Center(child: Text('Tall')),
                      ),
                      Container(
                        width: 50,
                        color: Colors.blue,
                        child: Center(child: Text('Medium\nText')),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### **Navigation & Routing**
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(NavigationApp());
}

class NavigationApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigation Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      initialRoute: '/',
      routes: {
        '/': (context) => HomeScreen(),
        '/details': (context) => DetailsScreen(),
        '/profile': (context) => ProfileScreen(),
      },
      onGenerateRoute: (settings) {
        // Handle dynamic routes
        if (settings.name == '/user/:id') {
          final id = settings.name!.split('/').last;
          return MaterialPageRoute(
            builder: (context) => UserScreen(userId: id),
          );
        }
        return null;
      },
    );
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                // 1. PUSH (Go to new screen)
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => DetailsScreen(),
                    settings: RouteSettings(arguments: 'Hello from Home'),
                  ),
                );
              },
              child: Text('Push to Details'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // 2. PUSH NAMED (Using route names)
                Navigator.pushNamed(
                  context,
                  '/details',
                  arguments: 'Named Route Data',
                );
              },
              child: Text('Push Named Route'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () async {
                // 3. PUSH WITH RESULT
                final result = await Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => SelectionScreen()),
                );
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('You selected: $result')),
                );
              },
              child: Text('Get Result from Screen'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // 4. REPLACE (Replace current screen)
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(builder: (context) => DetailsScreen()),
                );
              },
              child: Text('Replace with Details'),
            ),
          ],
        ),
      ),
    );
  }
}

class DetailsScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments;
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Details'),
        leading: IconButton(
          icon: Icon(Icons.arrow_back),
          onPressed: () {
            Navigator.pop(context);
          },
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Arguments: $args'),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // POP (Go back)
                Navigator.pop(context, 'Returned from Details');
              },
              child: Text('Go Back'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // POP UNTIL (Go back multiple screens)
                Navigator.popUntil(context, (route) => route.isFirst);
              },
              child: Text('Go to Home'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // PUSH AND REMOVE UNTIL (Clear stack)
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(builder: (context) => HomeScreen()),
                  (route) => false,
                );
              },
              child: Text('Clear Stack and Go Home'),
            ),
          ],
        ),
      ),
    );
  }
}

class SelectionScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Select an option')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, 'Option A');
              },
              child: Text('Option A'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, 'Option B');
              },
              child: Text('Option B'),
            ),
          ],
        ),
      ),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Profile')),
      body: Center(
        child: Text('Profile Screen'),
      ),
    );
  }
}

class UserScreen extends StatelessWidget {
  final String userId;
  
  UserScreen({required this.userId});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
      body: Center(
        child: Text('Details for user $userId'),
      ),
    );
  }
}
```

---

## **5. Form Handling & Validation**
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(FormApp());
}

class FormApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Form Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: FormScreen(),
    );
  }
}

class FormScreen extends StatefulWidget {
  @override
  _FormScreenState createState() => _FormScreenState();
}

class _FormScreenState extends State<FormScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  
  String _selectedGender = 'Male';
  bool _termsAccepted = false;
  DateTime? _selectedDate;
  
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
  
  Future<void> _selectDate(BuildContext context) async {
    final DateTime? picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(1900),
      lastDate: DateTime.now(),
    );
    if (picked != null && picked != _selectedDate) {
      setState(() {
        _selectedDate = picked;
      });
    }
  }
  
  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      if (!_termsAccepted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Please accept terms and conditions')),
        );
        return;
      }
      
      // Form is valid
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Form submitted successfully!')),
      );
      
      print('Name: ${_nameController.text}');
      print('Email: ${_emailController.text}');
      print('Gender: $_selectedGender');
      print('Date: $_selectedDate');
      
      // Clear form
      _formKey.currentState!.reset();
      setState(() {
        _termsAccepted = false;
        _selectedDate = null;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Form Demo')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(20),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // NAME FIELD
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: 'Full Name',
                  hintText: 'Enter your full name',
                  prefixIcon: Icon(Icons.person),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  if (value.length < 3) {
                    return 'Name must be at least 3 characters';
                  }
                  return null;
                },
              ),
              SizedBox(height: 20),
              
              // EMAIL FIELD
              TextFormField(
                controller: _emailController,
                decoration: InputDecoration(
                  labelText: 'Email Address',
                  hintText: 'Enter your email',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.emailAddress,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
              ),
              SizedBox(height: 20),
              
              // PASSWORD FIELD
              TextFormField(
                controller: _passwordController,
                decoration: InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter your password',
                  prefixIcon: Icon(Icons.lock),
                  suffixIcon: Icon(Icons.visibility),
                  border: OutlineInputBorder(),
                ),
                obscureText: true,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter password';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
              ),
              SizedBox(height: 20),
              
              // GENDER RADIO BUTTONS
              Text('Gender', style: TextStyle(fontWeight: FontWeight.bold)),
              Row(
                children: [
                  Radio(
                    value: 'Male',
                    groupValue: _selectedGender,
                    onChanged: (value) {
                      setState(() {
                        _selectedGender = value.toString();
                      });
                    },
                  ),
                  Text('Male'),
                  SizedBox(width: 20),
                  Radio(
                    value: 'Female',
                    groupValue: _selectedGender,
                    onChanged: (value) {
                      setState(() {
                        _selectedGender = value.toString();
                      });
                    },
                  ),
                  Text('Female'),
                ],
              ),
              SizedBox(height: 20),
              
              // DATE PICKER
              Text('Date of Birth', style: TextStyle(fontWeight: FontWeight.bold)),
              SizedBox(height: 10),
              ElevatedButton(
                onPressed: () => _selectDate(context),
                child: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Icon(Icons.calendar_today),
                    SizedBox(width: 10),
                    Text(_selectedDate == null 
                      ? 'Select Date' 
                      : 'Selected: ${_selectedDate!.toLocal()}'.split(' ')[0]),
                  ],
                ),
              ),
              SizedBox(height: 20),
              
              // CHECKBOX
              Row(
                children: [
                  Checkbox(
                    value: _termsAccepted,
                    onChanged: (value) {
                      setState(() {
                        _termsAccepted = value!;
                      });
                    },
                  ),
                  Expanded(
                    child: Text(
                      'I agree to the terms and conditions',
                      style: TextStyle(fontSize: 14),
                    ),
                  ),
                ],
              ),
              SizedBox(height: 20),
              
              // DROPDOWN
              Text('Country', style: TextStyle(fontWeight: FontWeight.bold)),
              SizedBox(height: 10),
              DropdownButtonFormField<String>(
                value: 'USA',
                decoration: InputDecoration(
                  border: OutlineInputBorder(),
                  contentPadding: EdgeInsets.symmetric(horizontal: 10, vertical: 15),
                ),
                items: ['USA', 'Canada', 'UK', 'Australia', 'India']
                  .map((country) => DropdownMenuItem(
                    value: country,
                    child: Text(country),
                  ))
                  .toList(),
                onChanged: (value) {
                  print('Selected: $value');
                },
              ),
              SizedBox(height: 30),
              
              // SUBMIT BUTTON
              Center(
                child: ElevatedButton(
                  onPressed: _submitForm,
                  style: ElevatedButton.styleFrom(
                    padding: EdgeInsets.symmetric(horizontal: 40, vertical: 15),
                  ),
                  child: Text('Submit Form'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## **6. State Management Basics**

### **Using setState()**
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(StateManagementApp());
}

class StateManagementApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'State Management',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: TodoApp(),
    );
  }
}

class TodoApp extends StatefulWidget {
  @override
  _TodoAppState createState() => _TodoAppState();
}

class _TodoAppState extends State<TodoApp> {
  final List<TodoItem> _todos = [];
  final _textController = TextEditingController();
  
  void _addTodo() {
    if (_textController.text.isNotEmpty) {
      setState(() {
        _todos.add(TodoItem(
          title: _textController.text,
          isCompleted: false,
        ));
        _textController.clear();
      });
    }
  }
  
  void _toggleTodo(int index) {
    setState(() {
      _todos[index].isCompleted = !_todos[index].isCompleted;
    });
  }
  
  void _removeTodo(int index) {
    setState(() {
      _todos.removeAt(index);
    });
  }
  
  @override
  Widget build(BuildContext context) {
    final completedCount = _todos.where((todo) => todo.isCompleted).length;
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Todo App (${_todos.length} items)'),
        actions: [
          Chip(
            label: Text('$completedCount/${_todos.length}'),
            backgroundColor: Colors.white,
          ),
        ],
      ),
      body: Column(
        children: [
          // INPUT SECTION
          Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _textController,
                    decoration: InputDecoration(
                      hintText: 'Add a new todo',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (_) => _addTodo(),
                  ),
                ),
                SizedBox(width: 10),
                ElevatedButton(
                  onPressed: _addTodo,
                  child: Icon(Icons.add),
                ),
              ],
            ),
          ),
          
          // TODO LIST
          Expanded(
            child: ListView.builder(
              itemCount: _todos.length,
              itemBuilder: (context, index) {
                final todo = _todos[index];
                return Dismissible(
                  key: UniqueKey(),
                  background: Container(color: Colors.red),
                  onDismissed: (_) => _removeTodo(index),
                  child: ListTile(
                    leading: Checkbox(
                      value: todo.isCompleted,
                      onChanged: (_) => _toggleTodo(index),
                    ),
                    title: Text(
                      todo.title,
                      style: TextStyle(
                        decoration: todo.isCompleted 
                          ? TextDecoration.lineThrough 
                          : TextDecoration.none,
                        color: todo.isCompleted ? Colors.grey : Colors.black,
                      ),
                    ),
                    trailing: IconButton(
                      icon: Icon(Icons.delete, color: Colors.red),
                      onPressed: () => _removeTodo(index),
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class TodoItem {
  String title;
  bool isCompleted;
  
  TodoItem({required this.title, required this.isCompleted});
}
```
