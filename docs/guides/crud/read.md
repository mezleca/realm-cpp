# CRUD - Read - C++ SDK
You can read back the data that you have stored in Realm by finding,
filtering, and sorting objects.

A read from a realm generally consists of the following
steps:

- Get all objects of a certain type from the realm.
- Optionally, filter the results.

Query operations return a
results collection. These
collections are live, meaning they always contain the latest
results of the associated query.

## Read Characteristics
Design your app's data access patterns around these three key
read characteristics to read data as efficiently as possible.

### Results Are Not Copies
Results to a query are not copies of your data. Modifying
the results of a query modifies the data on disk
directly. This memory mapping also means that results are
**live**: that is, they always reflect the current state on
disk.

### Results Are Lazy
Realm only runs a query when you actually request the
results of that query. This lazy evaluation enables you to write
highly performant code for handling large data sets and complex
queries. You can chain several filter operations without requiring
extra work to process the intermediate state.

### References Are Retained
One benefit of Realm's object model is that
Realm automatically retains all of an object's
relationships as direct
references. This enables you to traverse your graph of relationships
directly through the results of a query.

A **direct reference**, or pointer, allows you to access a
related object's properties directly through the reference.

Other databases typically copy objects from database storage
into application memory when you need to work with them
directly. Because application objects contain direct
references, you are left with a choice: copy the object
referred to by each direct reference out of the database in
case it's needed, or just copy the foreign key for each
object and query for the object with that key if it's
accessed. If you choose to copy referenced objects into
application memory, you can use up a lot of resources for
objects that are never accessed, but if you choose to only
copy the foreign key, referenced object lookups can cause
your application to slow down.

Realm bypasses all of this using zero-copy
live objects. Realm object accessors point
directly into database storage using memory mapping, so there is no
distinction between the objects in Realm and the results
of your query in application memory. Because of this, you can traverse
direct references across an entire realm from any query result.

### Limiting Query Results
As a result of lazy evaluation, you do not need any special mechanism to
limit query results with Realm. For example, if your query
matches thousands of objects, but you only want to load the first ten,
simply access only the first ten elements of the results collection.

### Pagination
Thanks to lazy evaluation, the common task of pagination becomes quite
simple. For example, suppose you have a results collection associated
with a query that matches thousands of objects in your realm. You
display one hundred objects per page. To advance to any page, simply
access the elements of the results collection starting at the index that
corresponds to the target page.

## Read Realm Objects
### Query All Objects of a Given Type
To query for objects of a given type in a realm, pass the object type
`YourClassName` to the realm::query<T> member function.

This returns a Results object
representing all objects of the given type in the realm.

```cpp
auto managedBusinesses = realm.objects<realm::Business>();

```

### Filter Queries Based on Object Properties
A filter selects a subset of results based on the value(s) of one or
more object properties. Realm provides a full-featured
query engine that you can use to define filters.

```cpp
auto businessesNamedMongoDB = managedBusinesses.where(
    [](auto &business) { return business.name == "my business"; });

```

### Supported Query Operators
Currently, the Realm C++ SDK supports the following query operators:

- Equality (`==`, `!=`)
- Greater than/less than (`>`, `>=`, `<`, `<=`)
- Compound queries (`||`, `&&`)

### Check the Size of the Results Set and Access Results
Realm Results exposes member
functions to work with results. You may want to check the size of a results
set, or access the object at a specific index.

```cpp
auto managedBusinesses = realm.objects<realm::Business>();
auto businessesNamedMongoDB = managedBusinesses.where(
    [](auto &business) { return business.name == "my business"; });
CHECK(businessesNamedMongoDB.size() >= 1);
auto mongoDB = businessesNamedMongoDB[0];

```

Additionally, you can iterate through the results, or observe a results
set for changes.

### Read a Map Property
You can iterate and check the values of a realm map property
as you would a standard C++ [map](https://en.cppreference.com/w/cpp/container/map):

```cpp
auto employees = realm.objects<realm::Employee>();
auto employeesNamedTommy = employees.where(
    [](auto &employee) { return employee.firstName == "Tommy"; });
auto tommy = employeesNamedTommy[0];
// You can get an iterator for an element matching a key using `find()`
auto tuesdayIterator = tommy.locationByDay.find("Tuesday");

// You can access values for keys like any other map type
auto mondayLocation = tommy.locationByDay["Monday"];

```

### Read a Set Property
You can iterate, check the size of a set, and find values in a set property:

```cpp
auto repositories = realm.objects<realm::Repository>();

auto repositoriesNamedDocsRealm = repositories.where([](auto &repository) {
  return repository.ownerAndName == "mongodb/docs-realm";
});

auto docsRealm = repositoriesNamedDocsRealm[0];

// You can check the size of the set
auto numberOfPullRequests = docsRealm.openPullRequestNumbers.size();

// Find an element in the set whose value is 3064
auto it = managedDocsRealm.openPullRequestNumbers.find(3064);

// Get a copy of the set that exists independent of the managed set
auto openRealmPullRequests = docsRealm.openPullRequestNumbers.detach();

```

## Sort Lists and Query Results
A sort operation allows you to configure the order in which Realm returns
list objects and query results. You can sort based on one or more properties
of the objects in the list or results collection. Realm only guarantees a
consistent order if you explicitly sort.

Unlike using [std::sort](https://en.cppreference.com/w/cpp/algorithm/sort),
Realm's sort implementation preserves the lazy loading of objects. It does
not pull the entire set of results or list objects into memory, but only
loads them into memory when you access them.

To sort, call the `.sort()` function on a list or results set with one or more
sort_descriptors.

A `sort_descriptor` includes:

- The desired key path to sort by, as a string.
- A bool to specify sort order, where``true`` is ascending and `false`
is descending.

In this example, we sort a results set on `priority` in descending order.

```cpp
auto items = realm.objects<realm::Item>();

// Sort with `false` returns objects in descending order.
auto itemsSorted = items.sort("priority", false);

```

You can also sort a list or results set by multiple sort descriptors. In
this example, we sort a list property on `assignee` in ascending order,
and then on `priority` in descending order.

```cpp
auto sortedListProperty =
    specificProject.items.sort({{"assignee", true}, {"priority", false}});

```
