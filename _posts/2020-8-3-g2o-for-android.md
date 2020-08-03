---
layout: post
title: Build g2o for Android
categories: [SLAM, Android]
---

## Get g2o source code 

https://github.com/RainerKuemmerle/g2o

## Build g2o for Android 10

1. Create a build_android.sh script into g2o source folder

2. Create a new folder build

3. Edit build_android.sh

```sh
cd build
rm -r *

cmake \
-DANDROID_ABI=arm64-v8a \ 
-DANDROID_NDK=/home/zhaoqun/Android/Sdk/ndk/21.3.6528147 \
-DCMAKE_TOOLCHAIN_FILE=/home/zhaoqun/Android/Sdk/ndk/21.3.6528147/build/cmake/android.toolchain.cmake \
-DANDROID_NATIVE_API_LEVEL=27 \ #minimal Android api level you want to support
-DCMAKE_BUILD_TYPE=Release \
-DEIGEN3_INCLUDE_DIR=/home/zhaoqun/Documents/eigen-3.3.7 \
-DEIGEN3_VERSION_OK=ON ..

make -j8
```

4. Find the generated shared library in {g2o source folder}/lib

## Use g2o in Android source code

1. Copy the {g2o source folder}/config.h into {g2o source folder}/g2o and include the {g2o source folder}/g2o folder in your Android cpp source folder. 

2. Copy the .so libs into the Android cpp source folder as you need, and target_link_libraries() to them.

3. A demo Android app project which is compatible with Android 10 and arm64-v8a can be found at this github [repo](https://github.com/ZhaoqunZhong/g2o-android-test).