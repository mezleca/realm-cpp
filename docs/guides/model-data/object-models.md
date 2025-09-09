# Object Models - C++ SDK
Realm SDK applications model data as objects composed of
field-value pairs that each contain one or more supported data types.

## Object Types and Schemas
Every database object has an *object type* that refers to the object's
class. Objects of the same type share an object schema that defines the properties and relationships of those
objects.

### Database Schema
A **database schema** is a list of valid object schemas that the database may
contain. Every database object must conform to an object type that's included
in its database's schema.

When opening a database, you must specify which models are available by
passing the models to the template you use to open the database. Those
models must have schemas, and this list of schemas becomes the database schema.

If the database already contains data when you open it, the SDK
validates each object to ensure that an object schema was provided for
its type and that it meets all of the constraints specified in the schema.

For more information about how to open the database, refer to
Configure & Open a Realm - C++ SDK

### Object Model
Your object model is the core structure that gives the database
information about how to interpret and store the objects in your app.
The C++ SDK object model is a regular C++ class or a struct that contains
a collection of properties. The properties that you want to persist must
use supported data types. Properties
are also the mechanism for establishing relationships between object types.

When you define your C++ class or struct, you must also provide an
object schema. The schema is a C++ macro that gives the SDK information
about which properties to persist, and what type of database object it
is.

You must define your SDK object model within the `realm` namespace.

### Object Schema
A C++ SDK **object schema** maps properties for a specific object type.
The SDK schemas are macros that give the SDK the information it needs to
store and retrieve the objects. A schema must accompany every object model
you want to persist, and it may be one of:

- `REALM_SCHEMA`
- `REALM_EMBEDDED_SCHEMA`
- `REALM_ASYMMETRIC_SCHEMA`

You must define the schema and your object model within the `realm` namespace.

## Define a New Object Type
In the C++ SDK, you can define your models as regular C++ structs or classes.
Provide an Object Schema with the object type name and
the names of any properties that you want to persist to the database. When you
add the object to the database, the SDK ignores any properties that you omit
from the schema.

You must declare your object and the schema within the `realm` namespace.
You must then use the `realm` namespace when you initialize and perform CRUD
operations with the object.

```cpp
namespace realm {
struct Dog {
  std::string name;
  int64_t age;
};
REALM_SCHEMA(Dog, name, age)

struct Person {
  realm::primary_key<int64_t> _id;
  std::string name;
  int64_t age;

  // Create relationships by pointing an Object field to another struct or class
  Dog *dog;
};
REALM_SCHEMA(Person, _id, name, age, dog)
}  // namespace realm

```

> Note:
> Class names are limited to a maximum of 57 UTF-8 characters.
>

### Specify a Primary Key
You can designate a property as the **primary key** of your object.

Primary keys allow you to efficiently find, update, and upsert objects.

Primary keys are subject to the following limitations:

- You can define only one primary key per object model.
- Primary key values must be unique across all instances of an object
in the database. The C++ SDK throws an error if you try to
insert a duplicate primary key value.
- Primary key values are immutable. To change the primary key value of
an object, you must delete the original object and insert a new object
with a different primary key value.
- Embedded objects cannot define a primary key.

The C++ SDK supports primary keys of the following types, and their
optional variants:

- `int64_t`
- `realm::object_id`
- `realm::uuid`
- `std::string`

Additionally, a required `realm::enum` property can be a primary key, but
`realm::enum` cannot be optional if it is used as a primary key.

Set a property as a primary key with the `primary_key` template:

```cpp
struct Person {
  realm::primary_key<int64_t> _id;
  std::string name;
  int64_t age;

  // Create relationships by pointing an Object field to another struct or class
  Dog *dog;
};
REALM_SCHEMA(Person, _id, name, age, dog)

```

### Ignore a Property
Your model may include properties that the database does not store.

The database ignores any properties not included in the Object Schema.

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

### Define an Embedded Object
An **embedded object** is a special type of object that models complex
data about a specific object. Embedded objects are similar to
relationships, but they provide additional constraints and
map more naturally to the denormalized document model

The C++ SDK enforces unique ownership constraints that treat each embedded
object as nested data inside of a single, specific parent object. An
embedded object inherits the lifecycle of its parent object and cannot
exist as an independent database object. The SDK automatically deletes
embedded objects if their parent object is deleted or when overwritten
by a new embedded object instance.

You can declare an object as an embedded object
that does not have a lifecycle independent of the object in which it
is embedded. This differs from a to-one
or to-many relationship, in which the
related objects have independent lifecycles.

Provide a `REALM_EMBEDDED_SCHEMA` with the struct or class name and the
names of any properties that you want the database to persist.

Define a property as an embedded object on the parent object by setting
a pointer to the embedded object's type.

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

