# React to Changes - C++ SDK
All Realm objects are **live objects**, which means they
automatically update whenever they're modified. Realm emits a
notification event whenever any property changes. You can register a
notification handler to listen for these notification events, and update
your UI with the latest data.

## Register an Object Change Listener
You can register a notification handler on a specific object
within a realm. Realm notifies your handler:

- When the object is deleted.
- When any of the object's properties change.

```cpp
auto token = object.observe([&](auto&& change) { ... }
```

The handler receives an object_change
object that contains information about the changes, such as whether the
object was deleted. It may include a list
of PropertyChange
objects that contain information about what fields changed, the new values
of those fields (except on List properties), and potentially the old values
of the fields.

```cpp
if (change.error) {
  rethrow_exception(change.error);
}
if (change.is_deleted) {
  std::cout << "The object was deleted.\n";
} else {
  for (auto& propertyChange : change.property_changes) {
    std::cout << "The object's " << propertyChange.name
              << " property has changed.\n";
    auto newPropertyValue =
        std::get<std::string>(*propertyChange.new_value);
    std::cout << "The new value is " << newPropertyValue << "\n";
  }
}

```

When you make changes, refresh() the
realm to emit a notification.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

// Create an object and move it into the database.
auto dog = realm::Dog{.name = "Max"};
realm.write([&] { realm.add(std::move(dog)); });

auto dogs = realm.objects<realm::Dog>();
auto specificDog = dogs[0];
//  Set up the listener & observe object notifications.
auto token = specificDog.observe([&](auto&& change) {
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
        auto newPropertyValue =
            std::get<std::string>(*propertyChange.new_value);
        std::cout << "The new value is " << newPropertyValue << "\n";
      }
    }
  } catch (std::exception const& e) {
    std::cerr << "Error: " << e.what() << "\n";
  }
});

// Update the dog's name to see the effect.
realm.write([&] { specificDog.name = "Wolfie"; });

// Deleting the object triggers a delete notification.
realm.write([&] { realm.remove(specificDog); });

// Refresh the database after the change to trigger the notification.
realm.refresh();

// Unregister the token when done observing.
token.unregister();

```

## Register a Collection Change Listener
You can register a notification handler on a collection. A collection is
a list, map, or set property that can contain any supported data type, including primitives or other objects.

Realm notifies your handler whenever a write transaction
removes, adds, or changes objects in the collection.

Notifications describe the changes since the prior notification with
three lists of indices: the indices of the objects that were removed,
inserted, and modified.

> Important:
> In collection notification handlers, always apply changes
in the following order: deletions, insertions, then
modifications. Handling insertions before deletions may
result in unexpected behavior.
>

Collection notifications provide a collection_change
struct that reports the index of the objects that are removed, added, or
modified. It can also notify you if the collection was deleted.

In this example, the `Person` object has a list of `Dog` objects as a property:

```cpp
struct Person {
  std::string name;
  std::vector<Dog*> dogs;
};
REALM_SCHEMA(Person, name, dogs)

```

Removing a dog, adding a new dog, or modifying a dog triggers the notification
handler:

```cpp
//  Set up the listener & observe a collection.
auto token = person.dogs.observe([&](auto&& changes) {
  if (changes.collection_root_was_deleted) {
    std::cout << "The collection was deleted.\n";
  } else {
    // Handle deletions, then insertions, then modifications.
    for (auto& collectionChange : changes.deletions) {
      std::cout << "The object at index " << std::to_string(collectionChange)
                << " was removed\n";
    }
    for (auto& collectionChange : changes.insertions) {
      std::cout << "The object at index " << std::to_string(collectionChange)
                << " was inserted\n";
    }
    for (auto& collectionChange : changes.modifications) {
      std::cout << "The object at index " << std::to_string(collectionChange)
                << " was modified\n";
    }
  }
});

// Remove an object from the collection, and then add an object to see
// deletions and insertions.
realm.write([&] {
  person.dogs.clear();
  person.dogs.push_back(&dog2);
});

// Modify an object to see a modification.
realm.write([&] { dog2.age = 2; });

// Refresh the realm after the change to trigger the notification.
realm.refresh();

// Unregister the token when done observing.
token.unregister();

```

## Register a Results Collection Change Listener
You can register a notification handler on a results collection.

Realm notifies your handler whenever a write transaction
removes, adds, or changes objects in the collection.

Notifications describe the changes since the prior notification with
three lists of indices: the indices of the objects that were deleted,
inserted, and modified.

> Important:
> In collection notification handlers, always apply changes
in the following order: deletions, insertions, then
modifications. Handling insertions before deletions may
result in unexpected behavior.
>

Results collection notifications provide a results_change struct that
reports the index of the objects that are deleted, added, or modified.
It can also notify you if the collection was deleted.

```cpp
// Get a results collection to observe
auto dogs = realm.objects<realm::Dog>();
//  Set up the listener & observe results notifications.
auto token = dogs.observe([&](auto&& changes) {
  try {
    if (changes.collection_root_was_deleted) {
      std::cout << "The collection was deleted.\n";
    } else {
      // Handle deletions, then insertions, then modifications.
      for (auto& resultsChange : changes.deletions) {
        std::cout << "The object at index " << std::to_string(resultsChange)
                  << " was deleted\n";
      }
      for (auto& resultsChange : changes.insertions) {
        std::cout << "The object at index " << std::to_string(resultsChange)
                  << " was inserted\n";
      }
      for (auto& resultsChange : changes.modifications) {
        std::cout << "The object at index " << std::to_string(resultsChange)
                  << " was modified\n";
      }
    }
  } catch (std::exception const& e) {
    std::cerr << "Error: " << e.what() << "\n";
  }
});

// Delete and then add an object to see deletions and insertions.
realm.write([&] {
  realm.remove(firstDog);
  realm.add(std::move(dog2));
});

// Modify an object to see a modification.
realm.write([&] { dog2.age = 2; });

// Refresh the database after the change to trigger the notification.
realm.refresh();

// Unregister the token when done observing.
token.unregister();

```

## Stop Watching for Changes
Observation stops when the token returned by an `observe` call becomes
invalid. You can explicitly invalidate a token by calling its
`unregister()` member function.

```cpp
// Unregister the token when done observing.
token.unregister();

```

> Important:
> Notifications stop when the token's destructor is called. For example,
if the token is in a local variable that goes out of scope. You can
use `std::move` to transfer the token to a variable in a different
scope.
>

## Change Notification Limits
Changes in nested documents deeper than four levels down do not trigger
change notifications.

If you have a data structure where you need to listen for changes five
levels down or deeper, workarounds include:

- Refactor the schema to reduce nesting.
- Add something like "push-to-refresh" to enable users to manually refresh data.
