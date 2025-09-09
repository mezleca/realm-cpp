# CRUD - Create - C++ SDK
## Write Transactions
Realm uses a highly efficient storage engine
to persist objects. You can **create** objects in a realm,
**update** objects in a realm, and eventually **delete**
objects from a realm. Because these operations modify the
state of the realm, we call them writes.

Realm handles writes in terms of **transactions**. A
transaction is a list of read and write operations that
Realm treats as a single indivisible operation. In other
words, a transaction is *all or nothing*: either all of the
operations in the transaction succeed or none of the
operations in the transaction take effect.

All writes must happen in a transaction.

A realm allows only one open transaction at a time. Realm
blocks other writes on other threads until the open
transaction is complete. Consequently, there is no race
condition when reading values from the realm within a
transaction.

When you are done with your transaction, Realm either
**commits** it or **cancels** it:

- When Realm **commits** a transaction, Realm writes
all changes to disk.
- When Realm **cancels** a write transaction or an operation in
the transaction causes an error, all changes are discarded
(or "rolled back").

## Create a New Object
### Create an Object
To create an object, you must instantiate it using the `realm` namespace.
Move the object into the realm using the
Realm.add() function
inside of a write transaction.

When you move an object into a realm, this consumes the object as an
rvalue. You must use the managed object for any data access or observation.
In this example, copying the `dog` object into the realm consumes
it as an rvalue. You can return the managed object to continue to work
with it.

```cpp
// Create an object using the `realm` namespace.
auto dog = realm::Dog{.name = "Rex", .age = 1};

std::cout << "dog: " << dog.name << "\n";

// Open the database with compile-time schema checking.
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

// Persist your data in a write transaction
// Optionally return the managed object to work with it immediately
auto managedDog = realm.write([&] { return realm.add(std::move(dog)); });

```

#### Model
For more information about modeling an object, refer to:
Define a New Object Type.

```cpp
namespace realm {
struct Dog {
  std::string name;
  int64_t age;
};
REALM_SCHEMA(Dog, name, age)
}  // namespace realm

```

### Create an Embedded Object
To create an embedded object, assign the raw pointer of the embedded
object to a parent object's property. Move the parent object into
the realm using the Realm.add() function
inside of a write transaction.

In this example, we assign the raw pointer of the embedded object -
`ContactDetails *` - to the embedded object property of the parent
object - `Business.contactDetails`.

Then, we add the `business` object to the realm. This copies the
`business` and `contactDetails` objects to the realm.

Because `ContactDetails` is an embedded object, it does not have
its own lifecycle independent of the main `Business` object.
If you delete the `Business` object, this also deletes the
`ContactDetails` object.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

auto contactDetails = realm::ContactDetails{
    .emailAddress = "email@example.com", .phoneNumber = "123-456-7890"};
auto business = realm::Business();
business._id = realm::object_id::generate();
business.name = "my business";
business.contactDetails = &contactDetails;

realm.write([&] { realm.add(std::move(business)); });

```

#### Model
For more information about modeling an embedded object, refer to:
Define an Embedded Object.

```cpp
namespace realm {
struct ContactDetails {
  // Because ContactDetails is an embedded object, it cannot have its own _id
  // It does not have a lifecycle outside of the top-level object
  std::string emailAddress;
  std::string phoneNumber;
};
REALM_EMBEDDED_SCHEMA(ContactDetails, emailAddress, phoneNumber)

struct Business {
  realm::object_id _id;
  std::string name;
  ContactDetails *contactDetails;
};
REALM_SCHEMA(Business, _id, name, contactDetails)
}  // namespace realm

```

### Create an Object with a To-One Relationship
To create an object with a to-one relationship to another object,
assign the raw pointer of the related object to the relationship
property of the main object. Move the object into the realm using
the Realm.add() function
inside of a write transaction.

In this example, we assign the raw pointer of the related object -
`FavoriteToy *` - to the relationship property of the main object
- `Dog.favoriteToy`. Then, when we add the `dog` object to the
realm, this copies both the `dog` and `favoriteToy` to the realm.

The related `favoriteToy` object has its own lifecycle independent
of the main `dog` object. If you delete the main object, the related
object remains.

```cpp
auto config = realm::db_config();
auto realmInstance = realm::db(std::move(config));

