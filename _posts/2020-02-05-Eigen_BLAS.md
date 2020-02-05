---
layout: post
title:  Eigen BLAS Note
comments: true
category: c++
---

Basic Linear Algebra Subprograms (BLAS) are small pieces of code, performing linear algebra operations such as vector addition, dot product, matrix multiplication, etc. The implementation takes system architecture (such as cache and instruction set) into consideration and is optimized for each hardware platform. Since BLAS libs are generally independent of application logic, so they are maintained separately and serve as a building block for many numerical programs.

[Eigen](!http://eigen.tuxfamily.org/index.php?title=Main_Page) is a popular library for linear algebra, including vectors, matrices, numerical solvers and related algorithms. While using Eigen in my project, I was uncertain if Eigen embeds BLAS into its design and makes the best usage of the hardware. 

According to the [document](!http://eigen.tuxfamily.org/index.php?title=FAQ#How_does_Eigen_compare_to_BLAS.2FLAPACK.3F), Eigen is as good as most BLAS libraries and supports integration with them (see [doc](!https://eigen.tuxfamily.org/dox/TopicUsingBlasLapack.html)). It has several features:

* **The BLAS-like behavior is enabled automatically in Eigen**, such as vectorization and GEMM. (Though need to specify RELEASE build and *-march=native*)
* It has built-in support for sparse matrix and fixed-size vectors/matrices, which is very common in graphic programs.

However, there are still some problems:

* Eigen only has parallel support for matrix multiplication. **So for other operations, such as vector addition and dot product, it is sequential**.
* Need to take care of memory allocation and many small tips in order to obtain optimal performance. See [document](!http://eigen.tuxfamily.org/index.php?title=FAQ#Optimization). 

In conclusion, three things need to take care of:

* Specify RELEASE build and *-march=native* to improve your Eigen program performance.
* Simple algebra operations are not parallelized, need to do it by your self.
* Read [document](!http://eigen.tuxfamily.org/index.php?title=FAQ#Optimization) to avoid common Eigen performance pit fall.