---
layout: post
title:  OrbSlam3 on Android 10
categories: [SLAM,Android]
---

This post keeps track of how I implemented the recently released OrbSlam 3 on an Android 10 smartphone, and some thinking about the process. 

## Organize Cmake project

- Typical structiure uses **cmake_minimum_required(), project(), add_library(), target_link_libraries(), target_include_directories()**
- add_subdirectory() will inherit the ${} macros defined before it, and the parent file which includes the add_subdirectory() can refer to the library it generates directly. 
- Proper command to copy intermediate shared library files to a location is 
```sh
add_custom_command(TARGET shared-library-name POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:shared-library-name> ${PROJECT_SOURCE_DIR}/lib/new-shared-library-name(e.g. shared-library-name or libshared-library-name.so) 
        )
```
- It seems that Android 10 will only pack .so files into the final apk if the .so files are located in **src/main/jniLibs**
- Organize the cmake tree in layers and bottom-up fashion to save the compiling time for large cmake project. 


## Build dependency libs on Android 10

### boost 
Orbslam3 used boost serialization in its code. I already mentioned how to build boost for this task in another [post](https://zhaoqunzhong.github.io/boost-for-android/). There are two questions you might ask though: 
1. Why static library instead of shared library
	The author answered the question in the repo's issue section. The main reason was his personal choice and his task at hand. 
2. Why use the generated wserialization instead of serialization file
	The w prefix stands for "wide char" version, which has something to do with the target platform data type. I tried both, only the w version worked for Android 10, I didn't dig the reason though. 

There are also other choices for the serialization task, like Google protobuf, which should has a better support for Android platform.

### g2o
I mentioned the process for building g2o and a working demo in another [post](https://zhaoqunzhong.github.io/g2o-for-android/)

### Dbow2
There exists newer optimized version of the DBOW2/DBOW3, at https://github.com/rmsalinas/fbow. 

## Optimization and speed up 

### Vocabulary file .bin instead of .txt 
It takes about 1 min to load the >100Mb .txt vocabulary file on Android, while the .bin version takes about 1s. Notice that Vins-Mono used the .bin verison for their relocalization. 

## Compiler and language related errors (unsolved)

There are two compile errors related to Eigen3.3.7 library and c++11 std map library, both caused by the assert action in the library code. Solutions:
1. eigen-3.3.7\Eigen\src\Core\AssignEvaluator.h
	Comment out line 833 
```cpp
EIGEN_STATIC_ASSERT_SAME_MATRIX_SIZE(ActualDstTypeCleaned,Src)
```
2. ${ndk root}\toolchains\llvm\prebuilt\windows-x86_64\sysroot\usr\include\c++\v1\map
	Comment out line 911
```cpp
static_assert((is_same<typename allocator_type::value_type, value_type>::value),
                  "Allocator::value_type must be same type as value_type");
```
Although this temporary fixes the compile errors, it's really terrible to mess up with the library source code, which might affects other projects on this computer. 