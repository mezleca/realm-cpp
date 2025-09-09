# Install the C++ SDK
Realm SDK for C++ enables client applications written in C++ to access
data stored on devices. This page details how to
install the C++ SDK in your project and get started.

## Requirements
- Minimum C++ standard: C++17.
- For development on macOS: Xcode 11.x or later.
- For development on Windows: Microsoft Visual C++ (MSVC).
- Otherwise, we recommend git and [CMake](https://cmake.org).

## Install
> Tip:
> The SDK uses Realm Core database for device data persistence. When you
install the C++ SDK, the package names reflect Realm naming.
>

#### Swiftpm

When developing with Xcode, you can use Swift Package Manager (SPM) to
install realm-cpp.

##### Add Package Dependency
In Xcode, select `File` > `Add Packages...`.

##### Specify the Repository
Copy and paste the following into the search/input box.

```sh
https://github.com/realm/realm-cpp
```

##### Select the Package Products
Under Package Product, select `realm-cpp-sdk`. Under
Add to Target, select the target you would like to add
the SDK to. For example, the target might be the main executable of
your app. Click Add Package.

#### Cmake

You can use CMake with the FetchContent module to manage the SDK and its
dependencies in your C++ project.

Create or modify your `CMakeLists.txt` in the root directory of your
project:

1. Add `Include(FetchContent)` to include the FetchContent module
in your project build.
2. Use `FetchContent_Declare` to locate the SDK dependency
and specify the version tag you want to use.
3. Use the `FetchContent_MakeAvailable()` command to check whether
the named dependencies have been populated, and if not, populate them.
4. Finally, `target_link_libraries()` links the SDK dependency to
your target executable.

To get the most recent version tag, refer to the releases on GitHub:
[realm/realm-cpp](https://github.com/realm/realm-cpp/releases).

Set the minimum C++ standard to 17 with `set(CMAKE_CXX_STANDARD 17)`.

In a Windows install, add the required compiler flags listed below.

```cmake
cmake_minimum_required(VERSION 3.15)

project(MyDeviceSDKCppProject)

# Minimum C++ standard
set(CMAKE_CXX_STANDARD 17)

# In a Windows install, set these compiler flags:
if(MSVC)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:preprocessor /bigobj")
endif()

# Include the FetchContent module so you can download the C++ SDK
Include(FetchContent)

# Declare the version of the C++ SDK you want to download
FetchContent_Declare(
  cpprealm
  GIT_REPOSITORY https://github.com/realm/realm-cpp.git
  GIT_TAG        v1.0.0
)

# The MakeAvailable command ensures the named dependencies have been populated
FetchContent_MakeAvailable(cpprealm)

# Create an executable target called myApp with the source file main.cpp
add_executable(myApp main.cpp)

target_link_libraries(myApp PRIVATE cpprealm)
```

Run CMake in a gitignored directory, such as `build`, to generate the build
configurations that you can then use to compile your app:

```bash
# build/ is in .gitignore
mkdir build
cd build
cmake .. # Create Makefile by reading the CMakeLists.txt in the parent directory (../)
make # Actually build the app
```

You can use CMake to generate more than simple Makefiles by using the `-G`
flag. See the [CMake documentation](https://cmake.org/documentation/) for more
information.

## Usage
### Include the Header
Make the C++ SDK available in your code by including the
`cpprealm/sdk.hpp` header in the translation unit where you want to use it:

```cpp
#include <cpprealm/sdk.hpp>

```

## Build an Android App
The C++ SDK supports building Android apps. To build an Android
app:

- Add `<uses-permission android:name="android.permission.INTERNET" />` to your `AndroidManifest.xml`
- Add the subdirectory of the C++ SDK to your native library's `CMakeLists.txt`
and link it as a target library: `set(CMAKE_CXX_STANDARD 17)
add_subdirectory("realm-cpp")
...
target_link_libraries(
   # Specifies the target library.
   myapplication
   # make sure to link the C++ SDK.
   cpprealm
)`
- Ensure that the git submodules are initialized inside of the `realm-cpp` folder before building.
- When instantiating the database or the SDK App, you must pass the `filesDir.path` as the `path`
parameter in the respective constructor or database open template.

For an example of how to use the C++ SDK in an Android app, refer to
the [Android RealmExample App](https://github.com/realm/realm-cpp/tree/main/examples/Android)
in the `realm-cpp` GitHub repository.

Specifically, refer to the `MainActivity.kt` & `native-lib.cpp` files
in the Android example app for code examples.
