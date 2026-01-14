# **Flutter Core**

## **1. Flutter Widget Lifecycle**

### **StatelessWidget Lifecycle**
```dart
class MyStatelessWidget extends StatelessWidget {
  // Constructor → build() → disposed
  
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### **StatefulWidget Lifecycle**
```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  // 1. INITIALIZATION
  @override
  void initState() {
    super.initState();
    print('Widget created - setup listeners, init data');
  }
  
  // 2. BUILD/UPDATE CYCLES
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('Called when dependencies change (InheritedWidget)');
  }
  
  @override
  void didUpdateWidget(MyWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('Parent widget rebuilt with new configuration');
  }
  
  @override
  Widget build(BuildContext context) {
    print('Rebuilding UI');
    return Container();
  }
  
  // 3. DISPOSAL
  @override
  void dispose() {
    print('Widget removed - cleanup resources');
    super.dispose();
  }
}
```

## **2. Difference between InheritedWidget and Provider**
```dart
// INHERITEDWIDGET (Low-level, manual)
class MyInheritedWidget extends InheritedWidget {
  final String data;
  
  MyInheritedWidget({required this.data, required Widget child}) 
    : super(child: child);
  
  static MyInheritedWidget of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>()!;
  }
  
  @override
  bool updateShouldNotify(MyInheritedWidget oldWidget) {
    return data != oldWidget.data;
  }
}

// PROVIDER (Wrapper on InheritedWidget, simpler)
Provider(
  create: (context) => MyModel(),
  child: Consumer<MyModel>(
    builder: (context, model, child) => Text(model.data),
  ),
);
```

**Key Difference**: Provider simplifies InheritedWidget with less boilerplate, dependency injection, and better state management patterns.

## **3. Hot Reload vs Hot Restart**
```dart
// Example showing the difference
class CounterApp extends StatelessWidget {
  // With Hot Reload (preserves state):
  // Change this color, app keeps counter value
  Color bgColor = Colors.blue; // ← Change to Colors.red
  
  // With Hot Restart (resets state):
  // Counter resets to 0, full app reload
}

// Hot Reload: 
// - Updates code, preserves state (seconds)
// - For UI changes, minor logic

// Hot Restart:
// - Full app reload, resets state (~10-20 seconds)
// - For plugin changes, initial state changes
```

## **4. Keys in Flutter - Types & Examples**
```dart
// 1. VALUE KEY (identifies widgets by value)
ValueKey<int>(1);

// 2. OBJECT KEY (identifies by object identity)
ObjectKey(myObject);

// 3. UNIQUE KEY (unique each build - usually avoid)
UniqueKey();

// 4. GLOBAL KEY (access widget state globally)
GlobalKey<FormState> _formKey = GlobalKey();
_formKey.currentState?.save();

// 5. PAGESTORAGE KEY (preserves scroll position)
PageStorageKey<String>('page1');

// Usage Example - Essential in list reordering
ListView.builder(
  itemBuilder: (context, index) {
    return ListTile(
      key: ValueKey(users[index].id), // Preserves state during reorder
      title: Text(users[index].name),
    );
  },
);
```

## **5. Difference between Expanded and Flexible**
```dart
Row(
  children: [
    // FLEXIBLE - Can be flexible, but doesn't have to fill
    Flexible(
      flex: 1, // Relative weight
      fit: FlexFit.loose, // Can be smaller than available space
      child: Container(color: Colors.red, height: 50),
    ),
    
    // EXPANDED = Flexible with FlexFit.tight (must fill)
    Expanded(  // flex: 1 (default), fit: FlexFit.tight
      child: Container(color: Colors.blue), // Takes all remaining space
    ),
  ],
);
```

## **6. What is MediaQuery?**
```dart
class ResponsiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Access device information
    final size = MediaQuery.of(context).size;
    final orientation = MediaQuery.of(context).orientation;
    final padding = MediaQuery.of(context).padding;
    final textScale = MediaQuery.of(context).textScaleFactor;
    
    return Container(
      width: size.width * 0.8, // 80% of screen width
      height: orientation == Orientation.portrait ? 100 : 50,
      padding: EdgeInsets.only(top: padding.top), // Safe area
      child: Text(
        'Responsive Text',
        style: TextStyle(fontSize: 16 / textScale), // Adjust for accessibility
      ),
    );
  }
}
```

## **7. How does Flutter achieve 60fps?**
```dart
// Flutter's rendering optimization:
class OptimizedWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return const Scaffold(  // 1. Use const constructors
      body: CustomScrollView(  // 2. Efficient scrolling
        slivers: [
          SliverAppBar(),
          SliverList(  // 3. Lazy loading
            delegate: SliverChildBuilderDelegate(
              (context, index) => const ListItem(), // Const widget
              childCount: 1000,
            ),
          ),
        ],
      ),
    );
  }
}

// Key techniques:
// - Skia 2D rendering engine (GPU accelerated)
// - Widget caching (Element tree reuse)
// - Layout calculation in single pass
// - Layer tree optimization
// - Jank-free scheduling
```

---

# **Architecture (MVC / MVVM / MVP)**

## **1. MVC in Flutter Example**
```dart
// MODEL
class User {
  final String name;
  final String email;
  
  User({required this.name, required this.email});
}

// CONTROLLER
class UserController {
  final UserRepository _repository = UserRepository();
  
  Future<User> getUser(int id) => _repository.fetchUser(id);
  
  void updateUserName(User user, String newName) {
    // Business logic and validation
    if (newName.length > 2) {
      user = User(name: newName, email: user.email);
      _repository.updateUser(user);
    }
  }
}

// VIEW
class UserView extends StatelessWidget {
  final UserController controller;
  
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<User>(
      future: controller.getUser(1),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          return Text(snapshot.data!.name);
        }
        return CircularProgressIndicator();
      },
    );
  }
}
```

## **2. How MVVM differs from MVC**
```dart
// MVVM EXAMPLE
// MODEL (same as MVC)
class User { ... }

