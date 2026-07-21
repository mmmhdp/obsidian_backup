Name lookup, overload resolution, and semantic processes in `C++`.
# Building blocks
The building block of generic programming is an overload set.
```cpp
auto nth_power(std::integral auto x, unsigned n)
  -> decltype(x);

Matrix nth_power(Matrix x, unsigned n);

```
Overloading implies the need to associate a name with an entity:
```cpp
auto t = nth_power(x, n); // ???
```
# What can it be?
How does the language, during name resolution, determine how a name is bound to an entity?
```cpp
foo(s);
```
For example, how can the line above be interpreted?
It can be anything!
- default constructor
- constructor with one argument
- function call
- functional-style cast
- as you like)
```cpp
struct foo {
	int x = 1;
	foo() = default;
	static int s;
	foo(int x) { s = x; }
}

int foo::s;

TEST (....){
	foo(s); 
	// () here means nothing
	// could be written as foo s; Now it's clear why we 
	// get OK
	EXPECT_EQ(s.x, 1); // OK
}

TEST (....){
	int x = 2;
	delete new foo(s); 
	// here it's constructor with one argument
	EXPECT_EQ(foo::s, 2); // OK
}

int foo(int) { return 3; } 

TEST (....){
	int s = 0;
	foo(s);
	// it's function call now, remember, 
	// struct and fucn with a same name in one file!
	EXPECT_EQ(foo(s), 3); // OK
}

```

# Discussion
We see that name introduction is inherently context-dependent, meaning it belongs to the language's semantics.

It is impossible to determine the binding of a name to an entity based solely on syntax.
# Language syntax
From `C++` standard:
![[Pasted image 20260715073132.png]]
![[Pasted image 20260715072718.png]]
- The grammar production "A:B" means "A expands to B"
- Italic font denotes **non-terminal** symbols
- Regular fond denotes **terminal** symbols
- In the final `C++` program, you do not see non-terminals
- In the `C++` grammar, you always see a single non-terminal on the left side. This means it is **context-free** grammar.
# Simple question
Is this expression syntactically correct?
```cpp
auto *p = new int;
delete delete delete p;
```
# Let's use the syntax blindly
![[Pasted image 20260715073409.png]]
So grammatically this expressions are fine.
# The grammar problem: types
The expression has **semantic** problems:
```cpp
auto *p = new int;
delete delete delete p;

```
And we get an error:
```cpp
error: type `void` argument given to `delete`,
expected pointer 
delete delete delete p;
```
It is syntactically correct. It fails **type checking**.
Type checking is not governed by a context-free grammar.
# The grammar problem: ambiguities
Will this compile?
```cpp
delete []{
	return new int;
}();
```
From syntax, we conclude that such a production exists.
And moreover, it is listed first here:
![[Pasted image 20260715073950.png]]
# Syntactic ambiguities
![[Pasted image 20260715074120.png]]
So in such occasions, standard have to take one of the alternatives, since programming language have to be translated to the machine without any ambiguities. Thus standard committee in such cases take one of the variants as basis and this basis stated in standard itself.   
# Discussion
The language contains **context-dependent** constructs: for example, the meaning of square brackets is determined by what comes before and after them.

So why is a **context-free** grammar used for our language?

The reason is speed of parsing.
In worst case scenario syntactic resolution asymptotic of **context-free** grammar is `O(n^3)`.

But the alternative for **context-dependent** grammar is a `PSPACE`-problem class citizen ([[Hierarchy of complexity classes]]).

So we actually don't have a choice here.
# Local context dependency
Thus scheme for pipeline of code processing goes like: 

![[Pasted image 20260715075717.png]]
# Do we need to teach
Every programmer, when writing or reading code, acts as a kind of abstract `C++` execution machine:
```cpp
int foo() { return 1; } // looks like we forge constexpr here 

int main () {
	using X = int[foo()];
	strcut S { S () {X x;} } s ; 
}
```
Here we will get an error:
```
during RTL pass: expand 
in constructor `main::s::s()`:
internal compiler error: 
err -> using X + int [foo()];
```
# Discussion
Fine, we are agree we have some semantic in the language around name lookup.

