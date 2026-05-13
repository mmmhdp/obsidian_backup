Language standard, generic programming and program behavior
# The C++ programming language
General purpose, statically typed, statically compiled.

The are two main phases: 
- translation phase
- the execution phase of the translated program

TRANSLATION ---> EXECUTION

The `language standard` determines if the program can be translated.
# The language standard
A programming language is an agreement between programmer and compiler developer.

As such, it's documented in the language standard.

It is a standard, not a specific implementation, that is final and decisive argument on how a program is translated.

The current `C++` standard is `find_if_yourself`, the current word means it, but it's ISO standard.
# Syntax and semantics
The language standard is a list of `diagnosable rules`.

Rules can be `syntactic` (which can be checked by grammar):
```cpp
template<int i = 3<4> struct S; // syntax violation
```
And `semantic`,which are sometimes difficult to check: 
```cpp
int foo(int); // foo shall be defined somewhere

int bar(int x) { return foo(x);}
```

An implementation that satisfies the standard should ideally either translate the program or issue a diagnostic.
# Normative references
From edition to edition, absolute numbers can change.

`C++17` overloading is Chapter 16
`C++20` - it's 12

Therefore, the standard uses symbolic references.
In both these documents, the section is labeled as {over}, 
and subsections like `C++17`, 16.1 or `C++20`, 12.2 are labeled {over.load}

A typical reference looks like {C++17, over} or simply {over} if the standard is not important, it the latest one is meant, or if it is clear from the context.
# `MSB` example in with `__buildin_clz()`
Main theme - much faster realization with build in is much faster, but less portable between compilers.

And the worst part - in `C` language, it is difficult to achieve both at the same time, meaning speed and portability.
# Normal vector of development
From `common idioms` to `special assembly instructions`.
```cpp
find_msb: li a5, 31
     clzw a0, a0 # -march=rv64gc_zbb
     subw a0, a5, a0
     ret
```

From `macros` to `generic programming`.

"Almost every macro demonstrates a flaw in the programming language, in the program, or in the programmer" - `Bjarne Stroustrup`.

From compiler `intrinsics` to `standard library functions`:
```cpp
__buildit_clz -> template <class T> int countl_zero(T x);
```
# Let's try `C++`
We will be learning `C++23`, occasionally touching on `C++26`.
First attempt is occasionally `ill-formed`.
```cpp
template <typename T> int find_msb(T value) 
{
 int total_bits = sizeof(T) * CHAR_BIT;
 int leading_zeros = std::countl_zero(value);
 int msb_position = total_bits - 1 - leading_zeros;
 return msb_position;
}

assert(find_msb(1) == 0);
```
and here we get ERROR:
```cpp
error: no matching function for call to 'countl_zero'
	int leading_zeros = std::countl_zero(value);
```
The program is ill-formed since `countl_zero` have overload only for unsigned types.

# `Well-formed` and `ill-formed`
A program that cannot be translated is called ill-formed:
```cpp
int main () {
	12315859082; // well-formed
}
```
BUT:
```cpp
int main () {
	123934529345802812315859082; // ill-formed [lex.icon]
}
```

Sometime the program is ill-formed, but the compiler cannot diagnose it. In this case, we assign it the status `IFNDR`, relying on the assembler or the linker.
# Make type unsigned
When we generalize something with respect to type, we need operations on types, not just values.
```cpp
template <typename T> int find_msb(T value) 
{
 using U = std::make_unsigned_t<T>;
 int digits = std::numeric_limits<U>::digits;
 int leading_zeros = std::countl_zero(
							static_cast<U>(value));
 int msb_position = total_bits - 1 - leading_zeros;
 return msb_position;
}

assert(find_msb(1u << 31) == find_msb(-1) == 31);
```
# Make auto local variables
But better do not touch non-template parts of signature.
```cpp
template <typename T> int find_msb(T value) 
{
 using U = std::make_unsigned_t<T>;
 auto digits = std::numeric_limits<U>::digits;
 auto leading_zeros = std::countl_zero(
							static_cast<U>(value));
 int msb_position = total_bits - 1 - leading_zeros;
 return msb_position;
}

assert(find_msb(1u << 31) == find_msb(-1) == 31);
```
This way we still can be sure, that zero still returns in -1.
# What about user-defined types?
We would like to make overloads to non-integral types too.
```cpp
struct ModularInt {/*some weird stuff*/};
```

To do this, we must behave politely in the main version:

```cpp
int find_msb(std::integral auto value) 
{
 using U = std::make_unsigned_t<decltype(value)>;
 auto digits = std::numeric_limits<U>::digits;
 auto leading_zeros = std::countl_zero(
							static_cast<U>(value));
 int msb_position = digits - 1 - leading_zeros;
 return msb_position;
}
```
# Problem with unsigned char
```cpp
#include <bit>
#include <concepts>
#include <limits>
#include <type_traits>

int find_msb(std::integral auto value) {
  using U = std::make_unsigned_t<decltype(value)>;
  auto digits = std::numeric_limits<U>::digits;
  auto leading_zeros = std::countl_zero(static_cast<U>(value));
  int msb_position = digits - 1 - leading_zeros;
  return msb_position;
}

// ok
template int find_msb<unsigned>(unsigned);

// slightly worse (see asm)
template int find_msb<unsigned char>(unsigned char);

int main() {}
```
# Problem with unsigned char
```cpp
int find_msb<unsigned char>:
	movzx   edi, dil
	mov     eax, 31
	lzcnt   edx, edi
	sub     eax, edx
	test    edi, edi
	mov     edx, -1
	cmove   eax, edx
	ret   
```
```cpp
int find_msb<unsigned int>:
	mov     eax, 31
	lzcnt   edi, edi
	sub     eax, edi
	ret   
```
The compiler, even for 8-bit integers, uses 32-bit registers and therefore builds additional code to handle zero at the input.
# The grand master sacrifice the exchange
We can say that the function's behavior for input zero is undefined:
```cpp
#include <bit>
#include <concepts>
#include <limits>
#include <type_traits>

int find_msb(std::integral auto value) {
  if (value == 0) {
	  std::unreacheble();
  }
  using U = std::make_unsigned_t<decltype(value)>;

  auto digits = std::numeric_limits<U>::digits;
  auto leading_zeros = std::countl_zero(static_cast<U>(value));
  int msb_position = digits - 1 - leading_zeros;
  return msb_position;
}

// ok
template int find_msb<unsigned>(unsigned);

// slightly worse (see asm)
template int find_msb<unsigned char>(unsigned char);

int main() {}
```
# Assembler obviously improved
From:
```cpp
int find_msb<unsigned char>:
	movzx   edi, dil
	mov     eax, 31
	lzcnt   edx, edi
	sub     eax, edx
	test    edi, edi
	mov     edx, -1
	cmove   eax, edx
	ret   
```
To:
```cpp
int find_msb<unsigned int>:
	movzx   edi, dil
	mov     eax, 31
	lzcnt   edi, edi
	sub     eax, edi
	ret   
```
`Btw`, on bench difference with `3x` better in performance. 

This will allow the compiler to optimize much much better.
But it can cause `the death of your kitten` if the condition if violated.
# Kittens are in danger
We can say that the function's behavior for input zero is undefined and here kittens indeed in danger:
```cpp
#include <bit>
#include <concepts>
#include <limits>
#include <type_traits>

int find_msb(std::integral auto value) {
  if (value == 0) {
	  std::unreacheble();
  }
  using U = std::make_unsigned_t<decltype(value)>;

  auto digits = std::numeric_limits<U>::digits;
  auto leading_zeros = std::countl_zero(static_cast<U>(value));
  int msb_position = digits - 1 - leading_zeros;
  return msb_position;
}

// ok
template int find_msb<unsigned>(unsigned);

// slightly worse (see asm)
template int find_msb<unsigned char>(unsigned char);

int main() {}
```
# Document intent with contract
But, we can `state`, that the function's behavior for input zero is undefined (with `C++26` `contract` feature)
```cpp
#include <bit>
#include <concepts>
#include <limits>
#include <type_traits>

int find_msb(std::integral auto value) pre(value != 0) {
  if (value == 0) {
	  std::unreacheble();
  }
  using U = std::make_unsigned_t<decltype(value)>;

  auto digits = std::numeric_limits<U>::digits;
  auto leading_zeros = std::countl_zero(static_cast<U>(value));
  int msb_position = digits - 1 - leading_zeros;
  return msb_position;
}

// ok
template int find_msb<unsigned>(unsigned);

// slightly worse (see asm)
template int find_msb<unsigned char>(unsigned char);

int main() {}
```
We won't go into detail here, as we're in a hurry to get back to the dead kittens. 
# Abstract machine
The semantic descriptions in this document define a parameterized nondeterministic abstract machine {C++, intro.abstract}.
```cpp
int zero()  // external linkage
{
	int i = 0;
	int j = i * i + i * 3 + 42; // no side effects
	return i;
}
```
The entire language standard document defines the abstract machine.
The compiler does not generate code for it, but it considers it's behavior during optimizations. 
# Volatile qualifier
A special qualifier is needed to describe unpredictable effects in the abstract machine.
```cpp
int zero()  // external linkage
{
	int i = 0;
	volatile int j = i * i + i * 3 + 42; // unknown side effects
	return i;
}
```
`volatile` can be read as "may change unpredictably".
```cpp
const volatile int i = 0;
```
An initializer is required here (which is generally ignored) because const requires it.
# What the translator can do: the as-if rule
{intro.abstract}{...} conforming implementations are required to emulate (only) the observable behavior of the abstract machine.

What constitutes observable behavior?
- Access through volatile glvalues
- Data written into files
- The input and output dynamics of interactive devices

The compiler is allowed to do anything with the program as long as the observable behavior remains the same.