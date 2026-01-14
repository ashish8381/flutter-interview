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
