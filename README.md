# Measurements.jl

| **Documentation**                       | [**Package Evaluator**][pkgeval-link] | **Build Status**                          | **Code Coverage**               |
|:---------------------------------------:|:-------------------------------------:|:-----------------------------------------:|:-------------------------------:|
| [![][docs-stable-img]][docs-stable-url] | [![][pkg-0.5-img]][pkg-0.5-url]       | [![Build Status][travis-img]][travis-url] | [![][coveral-img]][coveral-url] |
| [![][docs-latest-img]][docs-latest-url] | [![][pkg-0.6-img]][pkg-0.6-url]       | [![Build Status][appvey-img]][appvey-url] | [![][codecov-img]][codecov-url] |

Introduction
------------

`Measurements.jl` is a package that allows you to define numbers with
[uncertainties](https://en.wikipedia.org/wiki/Measurement_uncertainty), perform
calculations involving them, and easily get the uncertainty of the result
according to
[linear error propagation theory](https://en.wikipedia.org/wiki/Propagation_of_uncertainty).
This library is written in [Julia](http://julialang.org/), a modern high-level,
high-performance dynamic programming language designed for technical computing.

When used in the
[Julia interactive session](http://docs.julialang.org/en/stable/manual/getting-started/),
it can serve also as an easy-to-use calculator.

### Features List ###

* Support for most mathematical operations available in Julia standard library
  and special functions
  from [`SpecialFunctions.jl`](https://github.com/JuliaMath/SpecialFunctions.jl)
  package, involving real and complex numbers.  All existing functions that
  accept `AbstractFloat` (and `Complex{AbstractFloat}` as well) arguments and
  internally use already supported functions can in turn perform calculations
  involving numbers with uncertainties without being redefined.  This greatly
  enhances the power of `Measurements.jl` without effort for the users
* Functional correlation between variables is correctly handled, so `x-x ≈
  zero(x)`, `x/x ≈ one(x)`, `tan(x) ≈ sin(x)/cos(x)`, `cis(x) ≈ exp(im*x)`,
  etc...
* Support for
  [arbitrary precision](http://docs.julialang.org/en/stable/manual/integers-and-floating-point-numbers/#arbitrary-precision-arithmetic)
  (also called multiple precision) numbers with uncertainties.  This is useful
  for measurements with very low relative error
* Define arrays of measurements and perform calculations with them.  Some linear
  algebra functions work out-of-the-box
* Propagate uncertainty for any function of real arguments (including functions
  based on
  [C/Fortran calls](http://docs.julialang.org/en/stable/manual/calling-c-and-fortran-code/)),
  using `@uncertain`
  [macro](http://docs.julialang.org/en/stable/manual/metaprogramming/)
* Function to get the derivative and the gradient of an expression with respect
  to one or more independent measurements
* Functions to calculate
  [standard score](https://en.wikipedia.org/wiki/Standard_score) and
  [weighted mean](https://en.wikipedia.org/wiki/Weighted_arithmetic_mean)
* Parse strings to create measurement objects
* Easy way to attach the uncertainty to a number using the `±` sign as infix
  operator.  This syntactic sugar makes the code more readable and visually
  appealing
* Extensible in combination with external packages: you can propagate errors of
  measurements with their physical units, perform numerical integration
  with [`QuadGK.jl`](https://github.com/JuliaMath/QuadGK.jl), numerical and
  automatic differentiation, and much more.
* Integration with [`Plots.jl`](https://github.com/JuliaPlots/Plots.jl).

Further features are expected to come in the future, see the section "How Can I
Help?" and the TODO list below.

The method used to handle functional correlation is described in this paper:

* M. Giordano, 2016, "Uncertainty propagation with functionally correlated
  quantities", [arXiv:1610.08716](http://arxiv.org/abs/1610.08716)
  (Bibcode:
  [`2016arXiv161008716G`](http://adsabs.harvard.edu/abs/2016arXiv161008716G))

If you use use this package for your research, please cite it.

### Documentation ###

The complete manual of `Measurements.jl` is available at
http://measurementsjl.readthedocs.io.  There, people interested in the details
of the package, in order integrate the package in their workflow, can can find a
technical appendix explaining how the package internally works.  You can also
download the PDF version of the manual from
https://media.readthedocs.org/pdf/measurementsjl/latest/measurementsjl.pdf.

Installation
------------

`Measurements.jl` is available for Julia 0.6 and later versions, and can be
installed with
[Julia built-in package manager](http://docs.julialang.org/en/stable/manual/packages/).
In a Julia session run the commands

```julia
julia> Pkg.update()
julia> Pkg.add("Measurements")
```

Older versions are also available for Julia 0.4 and 0.5.

Usage
-----

After installing the package, you can start using it with

```julia
using Measurements
```

The module defines a new `Measurement` data type.  `Measurement` objects can be
created with the two following constructors:

``` julia
measurement(value, uncertainty)
value ± uncertainty
```

where

* `value` is the nominal value of the measurement
* `uncertainty` is its uncertainty, assumed to be a
  [standard deviation](https://en.wikipedia.org/wiki/Standard_deviation).

They are both subtype of `AbstractFloat`.  Some keyboard layouts provide an easy
way to type the `±` sign, if your does not, remember you can insert it in Julia
REPL with `\pm` followed by `TAB` key.  You can provide `value` and
`uncertainty` of any subtype of `Real` that can be converted to `AbstractFloat`.
Thus, `measurement(42, 33//12)` and `pi ± 0.1` are valid.

`measurement(value)` creates a `Measurement` object with zero uncertainty, like
mathematical constants.  See below for further examples.

Every time you use one of the constructors above, you define a *new independent*
measurement.  Instead, when you perform mathematical operations involving
`Measurement` objects you create a quantity that is not independent, but rather
depends on really independent measurements.

Most mathematical operations are instructed, by
[operator overloading](https://en.wikipedia.org/wiki/Operator_overloading), to
accept `Measurement` type, and uncertainty is calculated exactly using analityc
expressions of functions’ derivatives.

In addition, it is possible to create a `Complex` measurement with
`complex(measurement(a, b), measurement(c, d))`.

``` julia
measurement(string)
```

`measurement` function has also a method that enables you to create a
`Measurement` object from a string.

This module extends many methods defined in Julia’s mathematical standard
library, and some methods from widespread third-party packages as well.  This is
the case for most special functions
in [`SpecialFunctions.jl`](https://github.com/JuliaMath/SpecialFunctions.jl)
package, and the `quadgk` integration routine
from [`QuadGK.jl`](https://github.com/JuliaMath/QuadGK.jl) package.  See the
full manual for details.

### Caveat about `±` Sign ###

The `±` infix operator is a convenient symbol to define quantities with
uncertainty, but can lead to unexpected results if used in elaborate expressions
involving many `±`s.  Use parantheses where appropriate to avoid confusion.  See
for example the following cases:

``` julia
julia> 7.5±1.2 + 3.9±0.9 # This is wrong!
11.4 ± 1.2 ± 0.9 ± 0.0

julia> (7.5±1.2) + (3.9±0.9) # This is correct
11.4 ± 1.5
```

Examples
--------

``` julia
julia> using Measurements

julia> a = measurement(4.5, 0.1)
4.5 ± 0.1

julia> b = 3.8 ± 0.4
3.8 ± 0.4

julia> 2a + b
12.8 ± 0.4472135954999579

julia> a - 1.2b
-0.05999999999999961 ± 0.49030602688525043

julia> l = measurement(0.936, 1e-3);

julia> T = 1.942 ± 4e-3;

julia> P = 4pi^2*l/T^2
9.797993213510699 ± 0.041697817535336676

julia> c = measurement(4)
4.0 ± 0.0

julia> a*c
18.0 ± 0.4

julia> sind(94 ± 1.2)
0.9975640502598242 ± 0.0014609761696991563

julia> x = 5.48 ± 0.67;

julia> y = 9.36 ± 1.02;

julia> log(2x^2 - 3.4y)
3.3406260917568824 ± 0.5344198747546611

julia> atan2(y, x)
1.0411291003154137 ± 0.07141014208254456
```

### Measurements from Strings ###

You can construct `Measurement` objects from strings.  Within parentheses there
is the uncertainty on the last digits.

```julia
julia> measurement("-12.34(56)")
-12.34 ± 0.56

julia> measurement("+1234(56)e-2")
12.34 ± 0.56

julia> measurement("123.4e-1 +- 0.056e1")
12.34 ± 0.56

julia> measurement("(-1.234 ± 0.056)e1")
-12.34 ± 0.56

julia> measurement("1234e-2 +/- 0.56e0")
12.34 ± 0.56

julia> measurement("-1234e-2")
-12.34 ± 0.0
```

### Correlation Between Variables ###

Here you can see examples of how functionally correlated variables are treated
within the package:

``` julia
julia> x = 8.4 ± 0.7

julia> x - x
0.0 ± 0.0

julia> x/x
1.0 ± 0.0

julia> x*x*x - x^3
0.0 ± 0.0

julia> sin(x)/cos(x) - tan(x)
-2.220446049250313e-16 ± 0.0 # They are equal within numerical accuracy
```

### `@uncertain` Macro ###

Macro `@uncertain` can be used to propagate uncertainty in arbitrary real- or
complex-valued functions of any number of real arguments, even in functions not
natively supported by this package.

``` julia
julia> @uncertain zeta(2 ± 0.13)
1.6449340668482273 ± 0.12188127308075564

julia> @uncertain log(9.4 ± 1.3, 58.8 ± 3.7)
1.8182372640255153 ± 0.11568300475873611

julia> log(9.4 ± 1.3, 58.8 ± 3.7)
1.8182372640255153 ± 0.11568300475593848
```

### Complex Measurements ###

Here are a few examples about uncertainty propagation of complex-valued
measurements.

``` julia
julia> u = complex(32.7 ± 1.1, -3.1 ± 0.2)

julia> v = complex(7.6 ± 0.9, 53.2 ± 3.4)

julia> 2u + v
(73.0 ± 2.3769728648009427) + (47.0 ± 3.4234485537247377)im

julia> sqrt(u * v)
(33.004702573592 ± 1.0831254428098636) + (25.997507418428984 ± 1.1082833691607152)im
```

### Arrays of Measurements ###

You can create arrays of `Measurement` objects and perform mathematical
operations on them:

``` julia
julia> A = [1.03 ± 0.14, 2.88 ± 0.35, 5.46 ± 0.97]
3-element Array{Measurements.Measurement{Float64},1}:
 1.03±0.14
 2.88±0.35
 5.46±0.97

julia> log.(A)
3-element Array{Measurements.Measurement{Float64},1}:
 0.0295588±0.135922
   1.05779±0.121528
   1.69745±0.177656

julia> cos.(A) .^ 2 .+ sin.(A) .^ 2
3-element Array{Measurements.Measurement{Float64},1}:
 1.0±0.0
 1.0±0.0
 1.0±0.0

julia> B = measurement.([174.8, 253.7, 626.6], [12.2, 19.4, 38.5])
3-element Array{Measurements.Measurement{Float64},1}:
 174.8±12.2
 253.7±19.4
 626.6±38.5

julia> sum(B)
1055.1 ± 44.80457565918909

julia> mean(B)
351.7 ± 14.93485855306303
```

### Derivative and Gradient ###

The package provides a convenient function, `Measurements.derivative`, that
returns the total derivative and the gradient of an expression with respect to
independent measurements.

``` julia
julia> x = 98.1 ± 12.7
98.1 ± 12.7

julia> y = 105.4 ± 25.6
105.4 ± 25.6

julia> z = 78.3 ± 14.1
78.3 ± 14.1

julia> Measurements.derivative(2x - 4y, x)
2.0

julia> Measurements.derivative(2x - 4y, y)
-4.0

julia> Measurements.derivative.(log1p(x) + y^2 - cos(x/y), [x, y, z])
3-element Array{Float64,1}:
   0.0177005
 210.793
   0.0       # The expression does not depend on z
```

### `stdscore` Function ###

You can get the distance in number of standard deviations between a real
measurement and its expected value (not a `Measurement`) using `stdscore`:

``` julia
julia> stdscore(1.3 ± 0.12, 1)
2.5000000000000004
```

You can also test the consistency of two real measurements by measuring the
standard score of their difference and zero.  This is what `stdscore` does if
both arguments are `Measurement` objects:

```julia
julia> stdscore((4.7 ± 0.58) - (5 ± 0.01), 0)
-0.5171645175253433

julia> stdscore(4.7 ± 0.58, 5 ± 0.01)
-0.5171645175253433
```

### `weightedmean` Function ###

Calculate the weighted and arithmetic means of your set of measurements with
`weightedmean` and `mean` respectively:

``` julia
julia> weightedmean((3.1±0.32, 3.2±0.38, 3.5±0.61, 3.8±0.25))
3.4665384454054498 ± 0.16812474090663868

julia> mean((3.1±0.32, 3.2±0.38, 3.5±0.61, 3.8±0.25))
3.4000000000000004 ± 0.2063673908348894
```

### Use with ``SIUnits.jl`` and ``Unitful.jl`` ###

Used together with third-party packages, ``Measurements.jl`` enables you to
perform calculations involving numbers with both uncertainty and physical unit.
For example, you can use [`SIUnits.jl`](https://github.com/Keno/SIUnits.jl) or
[`Unitful.jl`](https://github.com/ajkeller34/Unitful.jl).

``` julia
julia> using Measurements, SIUnits, SIUnits.ShortUnits

julia> hypot((3 ± 1)*m, (4 ± 2)*m) # Pythagorean theorem
5.0 ± 1.7088007490635064 m

julia> (50 ± 1)Ω * (13 ± 2.4)*1e-2*A # Ohm's Law
6.5 ± 1.20702112657567 kg m²s⁻³A⁻¹

julia> 2pi*sqrt((5.4 ± 0.3)*m / ((9.81 ± 0.01)*m/s^2)) # Pendulum's  period
4.661677707464357 ± 0.1295128435999655 s

julia> using Measurements, Unitful

julia> hypot((3 ± 1)*u"m", (4 ± 2)*u"m") # Pythagorean theorem
5.0 ± 1.7088007490635064 m

julia> (50 ± 1)*u"Ω" * (13 ± 2.4)*1e-2*u"A" # Ohm's Law
6.5 ± 1.20702112657567 A Ω

julia> 2pi*sqrt((5.4 ± 0.3)*u"m" / ((9.81 ± 0.01)*u"m/s^2")) # Pendulum's period
4.661677707464357 ± 0.12951284359996548 s
```

Development
-----------

The package is developed at https://github.com/JuliaPhysics/Measurements.jl.  There
you can submit bug reports, make suggestions, and propose pull requests.

### How Can I Help? ###

Have a look at the TODO list below and the bug list at
https://github.com/JuliaPhysics/Measurements.jl/issues, pick-up a task, write great
code to accomplish it and send a pull request.  In addition, you can instruct
more mathematical functions to accept `Measurement` type arguments.  Please,
read the
[technical appendix](http://measurementsjl.readthedocs.io/en/latest/appendix.html)
of the complete documentation in order to understand the design of this package.
Bug reports and wishlists are welcome as well.

### TODO ###

* Add pretty printing: optionally print only the relevant significant digits
  ([issue #5](https://github.com/JuliaPhysics/Measurements.jl/issues/5))
* Other suggestions welcome `:-)`

### History ###

The ChangeLog of the package is available in
[NEWS.md](https://github.com/JuliaPhysics/Measurements.jl/blob/master/NEWS.md) file
in top directory.  There have been some breaking changes from time to time,
beware of them when upgrading the package.

License
-------

The `Measurements.jl` package is licensed under the MIT "Expat" License.  The
original author is Mosè Giordano.

Please, cite the paper Giordano 2016 (http://arxiv.org/abs/1610.08716) if you
employ this package in your research work.


[docs-latest-img]: https://img.shields.io/badge/docs-latest-blue.svg
[docs-latest-url]: http://measurementsjl.readthedocs.io/en/latest/

[docs-stable-img]: https://img.shields.io/badge/docs-stable-blue.svg
[docs-stable-url]: http://measurementsjl.readthedocs.io/en/stable/

[pkgeval-link]: http://pkg.julialang.org/?pkg=Measurements

[pkg-0.5-img]: http://pkg.julialang.org/badges/Measurements_0.5.svg
[pkg-0.5-url]: http://pkg.julialang.org/detail/Measurements.html
[pkg-0.6-img]: http://pkg.julialang.org/badges/Measurements_0.6.svg
[pkg-0.6-url]: http://pkg.julialang.org/detail/Measurements.html

[travis-img]: https://travis-ci.org/JuliaPhysics/Measurements.jl.svg?branch=master
[travis-url]: https://travis-ci.org/JuliaPhysics/Measurements.jl

[appvey-img]: https://ci.appveyor.com/api/projects/status/u8mg5dlhyb1vjcpe?svg=true
[appvey-url]: https://ci.appveyor.com/project/giordano/measurements-jl

[coveral-img]: https://coveralls.io/repos/github/JuliaPhysics/Measurements.jl/badge.svg?branch=master
[coveral-url]: https://coveralls.io/github/JuliaPhysics/Measurements.jl?branch=master

[codecov-img]: https://codecov.io/gh/JuliaPhysics/Measurements.jl/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/JuliaPhysics/Measurements.jl