// VIEWMODEL (replaces Controller, handles presentation logic)
class UserViewModel extends ChangeNotifier {
  User? _user;
  bool _loading = false;
  
  User? get user => _user;
  bool get loading => _loading;
  
  Future<void> loadUser() async {
    _loading = true;
    notifyListeners();
    
    _user = await UserRepository().fetchUser(1);
    _loading = false;
    notifyListeners();
  }
  
  void updateName(String name) {
    _user = User(name: name, email: _user!.email);
    notifyListeners(); // Auto-updates View
  }
}

// VIEW (observes ViewModel)
class UserView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => UserViewModel(),
      child: Consumer<UserViewModel>(
        builder: (context, viewModel, child) {
          if (viewModel.loading) return CircularProgressIndicator();
          return Text(viewModel.user?.name ?? 'No user');
        },
      ),
    );
  }
}
```

**Key Difference**: MVVM has **data binding** (View observes ViewModel automatically), while MVC requires manual UI updates.

## **3. Architecture Preference & Why**
```dart
// I prefer BLoC/Cubit for complex apps:
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  
  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

// Why:
// 1. Clear separation (Business Logic Component)
// 2. Testable (pure functions)
// 3. Predictable state changes
// 4. Works well with Streams
// 5. Great dev tools (BlocObserver)

// For simple apps: Provider/ChangeNotifier
// For medium apps: Riverpod
// For complex apps: BLoC/Redux
```

## **4. Separating UI and Business Logic**
```dart
// BAD: Mixed concerns
class BadWidget extends StatefulWidget {
  @override
  _BadWidgetState createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  List<User> users = [];
  
  Future<void> fetchUsers() async {
    // API call mixed with UI code
    final response = await http.get('api/users');
    // Business logic mixed
    users = parseUsers(response.body);
    setState(() {});
  }
  
  @override Widget build(BuildContext context) { ... }
}

// GOOD: Separated concerns
// lib/
//   data/        ← API calls, repositories
//   models/      ← Data classes
//   providers/   ← State management
//   services/    ← Business logic
//   views/       ← UI components

// View (only UI)
class UserListView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final users = context.watch<UserProvider>().users;
    return ListView.builder(...);
  }
}

// Provider (business logic)
class UserProvider extends ChangeNotifier {
  final UserRepository _repository;
  List<User> _users = [];
  
  Future<void> loadUsers() async {
    _users = await _repository.fetchUsers();
    notifyListeners();
  }
}
```

---

# **API Calling & Caching**

## **1. Calling REST APIs in Flutter**
```dart
import 'package:http/http.dart' as http;

class ApiService {
  static const String baseUrl = 'https://api.example.com';
  
  Future<User> fetchUser(int id) async {
    try {
      final response = await http.get(
        Uri.parse('$baseUrl/users/$id'),
        headers: {'Authorization': 'Bearer $token'},
      );
      
      if (response.statusCode == 200) {
        return User.fromJson(jsonDecode(response.body));
      } else {
        throw Exception('Failed to load user');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
  
  Future<User> createUser(User user) async {
    final response = await http.post(
      Uri.parse('$baseUrl/users'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(user.toJson()),
    );
    
    return User.fromJson(jsonDecode(response.body));
  }
}
```

## **2. Difference between http and Dio**
```dart
// HTTP PACKAGE (Simple)
final response = await http.get(Uri.parse(url));

// DIO PACKAGE (Feature-rich)
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: 5000,
  receiveTimeout: 3000,
));

// Dio advantages:
// 1. Interceptors (logging, auth, error handling)
dio.interceptors.add(LogInterceptor());
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    options.headers['Auth'] = token;
    return handler.next(options);
  },
));

// 2. FormData (easy file upload)
FormData formData = FormData.fromMap({
  'file': await MultipartFile.fromFile('path/to/file'),
});

// 3. Request/response transformers
// 4. Cancel requests
final cancelToken = CancelToken();
dio.get(url, cancelToken: cancelToken);
cancelToken.cancel(); // Cancel request
```

## **3. Caching API Responses**
```dart
import 'package:shared_preferences/shared_preferences.dart';
import 'dart:convert';

class CacheManager {
  // 1. IN-MEMORY CACHE
  final Map<String, dynamic> _memoryCache = {};
  
  // 2. DISK CACHE (SharedPreferences)
  Future<void> saveToCache(String key, dynamic data, Duration duration) async {
    final prefs = await SharedPreferences.getInstance();
    final cacheItem = {
      'data': data,
      'timestamp': DateTime.now().add(duration).toIso8601String(),
    };
    await prefs.setString(key, jsonEncode(cacheItem));
    _memoryCache[key] = cacheItem;
  }
  
  Future<dynamic> getFromCache(String key) async {
    // Check memory first
    if (_memoryCache.containsKey(key)) {
      final item = _memoryCache[key];
      if (DateTime.parse(item['timestamp']).isAfter(DateTime.now())) {
        return item['data'];
      }
    }
    
    // Check disk
    final prefs = await SharedPreferences.getInstance();
    final cached = prefs.getString(key);
    if (cached != null) {
      final item = jsonDecode(cached);
      if (DateTime.parse(item['timestamp']).isAfter(DateTime.now())) {
        _memoryCache[key] = item;
        return item['data'];
      }
    }
    
    return null; // Cache miss
  }
}

// Usage with API call
Future<User> getUserWithCache(int id) async {
  final cacheKey = 'user_$id';
  final cached = await CacheManager().getFromCache(cacheKey);
  
  if (cached != null) {
    return User.fromJson(cached);
  }
  
  final user = await ApiService().fetchUser(id);
  await CacheManager().saveToCache(cacheKey, user.toJson(), Duration(hours: 1));
  return user;
}
```

## **4. GraphQL vs REST**
```dart
// REST EXAMPLE (Multiple endpoints)
GET /users/1
GET /users/1/posts
GET /users/1/friends

