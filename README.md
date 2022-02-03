scala-lsgo-benchmark
====================

[![License](http://img.shields.io/:License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.html)
[![Scala](http://img.shields.io/:Scala-2.13.7-green.svg)]()

**scala-lsgo-benchmark** is a Scala implementation of the test suite used in the competitions organised by the [IEEE Task Force on Large Scale Global Optimization (LGSO)](http://www.tflsgo.org) as part of the IEEE Congresses on Evolutionary Computation (CEC).

### Original code

This implementation of the test suite is based on the original C++ implementation which is available here: [lsgo_2013_benchmarks_original.zip](http://www.tflsgo.org/assets/cec2018/lsgo_2013_benchmarks_original.zip). A Matlab implementation and a Java wrapper are also available in the zip file.

Also a Python wrapper for the C++ implementation of the test suite can be found here: https://pypi.python.org/pypi/cec2013lsgo

Detailed information about these benchmark functions is provided in the following technical report:

[X. Li, K. Tang, M. Omidvar, Z. Yang and K. Qin, “Benchmark Functions for the CEC’2013 Special Session and Competition on Large Scale Global Optimization,” Technical Report, Evolutionary Computation and Machine Learning Group, RMIT University, Australia, 2013](http://www.tflsgo.org/assets/cec2018/cec2013-lsgo-benchmark-tech-report.pdf)

Setup
-----

In order to use the **scala-lsgo-benchmark** in your Scala project add the following to your `build.sbt`:

```scala
libraryDependencies += "org.apache.commons" % "commons-math3" % "3.3" // this is optional

externalResolvers += "Scala LSGO2013 packages" at "https://maven.pkg.github.com/xoanpardo/scala-lsgo-benchmarks"
libraryDependencies += "xoanpardo" %% "scala-lsgo-benchmarks" % "0.1.0"
```
The Apache Commons Math dependency is only necessary to generate random samples. If you do not plan to use random samples in your code or they are generated by other means it can be skipped.

> **Note**: the original `cdatafiles` from the C++ implementation are included in the distribution package for convenience.

### Configuring credentials

The distribution package is stored in the [Github Packages Registry](https://help.github.com/en/articles/about-github-package-registry). To install the package and use it as a dependency in your projects is necessary to first configure the credentials to access the Github API. The steps are the following:

1. In your Github account [create a PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-token) (Personal Access Token) with the `read:packages` [permissions](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries).
2. Store the PAT somewhere safe (e.g. NOT in the repository of your project) and make it accessible to `sbt`. There are different ways of doing this, a common approach is to have a global `$HOME/.sbt/1.0/github.sbt` file with the following contents:  

```scala
credentials += Credentials("GitHub Package Registry", "maven.pkg.github.com", "<GITHUB USERNAME>", "<GITHUB TOKEN>")
```

Usage
-----

**scala-lsgo-benchmark** is very easy to use. For each function class in the benchmark there is a companion object to make it easier to create function instances. 

```scala
import gal.udc.gac.lsgo2013.Benchmark._

val f1 = F1() // F1 to F15 companion objects are available
val f10 = F10()
```

Also, a benchmark constructor is available to get an instance of all benchmark functions at once.

```scala
import gal.udc.gac.lsgo2013.Benchmark._

val lsgo2013 = LSGO2013() // benchmark constructor, returns a Vector of function instances
val f1 = lsgo2013(0) // another way to get a function instance (index is the function ID minus 1)
val f10 = lsgo2013(9)
```

### Getting information about a function

For each benchmark function the following information is accessible:
- `id` the function name.
- `info` an instance of `SearchSpaceParameters` containing the search space dimensionality and boundaries.

This information can be obtained in two ways, directly from an instance of the function or from the companion object of its class. The following are examples of both.

```scala
import gal.udc.gac.lsgo2013.Benchmark._

val f1 = F1()
val info = f1.info // dimension and boundaries of the search space obtained directly from the function instance
println(s"${f1.id}: (dim=${info.dim}, lower=${info.lower}, upper=${info.upper})") // F1: (dim=1000, lower=-100.0, upper=100.0)
```

```scala
import gal.udc.gac.lsgo2013.Benchmark._

val info = F1.info // dimension and boundaries of the search space obtained from the companion object
println(s"${F1.id}: (dim=${info.dim}, lower=${info.lower}, upper=${info.upper})") // F1: (dim=1000, lower=-100.0, upper=100.0)
```

### Getting information about all benchmark functions

```scala
import gal.udc.gac.lsgo2013.Benchmark._

val lsgo2013 = LSGO2013() // benchmark constructor
lsgo2013.foreach( f =>
  println(s"${f.id}: (dim=${f.info.dim}, lower=${f.info.lower}, upper=${f.info.upper})")
)
```

### Evaluating a function

Every function class is implemented by extending a base function definition `FitnessFunction` and overriding its `apply` method, so they are evaluated in the same way as any other function. The following is an example of how to evaluate a function at zero. 

```scala
import gal.udc.gac.lsgo2013.Benchmark._

val f1 = F1() // f1 is a FitnessFunction instance
val zero = Vector.fill(f1.info.dim)(0.0) // vector of zeros
val expected = 2.09833896353343505859E+11 // expected result of f1(zero)
assert(f1(zero) == expected) // invoking the apply() method of F1
```

### Evaluating all benchmark functions at once

This is an extended version of the previous example in which all the benchmark functions are evaluated at zero.

```scala
import gal.udc.gac.lsgo2013.Benchmark._
import gal.udc.gac.lsgo2013.dim // default dimension for all functions

val zero = Vector.fill(dim)(0.0) // vector of zeros
val tolerance: Double = 8.0E-16 // maximum allowed relative round-off error
val solutions = Vector( // expected results (taken from the C++ implementation)
  2.09833896353343505859E+11,
  4.76203116166061372496E+04,
  2.17290025349525564025E+01,
  1.07955147656065953125E+14,
  4.84191483329246416688E+07,
  1.07773246530947787687E+06,
  9.93826981321072625000E+14,
  5.72227150187806412800E+18,
  6.00160320250193595886E+09,
  9.81154816487000137568E+07,
  1.04485201647212016000E+17,
  1.71135423694972143555E+12,
  8.27380048985966720000E+16,
  4.40797968120962457600E+18,
  2.39389233661550150000E+15)

val lsgo2013 = LSGO2013() // benchmark constructor
lsgo2013.zip(solutions) foreach { case (f, expected) => // every function is zipped with its expected result
  assert(math.abs(f(zero) - expected) / expected <= tolerance, s"Round-off error greater than tolerance at ${f.id}")
}
```

Additional utilities
--------------------

Included with the test suite there are some helper utilities to generate random samples, to scale vectors to search space boundaries, to read the `cdatafiles` included in the distribution package and to measure execution times.
In the following there are some examples showing how to use them. For more examples refer to the tests source code in the `src/test/scala` directory.

### Evaluating a random solution

The following example shows how to generate a random sample scaled to the search space boundaries of a function. 

```scala
import gal.udc.gac.lsgo2013.Benchmark._
import gal.udc.gac.lsgo2013.util.Random.reals
import gal.udc.gac.lsgo2013.util.SearchSpace._

val f1 = F1()
val random = Vector.fill(f1.info.dim)(reals.sample()).bound(BoundFunction.scale)(f1.info) // Random vector scaled to the search space boundaries
val sol = f1(random)
```

The utilities used in the example are:
* `reals`, a wrapper of [UniformRealDistribution](https://commons.apache.org/proper/commons-math/javadocs/api-3.3/org/apache/commons/math3/distribution/UniformRealDistribution.html) configured to generate random values from a uniform distribution in the range [0.0, 1.0) using a thread-safe [MersenneTwister](https://commons.apache.org/proper/commons-math/javadocs/api-3.3/org/apache/commons/math3/random/MersenneTwister.html) pseudo-random number generator. 
  Also included in the library there are `integer`, the counter-part of `reals` for a binary uniform distribution of integers in the range [0, 1] and
  `generator`, a wrapper of [RandomDataGenerator](https://commons.apache.org/proper/commons-math/javadocs/api-3.3/org/apache/commons/math3/random/RandomDataGenerator.html) configured with the same pseudo-random number generator as `reals`.
* `bound`, a method to bound vectors to the search space boundaries that supports different bounding strategies. Vectors in the search space are represented in the library as instances of `Element`, that is an alias for `Vector[Double]`.
  The `bound` method is added to `Element` instances through an implicit definition and the bounding strategy to be applied is provided as an argument of type `BoundingFunction`. 
  In the example, `BoundFunction.scale` is the bounding strategy that scales a vector of uniform samples in [0.0, 1.0) to the search space boundaries of `F1`.


### Evaluating an optimal solution

The following example shows how to evaluate a function with its optimal solution. The optimal solution is read from the `cdatafile` included in the distribution package.

```scala
import gal.udc.gac.lsgo2013.Benchmark._
import gal.udc.gac.lsgo2013.util.DataFileReader.CDataFileOps
object Reader extends CDataFileOps

val f1 = F1()
val optimum = Reader.readOptimal(f1.id) // reads the optimum solution from the cdatafile included in the distribution package
val expected = 0.0 // expected result of f1(optimum)
assert(f1(optimum) == expected)
```

The `Reader` object is defined by extending the trait `CDataFileOps` which implements the methods to read the `cdatafiles` included in the distribution package. 
There are methods in `CDataFileOps` for each type of `cdatafile` (i.e. `readPermutations`, `readSubcomponents`, etc). 
This trait is mixed into the implementation of the benchmark function classes for each function to read its own `cdatafiles` when necessary.

### Measuring execution time

For convenience, the library includes also an utility function `duration` to measure execution times.
Execution times returned by `duration` are instances of [FiniteDuration](https://www.scala-lang.org/api/2.13.7/scala/concurrent/duration/FiniteDuration.html).

The function has two implementations, one for measuring the execution time of functions or sequences of instructions that return a value and another for functions or sequences of instructions that do not.
The following is an example of the former.


```scala
import scala.concurrent.duration._
import gal.udc.gac.lsgo2013.Benchmark._
import gal.udc.gac.lsgo2013.util.Random.reals
import gal.udc.gac.lsgo2013.util.SearchSpace._
import gal.udc.gac.lsgo2013.util.duration

val f10 = LSGO2013()(9)
val random = Vector.fill(f10.info.dim)(reals.sample()).bound(BoundFunction.scale)(f10.info) // Random vector scaled to search space boundaries
val (sol, t) = duration(f10(random))  // sol = result of f10(random), t = evaluation time in nanoseconds (an instance of FiniteDuration)
println(s"${f10.id} evaluation result is $sol and took ${t.toUnit(MILLISECONDS)}ms")
```


In this second example we are only interested in the average evaluation time and not in the result of function evaluations, so the second form of the `duration` function is used.

```scala
import scala.concurrent.duration._
import gal.udc.gac.lsgo2013.Benchmark._
import gal.udc.gac.lsgo2013.util.Random.reals
import gal.udc.gac.lsgo2013.util.SearchSpace._
import gal.udc.gac.lsgo2013.util.duration

val f1 = LSGO2013()(0)
val random = Vector.fill(f1.info.dim)(reals.sample()).bound(BoundFunction.scale)(f1.info) // Random vector scaled to search space boundaries
val repeats = 100 // number of function evaluations
val t = duration { // t = evaluation time in nanoseconds (an instance of FiniteDuration)
  (1 to repeats).foreach(_ => f1(random))
}
println(s"Average time of $repeats evaluations = ${t.toUnit(MILLISECONDS)/repeats}ms")
```

> **Note**: Scala runs in the JVM so, although not included in the examples, it is recommended to perform some evaluations to warm up the JIT compiler before measuring execution times.

Validation of the implementation 
--------------------------------

Several tests have been performed to validate the correctness of the Scala implementation by comparing its results with the original C++ implementation:
* **Individual tests**: the base functions (i.e. Sphere, Rastrigin, Ackley, ...) and the transformation functions (i.e. shifting, ill-conditioning, irregularities, rotation, ...) have been tested individually.
* **Combined tests**: the transformation functions have been tested in combination with each other (e.g. shifting + oscillation + asymmetry + ill-conditioning).
* **Full tests**: the benchmark functions has been tested at zero, at their optimal and at random samples.

All the tests that use random samples have been run a minimum of 1.000 times using the same random samples for both the C++ and Scala implementations and the results compared. 

For individual tests, it has been found that the result is accurate most of the time and that only for some base functions and transformations, there are a small number of 
runs (< 5%) with relative errors below or around the [machine epsilon](https://en.wikipedia.org/wiki/Machine_epsilon) (approximated as 2.220446049250313080847263336182E-16 in the computer 
where the tests were run using the following code):

```scala
var e = 1.0
while (1.0 + e > 1.0) e *= 0.5
e *= 2.0
e
```
For combined and full tests, the results are also accurate in most cases and only a small number of runs (< 5%) have accumulated round-off errors that in all cases are lower than 2.0E-14.
The exception is the Ackley-based functions f3, f6 and f10, which have shown the worst results (e.g. f10 is the worst case with an accuracy of 17% and round-off errors up to 6.0E-13).

The source code for the tests is available in `src/main/resources/c` for the C++ implementation and in `src/test/scala` for the Scala implementation. 
Contact the author if you are interested in the detailed results of the validation tests.

License
-------

This code is open source software licensed under the [GPLv3 License](https://www.gnu.org/licenses/gpl-3.0.html).

Contact
-------

**Xoán C. Pardo**  <[xoan.pardo@udc.gal](mailto:xoan.pardo@udc.gal)> \
Computer Architecture Group / CITIC\
Universidade da Coruña (UDC)\
Spain

[![ORCID](http://img.shields.io/:ORCID-0000--0001--8577--6980-yellow.svg)](https://orcid.org/0000-0001-8577-6980)
[![ResearchID](http://img.shields.io/:ResearchID-D--8250--2015-green.svg)](https://publons.com/researcher/2420618/xoan-c-pardo/)

[mailto:xoan.pardo@udc.gal]: xoan.pardo@udc.gal