auto favoriteToy = realm::FavoriteToy{
    ._id = realm::uuid("68b696c9-320b-4402-a412-d9cee10fc6a5"),
    .name = "Wubba"};

auto dog = realm::Dog{
    ._id = realm::uuid("68b696d7-320b-4402-a412-d9cee10fc6a3"),
    .name = "Lita",
    .age = 10,
    .favoriteToy = &favoriteToy};

realmInstance.write([&] { realmInstance.add(std::move(dog)); });

```

You can optionally create an inverse relationship to refer to the main object
from the related object. For more information, refer to:
Create an Object with an Inverse Relationship.

#### Model
For more information about modeling a to-one relationship, refer to:
Define a To-One Relationship.

```cpp
struct FavoriteToy {
  realm::primary_key<realm::uuid> _id;
  std::string name;
};
REALM_SCHEMA(FavoriteToy, _id, name)

struct Dog {
  realm::primary_key<realm::uuid> _id;
  std::string name;
  int64_t age;

  // Define a relationship as a link to another SDK object
  FavoriteToy* favoriteToy;
};
REALM_SCHEMA(Dog, _id, name, age, favoriteToy)

```

### Create an Object with a To-Many Relationship
To create an object with a to-many relationship to one or more objects:

- Initialize the main object and the related objects
- Use the push_back
member function available to the Realm object lists
to append the raw pointers of the related objects to the main object's
list property
- Move the object into the realm using the
Realm.add() function
inside of a write transaction.

In this example, we append the raw pointers of the related objects -
`Employee *` - to the relationship property of the main object
- `Company.employees`. This creates a one-way connection from the
`Company` object to the `Employee` objects.

Then, we add the `Company` to the realm. This copies the
`Company` and `Employee` objects to the realm.

The related `Employee` objects have their own lifecycle independent
of the main `Company` object. If you delete the main object, the
related objects remain.

```cpp
auto config = realm::db_config();
auto realmInstance = realm::db(std::move(config));

auto employee1 = realm::Employee{
    ._id = 23456, .firstName = "Pam", .lastName = "Beesly"};

auto employee2 = realm::Employee{
    ._id = 34567, .firstName = "Jim", .lastName = "Halpert"};

auto company =
    realm::Company{._id = 45678, .name = "Dunder Mifflin"};

// Use the `push_back` member function available to the
// `ListObjectPersistable<T>` template to append `Employee` objects to
// the `Company` `employees` list property.
company.employees.push_back(&employee1);
company.employees.push_back(&employee2);

realmInstance.write([&] { realmInstance.add(std::move(company)); });

```

You can optionally create an inverse relationship to refer to the main object
from the related object. For more information, refer to:
Create an Object with an Inverse Relationship.

#### Model
For more information about modeling a to-many relationship, refer to:
Define a To-Many Relationship.

```cpp
namespace realm {
struct Employee {
  realm::primary_key<int64_t> _id;
  std::string firstName;
  std::string lastName;

  // You can use this property as you would any other member
  // Omitting it from the schema means the SDK ignores it
  std::string jobTitle_notPersisted;
};
// The REALM_SCHEMA omits the `jobTitle_notPersisted` property
// The SDK does not store and cannot retrieve a value for this property
REALM_SCHEMA(Employee, _id, firstName, lastName)
}  // namespace realm

```

```cpp
struct Company {
  int64_t _id;
  std::string name;
  // To-many relationships are a list, represented here as a
  // vector container whose value type is the SDK object
  // type that the list field links to.
  std::vector<Employee*> employees;
};
REALM_SCHEMA(Company, _id, name, employees)

