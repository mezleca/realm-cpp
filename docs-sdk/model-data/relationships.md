# Relationships - C++ SDK
Realm doesn't use bridge tables or explicit joins to define
relationships as you would in a relational database. Realm
handles relationships through embedded objects or reference properties to
other Realm objects. You read from and write to these
properties directly. This makes querying relationships as performant as
querying against any other property.

## Relationship Types
Realm supports **to-one**, **to-many**, and **inverse**
relationships. Realm also provides a special type of object, called an
embedded object, that is conceptually
similar to a relationship but provides additional constraints.

### To-One Relationship
A **to-one** relationship means that an object relates to one other object.
You define a to-one relationship for an object type in its object
model. Specify a property where the type is the related Realm
object type. For example, a dog might have a to-one relationship with
a favorite toy.

### To-Many Relationship
A **to-many** relationship means that an object relates to more than one
other object. In Realm, a to-many relationship is a list of
references to other objects. For example, a person might have many dogs.

You can represent a to-many relationship between two Realm
types as a list, map, or a set. Lists, maps, and sets are mutable: within
a write transaction, you can add and remove elements to and from these
collection types. Lists, maps, and sets are not associated with a query
and are declared as a property of the object model.

### Inverse Relationship
Relationship definitions in Realm are unidirectional. An
**inverse relationship** links an object back to an object that refers
to it.

An inverse relationship property is an automatic backlink relationship.
Realm automatically updates implicit relationships whenever an object is
added or removed in a corresponding to-many list or to-one relationship
property. You cannot manually set the value of an inverse relationship
property.

## Declare Relationship Properties

### Define a To-One Relationship
A **to-one** relationship maps one property to a single instance of
another object type. For example, you can model a dog having at most one
favorite toy as a to-one relationship.

Setting a relationship field to null removes the connection between objects.
Realm does not delete the referenced object, though, unless it is
an embedded object.

> Important:
> When you declare a to-one relationship in your object model, it must
be an optional property. If you try to make a to-one relationship
required, Realm throws an exception at runtime.
>

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

### Define a To-Many Relationship
A **to-many** relationship maps one property to zero or more instances
of another object type. For example, you can model a company having any
number of employees as a to-many relationship.

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

### Define an Inverse Relationship
To define an inverse relationship, use `linking_objects` in your object
model. The `linking_objects` definition specifies the object type and
property name of the relationship that it inverts.

In this example, we define a `Person` having a to-one relationship with
a `Dog`. The `Dog` has an inverse relationship to any `Person`
objects through its `owners` property.

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
