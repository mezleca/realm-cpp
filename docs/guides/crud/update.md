# CRUD - Update - C++ SDK
Updates to Realm Objects must occur within write transactions. For
more information about write transactions, see: Write
Transactions.

## Update an Object
You can modify properties of a Realm object inside of a write transaction.

```cpp
// Query for the object you want to update
auto dogs = realm.objects<realm::Dog>();
auto dogsNamedMaui =
    dogs.where([](auto &dog) { return dog.name == "Maui"; });
CHECK(dogsNamedMaui.size() >= 1);
// Access an object in the results set.
auto maui = dogsNamedMaui[0];

std::cout << "Dog " << maui.name.detach() << " is " << maui.age.detach()
          << " years old\n";

// Assign a new value to a member of the object in a write transaction
int64_t newAge = 2;
realm.write([&] { maui.age = newAge; });

```

**Model**

This example uses the following model:

```cpp
struct Dog {
  std::string name;
  int64_t age;
};
REALM_SCHEMA(Dog, name, age)

```

### Update an Embedded Object Property
To update a property in an embedded object, modify the property in a
write transaction.

```cpp
auto managedBusinesses = realm.objects<realm::Business>();
auto businessesNamedmyBusiness = managedBusinesses.where(
    [](auto &business) { return business.name == "my business"; });
CHECK(businessesNamedmyBusiness.size() >= 1);
auto myBusiness = businessesNamedmyBusiness[0];

realm.write(
    [&] { myBusiness.contactDetails->emailAddress = "info@example.com"; });

std::cout << "New email address: "
          << myBusiness.contactDetails->emailAddress.detach() << "\n";

```

**Model**

This example uses the following model:

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

### Overwrite an Embedded Object Property
To overwrite an embedded object, reassign the embedded object property
to the raw pointer of a new instance in a write transaction.

```cpp
auto businesses = realm.objects<realm::Business>();
auto theBusinesses = businesses.where(
    [](auto &business) { return business.name == "my business"; });
auto theBusiness = theBusinesses[0];

realm.write([&] {
  auto newContactDetails = realm::ContactDetails{
      .emailAddress = "info@example.com", .phoneNumber = "234-567-8901"};
  // Overwrite the embedded object
  theBusiness.contactDetails = &newContactDetails;
});

```

**Model**

This example uses the following model:

```cpp
auto managedBusinesses = realm.objects<realm::Business>();
auto businessesNamedmyBusiness = managedBusinesses.where(
    [](auto &business) { return business.name == "my business"; });
CHECK(businessesNamedmyBusiness.size() >= 1);
auto myBusiness = businessesNamedMyBusiness[0];

realm.write(
    [&] { myBusiness.contactDetails->emailAddress = "info@example.com"; });

std::cout << "New email address: "
          << myBusiness.contactDetails->emailAddress.detach() << "\n";

```

### Update an Inverse Relationship
You can't update an inverse relationship property directly. Instead, an
inverse relationship automatically updates by changing assignment through
its relevant related object.

In this example, a `Person` object has a to-one relationship to a
`Dog` object, and `Dog` has an inverse relationship to `Person`. The
inverse relationship automatically updates when the `Person` object updates
its `Dog` relationship.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

auto dog = realm::Dog{.name = "Wishbone"};

auto [joe] = realm.write([&realm]() {
  auto person =
      realm::Person{.name = "Joe", .age = 27, .dog = nullptr};
  return realm.insert(std::move(person));
});

realm.write([&dog, joe = &joe]() { joe->dog = &dog; });

// After assigning `&dog` to jack's `dog` property,
// the backlink automatically updates to reflect
// the inverse relationship through the dog's `owners`
// property.
CHECK(joe.dog->owners.size() == 1);

```

**Model**

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

### Update a Map Property
You can update a realm map property
as you would a standard C++ [map](https://en.cppreference.com/w/cpp/container/map):

```cpp
// You can check that a key exists using `find`
auto findTuesday = tommy.locationByDay.find("Tuesday");
if (findTuesday != tommy.locationByDay.end())
  realm.write([&] {
    tommy.locationByDay["Tuesday"] =
        realm::Employee::WorkLocation::HOME;
  });
;

```

**Model**

This example uses the following model:

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

### Update a Set Property
You can update a set property
in a write transaction. You can `.insert()` elements into the set, `erase()`
elements from a set, or use `std::set` algorithms to mutate sets:

- [std::set_union](https://en.cppreference.com/w/cpp/algorithm/set_union)
- [std::set_intersection](https://en.cppreference.com/w/cpp/algorithm/set_intersection)
- [std::set_difference](https://en.cppreference.com/w/cpp/algorithm/set_difference)
- [std::set_symmetric_difference](https://en.cppreference.com/w/cpp/algorithm/set_symmetric_difference)

```cpp
// Add elements to the set in a write transaction
realm.write([&] { managedDocsRealm.openPullRequestNumbers.insert(3066); });
CHECK(managedDocsRealm.openPullRequestNumbers.size() == 4);

// Use std::set algorithms to update a set
// In this example, use std::set_union to add elements to the set
// 3064 already exists, so it won't be added, but 3065 and 3067 are
// unique values and will be added to the set.
auto newOpenPullRequests = std::set<int64_t>({3064, 3065, 3067});
realm.write([&] {
  std::set_union(
      docsRealm.openPullRequestNumbers.begin(),
      docsRealm.openPullRequestNumbers.end(), newOpenPullRequests.begin(),
      newOpenPullRequests.end(),
      std::inserter(managedDocsRealm.openPullRequestNumbers,
                    managedDocsRealm.openPullRequestNumbers.end()));
});
CHECK(managedDocsRealm.openPullRequestNumbers.size() == 6);

// Erase elements from a set
auto it3065 = managedDocsRealm.openPullRequestNumbers.find(3065);
CHECK(it3065 != managedDocsRealm.openPullRequestNumbers.end());
realm.write([&] { managedDocsRealm.openPullRequestNumbers.erase(it3065); });

```

**Model**

This example uses the following model:

```cpp
namespace realm {
struct Repository {
  std::string ownerAndName;
  std::set<int64_t> openPullRequestNumbers;
};
REALM_SCHEMA(Repository, ownerAndName, openPullRequestNumbers)
}  // namespace realm

```
