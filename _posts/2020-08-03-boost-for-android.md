---
layout: post
title: Build boost for Android
categories: [SLAM, Android]
---

There is an awsome github [repo](https://github.com/moritz-wundke/Boost-for-Android) which does exactly what the title says. This post mainly introduce some detail about how to use it.

## Test Environment

Ubuntu 18

## Steps

1. Make sure Android NDK is installed, via Android Studio or stand-alone installation. $(NDK_ROOT) is needed for compilation.

2. Read build_android.sh and config compilation options as illustrated. 

	./build-android.sh $(NDK_ROOT) \
	--boost=1.73.0 \
	--with-libraries=serialization \
	--arch=arm64-v8a \
	--target-version=29 \

3. Find your generated libs in /build/out/{the ABI you chose}/lib, and the include files in /build/out/{the ABI you chose}/include. 