What about our main building block and how will we get to names bound to entities?
# Overview of overload resolution rules 
1. The set of `overloaded  names` is selected.
2. The set of `candidates` is selected.
3. From the candidate set, the `viable candidates` for this overload are chosen.
4. The best candidate is selected based on the `chains of implicit conversions` for each parameter.
5. If a best candidate `exists and is unique`, the overload resolution succeeds; otherwise, the program is ill-formed.
# Which entity?
```cpp
void foo(int); // 1 
//-> func with single int arg and no return value

struct foo { foo(int); }; // 2
// struct with constructor which takes int arg
// but it's not defined, just declared

foo(0); // ??? so it's 1
// it's function call, since as I remember, all that looks like a function is a function and all that looks like function call is a function call

foo{0}; // ??? so it's 2
// and here it's call to constructor, since foo is a 
// POD class and have constructor from initializer list
// by default, so we should get error here

// and i was right, lol, but reasoning...

```
# Interaction with parser
![[Pasted image 20260716213512.png]]
```cpp
void foo(int); // 1
struct foo { foo(int); }; // 2
foo(0); // -> 1
foo{0}; // FAIL

```
Here foo is a `discarded function pointer`.
**Overload set doesn't include TYPES**, you know.
So the only candidate for **foo** here is a function pointer from perspective of an analyzer.
# Non-overloadable names
![[Pasted image 20260716213904.png]]
The same name can refer to `different entities`.
Some of that entities may be `overloadable` or not.
```cpp
struct foo{};

template <typename T>
T foo(T x); // OK

template <typename T>
struct foo {}; // FAIL

int foo; // FAIL
```
# Indistinguishable declarations
There are **two broad categories**:
- Those for which `overloading is prohibited at the declaration level`:
  ```cpp
  int b();
  const int b(); // error, overloading prohibited
  
  ```
- Those for which a` different declaration is counted as another declaration`:
  ```cpp
  int f(int);
  int f(const int) // ok, re-declares f(int)
  // It's not overloading at all, const is not a part
  // of name mangling (but remember, not a part of 
  // mangling only external const and external const
  // for arg is not part of mangling)
  
  ```

The logic behind this process relates to the name mangling rules.

More tricky examples:
```cpp
int a();
int a() noexcept; // FAIL 
// prohibited function and same func with noexcept

int b();
const int b(); // FAIL
// int and const int in return type fails,
// since return value type is not a part of name 
// mangling

struct S {
	static void c();
	void c(); // FAIL
	// static void and void with same func name  
	
	int d() &;
	int d(); // FAIL
	// same name with and without ref annotation
}

```
# A beginner-level question
```cpp
struct B {
	int f(int) { return 1; }
};

struct D : B {
	int f(const char*) { return 2;} 
};

D d;

d.f(0); // 1 or 2?
// here answer is 2
// f(int) goes into shadow(of f(const char*))
// in another scope (in scope of class B)
// so actually we have only f(const char*) in
// overload set
```
# A mid-level question
What is minimal change to code to get `1` in previous task?
Answer is, since problems relies in scopes, introduce names from relative scope for resolution too.
# Names can be hidden in scopes
```cpp
struct B {
	int f(int) { return 1; }
};

struct D : B {
	using B::f; // introduce name into space
	int f(const char*) { return 2;} 
};

D d;

d.f(0); // Now answer is 1
```
# Different scopes vary greatly
Types of scopes:
- namespace scope
- template parameters scope
- enum scope
- class scope
- function parameters scope
- block scope
- lambda scope
  
```cpp
namespace A { int i; }

namespace A1 {
	using A::i;
	using A::i; 
	// OK: double declaration in NAMESPACE SCOPE
}

void f() {
	using A::i;
	using A::i; 
	// error: double declaration in FUNCTION SCOPE
}

```
# And something else also affect lookup
```cpp
namespace B { int x = 0; }
namespace C { int x = 0; }

namespace A {
	using namespace B;
	
	void f() {
		using C::x;
		A::x = 1; // ???	
		x = 2; // ???
		// Can this names in theory be different?
	}
}

```
# Names in the language can be qualified?
And answer if yes.

`A::x` is a `qualified name`(**any name with `::` in type is named qualified**). It's `lookup is performed within the qualifying namespace`.

`x` is an `unqualified name`. It's `lookup proceeds from the inside out.

