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
# We must be conservative
```cpp
int nonzero(int i) // external linkage
{
	return i * i  + i * 3 + 42; // no side effects, but...
}
```
The compiler does not know whether this expression will be used later to produce a side effect.

The scope within which the compiler can track all connections is called a `translation unit`.

For `C` language, it's a file.
For `C++` language, we will have a lecture about that.
# Discussion
Will be there a side effect?
You will be able to figure out, who is right here, if you open up a standard.
Patch to `clang/gcc`, huh?
```cpp
volatile std::nullptr_t a = nullptr;
int *b;
b = a;

// clang
mov qword ptr [rsp - 8], 0
xor eax, eax

// gcc
mov QWORD PTR [rsp - 8], 0
mov rax, QWORD PTR [rsp - 8]
xor eax, eax
ret
```
# How conservative?
```cpp
int foo (int *a, double *d, int n)
{
|\\or here?
|	for (int i = 1; i < n; ++i)	
|	{
--------d[2] = 1.0; \\ where you prefer to jump as a coml
|		a[i] += i * i;                               piler
|	}
|	
|\\here?
	return d[a[0]];
}
```
# Strict aliasing
```cpp
int foo(int *a, double* d, int n){....
```
{basic.val} if a program attempts to access the stored value of an object through a glvalue whose type is not similar to one of the following types the behavior is undefined:
1. The dynamic type of the object
2. A type that is the singed or unsigned type corresponding to the dynamic type of the object
3. a char, unsigned char, or std::byte type
There is a very dubious option `-fno-strict-aliasing` that blocks the compilers's strict aliasing-based optimizations.
# Details of the abstract machine
The abstract machine is:
- Parameterized:
```cpp
int nbits = std::numeric_limits<unsigned int>::digits;
```
- Non-deterministic:
```cpp
int is_equal = (+"abc" == +"abc");
```
- Not defined everywhere (and this is very interesting).
# Undefined behavior is important
Undefined behavior gives the optimizer free rein.
```cpp
int foo(int *a, double *d, int n);

int arr[0] = {/*some initializer*/}
double *d = (double *)(&arr[0]);

foo(arr, d, 10);
// this function will be optimized
// as if arr and d do not overlap!!!
```
No one will warn you.
For the optimizer, undefined behavior `does not exist`.
# Undefined behavior is dangerous
{intro.abstract} A conforming implementation executing a well-formed program shall produce the same observable behavior {...}.
**However**, if any such execution contains an undefined operation, this document places no requirement on the implementation executing that program with that input(**not even with regard to operations preceding the first undefined operation**).
```cpp
int k, satd = 0, dd, d[16];

/* some code here */

for (dd = d[k = 0]; k < 16; dd = d[++k])// how do you think?
{
	satd += (dd < 0 ? -dd : dd);
} 
```
# The compiler is blind to UB
The compiler always behaves as if UB does not exists:
```cpp
int f() {
	int i; int j = 0;
	for (i = 1; i > 0; i += i){
		++j;
	}
	return j;
}
```
A legitimize optimization of this code is again an infinite loop:
```cpp
f():
.L2: jmp    .L2
```
# Freedom of optimization around UB
"The whole concept of "use undefined C behavior to change code generation" is complete and utter BS. It's wrong. It's stupid. And a compiler shouldn't do it." - `Linus Torvalds`.

```cpp
int ubranch (int n)
{
	int k = 1;
	switch(n) {
		case 0: k = 7; break;	
		case 2: k = 0; break; // this is UB, so...
		case 9: k = 4; break;	
	}
	return n / k;
}

// will be in assebmler in form of:
ubranch (int):
	        li       a1, 9
	        beq      a0, a1, .LBB0_5
	        li       a1, 2
	        beq      a0, a1, .LBB0_4
	        li       a1, 1
	        bnez     a0, .LBB0_6
	        li       a1, 7
	        divw     a0, a0. a1
	        ret
.LBB0_4:    divw     a0, a0, zero
	        ret
.LBB0_5:    li       a1, 4
.LBB0_6:    divw     a0, a0, a1
		    ret 
```
Since case with `n == 2` cause an UB, compiler is really sure about your intent to say with that UB, that somehow you know, that n is never really becomes equal to 2 outside, since `n == 2` cause UB. So compiler uses that assumption about UB to optimize that code as if n CAN NOT EVER EVER BECOME 2, so, as an example, it could silently throw this line away like that:
```cpp
ubranch (int):
	        li       a1, 9
	        beq      a0, a1, .LBB0_2
	        li       a1, 1
	        beqz     a0, .LBB0_3
			j        .LBB0_4
.LBB0_2:    li       a1, 4
	        bnez     a0, .LLB0_4
.LBB0_3:    li       a1, 7
.LBB0_4:    divw     a0, a0, a1
		    ret 
```
# A golden opportunity to score a 10
We will study C++, most industrial compilers are written in C++.

Integrate into any industrial compiler a patch that removes code along paths leading to UB, which was not previously removed.

For example, be the first to exploit the following UB (via Vladislav Belov):
```cpp
void foo(short const * const p);

int test2(short p1)
{
	const short p2 = p1; 
	foo(&p2); // UB if foo changes p2
	
	if (p1 == p2) return 14;
	
	return 42; // this branch to be disregarded 
}
```
# Let me scary you now
Suppose you wrote code that "protects you from UB:
```cpp
int foo(int *a, int base, int off)
{
	if (off > 0 && base > base + off) return 42;
	// you thought you protected yourself from
	// overflow of int, but since it's UB...
	return a[base + off];
}
```
In assembly, you suddenly see a strange thing: the compiler has erased all the checks:
```cpp
foo(int*, int, int):
add    esi, edx
movsxd rax, esi
mov    eax, dword ptr [rdi + 4 * rax]
ret
```
# The compiler removed my code...
It can do it for two reasons:
- Due to the as-if rule, if your code did not affect side effects.
- If your code was on path that also contains UB:
```cpp
int foo(bool c) // foo(true) == 42
{
	int x, y;
	y = c ? x : 42; // read from uninitialize var is UB 
	return y;
}
```
This irritates many people, and in `C++26` a new type behavior was defined.
# Erroneous behavior (`C++26`)
```cpp
char foo()
{
	char a;
	char b [[indeterminate]];
	
	char c = a + 1; // erroneous, not undefined from C++26
	char d = b + 1; // still UB
	
	return c + d;
}
```
Well-defined behavior that the implementation is recommended to diagnose {defns.erroneous}.
# Erroneous value (`C++26`)
```cpp
unsigned char c;
unsinged char d = c;
// no erroneous behavior
// but d has an erroneous value

assert(c == d)
// holds, both integral promotions
// have erroneous behavior
```
When storage for an object with automatic or dynamic storage duration is obtained, the bytes comprising the storage for the object have the following initial value:
- if the object has dynamic storage duration, or {...} marked with the {{indeterminate}} attribute, the bytes have **indeterminate values**.
- otherwise, the bytes have **erroneous values**, where each value is determined by the implementation independently of the state of the program.
# Homework assignment
All tasks are based on C++23.

1. Justify the situation with volatile `nullptr_t` using the standard:
```cpp
volatile std::nullptr_t a = nullptr; int *b; b = a;
```
it't actually possible patch to compiler too btw.
2. Find another elegant example where dereferencing a null pointer is valid, before your classmates do:
```cpp
int *p = nullptr;
std::println("{}", typeid(*p).name()); 
// not even exception
```
3. Find how to write proper protection code for:
```cpp
int foo(int *a, int base, int off) 
{
	return a[base + off];
}
```