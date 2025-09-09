# CRUD - Delete - C++ SDK
## Delete Realm Objects
Deleting Realm Objects must occur within write transactions. For
more information about write transactions, see: Write Transactions.

### Delete an Object
To delete an object from a realm, pass the object to
Realm.remove() function
inside of a write transaction.

```cpp
realm.write([&] { realm.remove(managedDog); });

```

### Delete an Inverse Relationship
You can't delete an inverse relationship directly. Instead, an
inverse relationship automatically updates by removing the relationship
through the related object.

In this example, a `Person` has a to-one relationship to a `Dog`,
and the `Dog` has an inverse relationship to `Person`.
Setting the `Person.dog` relationship to `nullptr` removes the inverse
relationship from the `Dog` object.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

auto dog = realm::Dog{.name = "Wishbone"};

auto [joe] = realm.write([&realm]() {
  auto person =
      realm::Person{.name = "Joe", .age = 27, .dog = nullptr};
  return realm.insert(std::move(person));
});

// Assign an object with an inverse relationship
// to automatically set the value of the inverse relationship
realm.write([&dog, joe = &joe]() { joe->dog = &dog; });
CHECK(joe.dog->owners.size() == 1);

// ... Later ...

// Removing the relationship from the parent object
// automatically updates the inverse relationship
realm.write([joe = &joe]() { joe->dog = nullptr; });
CHECK(realm.objects<realm::Dog>()[0].owners.size() == 0);

```

#### Model
This example uses the following model:

```cpp
struct Dog;
struct Person {
  realm::primary_key<int64_t> _id;
  std::string name;
  int64_t age = 0;
  Dog* dog;
};
REALM_SCHEMA(Person, _id, name, age, dog)
struct Dog {
  realm::primary_key<int64_t> _id;
  std::string name;
  int64_t age = 0;
  linking_objects<&Person::dog> owners;
};
REALM_SCHEMA(Dog, _id, name, age, owners)

```

### Delete Map Keys/Values
To delete a map key,
pass the key name to `erase()`:

```cpp
realm.write([&] { tommy.locationByDay.erase("Tuesday"); });

```

### Delete Set Values
You can delete a set element
with `erase()`, or remove all elements from a set with `clear()`.

```cpp
// Remove an element from the set with erase()
auto it3064 = managedDocsRealm.openPullRequestNumbers.find(3064);
CHECK(it3064 != managedDocsRealm.openPullRequestNumbers.end());
realm.write([&] { managedDocsRealm.openPullRequestNumbers.erase(it3065); });
CHECK(managedDocsRealm.openPullRequestNumbers.size() == 4);

// Clear the entire contents of the set
realm.write([&] { managedDocsRealm.openPullRequestNumbers.clear(); });
CHECK(managedDocsRealm.openPullRequestNumbers.size() == 0);

```
