# Quick Start - C++ SDK
This Quick Start demonstrates how to use Realm with the Realm C++ SDK.
Before you begin, ensure you have Installed the C++ SDK.

## Import Realm
Make the Realm C++ SDK available in your code by including the
`cpprealm/sdk.hpp` header in the translation unit where you want to use it:

```cpp
#include <cpprealm/sdk.hpp>

```

## Define Your Object Model
You can define your object model directly in code.

```cpp
namespace realm {
struct Todo {
  realm::primary_key<realm::object_id> _id{realm::object_id::generate()};
  std::string name;
  std::string status;
};
REALM_SCHEMA(Todo, _id, name, status);
}  // namespace realm

```

## Open a Realm
When you open a realm, you must specify a db_config. You can
optionally open a realm at a specific path.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

```

For more information, see: Configure and Open a Realm.

## Create, Read, Update, and Delete Objects
Once you have opened a realm, you can modify it and its objects in a write transaction
block.

To instantiate a new Todo object and add it to the realm in a write block:

```cpp
auto todo = realm::Todo{.name = "Create my first todo item",
                              .status = "In Progress"};

realm.write([&] { realm.add(std::move(todo)); });

```

You can retrieve a live results collection of
all todos in the realm:

```cpp
auto todos = realm.objects<realm::Todo>();

```

You can also filter that collection using where:

```cpp
auto todosInProgress = todos.where(
    [](auto const& todo) { return todo.status == "In Progress"; });

```

To modify a todo, update its properties in a write transaction block:

```cpp
auto todoToUpdate = todosInProgress[0];
realm.write([&] { todoToUpdate.status = "Complete"; });

```

Finally, you can delete a todo:

```cpp
realm.write([&] { realm.remove(specificTodo); });

```

## Watch for Changes
You can watch an object for changes with the `observe` method.

```cpp
auto token = specificTodo.observe([&](auto&& change) {
  try {
    if (change.error) {
      rethrow_exception(change.error);
    }
    if (change.is_deleted) {
      std::cout << "The object was deleted.\n";
    } else {
      for (auto& propertyChange : change.property_changes) {
        std::cout << "The object's " << propertyChange.name
                  << " property has changed.\n";
      }
    }
  } catch (std::exception const& e) {
    std::cerr << "Error: " << e.what() << "\n";
  }
});

```

## Close a Realm
To close a realm and release all underlying resources, call `db::close()`.
Closing the database invalidates any remaining objects.

```cpp
realm.close();

```