```cpp
namespace B { int x = 0; }
namespace C { int x = 0; }

namespace A {
	using namespace B;
	
	void f() {
		using C::x;
		A::x = 1; // sets B::x
		x = 2; // sets C::x
	}
}

```
# Such different names
List of types of names in `C++` language with naive definitions:
- `identifier`- correct sequence of symbols (digits, underscores, characters and so on). In standard identifier has a lots of synonyms, but same meaning. `Template-name`, or more general all with form `smth-name` as an example:
```cpp
x, x1, MyArr, T, f, N, A, Foo_boo_52
```
- `template-id` - template name with `<>` after it:
```cpp
MyArr<T, 1>, f<N::A>(N::A());
// MyArr - identifier
// MyArr<T, 1> - template-id 
```
- `unqualified-id`- generalization of identifier
```cpp
~Foo, operator "" _y("f")
MyArr // also
MyArr<T, 1> // and this too
```
- `qualified-id` - names with qualification via `::`:
```cpp
::std::cout, Foo::~Foo
```
- `nested-name-specifier`
```cpp
this->f
Myarr<T, 1>::PFoo->~A<char>();
```
- `terminal name`- last name in identifier from the lexical perspective:
```cpp
this->f // f here is a terminal name 
```
- `qualified name`- terminal name in `qualified-id` or in `member qualified name`
```cpp
this->f // f here is a qualified name 
::std::cout // cout here is a qualified name
```
- `unqualified name`- this type of names aren't have `::`,`.` or `->` before them:
```cpp
this->f // this here is a unqualified name 
```
# General rules for name lookup
1. In any scope, single search is preferred; it is a common subroutine. 
2. If the scope is a class or template, base class lookup is performed there.
3. If the name is unqualified, unqualified lookup is conducted.
4. After unqualified lookup, if necessary, it is repeated as argument-dependent lookup (will discuss later).
5. If the name is qualified, qualified lookup is performed through nested scopes.
```bash
1. -> simple search
2. -> search
3. -> unqualified
4. -> ADL 
(but unqualified here is subroutine)
5. -> qualified
(and here unqualified and ADL could be 
called as subroutines on demand)
```
# Single search (simple lookup)
Simple lookup (single search) in `scope S` for a `name N`,
introduced at `point P`, finds all declarations of `N` in that scope:
```cpp
namespace N {
	int X = 1;
}

int main() {
	int N = 2;
	N += N::X; // OK
}

```
![[Pasted image 20260719140334.png]]
Even simple lookup is not so simple, as it restricts different names and entities.
# Search (base lookup)
Base class lookup is performed in classes or class templates.
![[Pasted image 20260719140811.png]]
For a `name N`, as `set S` is constructed, consisting of a set of declarations and a set of sub-objects.

It is built slightly differently depending on whether the name is looked up inside the class or from outside.

Primarily, it is a simple lookup within the class's scope.
# Lookup in base classes
Base class lookup is performed in classes or class templates.

The set of sub-objects combines the (possibly virtual) base classes of the class.
![[Pasted image 20260719141632.png]]
```cpp
struct A { int x; }

struct D : A {};

struct B { double x; };

struct C : public A, public B {};
// S = {invalid, A in C, B in C}
```
Even if result is defined, it's use can be erroneous:
```cpp
struct E : public A, public D {};
// here compiler will give an error,
// since x for A and x in D from it's base A 
// are the same from compilers perspective 
// but he have to chose one of them
```
# Lookup exercise
![[Pasted image 20260719142438.png]]
```cpp
struct A {
	struct T { int x = 1; };
};

struct S { int x = 2; };

template <typename T>
struct B : A {
	int f() {
		T t; return t.x;	
	}
};

B<S> B;

std::cout << B.f() << std::endl; // 1 or 2 ???
// definitely, 1

```
# Semantic processes
The compiler actually evaluates things **top-down**.
![[Pasted image 20260719143326.png]]
A person thinks about the process as starting from the **bottom-up**.
# Special rule for using namespace
During unqualified name lookup, the names appear as if they were declared in the nearest enclosing namespace which contains **both** the using-directive and the nominated namespace `[namespace.udir]`:
```cpp
namespace C { int x = 0; }
namespace B { int x = 0; }

// so B::x actually appear as part of 
// that nearest scope then goes from f()

namespace A {
	using namespace B;
	using C::x;
	void f() {
		x = 2;	 // ok C::x
	}
}
```
What happens if the using namespace is moved higher, or if namespace `B` is nested inside `A`?
# interesting corollary
```cpp
namespace C { int x = 0; }

namespace A {
	namespace B { int x = 0; }
	using C::x;
	void f() {
		x = 2;	 // now error, both x found
	}
}
```
# Qualified lookup
Qualified lookup in a class, enumeration, or namespace performs base class lookup within them.

