# Configure & Open a Realm - C++ SDK
A **realm** is the core data structure used to organize data in
Realm. A realm is a collection of the objects that you use
in your application, called Realm objects, as well as additional metadata
that describe the objects.

When opening a realm, you must specify a db_config.
The `db_config` may contain information such as:

- An optional path where the realm is stored on device
- An optional list of models that the realm should manage
- An optional scheduler if you need to customize the run loop

You can use the default `db_config` constructor if you do not need
to specify a realm file path or other
configuration details.

## Realm Files
Realm stores a binary encoded version of every object and type in a
realm in a single `.realm` file. The file is located at a specific
path that you can define when you open the realm.
You can open, view, and edit the contents of these files with
.

### Auxiliary Files
The SDK creates additional files for each realm:

- **realm files**, suffixed with "realm", e.g. default.realm:
contain object data.
- **lock files**, suffixed with "lock", e.g. default.realm.lock:
keep track of which versions of data in a realm are
actively in use. This prevents realm from reclaiming storage space
that is still used by a client application.
- **note files**, suffixed with "note", e.g. default.realm.note:
enable inter-thread and inter-process notifications.
- **management files**, suffixed with "management", e.g. default.realm.management:
internal state management.

### Open the Default Realm
Opening a realm without specifying an optional path opens the default realm
in the current directory.

When opening a realm, the C++ SDK can automatically infer which
models are available in the project.

```cpp
auto config = realm::db_config();
auto realm = realm::db(std::move(config));

```

> Tip:
> When building an Android app that uses the Realm C++ SDK,
you must pass the `filesDir.path` to the `path` parameter in the
db_config
constructor. For more information, refer to: Build an Android App.
>

### Open a Realm at a File Path
You can use `set_path()` to specify a path for the `db_config`
to use when opening the realm.

```cpp
auto relative_realm_path_directory = "custom_path_directory/";
std::filesystem::create_directories(relative_realm_path_directory);
// Construct a path
std::filesystem::path path =
    std::filesystem::current_path().append(relative_realm_path_directory);
// Add a name for the database file
path = path.append("employee_objects");
// Add the .realm extension
path = path.replace_extension("realm");
// Set the path on the config, and open the database at the path
auto config = realm::db_config();
config.set_path(path);
auto realmInstance = realm::db(std::move(config));

```

> Tip:
> When building an Android app that uses the Realm C++ SDK,
you must pass the `filesDir.path` as the `path` parameter in the
db_config
constructor. For more information, refer to: Build an Android App.
>

## Provide a Subset of Models to a Realm
> Tip:
> Some applications have tight constraints on their memory footprints.
To optimize your realm memory usage for low-memory environments, open the
realm with a subset of models.
>

By default, the C++ SDK automatically adds all object types that have a
Object Schema in your executable to your
Database Schema.

However, if you want the realm to manage only a subset of models, you
can specify those models by passing them into the template parameter list
of the `realm::open()` function.

```
auto config = realm::db_config();
auto realm = realm::open<realm::Dog>(std::move(config));

```

## Close a Realm
To avoid memory leaks, close the database when you're done using it. Closing
the database invalidates any remaining objects. Close the database with
`db::close()`.

```cpp
// Create a database configuration.
auto config = realm::db_config();
auto realm = realm::db(config);

// Use the database...

// ... later, close it.
realm.close();

// You can confirm that the database is closed if needed.
CHECK(realm.is_closed());

// Objects from the database become invalidated when you close the database.
CHECK(specificDog.is_invalidated());

```
