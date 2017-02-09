# The Art of C++ / Operators

[![Release](https://img.shields.io/github/release/taocpp/operators.svg)](https://github.com/taocpp/operators/releases/latest)
[![License](https://img.shields.io/github/license/taocpp/operators.svg)](#license)
[![TravisCI](https://travis-ci.org/taocpp/operators.svg)](https://travis-ci.org/taocpp/operators)
[![Coverage](https://img.shields.io/coveralls/taocpp/operators.svg)](https://coveralls.io/github/taocpp/operators)
[![Issues](https://img.shields.io/github/issues/taocpp/operators.svg)](https://github.com/taocpp/operators/issues)

The Art of C++ / Operators is a zero-dependency C++11 single-header library that provides highly efficient, move aware operators for arithmetic data types.

### Table of content

[Preface](#preface)<br/>
[Rationale](#rationale)<br/>
[Example](#example)<br/>
[Requirements](#requirements)<br/>
[Installation](#installation)<br/>
[Provided templates](#provided-templates)<br/>
[Commutativity](#commutativity)<br/>
[noexcept](#noexcept)<br/>
[License](#license)

## Preface

The Art of C++ / Operators is the next logical step in the evolution of the
[Boost.Operators](http://www.boost.org/doc/libs/1_53_0/libs/utility/operators.htm) library,
whose maintainer I became in 2002. Since then, C++ has changed significiantly and
with C++11, the time has come for a complete rewrite and to get rid of some
*very* old legacy code and work-arounds.

## Rationale

Overloaded operators for class types typically occur in groups.
If you can write `x + y`, you probably also want to be able to write `x += y`.
If you can write `x < y`, you also want `x > y`, `x >= y`, and `x <= y`.
Moreover, unless your class has really surprising behavior, some of these related operators
can be defined in terms of others (e.g. `x >= y` &equiv; `!(x < y)`).

Replicating these operators for multiple classes is both tedious and error-prone.
The Art of C++ / Operators templates help by generating operators for you based on
other operators you've defined in your class.

The generated operators are overloaded to take full advantage of move-aware types
and are carefully written to allow the compiler to apply important optimizations
to avoid unneccessary temporary objects. All generated operators are automatically
marked `noexcept` when the underlying base operations are themself marked as
`noexcept`.

## Example

```c++
#include <tao/operators.hpp>

class MyInt
  : tao::operators::commutative_addable< MyInt >,
    tao::operators::multipliable< MyInt, double >
{
public:
  // create a new instance of MyInt
  MyInt( const int v ) noexcept;

  // copy and move constructor
  MyInt( const MyInt& v ) noexcept;
  MyInt( MyInt&& v ) noexcept; // optional

  // copy and move assignment
  MyInt& operator=( const MyInt& v ) noexcept;
  MyInt& operator=( MyInt&& v ) noexcept; // optional

  // addition of another MyInt
  MyInt& operator+=( const MyInt& v ) noexcept;
  MyInt& operator+=( MyInt&& v ) noexcept; // optional

  // multiplication by a scalar
  MyInt& operator*=( const double v ) noexcept;
};
```

then the library's templates *generate* the following operators:

```c++
// generated by tao::operators::commutative_addable< MyInt >
MyInt   operator+( const MyInt& lhs, const MyInt& rhs ) noexcept;
MyInt&& operator+( const MyInt& lhs, MyInt&&      rhs ) noexcept;
MyInt&& operator+( MyInt&&      lhs, const MyInt& rhs ) noexcept;
MyInt&& operator+( MyInt&&      lhs, MyInt&&      rhs ) noexcept;

// generated by tao::operators::multipliable< MyInt, double >
MyInt   operator*( const MyInt& lhs, const double& rhs ) noexcept;
MyInt   operator*( const MyInt& lhs, double&&      rhs ) noexcept;
MyInt&& operator*( MyInt&&      lhs, const double& rhs ) noexcept;
MyInt&& operator*( MyInt&&      lhs, double&&      rhs ) noexcept;
```

>Note: The `// optional` in `class MyInt` above marks methods
>that you typically only add when your class benefits from an
>rvalue reference parameter. If there is no benefit for the
>implementation, you can just omit these methods. If you leave
>them out, The Art of C++ / Operators will simply call the corresponding
>non-movable version that takes the parameter by const lvalue
>reference.

## Requirements

Requires C++11 or newer. Tested with:

* GCC 4.7+
* Clang 3.2+
* Visual Studio 2015

Remember to enable C++11, e.g., provide `-std=c++11` or similar options.

>Note: If you use or test the library with other compilers/versions,
>e.g., Visual C++, Intel C++, or any other compiler we'd like to hear from you.

>Note: For compilers that don't support `noexcept`, see chapter [noexcept](#noexcept).

## Installation

The Art of C++ / Operators is a single-header library. There is nothing to build or install,
just copy the header somewhere and include it in your code.

## Provided templates

The following table gives an overview of the available templates.
Note that the "Provides" and "Requires" columns are just a basic overview.
Multiple overloads per provided operator might exist to ensure the most
efficient implementation for each case, exploiting move-semantics when
possible and (unless explicitly disabled) pass-through of temporary
values to avoid creating new temporaries.

Each overload of an operator is marked `noexcept` when the required operation(s)
that are used to implement it are also marked `noexcept`.

<table>

  <tr>
    <th>Template</th><th>Provides</th><th>Requires</th>
  </tr>

  <!-- equality_comparable -->
  <tr valign="top">
    <td>
      <code>equality_comparable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;!=&nbsp;T</code>
    </td><td>
      <code>T&nbsp;==&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>equality_comparable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;!=&nbsp;U</code><br/>
      <code>U&nbsp;==&nbsp;T</code><br/>
      <code>U&nbsp;!=&nbsp;T</code>
    </td><td>
      <code>T&nbsp;==&nbsp;U</code>
    </td>
  </tr>

  <!-- less_than_comparable -->
  <tr valign="top">
    <td>
      <code>less_than_comparable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&gt;&nbsp;T</code><br/>
      <code>T&nbsp;&lt;=&nbsp;T</code><br/>
      <code>T&nbsp;&gt;=&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&lt;&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>less_than_comparable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&lt;=&nbsp;U</code><br/>
      <code>T&nbsp;&gt;=&nbsp;U</code><br/>
      <code>U&nbsp;&lt;&nbsp;T</code><br/>
      <code>U&nbsp;&gt;&nbsp;T</code><br/>
      <code>U&nbsp;&lt;=&nbsp;T</code><br/>
      <code>U&nbsp;&gt;=&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&lt;&nbsp;U</code><br/>
      <code>T&nbsp;&gt;&nbsp;U</code>
    </td>
  </tr>

  <!-- equivalent -->
  <tr valign="top">
    <td>
      <code>equivalent&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;==&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&lt;&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>equivalent&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;==&nbsp;U</code>
    </td><td>
      <code>T&nbsp;&lt;&nbsp;U</code><br/>
      <code>T&nbsp;&gt;&nbsp;U</code>
    </td>
  </tr>

  <!-- partially_ordered -->
  <tr valign="top">
    <td>
      <code>partially_ordered&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&gt;&nbsp;T</code><br/>
      <code>T&nbsp;&lt;=&nbsp;T</code><br/>
      <code>T&nbsp;&gt;=&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&lt;&nbsp;T</code><br/>
      <code>T&nbsp;==&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>partially_ordered&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&lt;=&nbsp;U</code><br/>
      <code>T&nbsp;&gt;=&nbsp;U</code><br/>
      <code>U&nbsp;&lt;&nbsp;T</code><br/>
      <code>U&nbsp;&gt;&nbsp;T</code><br/>
      <code>U&nbsp;&lt;=&nbsp;T</code><br/>
      <code>U&nbsp;&gt;=&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&lt;&nbsp;U</code><br/>
      <code>T&nbsp;&gt;&nbsp;U</code><br/>
      <code>T&nbsp;==&nbsp;U</code>
    </td>
  </tr>

  <!-- addable -->
  <tr valign="top">
    <td>
      <code>commutative_addable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;+&nbsp;T</code>
    </td><td>
      <code>T&nbsp;+=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_addable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;+&nbsp;U</code><br/>
      <code>U&nbsp;+&nbsp;T</code>
    </td><td>
      <code>T&nbsp;+=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>addable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;+&nbsp;T</code>
    </td><td>
      <code>T&nbsp;+=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>addable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;+&nbsp;U</code>
    </td><td>
      <code>T&nbsp;+=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>addable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;+&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;+=&nbsp;T</code>
    </td>
  </tr>

  <!-- subtractable -->
  <tr valign="top">
    <td>
      <code>subtractable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;-&nbsp;T</code>
    </td><td>
      <code>T&nbsp;-=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>subtractable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;-&nbsp;U</code>
    </td><td>
      <code>T&nbsp;-=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>subtractable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;-&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;-=&nbsp;T</code>
    </td>
  </tr>

  <!-- multipliable -->
  <tr valign="top">
    <td>
      <code>commutative_multipliable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;*&nbsp;T</code>
    </td><td>
      <code>T&nbsp;*=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_multipliable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;*&nbsp;U</code><br/>
      <code>U&nbsp;*&nbsp;T</code>
    </td><td>
      <code>T&nbsp;*=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>multipliable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;*&nbsp;T</code>
    </td><td>
      <code>T&nbsp;*=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>multipliable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;*&nbsp;U</code>
    </td><td>
      <code>T&nbsp;*=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>multipliable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;*&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;*=&nbsp;T</code>
    </td>
  </tr>

  <!-- dividable -->
  <tr valign="top">
    <td>
      <code>dividable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;/&nbsp;T</code>
    </td><td>
      <code>T&nbsp;/=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>dividable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;/&nbsp;U</code>
    </td><td>
      <code>T&nbsp;/=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>dividable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;/&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;/=&nbsp;T</code>
    </td>
  </tr>

  <!-- modable -->
  <tr valign="top">
    <td>
      <code>modable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;%&nbsp;T</code>
    </td><td>
      <code>T&nbsp;%=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>modable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;%&nbsp;U</code>
    </td><td>
      <code>T&nbsp;%=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>modable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;%&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;%=&nbsp;T</code>
    </td>
  </tr>

  <!-- andable -->
  <tr valign="top">
    <td>
      <code>commutative_andable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&amp;&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&amp;=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_andable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&amp;&nbsp;U</code><br/>
      <code>U&nbsp;&amp;&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&amp;=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>andable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&amp;&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&amp;=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>andable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&amp;&nbsp;U</code>
    </td><td>
      <code>T&nbsp;&amp;=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>andable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;&amp;&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;&amp;=&nbsp;T</code>
    </td>
  </tr>

  <!-- orable -->
  <tr valign="top">
    <td>
      <code>commutative_orable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;|&nbsp;T</code>
    </td><td>
      <code>T&nbsp;|=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_orable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;|&nbsp;U</code><br/>
      <code>U&nbsp;|&nbsp;T</code>
    </td><td>
      <code>T&nbsp;|=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>orable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;|&nbsp;T</code>
    </td><td>
      <code>T&nbsp;|=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>orable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;|&nbsp;U</code>
    </td><td>
      <code>T&nbsp;|=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>orable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;|&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;|=&nbsp;T</code>
    </td>
  </tr>

  <!-- xorable -->
  <tr valign="top">
    <td>
      <code>commutative_xorable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;^&nbsp;T</code>
    </td><td>
      <code>T&nbsp;^=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_xorable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;^&nbsp;U</code><br/>
      <code>U&nbsp;^&nbsp;T</code>
    </td><td>
      <code>T&nbsp;^=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>xorable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;^&nbsp;T</code>
    </td><td>
      <code>T&nbsp;^=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>xorable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;^&nbsp;U</code>
    </td><td>
      <code>T&nbsp;^=&nbsp;U</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>xorable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>U&nbsp;^&nbsp;T</code>
    </td><td>
      <code>T(&nbsp;U&nbsp;)</code><br/>
      <code>T&nbsp;^=&nbsp;T</code>
    </td>
  </tr>

  <!-- left_shiftable -->
  <tr valign="top">
    <td>
      <code>left_shiftable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&lt;&lt;&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&lt;&lt;=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>left_shiftable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&lt;&lt;&nbsp;U</code>
    </td><td>
      <code>T&nbsp;&lt;&lt;=&nbsp;U</code>
    </td>
  </tr>

  <!-- right_shiftable -->
  <tr valign="top">
    <td>
      <code>right_shiftable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&gt;&gt;&nbsp;T</code>
    </td><td>
      <code>T&nbsp;&gt;&gt;=&nbsp;T</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>right_shiftable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>T&nbsp;&gt;&gt;&nbsp;U</code>
    </td><td>
      <code>T&nbsp;&gt;&gt;=&nbsp;U</code>
    </td>
  </tr>

  <!-- incrementable -->
  <tr valign="top">
    <td>
      <code>incrementable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T++</code>
    </td><td>
      <code>++T</code>
    </td>
  </tr>

  <!-- decrementable -->
  <tr valign="top">
    <td>
      <code>decrementable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>T--</code>
    </td><td>
      <code>--T</code>
    </td>
  </tr>

</table>

The following templates provide common groups of related operations.
For example, since a type which is left shiftable is usually also
right shiftable, the `shiftable` template provides the combined operators
of both.

<table>

  <tr>
    <th>Template</th><th>Provides</th>
  </tr>

  <!-- totally_ordered -->
  <tr valign="top">
    <td>
      <code>totally_ordered&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>equality_comparable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>less_than_comparable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>totally_ordered&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>equality_comparable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>less_than_comparable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- ring -->
  <tr valign="top">
    <td>
      <code>commutative_ring&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>commutative_addable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>subtractable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>commutative_multipliable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>commutative_addable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>subtractable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>subtractable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>commutative_multipliable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>ring&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>commutative_addable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>subtractable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>multipliable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>commutative_addable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>subtractable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>subtractable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>multipliable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- field -->
  <tr valign="top">
    <td>
      <code>field&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>commutative_ring&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>dividable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>field&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>commutative_ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>dividable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>dividable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- ordered_commutative_ring -->
  <tr valign="top">
    <td>
      <code>ordered_commutative_ring&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>commutative_ring&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>totally_ordered&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>ordered_commutative_ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>commutative_ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>totally_ordered&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- ordered_ring -->
  <tr valign="top">
    <td>
      <code>ordered_ring&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>ring&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>totally_ordered&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>ordered_ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>ring&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>totally_ordered&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- ordered_field -->
  <tr valign="top">
    <td>
      <code>ordered_field&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>field&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>totally_ordered&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>ordered_field&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>field&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>totally_ordered&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- bitwise -->
  <tr valign="top">
    <td>
      <code>commutative_bitwise&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>commutative_andable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>commutative_orable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>commutative_xorable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>commutative_bitwise&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>commutative_andable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>commutative_orable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>commutative_xorable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>bitwise&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>andable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>orable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>xorable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>bitwise&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>andable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>orable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>xorable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>bitwise_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>andable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>orable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>xorable_left&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- shiftable -->
  <tr valign="top">
    <td>
      <code>shiftable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>left_shiftable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>right_shiftable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr><tr valign="top">
    <td>
      <code>shiftable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td><td>
      <code>left_shiftable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code><br/>
      <code>right_shiftable&lt;&nbsp;T,&nbsp;U&nbsp;&gt;</code>
    </td>
  </tr>

  <!-- unit_steppable -->
  <tr valign="top">
    <td>
      <code>unit_steppable&lt;&nbsp;T&nbsp;&gt;</code>
    </td><td>
      <code>incrementable&lt;&nbsp;T&nbsp;&gt;</code><br/>
      <code>decrementable&lt;&nbsp;T&nbsp;&gt;</code>
    </td>
  </tr>

</table>

## Commutativity

For some templates, there are both [commutative](https://en.wikipedia.org/wiki/Commutative_property)
and non-commutative versions available. If the class you are writing is
commutative wrt an operation, you should prefer the commutative template,
i.e., the one which has `commutative_` at the beginning.

It will be *more efficient* in some cases because it can avoid to create an
extra temporary for the result and it has *fewer requirements*.

The one-argument version of the commutative template provides the same
operators as the non-commutative one, but you can see from the result type
in which cases creating a temporary (returning `T`) can be avoided
(returning `T&&`).

For the two-argument version, `commutative_{OP}< T, U >` provides the operators
of both `{OP}< T, U >` and `{OP}_left< T, U >`, again the return type indicates
those cases where an extra temporary is avoided.

## noexcept

If your compiler does not support `noexcept`, the following might be a viable
work-around:

```c++
#include <utility> // make sure it's included before the following!
#define noexcept(...)
#include <tao/operators.hpp>
#undef noexcept
```

With this little hack, The Art of C++ / Operators can be used with GCC 4.4+.

## License

The Art of C++ is certified [Open Source](http://www.opensource.org/docs/definition.html) software. It may be used for any purpose, including commercial purposes, at absolutely no cost. It is distributed under the terms of the [MIT license](http://www.opensource.org/licenses/mit-license.html) reproduced here.

> Copyright (c) 2013-2017 Daniel Frey
>
> Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
