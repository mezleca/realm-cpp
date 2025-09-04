# Supported Types - C++ SDK
The Realm C++ SDK currently supports these property types.

Optionals use the class template
[std::optional](https://en.cppreference.com/w/cpp/utility/optional).

## Property Cheat Sheet
You can use the following types to define your object model
properties.

|Type|Required|Optional|
| --- | --- | --- |
|Bool|`bool boolName;`|`std::optional<bool> optBoolName;`|
|Int64|`int64_t intName;`|`std::optional<int64_t> optIntName;`|
|Double|`double doubleName;`|`std::optional<double> optDoubleName;`|
|String|`std::string stringName;`|`std::optional<std::string> optStringName;`|
|Enum|`Enum enumName;`|`std::optional<Enum> optEnumName;`|
|Binary Data|`std::vector<std::uint8_t> binaryDataName;`|`std::optional<std::vector<std::uint8_t>> optBinaryDataName;`|
|Date|`std::chrono::time_point<std::chrono::system_clock> dateName;`|`std::optional<std::chrono::time_point<std::chrono::system_clock>> optDateName;`|
|Decimal128|`realm::decimal128 decimal128Name;`|`std::optional<realm::decimal128> optDecimal128Name;`|
|UUID|`realm::uuid uuidName;`|`std::optional<realm::uuid> optUuidName;`|
|Object ID|`realm::object_id objectIdName;`|`std::optional<realm::object_id> optObjectIdName;`|
|Mixed Data Type|`realm::mixed mixedName;`|N/A|
|Map|`std::map<std::string, SomeType> mapName;`|N/A|
|List|`std::vector<SomeType> listTypeName;`|N/A|
|Set|`std::set<SomeType> setTypeName;`|N/A|
|User-defined Object|N/A|`MyClass* opt_obj_name;`|
|User-defined Embedded Object|N/A|`MyEmbeddedClass* opt_embedded_object_name;`|

### Supported Type Implementation Details
Some of the supported types above are aliases for:

- `mixed`: A union-like object that can represent a value any of the
supported types. It is implemented using the class template
[std::variant](https://en.cppreference.com/w/cpp/utility/variant).
This implementation means that a `mixed` property holds a value of
one of its alternative types, or in the case of error - no value.
- For dates, use the [chrono library](https://en.cppreference.com/w/cpp/chrono)
to store a `time_point` relative to the `system_clock`:
`<std::chrono::time_point<std::chrono::system_clock>>`

## Map/Dictionary
The
Map
is an associative array that contains key-value pairs with unique keys.

You can declare a Map as a property of an object:

```cpp
namespace realm {
struct Dog {
  std::string name;
  std::map<std::string, std::string> favoriteParkByCity;
};
REALM_SCHEMA(Dog, name, favoriteParkByCity)
}  // namespace realm

```

String is the only supported type for a map key, but map values can be:

- Required versions of any of the SDK's supported data types
- Optional user-defined object links
- Optional embedded objects

Realm disallows the use of `.` or `$` characters in map keys.
You can use percent encoding and decoding to store a map key that contains
one of these disallowed characters.

```cpp
// Percent encode . or $ characters to use them in map keys
auto mapKey = "Monday.Morning";
auto encodedMapKey = "Monday%2EMorning";

```

## Collection Types
Realm has several types to represent groups of objects,
which we call **collections**. A collection is an object that contains
zero or more instances of one Realm type. Realm collections are **homogenous**:
all objects in a collection are of the same type.

You can filter and sort any collection using Realm's
query engine. Collections are
live, so they always reflect the current state
of the realm instance on the current thread. You can also
listen for changes in the collection by subscribing to collection
notifications.

### Results
The C++ SDK Results
collection is a type representing objects retrieved from queries. A
`Results` collection represents the lazily-evaluated results of a
query operation. Results are immutable: you cannot add or remove elements
to or from the results collection. Results have an associated query that
determines their contents.

For more information, refer to the Read documentation.

### Set
The C++ SDK set collection represents a
to-many relationship containing
distinct values. A C++ SDK set supports the following types (and their
optional versions):

- Binary Data
- Bool
- Double
- Date
- Int64
- Mixed
- ObjectId
- Object
- String
- UUID

Like the C++ [std::set](https://en.cppreference.com/w/cpp/container/set),
the C++ SDK set is a generic type that is parameterized on the type it stores.

You can only call set mutation methods during a write transaction. You can
register a change listener on a mutable set.

### Collections as Properties
The C++ SDK also offers several collection types you can use as properties
in your data model:

1. List, a type representing
to-many relationships in models.
2. Set,
a type representing to-many relationships
in models where values are unique.
3. linking_objects, a type
representing inverse relationships in models.
4. Map,
a type representing an associative array of key-value pairs with unique keys.

### Collections are Live
Like live objects, Realm collections
are usually **live**:

- Live results collections always reflect the current results of the associated query.
- Live lists always reflect the current state of the relationship on the realm instance.

The one case when a collection is **not** live is when the collection is
unmanaged. For example, a List property of a Realm object that has not
been added to a realm yet or that has been moved from a realm is not live.

Combined with collection notifications, live collections enable
clean, reactive code. For example, suppose your view displays the
results of a query. You can keep a reference to the results collection
in your view code, then read the results collection as needed without
having to refresh it or validate that it is up-to-date.

> Important:
> Since results update themselves automatically, do not
store the positional index of an object in the collection
or the count of objects in a collection. The stored index
or count value could be outdated by the time you use
it.
>
