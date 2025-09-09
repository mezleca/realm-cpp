# Logging - C++ SDK
You can set or change your app's log level to develop or debug your
application. You might want to change the log level to log different
amounts of data depending on the app's environment.

## Set the Realm Log Level
You can set the level of detail reported by the Realm C++ SDK. Pass a
realm::logger::level
to the `set_default_level_threshold()` member function:

```cpp
auto logLevel = realm::logger::level::info;
realm::set_default_level_threshold(logLevel);

```

## Customize the Logging Function
To set a custom logger function, create a
realm::logger
and override the virtual `do_log()` member function:

```cpp
struct MyCustomLogger : realm::logger {
  // This could be called from any thread, so may not output visibly to the
  // console. Handle output in a queue or other cross-thread context if needed.
  void do_log(realm::logger::level level, const std::string &msg) override {
    std::cout << "Realm log entry: " << msg << std::endl;
  }
};

```

Then, initialize an instance of the logger and set it as the default logger
for your realm:

```cpp
auto config = realm::db_config();
auto thisRealm = realm::db(config);
auto myLogger = std::make_shared<MyCustomLogger>();
realm::set_default_logger(myLogger);

```
