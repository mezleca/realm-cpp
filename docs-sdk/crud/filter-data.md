# Filter Data - C++ SDK
To filter data in your realm, you can leverage
Realm's query engine. You can also sort filtered data. For an example of
how to sort query results, refer to Sort Lists and Query Results.

> Note:
> The C++ SDK does not yet support the full range of query expressions
that the other SDKs provide.
>

## About the Examples on This Page
The examples in this page use a simple data set for a
todo list app. The two Realm object types are `Project`
and `Item`. An `Item` has:

- A name
- A completed flag
- An optional assignee's name
- A number representing priority, where higher is more important
- A count of minutes spent working on it

A `Project` has a name and a to-many relationship to zero or more `Items`.

The schemas for `Project` and `Item` are:

```cpp
struct Item {
  std::string name;
  bool isComplete;
  std::optional<std::string> assignee;
  int64_t priority;
  int64_t progressMinutes;
};
REALM_SCHEMA(Item, name, isComplete, assignee, priority, progressMinutes)

struct Project {
  std::string name;
  std::vector<Item*> items;
};
REALM_SCHEMA(Project, name, items)

```

You can set up the realm for these examples with the following code:

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

auto item1 = realm::Item{.name = "Save the cheerleader",
                         .assignee = std::string("Peter"),
                         .isComplete = false,
                         .priority = 6,
                         .progressMinutes = 30};

auto project = realm::Project{.name = "New project"};

project.items.push_back(&item1);

realm.write([&] { realm.add(std::move(project)); });

auto items = realm.objects<realm::Item>();
auto projects = realm.objects<realm::Project>();

```

## Filter Data
### Comparison Operators
Value comparisons

|Operator|Description|
| --- | --- |
|==|Evaluates to `true` if the left-hand expression is equal to the right-hand expression.|
|>|Evaluates to `true` if the left-hand numerical or date expression is greater than the right-hand numerical or date expression. For dates, this evaluates to `true` if the left-hand date is later than the right-hand date.|
|>=|Evaluates to `true` if the left-hand numerical or date expression is greater than or equal to the right-hand numerical or date expression. For dates, this evaluates to `true` if the left-hand date is later than or the same as the right-hand date.|
|<|Evaluates to `true` if the left-hand numerical or date expression is less than the right-hand numerical or date expression. For dates, this evaluates to `true` if the left-hand date is earlier than the right-hand date.|
|<=|Evaluates to `true` if the left-hand numeric expression is less than or equal to the right-hand numeric expression. For dates, this evaluates to `true` if the left-hand date is earlier than or the same as the right-hand date.|
|!=|Evaluates to `true` if the left-hand expression is not equal to the right-hand expression.|

> Example:
> The following example uses the query engine's
comparison operators to:
>
> - Find high priority tasks by comparing the value of the `priority` property
value with a threshold number, above which priority can be considered high.
> - Find just-started or short-running tasks by seeing if the `progressMinutes`
property falls within a certain range.
> - Find unassigned tasks by finding tasks where the `assignee` property is
equal to `std::nullopt`.
> - Find tasks assigned to specific teammates Ali or Jamie by seeing if the
`assignee` property is in a list of names.
>
> ```cpp
> auto highPriorityItems =
>     items.where([](auto const& item) { return item.priority > 5; });
>
> auto quickItems = items.where([](auto const& item) {
>   return item.progressMinutes > 1 && item.progressMinutes < 30;
> });
>
> auto unassignedItems = items.where(
>     [](auto const& item) { return item.assignee == std::nullopt; });
>
> auto aliOrJamieItems = items.where([](auto const& item) {
>   return item.assignee == std::string("Ali") ||
>          item.assignee == std::string("Jamie");
> });
>
> ```
>

### Logical Operators
You can use the logical operators listed in the following table to make compound
predicates:

|Operator|Description|
| --- | --- |
|&&|Evaluates to `true` if both left-hand and right-hand expressions are `true`.|
|!|Negates the result of the given expression.|
|\\|\\||Evaluates to `true` if either expression returns `true`.|

> Example:
> We can use the query language's logical operators to find
all of Ali's completed tasks. That is, we find all tasks
where the `assignee` property value is equal to 'Ali' AND
the `isComplete` property value is `true`:
>
> ```cpp
> auto completedItemsForAli = items.where([](auto const& item) {
>   return item.assignee == std::string("Ali") && item.isComplete == true;
> });
>
> ```
>

### String Operators
You can compare string values using these string operators.

|Operator|Description|
| --- | --- |
|.contains(_ value: String)|Evaluates to `true` if the left-hand string expression is found anywhere in the right-hand string expression.|
|==|Evaluates to `true` if the left-hand string is lexicographically equal to the right-hand string.|
|!=|Evaluates to `true` if the left-hand string is not lexicographically equal to the right-hand string.|

> Example:
> The following example uses the query engine's string operators to find:
>
> - Projects with names that contain 'ie'
>
> ```cpp
> auto containIe =
>     items.where([](auto const& item) { return item.name.contains("ie"); });
>
> ```
>