```

### Create an Object with an Inverse Relationship
To create an object with a inverse relationship to another object,
assign the raw pointer of the related object to the relationship
property of the main object. Move the object into the realm using the
Realm.add() function
inside of a write transaction.

In this example, we create two `Person` objects that each have a to-one
relationship to the same `Dog` object. The `Dog` has an inverse
relationship to each `Person` object. The inverse relationship backlink
is automatically updated when a linked `Person` object updates its
`Dog` relationship.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

auto dog = realm::Dog{.name = "Bowser"};

auto [jack, jill] = realm.write([&realm]() {
  auto person =
      realm::Person{.name = "Jack", .age = 27, .dog = nullptr};

  realm::Person person2;
  person2.name = "Jill";
  person2.age = 28;
  person2.dog = nullptr;

  return realm.insert(std::move(person), std::move(person2));
});

realm.write([&dog, jack = &jack]() { jack->dog = &dog; });

// After assigning `&dog` to jack's `dog` property,
// the backlink automatically updates to reflect
// the inverse relationship through the dog's `owners`
// property
CHECK(jack.dog->owners.size() == 1);

realm.write([&dog, jill = &jill]() { jill->dog = &dog; });

// After assigning the same `&dog` to jill's `dog`
// property, the backlink automatically updates
CHECK(jill.dog->owners.size() == 2);
CHECK(jack.dog->owners.size() == 2);

// Removing the relationship from the parent object
// automatically updates the inverse relationship
realm.write([jack = &jack]() { jack->dog = nullptr; });
CHECK(jack.dog == nullptr);
CHECK(jill.dog->owners.size() == 1);

```

#### Model
For more information about modeling an inverse relationship, refer to:
Define an Inverse Relationship.

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

### Create an Object with a Map Property
When you create an object that has a
map property,
you can set the values for keys in a few ways:

- Set keys and values on the object and then add the object to the realm
- Set the object's keys and values directly inside a write transaction

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

auto employee = realm::Employee{
    ._id = 8675309, .firstName = "Tommy", .lastName = "Tutone"};

employee.locationByDay = {
    {"Monday", realm::Employee::WorkLocation::HOME},
    {"Tuesday", realm::Employee::WorkLocation::OFFICE},
    {"Wednesday", realm::Employee::WorkLocation::HOME},
    {"Thursday", realm::Employee::WorkLocation::OFFICE}};

realm.write([&] {
  realm.add(std::move(employee));
  employee.locationByDay["Friday"] = realm::Employee::WorkLocation::HOME;
});

```

Realm disallows the use of `.` or `$` characters in map keys.
You can use percent encoding and decoding to store a map key that contains
one of these disallowed characters.

```cpp
// Percent encode . or $ characters to use them in map keys
auto mapKey = "Monday.Morning";
auto encodedMapKey = "Monday%2EMorning";

```

#### Model
For more information about supported map data types, refer to:
Map/Dictionary.

```cpp
namespace realm {
struct Employee {
  enum class WorkLocation { HOME, OFFICE };

  int64_t _id;
  std::string firstName;
  std::string lastName;
  std::map<std::string, WorkLocation> locationByDay;
};
REALM_SCHEMA(Employee, _id, firstName, lastName, locationByDay)
}  // namespace realm

```

### Create an Object with a Set Property
You can create objects that contain
set
properties as you would any Realm object, but you can only mutate a set
property within a write transaction. This means you can only set the value(s)
of a set property within a write transaction.

```cpp
auto realm = realm::db(std::move(config));

// Create an object that has a set property
auto docsRealmRepo =
    realm::Repository{.ownerAndName = "mongodb/docs-realm"};

// Add the object to the database and get the managed object
auto managedDocsRealm =
    realm.write([&]() { return realm.add(std::move(docsRealmRepo)); });

// Insert items into the set
auto openPullRequestNumbers = {3059, 3062, 3064};

realm.write([&] {
  for (auto number : openPullRequestNumbers) {
    // You can only mutate the set in a write transaction.
    // This means you can't set values at initialization,
    // but must do it during a write.
    managedDocsRealm.openPullRequestNumbers.insert(number);
  }
});

```

#### Model
For more information about supported set data types, refer to:
Set.

```cpp
namespace realm {
struct Repository {
  std::string ownerAndName;
  std::set<int64_t> openPullRequestNumbers;
};
REALM_SCHEMA(Repository, ownerAndName, openPullRequestNumbers)
}  // namespace realm

```