// GRAPHQL EXAMPLE (Single endpoint, flexible queries)
query {
  user(id: 1) {
    name
    email
    posts {
      title
      content
    }
    friends {
      name
    }
  }
}

// Flutter GraphQL Implementation
import 'package:graphql_flutter/graphql_flutter.dart';

final HttpLink httpLink = HttpLink('https://api.example.com/graphql');

final client = GraphQLClient(
  cache: GraphQLCache(),
  link: httpLink,
);

final query = gql('''
  query GetUser(\$id: ID!) {
    user(id: \$id) {
      id
      name
      email
    }
  }
''');

final result = await client.query(QueryOptions(
  document: query,
  variables: {'id': '1'},
));

// Differences:
// REST: Fixed endpoints, over/under-fetching
// GraphQL: Single endpoint, precise data fetching, real-time (subscriptions)
```

## **5. Handling API Loading & Error States**
```dart
class UserScreen extends StatefulWidget {
  @override
  _UserScreenState createState() => _UserScreenState();
}

class _UserScreenState extends State<UserScreen> {
  late Future<User> futureUser;
  
  @override
  void initState() {
    super.initState();
    futureUser = ApiService().fetchUser(1);
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: FutureBuilder<User>(
        future: futureUser,
        builder: (context, snapshot) {
          // 1. LOADING STATE
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          
          // 2. ERROR STATE
          else if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.error, size: 50, color: Colors.red),
                  SizedBox(height: 20),
                  Text('Error: ${snapshot.error}'),
                  SizedBox(height: 20),
                  ElevatedButton(
                    onPressed: () {
                      setState(() {
                        futureUser = ApiService().fetchUser(1);
                      });
                    },
                    child: Text('Retry'),
                  ),
                ],
              ),
            );
          }
          
          // 3. SUCCESS STATE
          else if (snapshot.hasData) {
            return UserProfile(user: snapshot.data!);
          }
          
          // 4. EMPTY STATE
          return Center(child: Text('No user found'));
        },
      ),
    );
  }
}
```

---

# **State Management**

## **1. Difference between setState, Provider, Bloc**
```dart
// 1. SETSTATE (Local state)
class CounterApp extends StatefulWidget {
  @override
  _CounterAppState createState() => _CounterAppState();
}

class _CounterAppState extends State<CounterApp> {
  int counter = 0;
  
  void increment() {
    setState(() {
      counter++; // Triggers rebuild of this widget only
    });
  }
  
  @override Widget build(BuildContext context) { ... }
}

// 2. PROVIDER (App-level state, simple)
class CounterProvider extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners(); // Notifies all listeners
  }
}

// Usage
Consumer<CounterProvider>(
  builder: (context, provider, child) => Text('${provider.count}'),
)

// 3. BLOC (Complex apps, event-driven)
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<Increment>((event, emit) => emit(state + 1));
    on<Decrement>((event, emit) => emit(state - 1));
  }
}

// When to use:
// - setState: Simple local state
// - Provider: App-wide simple state
// - Bloc: Complex state logic, multiple screens
```

## **2. When to use Bloc over Provider**
```dart
// USE BLOC WHEN:
// 1. Complex state logic
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginRequested>(_onLoginRequested);
    on<LogoutRequested>(_onLogoutRequested);
  }
  
  // Complex async operations with multiple states
  Future<void> _onLoginRequested(
    LoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    try {
      final user = await authRepository.login(event.email, event.password);
      emit(AuthSuccess(user));
    } catch (e) {
      emit(AuthFailure(e.toString()));
    }
  }
}

// 2. Need state history/time-travel debugging
// 3. Multiple async operations
// 4. Strict separation of business logic
// 5. Large teams (predictable state flow)

// USE PROVIDER WHEN:
// - Simple state changes
// - Small to medium apps
// - Quick prototyping
// - Simple dependency injection
```

## **3. Managing Global State**
```dart
// APPROACH 1: PROVIDER (Recommended)
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
        Provider(create: (_) => ApiService()),
        FutureProvider<UserProfile>(
          create: (_) => UserRepository().getProfile(),
          initialData: null,
        ),
      ],
      child: MyApp(),
    ),
  );
}

// APPROACH 2: RIVERPROD (Next-gen Provider)
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  
  void increment() => state++;
  void decrement() => state--;
}

// APPROACH 3: GET_IT (Service Locator)
final getIt = GetIt.instance;

void setup() {
  getIt.registerSingleton<AuthService>(AuthService());
  getIt.registerFactory<ApiService>(() => ApiService());
}

// Usage anywhere
final authService = getIt<AuthService>();
```

---

# **Performance & Optimization**

## **1. Causes of Unnecessary Widget Rebuilds**
```dart
// COMMON CULPRITS:
class BadPerformance extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 1. Creating objects in build()
    final service = ApiService(); // ← Creates new instance each build
    
    // 2. Not using const constructors
    return Container(
      child: Text('Hello'), // ← Not const
    );
  }
}

// SOLUTION:
class GoodPerformance extends StatelessWidget {
  // Move expensive operations outside build
  final ApiService service = ApiService();
  
  @override
  Widget build(BuildContext context) {
    return const Container(  // Use const
      child: Text('Hello'),  // Also const
    );
  }
}

// 3. Parent widget rebuilding unnecessarily
// 4. Not using keys in lists
// 5. Calling setState() on entire tree
```

## **2. Optimizing Large Lists**
```dart
// BAD: ListView with all children built immediately
ListView(
  children: List.generate(10000, (index) => HeavyWidget(index)),
)

