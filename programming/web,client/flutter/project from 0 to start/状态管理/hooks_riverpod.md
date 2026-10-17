``<https://pub.dev/packages/hooks_riverpod>\
<https://riverpod.dev/>\
<https://blog.csdn.net/ZuoYueLiang/article/details/122871775>\
<https://blog.csdn.net/qq_29689343/article/details/130806556>\
- [riverpod及代码生成](#riverpod及代码生成)
  - [搭建](#搭建)
  - [代码生成](#代码生成)
  - [Enabling riverpod\_lint/custom\_lint](#enabling-riverpod_lintcustom_lint)
    - [](#)
- [consumer](#consumer)
- [provider类型](#provider类型)
  - [provider](#provider)
    - [官方](#官方)
    - [小结](#小结)
      - [小结1](#小结1)
      - [小结2](#小结2)
  - [(Async)NotifierProvider](#asyncnotifierprovider)
    - [小结](#小结-1)
      - [小结1](#小结1-1)
      - [小结2](#小结2-1)
  - [StateNotifierProvider](#statenotifierprovider)
  - [FutureProvider](#futureprovider)
  - [StreamProvider](#streamprovider)
    - [官方](#官方-1)
    - [小结](#小结-2)
  - [StateProvider](#stateprovider)
    - [小结](#小结-3)
  - [ChangeNotifierProvider](#changenotifierprovider)

## riverpod及代码生成
### 搭建
```shell
flutter pub add hooks_riverpod
flutter pub add flutter_hooks
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
# 不必须 custom_lint 包含在riverpod_lint
flutter pub add dev:custom_lint
flutter pub add dev:riverpod_lint
```
### 代码生成
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

<!-- 生成前提供part，文件名.g.dart -->
part 'main.g.dart';
```
```shell
flutter pub run build_runner watch
```
### Enabling riverpod_lint/custom_lint
analysis_options.yaml添加如下
```yaml
analyzer:
  plugins:
    - custom_lint
```
* [查看错误列表](https://pub.dev/packages/riverpod_lint)

Those warnings will not show-up in the dart analyze command.
If you want to check those warnings in the CI/terminal, you can run the following:
```shell
dart run custom_lint
```
#### 

---

## consumer
`局部刷新`
<https://www.jianshu.com/p/2cd4dbbc5aeb>
## provider类型

### provider
#### 官方
* caching computations
* exposing a value to other providers (such as a Repository/HttpClient).
* offering a way for tests or widgets to override a value.
* reducing rebuilds of providers/widgets without having to use select.
#### 小结
* 只读,监控常量则不变，但如果监控对象是NotifierProvider，可以实现变化效果（provider对象并没有变化，还是表现为只读，其变化是由其监控的NotifierProvider提供的），可达到缓存数据和筛选数据的效果。
* 当单纯使用StateProvider监控部件状态，部件状态没有发生变化时也有可能发生重建，通过包裹一层provider，可保证部件状态未发生变化时不重建部件
##### 小结1
```dart
//NotifierProvider
class Todo {
  Todo(this.description, this.isCompleted);
  final bool isCompleted;
  final String description;
}
//代码生成
@riverpod
class Todos extends _$Todos {
  @override
  List<Todo> build() {
    return [];
  }

  void addTodo(Todo todo) {
    state = [...state, todo];
  }
  // TODO add other methods, such as "removeTodo", ...
}
//非代码生成
class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    return [];
  }

  void addTodo(Todo todo) {
    state = [...state, todo];
  }
  // TODO add other methods, such as "removeTodo", ...
}

final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(() {
  return TodosNotifier();
});

```
```dart
//通过provider筛选并缓存已完成的数据
//代码生成
@riverpod
List<Todo> completedTodos(CompletedTodosRef ref) {
  final todos = ref.watch(todosProvider);

  // we return only the completed todos
  return todos.where((todo) => todo.isCompleted).toList();
}
//非代码生成
final completedTodosProvider = Provider<List<Todo>>((ref) {
  // We obtain the list of all todos from the todosProvider
  final todos = ref.watch(todosProvider);

  // we return only the completed todos
  return todos.where((todo) => todo.isCompleted).toList();
});
```
```dart
Consumer(builder: (context, ref, child) {
  final completedTodos = ref.watch(completedTodosProvider);
  // TODO show the todos using a ListView/GridView/.../* SKIP */
  return Container();
  /* SKIP END */
});
```
##### 小结2
```dart
//代码生成
@riverpod
class PageIndex extends _$PageIndex {
  @override
  int build() {
    return 0;
  }

  void goToPreviousPage() {
    state = state - 1;
  }
}

// A provider which computes whether the user is allowed to go to the previous page
@riverpod
bool canGoToPreviousPage(CanGoToPreviousPageRef ref) {
  return ref.watch(pageIndexProvider) != 0;
}

class PreviousButton extends ConsumerWidget {
  const PreviousButton({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // We are now watching our new Provider
    // Our widget is no longer calculating whether we can go to the previous page.
    final canGoToPreviousPage = ref.watch(canGoToPreviousPageProvider);

    void goToPreviousPage() {
      ref.read(pageIndexProvider.notifier).goToPreviousPage();
    }

    return ElevatedButton(
      onPressed: canGoToPreviousPage ? goToPreviousPage : null,
      child: const Text('previous'),
    );
  }
}
//非代码生成

final pageIndexProvider = StateProvider<int>((ref) => 0);

// A provider which computes whether the user is allowed to go to the previous page
final canGoToPreviousPageProvider = Provider<bool>((ref) {
  return ref.watch(pageIndexProvider) != 0;
});

class PreviousButton extends ConsumerWidget {
  const PreviousButton({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // We are now watching our new Provider
    // Our widget is no longer calculating whether we can go to the previous page.
    final canGoToPreviousPage = ref.watch(canGoToPreviousPageProvider);

    void goToPreviousPage() {
      ref.read(pageIndexProvider.notifier).update((state) => state - 1);
    }

    return ElevatedButton(
      onPressed: canGoToPreviousPage ? goToPreviousPage : null,
      child: const Text('previous'),
    );
  }
}
```

### (Async)NotifierProvider
* exposing a state which can change over time after reacting to custom events.
* centralizing the logic for modifying some state (aka "business logic") in a single place, improving maintainability over time.

#### 小结
* NotifierProvider监测有复杂逻辑的对象
* AsyncNotifierProvider监测有复杂逻辑的异步对象，可代替FutureProvider
##### 小结1
```dart
//代码生成版本
@freezed
class Todo with _$Todo {
  factory Todo({
    required String id,
    required String description,
    required bool completed,
  }) = _Todo;
}

// This will generates a Notifier and NotifierProvider.
// The Notifier class that will be passed to our NotifierProvider.
// This class should not expose state outside of its "state" property, which means
// no public getters/properties!
// The public methods on this class will be what allow the UI to modify the state.
// Finally, we are using todosProvider(NotifierProvider) to allow the UI to
// interact with our Todos class.
@riverpod
class Todos extends _$Todos {
  @override
  List<Todo> build() {
    return [];
  }

  // Let's allow the UI to add todos.
  void addTodo(Todo todo) {
    // Since our state is immutable, we are not allowed to do `state.add(todo)`.
    // Instead, we should create a new list of todos which contains the previous
    // items and the new one.
    // Using Dart's spread operator here is helpful!
    state = [...state, todo];
    // No need to call "notifyListeners" or anything similar. Calling "state ="
    // will automatically rebuild the UI when necessary.
  }

  // Let's allow removing todos
  void removeTodo(String todoId) {
    // Again, our state is immutable. So we're making a new list instead of
    // changing the existing list.
    state = [
      for (final todo in state)
        if (todo.id != todoId) todo,
    ];
  }

  // Let's mark a todo as completed
  void toggle(String todoId) {
    state = [
      for (final todo in state)
        // we're marking only the matching todo as completed
        if (todo.id == todoId)
          // Once more, since our state is immutable, we need to make a copy
          // of the todo. We're using our `copyWith` method implemented before
          // to help with that.
          todo.copyWith(completed: !todo.completed)
        else
          // other todos are not modified
          todo,
    ];
  }
}
//非代码生成

// An immutable state is preferred.
// We could also use packages like Freezed to help with the implementation.
@immutable
class Todo {
  const Todo({
    required this.id,
    required this.description,
    required this.completed,
  });

  // All properties should be `final` on our class.
  final String id;
  final String description;
  final bool completed;

  // Since Todo is immutable, we implement a method that allows cloning the
  // Todo with slightly different content.
  Todo copyWith({String? id, String? description, bool? completed}) {
    return Todo(
      id: id ?? this.id,
      description: description ?? this.description,
      completed: completed ?? this.completed,
    );
  }
}

// The Notifier class that will be passed to our NotifierProvider.
// This class should not expose state outside of its "state" property, which means
// no public getters/properties!
// The public methods on this class will be what allow the UI to modify the state.
class TodosNotifier extends Notifier<List<Todo>> {
  // We initialize the list of todos to an empty list
  @override
  List<Todo> build() {
    return [];
  }

  // Let's allow the UI to add todos.
  void addTodo(Todo todo) {
    // Since our state is immutable, we are not allowed to do `state.add(todo)`.
    // Instead, we should create a new list of todos which contains the previous
    // items and the new one.
    // Using Dart's spread operator here is helpful!
    state = [...state, todo];
    // No need to call "notifyListeners" or anything similar. Calling "state ="
    // will automatically rebuild the UI when necessary.
  }

  // Let's allow removing todos
  void removeTodo(String todoId) {
    // Again, our state is immutable. So we're making a new list instead of
    // changing the existing list.
    state = [
      for (final todo in state)
        if (todo.id != todoId) todo,
    ];
  }

  // Let's mark a todo as completed
  void toggle(String todoId) {
    state = [
      for (final todo in state)
        // we're marking only the matching todo as completed
        if (todo.id == todoId)
          // Once more, since our state is immutable, we need to make a copy
          // of the todo. We're using our `copyWith` method implemented before
          // to help with that.
          todo.copyWith(completed: !todo.completed)
        else
          // other todos are not modified
          todo,
    ];
  }
}

// Finally, we are using NotifierProvider to allow the UI to interact with
// our TodosNotifier class.
final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(() {
  return TodosNotifier();
});
```
```dart

class TodoListView extends ConsumerWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // rebuild the widget when the todo list changes
    List<Todo> todos = ref.watch(todosProvider);

    // Let's render the todos in a scrollable list view
    return ListView(
      children: [
        for (final todo in todos)
          CheckboxListTile(
            value: todo.completed,
            // When tapping on the todo, change its completed status
            onChanged: (value) =>
                ref.read(todosProvider.notifier).toggle(todo.id),
            title: Text(todo.description),
          ),
      ],
    );
  }
}
```
##### 小结2
```dart
//代码生成
@freezed
class Todo with _$Todo {
  factory Todo({
    required String id,
    required String description,
    required bool completed,
  }) = _Todo;

  factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);
}

// This will generates a AsyncNotifier and AsyncNotifierProvider.
// The AsyncNotifier class that will be passed to our AsyncNotifierProvider.
// This class should not expose state outside of its "state" property, which means
// no public getters/properties!
// The public methods on this class will be what allow the UI to modify the state.
// Finally, we are using asyncTodosProvider(AsyncNotifierProvider) to allow the UI to
// interact with our Todos class.
@riverpod
class AsyncTodos extends _$AsyncTodos {
  Future<List<Todo>> _fetchTodo() async {
    final json = await http.get('api/todos');
    final todos = jsonDecode(json) as List<Map<String, dynamic>>;
    return todos.map(Todo.fromJson).toList();
  }

  @override
  FutureOr<List<Todo>> build() async {
    // Load initial todo list from the remote repository
    return _fetchTodo();
  }

  Future<void> addTodo(Todo todo) async {
    // Set the state to loading
    state = const AsyncValue.loading();
    // Add the new todo and reload the todo list from the remote repository
    state = await AsyncValue.guard(() async {
      await http.post('api/todos', todo.toJson());
      return _fetchTodo();
    });
  }

  // Let's allow removing todos
  Future<void> removeTodo(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.delete('api/todos/$todoId');
      return _fetchTodo();
    });
  }

  // Let's mark a todo as completed
  Future<void> toggle(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.patch(
        'api/todos/$todoId',
        <String, dynamic>{'completed': true},
      );
      return _fetchTodo();
    });
  }
}
//非代码生成

// An immutable state is preferred.
// We could also use packages like Freezed to help with the implementation.
@immutable
class Todo {
  const Todo({
    required this.id,
    required this.description,
    required this.completed,
  });

  factory Todo.fromJson(Map<String, dynamic> map) {
    return Todo(
      id: map['id'] as String,
      description: map['description'] as String,
      completed: map['completed'] as bool,
    );
  }

  // All properties should be `final` on our class.
  final String id;
  final String description;
  final bool completed;

  Map<String, dynamic> toJson() => <String, dynamic>{
        'id': id,
        'description': description,
        'completed': completed,
      };
}

// The Notifier class that will be passed to our NotifierProvider.
// This class should not expose state outside of its "state" property, which means
// no public getters/properties!
// The public methods on this class will be what allow the UI to modify the state.
class AsyncTodosNotifier extends AsyncNotifier<List<Todo>> {
  Future<List<Todo>> _fetchTodo() async {
    final json = await http.get('api/todos');
    final todos = jsonDecode(json) as List<Map<String, dynamic>>;
    return todos.map(Todo.fromJson).toList();
  }

  @override
  Future<List<Todo>> build() async {
    // Load initial todo list from the remote repository
    return _fetchTodo();
  }

  Future<void> addTodo(Todo todo) async {
    // Set the state to loading
    state = const AsyncValue.loading();
    // Add the new todo and reload the todo list from the remote repository
    state = await AsyncValue.guard(() async {
      await http.post('api/todos', todo.toJson());
      return _fetchTodo();
    });
  }

  // Let's allow removing todos
  Future<void> removeTodo(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.delete('api/todos/$todoId');
      return _fetchTodo();
    });
  }

  // Let's mark a todo as completed
  Future<void> toggle(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.patch(
        'api/todos/$todoId',
        <String, dynamic>{'completed': true},
      );
      return _fetchTodo();
    });
  }
}

// Finally, we are using NotifierProvider to allow the UI to interact with
// our TodosNotifier class.
final asyncTodosProvider =
    AsyncNotifierProvider<AsyncTodosNotifier, List<Todo>>(() {
  return AsyncTodosNotifier();
});
```
```dart

class TodoListView extends ConsumerWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // rebuild the widget when the todo list changes
    final asyncTodos = ref.watch(asyncTodosProvider);

    // Let's render the todos in a scrollable list view
    return switch (asyncTodos) {
      AsyncData(:final value) => ListView(
          children: [
            for (final todo in value)
              CheckboxListTile(
                value: todo.completed,
                // When tapping on the todo, change its completed status
                onChanged: (value) {
                  ref.read(asyncTodosProvider.notifier).toggle(todo.id);
                },
                title: Text(todo.description),
              ),
          ],
        ),
      AsyncError(:final error) => Text('Error: $error'),
      _ => const Center(child: CircularProgressIndicator()),
    };
  }
}
```
### StateNotifierProvider
（建议使用NotifierProvider）

### FutureProvider
（高级场景，建议使用NotifierProvider）

### StreamProvider
#### 官方
* listening to Firebase or web-sockets
* rebuilding another provider every few seconds
#### 小结
* 监听异步流数据接口
```dart
@riverpod
Stream<List<String>> chat(ChatRef ref) async* {
  // Connect to an API using sockets, and decode the output
  final socket = await Socket.connect('my-api', 4242);
  ref.onDispose(socket.close);

  var allMessages = const <String>[];
  await for (final message in socket.map(utf8.decode)) {
    // A new message has been received. Let's add it to the list of all messages.
    allMessages = [...allMessages, message];
    yield allMessages;
  }
}
//非代码生成
final chatProvider = StreamProvider<List<String>>((ref) async* {
  // Connect to an API using sockets, and decode the output
  final socket = await Socket.connect('my-api', 4242);
  ref.onDispose(socket.close);

  var allMessages = const <String>[];
  await for (final message in socket.map(utf8.decode)) {
    // A new message has been received. Let's add it to the list of all messages.
    allMessages = [...allMessages, message];
    yield allMessages;
  }
});
```
```dart
Widget build(BuildContext context, WidgetRef ref) {
  final liveChats = ref.watch(chatProvider);

  // Like FutureProvider, it is possible to handle loading/error states using AsyncValue.when
  return switch (liveChats) {
    // Display all the messages in a scrollable list view.
    AsyncData(:final value) => ListView.builder(
        // Show messages from bottom to top
        reverse: true,
        itemCount: value.length,
        itemBuilder: (context, index) {
          final message = value[index];
          return Text(message);
        },
      ),
    AsyncError(:final error) => Text(error.toString()),
    _ => const CircularProgressIndicator(),
  };
}
```
### StateProvider
（可读可写，高级场景建议使用NotifierProvider）
The state of a StateProvider is typically one of:

* an enum, such as a filter type
* a String, typically the raw content of a text field
* a boolean, for checkboxes
* a number, for pagination or age form fields
* You should not use StateProvider if:

your state needs validation logic
* your state is a complex object (such as a custom class, a list/map, ...)
* the logic for modifying your state is more advanced than a simple count++.

#### 小结
* 简单监测某个值的简单变化

### ChangeNotifierProvider
（建议使用NotifierProvider）
``