If nothing is found by qualified lookup for a member-qualified name that is the terminal name of nested-name-specifier and is not dependent, it undergoes unqualified lookup `[basic.lookup.qual.general]`.

Interestingly, qualified lookup is essentially just base class lookup, meaning it ignores `using namespace`.
# Stability of qualified lookup
```cpp
namespace C { int x = 0; }
namespace B { int x = 0; }
namespace A {
	using namespace B;
	using C::x;
	void f() {
		x = 2;	 // ok C::x
	}
}
```

```cpp
namespace C { int x = 0; }

namespace A {
	namespace B { int x = 0; }
	using C::x;
	void f() {
		x = 2;   // also ok C::x
	}
}
```
# What will we do with operators?
Usually, an operator can be located in any namespace:
```cpp
std::cout << "Hello!\n";
```
This could very well be equivalent to the following:
```cpp
operator<< (std::cout, "Hello!\n");
```
For this to work, it must be an operator from the std namespace:
```cpp
std::operator<< (std::cout, "Hello!\n");
```
But the compiler cannot deduce this from the notation `std::a << b`.
# Solution: ADL or Koenig Lookup
Andrew Koenig proposed a solution in the early 90s.

The compiler looks for the function name in the current and all enclosing namespaces.

If it is not found, the compiler looks for the function name in the namespaces of its arguments:
```cpp
namespace N {
	struct A;
	int f(A*);
}

int g(N::A *a) {
	int i = f(a);
	return i;
} 

```
But here the problem, because search was successful:
```cpp
typedef int f;

namespace N {
	struct A;
	int f(A*);
}

int g(N::A *a) {
	int i = f(a); 
	// here unqualified search WAS successful
	// and ADL lookup not even called!
	// here f(a) is just a functional-style cast int(a)
	return i;
} 

```
# Koenig lookup and templates
The following example does not work.
```cpp
namespace N {
	struct A;
	template <typename T> int f(A*);
}

int g(N::A *a){
	int i = f<int>(a); // FAIL!
	return i;
}
```
The reason of failure is that from compiler perspective `f` here is just an unqualified name, not a template-id as we expect it to be.

We may make it work if we introduce `f` as a name of a template function:
```cpp
namespace N {
	struct A;
	template <typename T> int f(A*);
}

template <typename T> void f(int);
// no matter what signature

int g(N::A *a){
	int i = f<int>(a); // now everything is ok
	return i;
}
```
# Hidden friend
Friend functions, defined inside the class are looked up bey ADL only
```cpp
struct X {
	friend bool operator==(X lhs, x rhs){
		return (lhs.data == rhs.data);
	}
};

struct Y {
	operator X() const { return X{}; }
};

X a,b;
Y c,d; 

(a == b); // OK, but 
(c == d); // FAIL
```
# Overview of overload resolution rules
1. The set of `overloaded names` is selected
2. The set of `candidates` is selected
3. From candidates set, the `viable candidates` for this overload are chosen
4. The best candidate is selected based on the `chains of implicit conversions` for each parameter
5. If best candidate `exists and is unique`, the overload resolution succeeds
- Otherwise, the program is ill-formed, no diagnostics required
# Note on candidates selection
Even during the candidate search phase, type deduction may be required to determine if a function template is a candidate:
```cpp
struct S { int x; };
struct U : S { using S::x; };

int bar (S x) { return 52; }
// is a candidate, but
// will fail at the end, no implicit cast from U to S

template <typename T> // NOTE!
int bar(T &&) {       // this part wiil be instantiate
	return 2;
}

int foo() { return bar(U{}); } // here!

// the reason is bar is a possible candidate,
// and compiler have to substitute T to be sure 
// that this candidate is viable

// so processes could overlap here,
// and we will see more of such thing
``` 
# Viability
More then enough to think about `viable` candidate as a candidate with correct number of parameters.