// GOOD: ListView.builder (lazy loading)
ListView.builder(
  itemCount: 10000,
  itemBuilder: (context, index) => HeavyWidget(index),
  // Only builds visible items
)

// BETTER: ListView with itemExtent
ListView.builder(
  itemCount: 10000,
  itemExtent: 50, // Fixed height for better performance
  itemBuilder: (context, index) => ListTile(title: Text('Item $index')),
)

// BEST: Slivers for complex scrolling
CustomScrollView(
  slivers: [
    SliverAppBar(...),
    SliverList(  // Most efficient
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
        childCount: 10000,
      ),
    ),
  ],
)
```

## **3. Difference between ListView and ListView.builder**
```dart
// LISTVIEW (All children built immediately)
ListView(
  children: [
    Container(height: 100, color: Colors.red),
    Container(height: 100, color: Colors.blue),
    Container(height: 100, color: Colors.green),
    // All 3 containers built even if only 1 is visible
  ],
)

// LISTVIEW.BUILDER (Lazy loading)
ListView.builder(
  itemCount: 1000, // Can be infinite
  itemBuilder: (context, index) {
    // Only builds widgets as they become visible
    return Container(
      height: 100,
      color: index.isEven ? Colors.red : Colors.blue,
    );
  },
)

// When to use:
// - ListView: Few items (≤ 50), all visible
// - ListView.builder: Many items, need lazy loading
```

## **4. Debugging Performance Issues**
```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class PerformanceDebug extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Performance Demo',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Performance Tools'),
        ),
        body: Column(
          children: [
            // 1. DEBUG PAINT
            ElevatedButton(
              onPressed: () {
                debugPaintSizeEnabled = !debugPaintSizeEnabled;
                // Shows layout boundaries
              },
              child: Text('Toggle Debug Paint'),
            ),
            
            // 2. PERFORMANCE OVERLAY
            ElevatedButton(
              onPressed: () {
                // Enable in MaterialApp:
                // MaterialApp(
                //   showPerformanceOverlay: true,
                // )
              },
              child: Text('Performance Overlay'),
            ),
            
            // 3. WIDGET INSPECTOR
            ElevatedButton(
              onPressed: () {
                // Enable with:
                // flutter run --profile
                // Then "i" key for inspector
              },
              child: Text('Widget Inspector'),
            ),
            
            // 4. TRACK REBUILDS
            const MyTrackedWidget(),
          ],
        ),
      ),
    );
  }
}

// Widget to track rebuilds
class MyTrackedWidget extends StatelessWidget {
  const MyTrackedWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    print('MyTrackedWidget rebuilt'); // Track in console
    
    return Container(
      child: const Text('Track me'),
    );
  }
}

// 5. PROFILING
// Run: flutter run --profile
// Use DevTools (chrome://inspect)
// Check:
// - GPU thread (rendering)
// - UI thread (Dart code)
// - Frame rendering time (<16ms for 60fps)
```

### **1. Difference between `Iterable`, `List`, and `Set` with examples**
```dart
// ITERABLE (Lazy evaluation)
Iterable<int> generateNumbers() sync* {
  for (int i = 0; i < 5; i++) {
    yield i; // Produced on-demand
  }
}

// LIST (Ordered, indexed, allows duplicates)
List<int> numbers = [1, 2, 3, 2, 1]; // OK
print(numbers[0]); // Index access
numbers.add(4); // Mutable

// SET (Unique, unordered)
Set<int> uniqueNumbers = {1, 2, 3, 2, 1}; // {1, 2, 3}
print(uniqueNumbers.contains(2)); // True
uniqueNumbers.add(4); // No duplicates
```

### **2. What are Generics and why are they useful?**
```dart
// WITHOUT GENERICS (Type unsafe)
class Box {
  dynamic content;
  Box(this.content);
}

// WITH GENERICS (Type safe, reusable)
class GenericBox<T> {
  T content;
  GenericBox(this.content);
  
  T getContent() => content;
}

// Usage
Box stringBox = Box('Hello'); // Can put anything
GenericBox<String> safeBox = GenericBox('Hello'); // Type safe
GenericBox<int> intBox = GenericBox(42); // Reusable

// Generic methods
T first<T>(List<T> items) => items.first;
String result = first<String>(['a', 'b', 'c']);
```

### **3. What are Streams and how do they differ from Futures?**
```dart
// FUTURE: Single async value
Future<String> fetchUserName(int id) async {
  await Future.delayed(Duration(seconds: 1));
  return 'User $id';
}

// STREAM: Multiple async values over time
Stream<int> countDown(int from) async* {
  for (int i = from; i >= 0; i--) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emit multiple values
  }
}

// Usage
Future<String> user = fetchUserName(1); // Resolves once
Stream<int> timer = countDown(10); // Emits 10, 9, 8...

timer.listen((count) {
  print('T-$count'); // Called 11 times
});

// Key differences:
// Future: Single value, async/await, one-time
// Stream: Multiple values, listen(), continuous
```

### **4. Explain Dart's type system (sound null safety)**
```dart
// TYPE HIERARCHY
// Object (root)
//   ↑
// dynamic (can be anything, opt-out of type checking)
//   ↑
// Never (bottom type - unreachable code)
//   ↑
// Null (null value)

// SOUND NULL SAFETY
String name = 'John'; // Non-nullable by default
String? nullableName = null; // Explicit nullable

// Flow analysis (smart promotion)
void printLength(String? text) {
  if (text != null) {
    print(text.length); // Type promoted to String
  }
}

// Late initialization
late String userName;
void initialize() {
  userName = 'Alice'; // Must initialize before use
}

// The "bang" operator (!) - use with caution
String? maybeNull;
String forced = maybeNull!; // Throws if null
```

---

## **Flutter Advanced Concepts**

### **5. How does Flutter's rendering engine work?**
```dart
// WIDGET → ELEMENT → RENDEROBJECT PIPELINE
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      width: 100,
      height: 100,
      color: Colors.blue,
    );
  }
}

