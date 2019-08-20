How to build reliable.io
========================

## Building on Windows

Download [premake 5](https://premake.github.io/download.html) and copy the **premake5** executable somewhere in your path. Please make sure you have at least premake5 alpha 13.

You need Visual Studio to build the source code. If you don't have Visual Studio 2015 you can [download the community edition for free](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx).

Once you have Visual Studio installed, go to the command line under the reliable.io/c directory and type:

    premake5 solution

This creates reliable.sln and opens it in Visual Studio for you.

Now you can build the library and run individual test programs as you would for any other Visual Studio solution.

## Building on MacOS and Linux

First, download and install [premake 5](https://premake.github.io/download.html). Please make sure you have at least premake5 alpha 13.

Now go to the command line under the reliable.io directory and enter:

    premake5 gmake

Which creates makefiles which you can use to build the source via:

    make all

Alternatively, you can use the following shortcuts to build and run test programs directly:

    premake5 test           // build and run unit tests

    premake5 soak           // run the soak test that exercises all library functionality

    premake5 fuzz           // run the fuzz test that tests the library is able to correctly handle random data

If you have questions please create an issue at https://github.com/networkprotocol/reliable.io and I'll do my best to help you out.

cheers

 - Glenn

## Building with CMake

Download and install [CMake](https://cmake.org/).

First clone this repository, then create a build folder and run cmake. In source and out of source build is supported.

```bash
# Clone the repository
git clone https://github.com/OlivierLDff/reliable.io
# Enter the folder and create a build folder (build folder is added in .gitignore)
cd reliable.io && mkdir build && cd build
# Run cmake
cmake ..
```

The `CMakeLists.txt` comes with multiples options for you to override when executing with CMake. simply add `-D<option>=<value>`.

### Parameters

CMake support multiple options.

- **RELIABLE_TARGET**: The name of the target that will be generated. *Default "reliable"*.
- **RELIABLE_BUILD_SHARED**: Build a shared library. *Default "OFF". [ON/OFF]*.
- **RELIABLE_FOLDER_PREFIX**: Prefix folder for all reliable.io generated targets in generated project (only decorative). *Default "reliable".*
- **RELIABLE_ENABLE_TESTS**: Enable the tests. This will create a target `${RELIABLE_TESTS_PREFIX}_test`. *Default "OFF". [ON/OFF]*.
- **RELIABLE_TESTS_PREFIX**: Prefix for every tests to avoid naming clashes in superbuild. *Default "reliable".*
- **RELIABLE_ENABLE_EXAMPLES**: Enable examples. This will create a target for each examples. *Default "OFF". [ON/OFF]*.
  - `${RELIABLE_EXAMPLES_PREFIX}soak`: The soak test that exercises all library functionality.
  - `${RELIABLE_EXAMPLES_PREFIX}fuzz`: The fuzz test that tests the library is able to correctly handle random data.
  - `${RELIABLE_EXAMPLES_PREFIX}stats`: Run stats.
- **RELIABLE_EXAMPLES_PREFIX**: Prefix for every examples to avoid naming clashes in superbuild. *Default "reliable".*
- **RELIABLE_ENABLE_INSTALL**: Enable install target. *Default "OFF". [ON/OFF]*.
- **RELIABLE_INSTALL_PREFIX**: Folder for all reliable.io headers in the install folder. *Default "reliable".*

If you want to enable everything:

```bash
cmake                                \
  -DRELIABLE_BUILD_SHARED=OFF      \
  -DRELIABLE_TARGET="reliable"  \
  -DRELIABLE_ENABLE_TESTS=ON       \
  -DRELIABLE_ENABLE_EXAMPLES=ON    \
  -DRELIABLE_ENABLE_INSTALL=ON     \
  ..
```

### Build

Depending on the generator use, you can use the generator specific command like `make` or `ninja` or `msbuild`. When building with a one release type generator you might need to add `-DCMAKE_BUILD_TYPE=Release` or `-DCMAKE_BUILD_TYPE=Debug`. Available config depends on your generator.

More generally you can simply use cmake build command.

```bash
## Equivalent of make all for any generator
cmake --build .
## Equivalent of "make reliable" in release mode
cmake --build . --target reliable --config Release
```

The `reliable` binary will be available in the `lib` folder of your build folder.

### Install

An install target is available for convenience and will deploy reliable.io on your system or any where you want.

```bash
## Specifify a custom install folder (optionnal)
cmake -DRELIABLE_ENABLE_INSTALL=ON -DCMAKE_INSTALL_PREFIX="/path/to/my/install/dir" ..
## Equivalent of "make install" (Debug)
cmake --build . --target install --config Debug
## Equivalent of "make install" (Release)
cmake --build . --target install --config Release
```

You can choose the install directory by setting **CMAKE_INSTALL_PREFIX** when configuring the project. By default on Unix system it is set to `/usr/local` and on Windows to `c:/Program Files/reliable`.

In this folder you will find an include folder ready to be included by another application. This is a copy of the `include` folder of this repository. A `lib` will be created with all the generated libraries. A `cmake` folder contain all the CMake scripts to find the package.

The installation prefix is also added to `CMAKE_SYSTEM_PREFIX_PATH` so that **find_package()**, **find_program()**, **find_library()**, **find_path()**, and **find_file()** will search the prefix for other software.

### Run Tests

To run the tests you need to compile the library as static, and set the `RELIABLE_ENABLE_TEST` flag to ON. Testing is done using CTest framework.

```bash
## Enable the tests
cmake -DRELIABLE_ENABLE_TESTS=ON ..
## Build all the tests and executables
cmake --build . --config Release
# Then run the tests
ctest -C Release
```