It's not the only thing happening here, on that step, but it's main one:
```cpp
// not viable
foo (int);

// viable
foo(int, ...);

// viable
foo(int, float, int = 0);

// not viable
foo(int, float, int);

int x, y;
foo(x, y);
```
- `[over.match.viable]` if there are **m** arguments it the list, all candidate functions having exactly **m** parameters are viable.
- A candidate function having fewer than **m** parameters is viable only if all parameters following the **mth** have default arguments. 
# The idea of building a chain
From a naive perspective, the conversion chain includes:
1. Highers priority: standard conversions
2. Slightly lower: user-defined conversions
3. Lowest priority: ellipsis (...)

Complexities arise when many different ones are combined:
```cpp
strcut S { S(int){} };

void foo(int); // 1
void foo(S);   // 2
void foo(...); // 3

foo(1); // -> 1
```
# Standard conversions
![[Pasted image 20260721202135.png]]

- Objects transformations (Exact match rank):
```cpp
int arr[10]; int *p = arr; // [conv.array]

```
- Qualifier adjustments (Exact match rank):
```cpp
int x; const int *px = &x; // [conv.qual]

```
- Promotions (Promotion rank):
```cpp
int res = true; // [conv.prom]
```
- Conversions (Conversion rank):
```cpp
float f = 1; // [conv.fpint]
```
# User-defined conversions
Defined by an implicit constructor or a conversion operator:
```cpp
struct A{
	operator int();    // 1
	operator double(); // 2
};

int i = A{}; // calls (1)

```
Here , `(1)` is better then `(2)` because it requires fewer standard conversions.

Chain of user-defined conversions have to contain **ONLY one** user defined conversion. Before and after this user-defined conversion could be placed unbounded amount of implicit conversions. Part after is a tail and this part is crucial, because: **if place it intuitively, a chain with a shorter tail is better**.
# Specifics of ICS (Implicit conversions sequence)
```cpp
struct S { S(long){} };

void foo(S) {}

int x = 42;

foo(x); // int -> long -> S wins

```
and another one:
```cpp
struct T { T(int){} };
struct S { S(T){} };

void foo(S) {}

int x = 42;

foo(x); // int -> T -> S fails

```
# Where name lookup resides
Name lookup occurs after alias substitution but before (other steps of) overload resolution:
```cpp
namespace S {
	using vector = std::vector<int>;
	void foo(vector) {}
}

int main() {
	foo(S::vector{}); // error
	// S::vector{} -> std::vector<int>
	// and ADL will look for foo() in std namespace 
	// and of course will fail
}

```
A typedef-name can also be introduced by an alias-declaration. The identifier following the using keyword is not looked up `[dcl.typedef, 2]`
# The bad reputation of aliases
Name lookup occurs `after` alias substitution but before (other steps of) overload resolution:
```cpp
namespace S {
// can be fixed wiht
	struct vector { std::vector<int> v; };
//  or
	struct vector : public std::vector<int> {} v;
	
	void foo(vector) {}
}

int main() {
	foo(S::vector{}); // OK!
}

```
This gives aliases their bad reputation in the language: 
they disappear too early.

# The bad reputation of unqualified names
```cpp
namespace A {
	struct std { struct Cout{}; static Cout cout; };
	
	void operator << (std::Cout, const char *){
		::std::cout << "World\n";
	}
}

int main() {
	using A::std;
	::std::cout << "Hello, "; // qualified std -> namespace
	std::cout << "Hello"; // unqualified std -> struct
}

// "Hello, World" on the screen

```
# Semantic processes so far
![[Pasted image 20260721211320.png]]
# Normal process
```cpp
template <typename T>
T min(T x, T y){
	return x < y ? x : y;
}

template <typename T>
T min(T x, T y, T z){
	auto t = min(x, y);
	return min(t, z);
}

min(1, 2, 3) -> min (1, 2) -> _|_
			 -> min (1, 3) -> _|_

```
![[Pasted image 20260721211631.png]]
This example about is somewhat called by Constantine like 
`normal semantic process`. In reality, if this is true, code base is really well designed. Otherwise semantic processes in real code be way different.
# Homework assignment
![[Pasted image 20260721211940.png]]
Use standard for this one.
And here should be `static_assert` or not?
![[Pasted image 20260721212255.png]]