// Process:
// 1. Widget tree (immutable configuration)
// 2. Element tree (mutable, manages lifecycle)
// 3. RenderObject tree (layout and painting)

// RENDEROBJECT EXAMPLE
class CustomBox extends SingleChildRenderObjectWidget {
  @override
  RenderObject createRenderObject(BuildContext context) {
    return RenderCustomBox();
  }
}

class RenderCustomBox extends RenderBox {
  @override
  void performLayout() {
    // Calculate size and position
    size = constraints.constrain(Size(100, 100));
  }
  
  @override
  void paint(PaintingContext context, Offset offset) {
    // Paint to canvas
    final paint = Paint()..color = Colors.blue;
    context.canvas.drawRect(offset & size, paint);
  }
}
```

### **6. What are CustomPaint and Canvas?**
```dart
class CustomChart extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(300, 200),
      painter: ChartPainter(),
    );
  }
}

class ChartPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;
    
    final path = Path()
      ..moveTo(0, size.height)
      ..lineTo(50, size.height * 0.7)
      ..lineTo(100, size.height * 0.5)
      ..lineTo(150, size.height * 0.8)
      ..lineTo(200, size.height * 0.3)
      ..lineTo(250, size.height * 0.6)
      ..lineTo(300, size.height)
      ..close();
    
    canvas.drawPath(path, paint);
    
    // Draw text
    final textStyle = TextStyle(color: Colors.black, fontSize: 12);
    final textSpan = TextSpan(text: 'Chart', style: textStyle);
    final textPainter = TextPainter(
      text: textSpan,
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(canvas, Offset(10, 10));
  }
  
  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

### **7. How to create custom animations?**
```dart
class BouncingBall extends StatefulWidget {
  @override
  _BouncingBallState createState() => _BouncingBallState();
}

class _BouncingBallState extends State<BouncingBall> 
    with SingleTickerProviderStateMixin {
  
  late AnimationController _controller;
  late Animation<double> _animation;
  late Animation<Offset> _position;
  
  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
    
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.bounceOut,
    );
    
    _position = Tween<Offset>(
      begin: Offset.zero,
      end: Offset(0, 1),
    ).animate(_animation);
  }
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.translate(
          offset: _position.value * 200,
          child: Container(
            width: 50,
            height: 50,
            decoration: BoxDecoration(
              color: Colors.red,
              shape: BoxShape.circle,
            ),
          ),
        );
      },
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### **8. Explain Flutter's navigation system**
```dart
// 1. IMPERATIVE NAVIGATION
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => SecondScreen(),
    settings: RouteSettings(arguments: 'Hello'),
  ),
);

// 2. NAMED ROUTES
MaterialApp(
  routes: {
    '/': (context) => HomeScreen(),
    '/details': (context) => DetailsScreen(),
    '/profile': (context) => ProfileScreen(),
  },
);

// Usage
Navigator.pushNamed(context, '/details', arguments: userId);

// 3. GENERATED ROUTES (Recommended for complex apps)
MaterialApp(
  onGenerateRoute: (settings) {
    if (settings.name == '/user/:id') {
      final id = settings.name!.split('/').last;
      return MaterialPageRoute(
        builder: (context) => UserScreen(id: id),
      );
    }
    return null;
  },
);

// 4. NAVIGATOR 2.0 (Declarative)
class AppRouter extends RouterDelegate
    with ChangeNotifier, PopNavigatorRouterDelegateMixin {
  
  @override
  Widget build(BuildContext context) {
    return Navigator(
      pages: [
        MaterialPage(child: HomeScreen()),
        if (showDetails) MaterialPage(child: DetailsScreen()),
        if (showProfile) MaterialPage(child: ProfileScreen()),
      ],
      onPopPage: (route, result) {
        // Handle back navigation
        return route.didPop(result);
      },
    );
  }
}
```

---

## **State Management Advanced**

### **9. Explain BLoC Pattern with Cubit**
```dart
// CUBIT (Simpler BLoC)
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  
  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

// FULL BLoC (Event-driven)
abstract class CounterEvent {}
class Increment extends CounterEvent {}
class Decrement extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<Increment>((event, emit) => emit(state + 1));
    on<Decrement>((event, emit) => emit(state - 1));
  }
}

// Usage with BlocConsumer
BlocConsumer<CounterCubit, int>(
  listener: (context, state) {
    // Side effects (navigation, dialogs)
    if (state == 10) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Reached 10!')),
      );
    }
  },
  builder: (context, state) {
    return Text('Count: $state');
  },
);

// BLOC SELECTOR (Optimized rebuilds)
BlocSelector<CounterCubit, int, bool>(
  selector: (state) => state > 5,
  builder: (context, isGreaterThan5) {
    return Text(isGreaterThan5 ? '> 5' : '≤ 5');
    // Only rebuilds when condition changes
  },
);
```

### **10. Riverpod vs Provider - Which and Why?**
```dart
// RIVERPROD EXAMPLE
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});

final userNameProvider = FutureProvider<String>((ref) async {
  final userId = ref.watch(authProvider).userId;
  return await userRepository.getUserName(userId);
});

// ADVANTAGES:
// 1. Compile-safe (no runtime errors)
// 2. Testable (override providers easily)
// 3. Flexible (combine providers)
final userProfileProvider = Provider<UserProfile>((ref) {
  final user = ref.watch(userProvider);
  final posts = ref.watch(userPostsProvider);
  return UserProfile(user: user, posts: posts);
});

// 4. Scoped providers
final scopedProvider = Provider((ref) {
  final parentValue = ref.watch(parentProvider);
  return parentValue * 2;
}, dependencies: [parentProvider]);

