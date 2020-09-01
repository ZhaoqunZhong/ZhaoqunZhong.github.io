---
layout: post
title:  Build SuiteSparse for Android 10+
categories: [Optimization, SLAM, Android]
---

## BLAS & LAPACK

There are several options for preparing BLAS & LAPACK library. 

1. Figure out a gnu compiling toolchain that support both c++ and gfortran, and compile original fortran based BLAS & LAPACK, integrate it in your c++ project. 
	
	The tricky part is android ndk build no longer supports gcc in recent versions, clang is default, but clang doesn't support fortran language. So it comes down to build a gnu toolchain that supports both c++ and fortran, and can be used by ndk-build. 

	But if you manage to do that, there exists several LAPACK interfaces can potentially increase the performance of the final library: Lapacke, Lapack++, openblas... They are called interfaces because they don't provide the fundemental math functions in c/c++, they just wrap the fortran implementations with c/c++ to utilize some modern language feature of c/c++. 

2. Use a libf2c library to transform fortran source files into c files

	Eigen3 uses this method to provide a blas implementation that utilize its own data structure. It's blas module is pure c/c++ and can be directly compiled, but for some reason its lapack module still keeps some fortran files. You can choose to translate them into c then compile them if you think the BLAS & LAPACK module can benefit from some Eigen design.  

3. Vendor provided library, these are recommanded if available since they are optimized based on the hardware.

	For Qualcomm cpu platforms there is QML(Qualcomm math library). But Qualcomm's website annources that QML-1.4.0 will be the end of it. I tested it on Android 10 & NDK-29, it still works. But it will break with future sdk versions sooner or later.

	For Intel platforms there is Inter MKL, which is recommanded by suitesparse's website against OpenBLas. But sadly it can't be used on Arm based architectures. 

## SuiteSparse

Original suitesparse source code uses makefile based building system. Thanks to [suitesparse-metis-for-windows](https://github.com/jlblancoc/suitesparse-metis-for-windows), with a little tweak, we can build it with ndk cmake toolchain. To solve its dependency on BLAS & LAPACK, you only need to link suitesparse to whatever lib of BLAS & LAPACK you choose. No target_include needed, since all BLAS & LAPACK libraries conform to a common interface. 

## Test the suitesparse libraries on unrooted android phone

1. The working directory would be /data/local/tmp. Copy everything related to test examples to this folder by adb push: libc++_shared.so, libQML-1.4.0.so, test data, and suitesparse libs. 

2. Set the c++ linking directory to /data/local/tmp to force the executables link libraries from this folder instead of looking from /system/lib64
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/local/tmp
```