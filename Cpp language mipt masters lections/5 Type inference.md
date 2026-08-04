How inference in the `C++` programming language works and how it is related to other semantic processes.

Hot take - type inference in `C++` is shit btw.

# Hindley-Miner style inference
In some programming languages (not `C++`) type inference looks like pure magic:
```haskell
f g [] = []
// function f here takes something called g
// and empty list - [] and returns empty list []

f g (x:xs) = g x : f g xs
// if function f takes something called g
// head of the list is x 
// and rest of the list is xs
// so list if I call it like 
f (*2) [1, 2, 3]
// initially x = 1, xs = [2, 3]

// so f here is a recursive function
// which takes an element and applies
// g to it, and then recursevly calles 
// itself on the rest of the list, so
// after 
x = 1, xs = [2 , 3]
x = 2, xs = [3]
x = 3, xs = []

// so we see the pattern
// second one
f (*2) [1, 2, 3] ->
// second one
[2] : f (*2) [2, 3] ->
// second one
[2, 4] : f(*2) [3] ->
// first one
[2, 4, 6] : f(*2) [] -> [2, 4, 6]

// more often such procedure called map
```
This infers principal type:
```haskell
f :: (A -> B) -> [A] -> [B]
```
How does it work?
# Unification procedure
```haskell
f g [] = []
f g (x:xs) = g x : f g xs
```
First lets assign variables:
```text
typeof(f) = A, typeof(g) = B,
typeof(x) = C, typeof(sx) = D

(lhs :) :: Z -> [Z] -> [Z],
(rhs :) :: Y -> [Y] -> [Y]
```
Now we need to solve the system of equations:
```text
D = [Z], C = Z, B = C -> Y,
A = B -> D -> E, E = [Y]
```
Through a series of substitutions we get the `principle type`:
```text
A = (Z -> Y) -> [Z] -> [Y]
```
# More `C++` style pseudo code
If this were in `C++`, it would look something like this (pseudo code):
```cpp
auto map(auto f, auto ls) {
	if (ls.empty()) {
		return ls;
	}
	
	auto x = head(ls);
	auto xs = tail(ls);
	
	return list{f(x), map(f, xs)};
}
```

From this, we could deduce (pseudo code):
```cpp
template <typename T, typename U, Callable F>
requires requires (F f, T t) { f(t) -> convertible_to<U>; }
list<U> map(F f, list<T> ls);
```
# Discussion
It seems like a dream.
Why not do it this way?
# Transform overload set again
What is principal type here? (mad...):
```cpp
template <class InputIt, class OutputIt, class UnaryOp>
OutputIt transform(InputIt first1, InputIt last1, 
				   OutputIt d_first, UnaryOp unary_op);
				   
template <class ExecutionPolicy,
		  class ForwardIt1,
		  class ForwardIt2,
	      class UnaryOp>
OutputIt transform(
	ExecutionPolicy&& policy,
	ForwardIt1 first1, ForwardIt1 last1, 
	ForwardIt2 d_first, UnaryOp unary_op);
	
template <class InputIt1,
		  class InputIt2,
		  class OutputIt,
		  class BinaryOp>
OutputIt transform(
	InputIt first1, InputIt last1,
	InputIt first2,
	OutputIt d_first, BinaryOp binary_op);

```
# You must choose
A language can have either overloading with a different number of arguments, or Hindley-Milner-style type inference.

This is why Haskell or OCAML do not have function overloading.

Instead, they have pattern matching.
It looks like overloading, but assumes the existence of a principal type:
```haskell
sumFirstN 0 _ = 0
sumFirstN _ [] = 0
sumFirstN 1 (x:_) = x
sumFirstN n (x:xs) = x + sumFirstN (n - 1) xs
```
# One step deduction
Generally, functions and variables in `C++` perform one-step deduction:
```cpp
void foo(std::integral auto x);

foo(5); // -> int

std::integral auto a = 5; // -> int

```
Here, the type is deduced in the immediate context of the call or initialization.
# Ambiguous deduction
Any ambiguity is resolved manually:
```cpp
template <typename T> 
T max(T x, T y) { .... }

auto x = max(1, 2); // -> int
auto x = max(1, 2.0); // -> FAIL

auto x = max<double>(1, 2.0); // -> double

```
In the last case, we essentially perform manual type substitution.

In these lectures, I consider direct substitution as a special case of a very simple deduction.
# Default arguments
Template parameters with default arguments can serve as hints for substitution (function parameters with default arguments are ignored during deduction):
```cpp
template <typename T = double> double foo (T x = 1.5);

auto v0 = foo(2.0); // -> double
auto v0 = foo(1); // -> int 
auto v0 = foo(); // -> double

```

Only the last line will break if the template's default parameter is removed.
# Type deduction after substitution
Type deduction inside a template function creates deduction points where the type can only be resolved after substitutuion:

```cpp
template <typename T>
T max(T x, T y) { .... }

template <typename T>
T min(T x, T y) { .... }

template <typename T>
bool test_minmax(const T& x, const T& y) {
	if (x > y) return test_minmax(y, x);
	return min(x, y) == x && max(x, y) == y;
}

```
Thus, name lookup, deduction, and substitution are engaged alternately.
# Normal process
We already encountered such a case of semantic process:
![[Pasted image 20260804204119.png]]
# Type decaying during deduction
During deduction, references and top-level cv-qualifiers are stripped:
```cpp
const int& a = 1; int b = 2; const int c = 1; int& d = b;

auto x = max(a, b); // -> int
auto y = max(c, d); // -> int

std::intergral auto z = a; // -> int
std::intergral auto w = d; // -> int
```
This is done to avoid generation functionally identical specializations.
# Deduction of elaborated types
if the template type is elaborated, the argument will be elaborated accordingly:
```cpp
template<typename T> T max(const T& x, const T& y);
auto a = max(1, 3); 
// -> int max<int>(const int&, const int&);

```
Elaborated deduction works differently with types - it preserves cv-qualifiers:
```cpp
template<typename T> void foo(T& x);

const int &a = 3;
foo(a); // -> void foo<const int>(const int& x);

const int c = 1;
auto& z = c; // -> const int& !!!

```
# Strange aspects of recursive deduction
This code works fine:
```cpp
auto sum(int i){
	 if (i == 1){
		 return i;
	 } else {
		 return sum(i - 1) + 1;
	 }
}

```
But this one is compilation error:
```cpp
auto sum(int i){
	if (i == 1){
		 return sum(i - 1) + i;
	} else {
		 return i;
	}
}

```
And this one also fails:
```cpp
auto sum(int i){
	return (i == 1) ? i : sum(i - 1) + 1;
}

```

The reason for such behavior explained in a standard: 
Once a non-discarded return statement **has been seen** in a function, however, the return type deduced from that statement **can be used** in the rest of the function, including in other return statements `[dcl.spec.auto.general]`

# Do compilers have eyes?
in the `C++23` standard, the expression "has been seen" appears only three times:
![[Pasted image 20260804210839.png]]

It's meaning is vague in all these cases.

But seems like it's just about real physical presence in the code, linewise.