// WHEN TO USE:
// - New projects: Riverpod
// - Existing Provider code: Stick with Provider
// - Complex dependency: Riverpod
```

### **11. Global State Management Patterns**
```dart
// APPROACH 1: SERVICE LOCATOR + CHANGE NOTIFIER
GetIt locator = GetIt.instance;

class AppServices {
  static void setup() {
    locator.registerLazySingleton(() => AuthService());
    locator.registerFactory(() => ApiService());
    locator.registerSingleton(CartManager());
  }
}

class CartManager extends ChangeNotifier {
  final List<Product> _items = [];
  
  void addProduct(Product product) {
    _items.add(product);
    notifyListeners();
  }
}

// APPROACH 2: REDUX
class AppState {
  final User? user;
  final List<Product> cart;
  final bool isLoading;
  
  AppState({this.user, this.cart = const [], this.isLoading = false});
}

// Reducer
AppState appReducer(AppState state, dynamic action) {
  if (action is LoginAction) {
    return state.copyWith(user: action.user);
  }
  return state;
}

// APPROACH 3: MOBX
class CartStore = _CartStore with _$CartStore;

abstract class _CartStore with Store {
  @observable
  List<Product> items = [];
  
  @action
  void addProduct(Product product) {
    items.add(product);
  }
  
  @computed
  double get totalPrice => items.fold(0, (sum, item) => sum + item.price);
}
```

---

## **Performance Optimization**

### **12. Optimize Build Methods**
```dart
class OptimizedWidget extends StatelessWidget {
  // Move expensive operations here
  final expensiveData = calculateExpensiveData();
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Use const widgets
        const HeaderWidget(),
        
        // 2. Extract to methods/widgets
        _buildContent(),
        
        // 3. Use builders for lists
        ListView.builder(
          shrinkWrap: true,
          physics: NeverScrollableScrollPhysics(),
          itemCount: 100,
          itemBuilder: (context, index) => ListItem(index),
        ),
      ],
    );
  }
  
  // Private method for better organization
  Widget _buildContent() {
    return Container();
  }
}

// OPTIMIZED LIST ITEM
class ListItem extends StatelessWidget {
  final int index;
  const ListItem(this.index, {Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return const SizedBox(
      height: 50,
      child: Text('Item'),
    ); // Keep simple
  }
}
```

### **13. Memory Management and Leak Prevention**
```dart
class LeakFreeWidget extends StatefulWidget {
  @override
  _LeakFreeWidgetState createState() => _LeakFreeWidgetState();
}

class _LeakFreeWidgetState extends State<LeakFreeWidget> {
  // COMMON LEAK SOURCES:
  
  // 1. Stream subscriptions
  StreamSubscription? _streamSub;
  final _controller = StreamController<int>();
  
  @override
  void initState() {
    super.initState();
    _streamSub = _controller.stream.listen((data) {
      // Handle data
    });
  }
  
  @override
  void dispose() {
    // ALWAYS cancel/close in dispose()
    _streamSub?.cancel();
    _controller.close();
    
    // 2. Animation controllers
    // 3. Timers
    // 4. Scroll controllers
    // 5. Focus nodes
    // 6. Text editing controllers
    
    super.dispose(); // Super last!
  }
  
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

// USE WIDGET BINDING OBSERVER
class MemoryObserver extends WidgetsBindingObserver {
  @override
  void didHaveMemoryPressure() {
    // Clear caches, dispose unused resources
    print('Memory pressure!');
  }
}
```

### **14. Optimize Images and Assets**
```dart
class OptimizedImageWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. USE CACHED NETWORK IMAGE
        CachedNetworkImage(
          imageUrl: 'https://example.com/image.jpg',
          placeholder: (context, url) => CircularProgressIndicator(),
          errorWidget: (context, url, error) => Icon(Icons.error),
          cacheKey: 'image_1', // Custom cache key
          maxWidth: 500, // Resize for performance
        ),
        
        // 2. ASSET BUNDLE OPTIMIZATION
        Image.asset(
          'assets/images/photo.jpg',
          width: 300, // Specify dimensions
          height: 200,
          fit: BoxFit.cover,
          cacheWidth: 600, // Resize before loading
          cacheHeight: 400,
        ),
        
        // 3. LAZY LOAD IMAGES
        LayoutBuilder(
          builder: (context, constraints) {
            return ListView.builder(
              itemCount: 100,
              itemBuilder: (context, index) {
                return Image.network(
                  'https://picsum.photos/200/300?image=$index',
                  loadingBuilder: (context, child, loadingProgress) {
                    if (loadingProgress == null) return child;
                    return CircularProgressIndicator();
                  },
                );
              },
            );
          },
        ),
      ],
    );
  }
}
```

---

## **Testing & Debugging**

### **15. Write Unit, Widget, and Integration Tests**
```dart
// UNIT TEST
void main() {
  group('Counter Calculator', () {
    test('Adds two numbers', () {
      expect(add(2, 3), 5);
    });
    
    test('Counter increments', () async {
      final cubit = CounterCubit();
      expect(cubit.state, 0);
      
      cubit.increment();
      await expectLater(cubit.stream, emits(1));
      
      cubit.close();
    });
  });
}

// WIDGET TEST
void main() {
  testWidgets('Counter increments smoke test', (tester) async {
    await tester.pumpWidget(MyApp());
    
    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);
    
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump(); // Rebuild
    
    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });
}

