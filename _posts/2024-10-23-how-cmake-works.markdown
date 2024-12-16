---
layout: post
title:  "how cmake's find_package works"
date:   2024-10-24 18:41:00 -0400
categories: cpp cmake
---

I spent 8 hours trying to use find_package to build a project with zeromq as a dependency because I was a bit confused about how the find_package function works. In theory, the find_package function works like this:
```cmake
find_package(package_name REQUIRED)

add_execuatble(main main.cpp)

target_link_libraries(main package_name)
```
In practice, you will likely see the error that looks like this[^1]:
![CMake Error](/images/cmake_error.png)
[^2]

It tries to tell you that it could not find the package configuration file that it usually reads to build a package.[^3] The way I would usually try to fix this after encountering this error is I would manually include the libraries and link the library to the target.

CMake has a list of built in `Find<package>.cmake` configuration files that it searches when the find_package function is called. The list of all such configuration files can be found in `<cmake_root>/share/cmake/Modules` path. If the package argument used in the find_package function call is in the list, everything will work smoothly as normal. At this point, I am going to assume that the cmake configuration file exists in the folder that you are working out of, but it's not in the directory that I mentioned earlier. I will later discuss how to generate this file if it does not exist. 

If the configuration file exists, it will likely be in some `share` subdirectory. In my example, the configuration files were in `<project root>/share` subdirectory that looked like this

```
share
├── cmake
│   └── cppzmq
│       ├── cppzmqConfig.cmake
│       ├── cppzmqConfigVersion.cmake
│       ├── cppzmqTargets.cmake
│       └── libzmq-pkg-config
│           └── FindZeroMQ.cmake
└── cppzmq
    └── examples
        ├── CMakeLists.txt
        ├── hello_world.cpp
        ├── multipart_messages.cpp
        └── pubsub_multithread_inproc.cpp
```
Currently we only care about the cmake subdirectory. At this point, there are 2 options: 1. Add the cmake subdirectory to cmake with a symlink or 2. Modify the CMAKE_MODULE_PATH environment variable. I personally prefer the 1st way. 

There are 2 ways of accomplishing it, you need to build with the CMAKE_INSTALL_PREFIX set to a popular directory in CMAKE_MODULE PATH. The rough way to accomplish this is
```cmake
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/usr/local
cmake --build build
sudo cmake --install build
```
Obviously, package managers make this process much easier because they will usually automatically copy the cmake configs to a CMAKE_MODULE_PATH elmininating the need to carefully move files to the default install prefix. 

To Modify the CMAKE_MODULE_PATH, do something like this. Note that I don't like this approach but it is valid

```cmake
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
``` 
[^4]

[^1]: Yes, I know I spelled cppzmq incorrectly, I was trying to prove a point
[^2]: I feel so silly writing this because it feels so obvious in retrospect. Like literally just read the instructions!
[^3]: If you don't see this error, stop reading now because the rest of the article will be moot.
[^4]: There are a bunch of other ways to add to cmake module path, but I didn't investigate any of them.