// INTEGRATION TEST
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  testWidgets('Full app test', (tester) async {
    app.main(); // Start app
    
    await tester.pumpAndSettle();
    
    await tester.tap(find.text('Login'));
    await tester.pumpAndSettle();
    
    await tester.enterText(find.byType(TextField).first, 'test@email.com');
    await tester.enterText(find.byType(TextField).last, 'password');
    
    await tester.tap(find.text('Submit'));
    await tester.pumpAndSettle(Duration(seconds: 2));
    
    expect(find.text('Welcome'), findsOneWidget);
  });
}
```

### **16. Debug Production Issues**
```dart
// ERROR MONITORING
void main() {
  // 1. SETUP ERROR HANDLING
  FlutterError.onError = (details) {
    // Send to crashlytics/sentry
    FirebaseCrashlytics.instance.recordFlutterError(details);
    Sentry.captureException(details.exception);
  };
  
  // 2. PLATFORM EXCEPTIONS
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack);
    return true;
  };
  
  // 3. CUSTOM ERROR WIDGET
  ErrorWidget.builder = (details) {
    return Material(
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.error_outline, color: Colors.red, size: 50),
            SizedBox(height: 20),
            Text('Something went wrong!'),
            Text(details.exception.toString()),
          ],
        ),
      ),
    );
  };
  
  runApp(MyApp());
}

// PERFORMANCE MONITORING
class PerformanceMonitor extends StatefulWidget {
  @override
  _PerformanceMonitorState createState() => _PerformanceMonitorState();
}

class _PerformanceMonitorState extends State<PerformanceMonitor>
    with WidgetsBindingObserver {
    
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.resumed) {
      // Log app foreground time
      Analytics().logEvent('app_foreground');
    }
  }
  
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
    
    // Monitor frame rate
    WidgetsBinding.instance.addTimingsCallback((List<FrameTiming> timings) {
      for (final timing in timings) {
        if (timing.totalSpan > Duration(milliseconds: 16)) {
          // Frame took too long
          debugPrint('Slow frame detected');
        }
      }
    });
  }
}
```

---

## **Advanced Topics**

### **17. Platform Channels & Native Integration**
```dart
// DART SIDE
class NativeBridge {
  static const platform = MethodChannel('com.example/native');
  
  Future<String> getBatteryLevel() async {
    try {
      final result = await platform.invokeMethod('getBatteryLevel');
      return 'Battery: $result%';
    } on PlatformException catch (e) {
      return 'Failed: ${e.message}';
    }
  }
  
  Future<void> showNativeDialog() async {
    await platform.invokeMethod('showDialog', {
      'title': 'Native Dialog',
      'message': 'From Flutter!',
    });
  }
  
  // EVENT CHANNEL (Native → Flutter)
  static const eventChannel = EventChannel('com.example/events');
  
  Stream<String> get nativeEvents {
    return eventChannel.receiveBroadcastStream().cast<String>();
  }
}

// ANDROID (Kotlin)
class MainActivity: FlutterActivity() {
  override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "com.example/native")
      .setMethodCallHandler { call, result ->
        when (call.method) {
          "getBatteryLevel" -> {
            val battery = getBatteryLevel()
            result.success(battery)
          }
          else -> result.notImplemented()
        }
      }
  }
}

// iOS (Swift)
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller = window?.rootViewController as! FlutterViewController
    let channel = FlutterMethodChannel(
      name: "com.example/native",
      binaryMessenger: controller.binaryMessenger
    )
    
    channel.setMethodCallHandler { call, result in
      if call.method == "getBatteryLevel" {
        UIDevice.current.isBatteryMonitoringEnabled = true
        let level = UIDevice.current.batteryLevel
        result.success(Int(level * 100))
      } else {
        result.notImplemented()
      }
    }
    
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

### **18. Code Generation & Build Runner**
```dart
// JSON SERIALIZATION
@JsonSerializable()
class User {
  final String name;
  final String email;
  final int age;
  
  User({required this.name, required this.email, required this.age});
  
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}

// RUN: flutter pub run build_runner build

// FREEZED (Immutable classes)
part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  const factory User({
    required String name,
    required String email,
    int? age,
  }) = _User;
  
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// GENERATED ROUTES (GoRouter)
part 'app_router.g.dart';

@TypedGoRoute<HomeRoute>(
  path: '/',
)
class HomeRoute extends GoRouteData {
  const HomeRoute();
  
  @override
  Widget build(BuildContext context, GoRouterState state) => HomeScreen();
}

@TypedGoRoute<UserRoute>(
  path: '/user/:id',
)
class UserRoute extends GoRouteData {
  const UserRoute({required this.id});
  
  final String id;
  
  @override
  Widget build(BuildContext context, GoRouterState state) => UserScreen(id: id);
}
```

### **19. CI/CD & Deployment**
```dart
// github/workflows/flutter.yml
name: Flutter CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.7.0'
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test
      - run: flutter build apk --release

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2
      - uses: subosito/flutter-action@v2
      - run: flutter pub get
      - run: flutter build appbundle --release
      - uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJson: ${{ secrets.GCP_SA_KEY }}
          packageName: com.example.app
          releaseFiles: build/app/outputs/bundle/release/app-release.aab
          track: internal
```

### **20. Security Best Practices**
```dart
class SecureApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: FutureBuilder(
          future: _checkSecurity(),
          builder: (context, snapshot) {
            if (snapshot.connectionState == ConnectionState.waiting) {
              return Center(child: CircularProgressIndicator());
            }
            return HomeScreen();
          },
        ),
      ),
    );
  }
  
  Future<bool> _checkSecurity() async {
    // 1. CHECK FOR ROOT/JAILBREAK
    if (await FlutterJailbreakDetection.jailbroken) {
      throw Exception('Device is rooted');
    }
    
    // 2. VERIFY SSL PINNING
    final dio = Dio();
    (dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate =
        (client) {
      client.badCertificateCallback =
          (X509Certificate cert, String host, int port) {
        // Implement certificate pinning
        return false;
      };
    };
    
    // 3. SECURE STORAGE
    const storage = FlutterSecureStorage();
    await storage.write(key: 'token', value: 'secure_token');
    
    // 4. OBFUSCATE CODE
    // Add to android/app/build.gradle:
    // minifyEnabled true
    // shrinkResources true
    
    return true;
  }
}
```

---

## **Behavioral Questions**

### **21. How do you handle large codebases?**
```dart
// PROJECT STRUCTURE
lib/
├── core/                  # Framework-agnostic code
│   ├── constants/
│   ├── errors/
│   ├── usecases/
│   └── utils/
├── features/              # Feature-based modules
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── products/
│       ├── data/
│       ├── domain/
│       └── presentation/
├── shared/               # Shared components
│   ├── widgets/
│   ├── services/
│   └── theme/
└── app.dart

// DEPENDENCY INJECTION
class AppModule extends Module {
  @override
  List<Bind> get binds => [
    Bind.factory((i) => ApiService()),
    Bind.singleton((i) => AuthRepository()),
    Bind.lazySingleton((i) => UserCubit(i())),
  ];
}

// CODE GENERATION FOR CONSISTENCY
// Use: flutter pub run build_runner build --delete-conflicting-outputs
// Enforce with: pre-commit hooks, lint rules
```

### **22. How do you ensure code quality?**
```dart
// ANALYSIS OPTIONS
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  exclude:
    - '**/*.g.dart'
    - '**/*.freezed.dart'
  strong-mode:
    implicit-casts: false
    implicit-dynamic: false
  
linter:
  rules:
    - always_declare_return_types
    - always_require_non_null_named_parameters
    - avoid_print
    - camel_case_types
    - empty_statements

# TESTS STRUCTURE
test/
├── unit/
│   ├── repositories/
│   └── usecases/
├── widget/
│   └── screens/
└── integration/
    └── app_test.dart

// CODE REVIEW CHECKLIST
// - [ ] Null safety followed
// - [ ] Widgets are const where possible
// - [ ] No print statements
// - [ ] Proper error handling
// - [ ] Tests written
// - [ ] Performance considered
```

### **23. How do you approach refactoring?**
```dart
// REFACTORING STRATEGY
class LegacyCode {
  // BEFORE
  void processUserData(Map<String, dynamic> data) {
    // 500 lines of mixed concerns
  }
}

// AFTER REFACTORING
class UserProcessor {
  final UserValidator _validator;
  final UserRepository _repository;
  final AnalyticsService _analytics;
  
  Future<User> processUserData(UserData data) async {
    // 1. Validate
    await _validator.validate(data);
    
    // 2. Process
    final user = await _repository.createUser(data);
    
    // 3. Track
    await _analytics.trackUserCreated(user);
    
    return user;
  }
}

// REFACTORING STEPS:
// 1. Write comprehensive tests
// 2. Extract methods/classes
// 3. Apply design patterns
// 4. Update dependencies
// 5. Run performance tests
// 6. Document changes
```

---

## **Practical Scenarios**

### **24. Build a real-time chat app architecture**
```dart
// ARCHITECTURE
class ChatApp {
  // LAYERS:
  // 1. Data Layer (Firebase/Socket.io)
  // 2. Domain Layer (Use cases, entities)
  // 3. Presentation Layer (BLoC, UI)
}

// IMPLEMENTATION
class ChatRepository {
  final FirebaseFirestore _firestore;
  final StreamController<Message> _messageController = StreamController();
  
  Stream<List<Message>> getMessages(String roomId) {
    return _firestore
        .collection('rooms/$roomId/messages')
        .orderBy('timestamp', descending: true)
        .limit(50)
        .snapshots()
        .map((snapshot) => snapshot.docs
            .map((doc) => Message.fromFirestore(doc))
            .toList());
  }
  
  Future<void> sendMessage(Message message) async {
    await _firestore
        .collection('rooms/${message.roomId}/messages')
        .add(message.toFirestore());
    
    // Local echo for immediate feedback
    _messageController.add(message);
  }
}

class ChatCubit extends Cubit<ChatState> {
  final ChatRepository _repository;
  StreamSubscription? _messageSubscription;
  
  ChatCubit(this._repository) : super(ChatInitial());
  
  void connect(String roomId) {
    _messageSubscription = _repository
        .getMessages(roomId)
        .listen((messages) {
          emit(ChatLoaded(messages));
        });
  }
  
  void sendMessage(String text) async {
    final message = Message(
      text: text,
      senderId: userId,
      timestamp: DateTime.now(),
    );
    
    await _repository.sendMessage(message);
  }
  
  @override
  Future<void> close() {
    _messageSubscription?.cancel();
    return super.close();
  }
}
```

### **25. Handle offline-first app with sync**
```dart
class OfflineFirstApp {
  // STRATEGY:
  // 1. Local database (Hive/SQLite)
  // 2. Sync manager
  // 3. Conflict resolution
}

// IMPLEMENTATION
class TodoRepository {
  final LocalDatabase _localDb;
  final RemoteApi _remoteApi;
  final SyncManager _syncManager;
  
  Future<List<Todo>> getTodos() async {
    // Check cache first
    final cached = await _localDb.getTodos();
    if (cached.isNotEmpty) {
      // Return cached, sync in background
      _syncManager.syncTodos();
      return cached;
    }
    
    // Fetch from network
    final remote = await _remoteApi.getTodos();
    await _localDb.saveTodos(remote);
    return remote;
  }
  
  Future<void> addTodo(Todo todo) async {
    // Optimistic UI update
    await _localDb.addTodo(todo.copyWith(status: SyncStatus.pending));
    
    try {
      final saved = await _remoteApi.addTodo(todo);
      await _localDb.updateTodo(saved.copyWith(status: SyncStatus.synced));
    } catch (e) {
      await _localDb.updateTodo(todo.copyWith(status: SyncStatus.failed));
    }
  }
}

enum SyncStatus { pending, synced, failed }
```
