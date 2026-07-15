Generic function and overloading sets. Basic concepts.
# Raising a number to power
"The first step is to get the algorithm right. The second step is to figure out which sorts of things (types) it works for" - `Alex Stepanov`.

Let's start with first one.
```cpp
unsigned nth_power(unsigned x, unsigned n); // returns x^n
```

How to write the body of this function?
# Getting the algorithm right
`Russian peasant nth_power`:
```cpp
unsigned nth_power(unsigned x, unsigned n)
{
	unsigned acc = 1;
	if ((x < 2) || (n == 1)) 
	{
		return x;
	}
	while (n > 0)
	{
		if ((n & 0x1) == 0x1) {
			acc *= x;
			n--;
		}
		
		x *= x; 
		n /= 2;
	}
	
	return acc;
}
```
# Figuring out possible generalizations
```cpp
template<typename T>
T nth_power(T x, unsigned n)
{
	T acc = 1;
	if ((x < 2) || (n == 1)) 
	{
		return x;
	}
	while (n > 0)
	{
		if ((n & 0x1) == 0x1) {
			acc *= x;
			n--;
		}
		
		x *= x; 
		n /= 2;
	}
	
	return acc;
}
```
# Naive generalization issues
```cpp
template<typename T>
T nth_power(T x, unsigned n)
{
	T acc = 1; 
	// is it really working fine, for maybe matrix?
	// and it's not, so assume we have smth like that 
	// instead
	T acc = id<T>();
	// but in reality, it's to much, to strict
	if ((x < 2) || (n == 1)) 
	{
		return x;
	}
	while (n > 0)
	{
		if ((n & 0x1) == 0x1) {
			acc *= x;
			n--;
		}
		
		x *= x; 
		n /= 2;
	}
	
	return acc;
}
```
# Introducing traits
```cpp
template<
	typename T,
	typename Trait = default_id_trait<T>
>
T nth_power(T x, unsigned n)
{
	// and now we can do this instead
	T acc = Trait::id();
	....	
}
```
But the problem is that in std we don't see such an approach in algorithms at all.

Maybe we should think more.
# Separating the clear part
```cpp
template<typename T>
T nth_power(T x, T acc, unsigned n)
{
	while (n > 0)
	{
		if ((n & 0x1) == 0x1) {
			acc *= x;
			n--;
		} else {
			x *= x; 
			n /= 2;
		}
	}
	
	return acc;
}

unsigned nth_power(unsigned x, unsigned n)
{
	if ((x < 2u) || (n == 1u))
	{
		return x;
	}
	return nth_power<unsigned>(x, 1u, n);
}
```
So all algorithms in std are like that - they don't check arguments and treat misuse of args as UB.
# Building blocks
**The building block of generic programming is an overload set**.
```cpp
template <typename T>
T nth_power(T x, T acc, unsigned n);

unsigned nth_power(unsigned x, unsigned n);
double nth_power(double x, unsigned n);
```
And we will encounter this repeatedly:
classes are designed with sets of overloaded constructors, operators, methods, etc.
# Back to Strings
What if we try to write `operator==` для `basic_string`?
One simple solution:
```cpp
template <typename CharT, typename Traits, typename Alloc>
bool 
operator==(const basic_string<CharT, Traits, Alloc>& lhs,
		   const basic_string<CharT, Traits, Alloc>& rhs)
{
	return lhs.compare(rhs) == 0;
}
```
What's wrong with it?

It is really inefficient.
Think about ("hello" == str), an extra copy is clearly created here.
We would prefer to overload it as a regular function.
# A Better Comparison Approach
The adopted solution (included in `libstdc++`) uses overloads.
```cpp
template <typename CharT, typename Traits, typename Alloc>
bool 
operator==(const basic_string<CharT, Traits, Alloc>& lhs,
		   const basic_string<CharT, Traits, Alloc>& rhs)
{
	return lhs.compare(rhs) == 0;
}

template <typename CharT, typename Traits, typename Alloc>
bool 
operator==(const CharT* lhs,
		   const basic_string<CharT, Traits, Alloc>& rhs)
{
	return rhs.compare(lhs) == 0;
}

template <typename CharT, typename Traits, typename Alloc>
bool 
operator==(const basic_string<CharT, Traits, Alloc>& lhs,
		   const CharT* rhs)
{
	return lhs.compare(rhs) == 0;
}
```
# Discussion
Are there any general principles for designing an overload set?
# Example of Good Design
Different but related types:
```cpp
void Foo(const char *s);
void Foo(std::string s) { Foo(s.c_str());}
```
Different number of parameters:
```cpp
auto s1 = twine("Hello", name).str();
auto s2 = twine("Hello", name, " ", surname).str();
```
Optimization:
```cpp
void vector<T>::push_back(const T&);
void vector<T>::push_back(T&&);
```
# Winter's Rules
1. A person should not be forced to mentally perform overload resolutions.
2. A single comment can describe the entire set.
3. **Each element in the overloading set does roughly the same thing**. (noted as main one)

Below is an example of very poor design:
```cpp
// for an integer argument n, returns the smallest 
// coprime number greater than 1.
// For a string argument (a comma-separated list of coprimes),
// deduces and returns the smallest possible n that 
// could have generated that list
int least_coprime(int n);
int least_coprime(const std::string& x);
```
# Overload Set: Transform
it this good overload set?
```cpp
template <
	class InputIt,
	class OutputIt,
	class UnaryOp>
OutputIt transform(
	InputIt first1, InputIt last1,
	OutputIt d_fisrt, UnaryOp unary_op);

template <
	class ExecutionPolicy,
	class ForwardIt1,
	class ForwardIt2,
	class UnaryOp>
ForwardIt2 transform(
	ExecutionPolicy&& policy,
	ForwardIt1 first1, ForwardIt1 last1,
	ForwardIt2 d_first, UnaryOp unary_op
);
// this normally called map

template <
	class InputIt1,
	class InputIt2,
	class OutputIt,
	class BinaryOp>
OutputIt transform(
	InputIt1 first1, InputIt1 last1,
	InputIt2 first2, OutputIt d_fisrt,
	BinaryOp binary_op);
// this normally called zip
```
So, maybe naming here could be more intuitive.
# Overload Set: String Constructors
Is this good overload set?
```cpp
basic_string();
basic_string(size_type count, CharT ch);

basic_string(const basic_string& other, size_type pos);
basic_string(const basic_string& other,
			 size_type pos,
			 size_type count);
basic_string(basic_string&& other,
			 size_type pos,
			 size_type count);
			 
basic_string(CharT* s, size_type count);
basic_string(CharT* s);

template<class InputIt> 
basic_string(InputIt first, InputIt last);

basic_string(basic_string&& other);
basic_string(const basic_string& other);

basic_string(std::initializer_list<CharT> ilist);

template<class StringViewLike> 
explicit basic_string(const StringViewLike& t);
template<class StringViewLike> 
explicit basic_string(const StringViewLike& t,
					  size_type pos,
					  size_type count);
```
# We must be nice
We must somehow encode the requirements for the interface of generic functions:
```cpp
template<typename T, typename U>
bool check_eq(T lhs, U rhs) {return (lhs == rhs);}
```

The simplest way to document this is with a requires constant:
```cpp
template<typename T, typename U>
requires is_equality_comparable<T, U>::value
bool check_eq(T lhs, U rhs) { return (lhs == rhs);}
```

Now, instead of an error inside the function, we simply exclude it from the overload set.
# Combining Constraints
Constraints are easy to combine:
```cpp
template<typename Iter>
	requires(
	is_forward_iterator<Iter>::value &&
	is_totally_ordered<typename Iter::value_type>::value)
Iter my_min_element(Iter first, Iter last)
{
	....
}
```
Here, both requirements must be met.
Furthermore, different error messages are shown depending on what wrong.
```cpp
note: 'is_forward_iterator::value' evaluated to false
note: 'is_totally_ordered<typename Iter::value_type, void>::value' evaluated to false
```
# Overloading by Constraints
You can overload based on constraints.
```cpp
struct Foo {
	template <typename Int>
		requires std::is_intergral<Int>::value
	Foo (Int x) {
		std::cout << "Creating int-like object\n";
	}
	
	template <typename Float>
		requires std::is_floating_point<Float>::value
	Foo (Float x) {
		std::cout << "Creating float-like object\n";
	}
	
};
```
What would be is both of them failed?
Improved diagnostics!
# Improving Diagnostics
If no overload is suitable, the failed constraints from each are displayed:
```cpp
struct S{};

Foo fs(S{});
```
The error message will be clear and informative:
```cpp
note:  constraints not satisfied
note: 'std::is_intergral::value' evaluated to false
note:  constraints not satisfied
note: 'std::is_floating_point::value' evaluated to false
```
# Complete coverage
We can use explicit constraints to discriminate between functions.
```cpp
template <typename T> requires (sizeof(T) > 4)
void foo(T x) { /* do smth with x */ }

template <typename T> requires (sizeof(T) <= 4)
void foo(T x) { /* do smth else with x */ }
```

The special status of constraints makes them part of overload resolution.

`[over.dcl]` **two function declarations of the same name refer to the same function if they are** in the same scope **and** have equivalent parameter declarations **and** equivalent trailing requires-clauses, **if any**.
# Sometime this goes to far
Expressions inside requires don't even require CT evaluation.

```cpp
consteval bool C() { return true; }

template<typename T> struct A {
	int f requires (C()) {return 1;}
	
	// this is not a redeclaration
	int f requires true {return 2;}
};
```
This means the analysis of requires clauses happens very early.
# Limitation of simple constraints
Alas, simple type traits are not ordered by how restrictive they are.

```cpp
template <typename It>
struct is_input_iterator: std::is_base_of <
	std::input_iterator_tag,
	typename std::iterator_traits<It>::iterator_category>{};
	
template <typename It>
struct is_random_iterator: std::is_base_of <
	std::random_access_iterator_tag,
	typename std::iterator_traits<It>::iterator_category>{};
```
These are just two different templates. And this leads to problems.

In practice, this would cause ambiguity for `std::vector::iterator`:
```cpp
template <typename It>
	requires is_input_iterator<It>::value
int my_distance(It first, It last)
{
	int n = 0;
	while (first != last) { ++first; ++n; }
	return n;
}

template <typename It>
	requires is_random_iterator<It>::value
int my_distance(It first, It last)
{
	return last - first;
}
```
Because from perspective of `std::vector::iterator`, both constraints are equally valid.

So, to overcome this limitations, we need more sophisticated instrument. And name of this one is `requires-expressions`.
Note - before this point we were discussing `requires-clauses`.
# Complex constraints
Let's return to a simple example:
```cpp
template <typename T, typename U> bool
	requires is_equality_comparable<T, U>::value
check_eq(T && lhs, U &&rhs) { return (lhs == rhs);}
```

The same can be written using a **requires-expression**:
```cpp
template <typename T, typename U> bool
	requires requires(T t, U u) { t == u; }	
check_eq(T && lhs, U &&rhs) { return (lhs == rhs);}
```
Yes, the `requires`-`requires` might look confusing.
But recall `noexcept-clause` and `noexcept-expression` (or `throw`-`nothrow`). 
# Even better diagnostics
```cpp
template <typename T, typename U> bool
	requires requires(T t, U u) { t == u; }	
check_eq(T && lhs, U &&rhs) { return (lhs == rhs);}
```
The expression:
```cpp
check_eq(std::string{"1"}, 1);
```
Yields:
```cpp
note: because 't == u' would be invalid
```
This not only states the constraint name, but also the specific ill-formed expression within it.
# The key difference of complex constraints
`Simple constraints` **are evaluated at compile time**:
```cpp
consteval int somepred() {return 14;}

// false here for requires-clause
template <typename T> requires (somepred() == 42)
bool foo(T&& lhs, U&& rhs);
```
`Complex constraints` **check the validity of an expression**, without evaluation:
```cpp
// true here for requires-expression
template <typename T> requires (T t) {somepred<T>() == 42;}
bool foo(T&& lhs, U&& rhs);
```
# Syntax of Complex Constraints
Think of complex constraint as a compile-time function returning true or false:
```cpp
requires (T t, U u) {
	t + u; 
	// true if t + u syntaxically possible [simple]
	
	typename T::inner; 
	// true if T::inner exists [type]
}
```
Think or each requirement inside it as a conjunct.
`Simple` requirements and `type` requirements are basic variants of complex constraints.

There are two more: `compound` and `nested`.
# Concepts: convertible_to
To define constraint system, `C++20` introduced the special keyword `concept`:
```cpp
template <class From, class Too>
concept convertible_to = std::is_convertible_v<From, To> &&
	requires { static_cast<To>(std::declval<From>())};
```
Think of a concept as an abbreviation for a requires-expression.
And, of course, many useful concepts are already in your standard library.
# You can check a concept
Any concept works as a compile-time predicate.
But doesn't need to be **called**. A concept is already a value:
```cpp
struct S {};
static_assert(std::move_constructible<S>); // ok
bool a = std::convertible_to<int, double>; // true
bool b = std::convertible_to<int, S>; // false
```
# Compound requirements
Compound requirements check type compatibility with expressions:
```cpp
requires requires(T x) { {*x}-> typename T::inner; }
```
A compound requirement can use concepts:
```cpp
requires requires(T x) { {*x}-> std::convertible_to<typename T::inner>; } // concept
```
There is also a special `noexcept` syntax:
```cpp
requires requires(T t) {
	{ ++t } noexcept;
}
```
# Nested constraints
Inside the requires-expression it can be repeated.
This is nested requirement:
```cpp
requires(T t){
	requires sizeof(T) == 4; 
	// calculated [nested]
	
	requires somepred<T>() == 42;  
	// consteval predicate [nested]
	
	requires noexcept(++t);
	//
}
```
Task: simplify this nested requires-clause:
```cpp
template <typename T> 
int foo(T t)
requires requires (T t) { requires noexcept(++t); } 
{
	return 42;
}
```
Solution:
```cpp
template <typename T>
int foo(T t)
  requires(noexcept(++t))
{
  return 42;
}
```
# Constraints on concepts
**Recursive concepts are not allowed**:
```cpp
template<bool b, bool... bs>
concept AllTrueRec = b && 
  ((sizeof...(bs) == 0) ? true AllTrueRes<bs...>); // ERROR
```
**Concepts cannot be directly constrained by other predicates**:
```cpp
template <typename T>
concept Inner = requires { typename T::inner; };

template <typename T> requires Inner<T>
concept InOuter = requires { typename T::outer; }; // ERROR

// but can be fixed with
template <typename T> 
concept InOuter =
requires Inner<T> &&
 requires { typename T::outer; }; // OK
```
# Syntax for using concepts
Basic syntax:
```cpp
template <typename T> requires std::integral<T> void foo(T);
```
Template parameter or local variable:
```cpp
template <std::integral T> void bar(T t);
void buzz(std::intergal auto T);
// still function template
std::integral auto x = buz(1);
```
You can restrict a class template as well:
```cpp
template <typename D> requires std::is_class_v<D>
class Foo { /* */};
```
# Funny abbreviations
Constraint for multiple arguments:
```cpp
template <typename T>
requires std::same_as<T, int>
struct S {};
```
Abbreviation syntax:
```cpp
template <std::same_as<int> T>
struct S {};
```
# Concepts on member functions
You can constraint member functions:
```cpp
template <typename D> struct Foo {
	bool empty() requires ranges::forward_range<D>;
};
```
We will see how is it used in ranges library:
```cpp
template <typename T> struct Foo {
	T val;
	bool empty () requires requires { val.empty(); } {
		return val.empty();	
	}
}
```
As shown above you can also use class members.
# Concept on `ctor` or `class`? Yes!
You can constraint only `ctors` without constraining type:
```cpp
template <typename T> requires requires(T x) { x.foo(); }
struct Foo {
	T t;
	Foo() requires std::default_initializable<T> : t() {}
	Foo(Foo &f) requires std::copyable<T> : t(f.t) {}
};
```
You can have both: generic constraint on type and specific constraints on `ctors`.
# Concept partial order
```cpp
template <typename T> concept Ord = 
	requires (T a, T b) {  a < b; };
template <typename T> concept Inc = 
	requires (T a) { ++a; };
template <typename T> concept Int = 
	std::is_same_v<T, int>;

template <typename T>
requires Ord<T> || Inc<T> || Int<T>
int foo(T x) { return 2; }

template <typename T>
	requires Ord<T> 
int foo(T x) { return 22; }

int foo(int x) { return 42; }

double y;

foo(y); // -> ??? answer is 22!
```
# Discussion
How does partial ordering work?
How does the compiler understand that `Ord` is more specialized than `(Ord || Void)`?

The compiler has minimalistic `theorem-prover` for that cause.
# Conjuncts and Disjuncts
Every concept of atomic constraints joined by logical operations (with the usual short-circuiting rules for them):
```cpp
template<typename T>
concept Strange = (sizeof(T) == 4) ||
	requires() {{T::value} -> convertible_to<bool>} &&
	T::value == true);
	
template <typename T> requires Strange<T>
void f(T);

f(1); // ok (lazy rules)
```
# Subsumes 
Subsume - читай как "более ограниченный".

**A constraint P subsumes a constraint Q if and only if:**
**for every disjunctive clause $P_i$ in the disjunctive normal form of $P$, $P_i$ subsumes every conjunctive clause $Q_j$ in the conjunctive normal form of $Q$** `[temp.constr.order]`.
```cpp
template <typename T>
concept Q1 = Q<T> || sizeof(T) == 4; // How do you think?

template <typename T>
concept P = Q<T> && R(T); // P subsumes Q and R
```
# Atomic constraints
**An atomic constraint A subsumes another atomic constraint B if and only if A and B are identical** `[temp.constr.order]`.
```cpp
template <typename T> constexpr bool Atomic = true;
template <typename T> concept C = Atomic<T>;
template <typename T> concept D = Atiomic<T*> && true;
```
Here compiler cannot determine between `D` and `C`, this is ill-formed.

Of course in the ideal world we would prefer this:
$A$ constraint $P$ subsumes a constraint $Q$ if and only if $Q$ implies $P$.
# Identical not similar
```cpp
template<typename T>
concept Foo = (sizeof(T)> 4) && std::is_intergral_v<T>;

template<typename T>
concept Bar = std::is_integral_v<T>;

template <Foo T> int f(T x) { return x + 1; }
template <Bar T> int f(T x) { return x + 1; }

int main() {
	return f(1ull); // FAIL
}

```
I WANT Foo to win, since it looks like Bar is subsumes Foo, but it's not like it at all.

To make this work as expected, we should use more constraints:
```cpp
template<typename T>
concept Int = std::is_integral_v<T>;

template<typename T>
concept Foo = (sizeof(T)> 4) && Int<T>;

template<typename T>
concept Bar = Int<T>;

template <Foo T> int f(T x) { return x + 1; }
template <Bar T> int f(T x) { return x + 1; }

int main() {
	return f(1ull); // FAIL
}
```
We can't subsume types, we can only subsume another concept.
# Golden rules of concept usage
If you want to have partial ordering to work correctly, just wright your own concept for every conjunct you have.

But all of that not that good. 
Here some more problems.
# Subsuming not automatic
```cpp
template<typename T> concept Foo = requries(T x) { x.foo(); };

template<typename T> concept Bar = requries(T x) { x.bar(); };

template<typename T> concept FooBar = requires(T x) {
	x.foo(); x.bar();
}

template <Foo T> int f(T x) {return x.foo(); }
template <FooBar T> int f(T x) {return x.foo() + 1; }

struct SBar {/* have both foo() and bar() */ }

f(SBar{}); // FAIL!
```
Пробелема тут сводится к тому, что данный случай сводится опять к сравению атомарных концептов, а в данном случае выходит, что они не одинаковы и ВСЁ, т.е. никакого порядка между ними автоматически установлено не будет.

Чинится это как и в примере ранее, введением концепта дял связи:
```cpp
template<typename T> concept Foo = requries(T x) { x.foo(); };

template<typename T> concept Bar = requries(T x) { x.bar(); };

template<typename T> concept FooBar = Foo<T> && Bar<T>;

template <Foo T> int f(T x) {return x.foo(); }
template <FooBar T> int f(T x) {return x.foo() + 1; }

struct SBar {/* have both foo() and bar() */ }

f(SBar{}); // OK!
```
# Subsumes
Now we can order constraints on subsuming:
```cpp
template <typename T>
concept InputIterator = Iterator<T> &&
  requires { typename iterator_category_t<T>; } &&
  DerivedFrom<iterator_category_t<T>, input_iterator_tag>;
  
template <typename T>
concept ForwardIterator = InputIterator<I> &&
  Incrementable<T> && Sentinel<I, I> &&
  DerivedFrom<iterator_category_t<T>, forward_iterator_tag>;
```
And so on, down to the random access iterator.
# Now overloading works
```cpp
template <InputIterator Iter>
int my_distance(Iter first, Iter last){
	int n = 0;
	while (first != last) { ++first; ++n; }
	return n;
}

template<RandomAccessIterator Iter>
int my_distance(Iter first, Iter last){
	return last - first;
}

```
Because `InputIterator` is less general (it is a sub-condition of `RandomAccessIterator`), there is no ambiguity here.
# Sutton's counterexample
Lets suppose we have this implementation of copy:
```cpp
template <
	IntputIterator In,
	OutputIterator<value_type_t<In>> Out
>
Out copy(In first, In last, Out out){
	// direct loop
}

template <ContIterator In, ContIterator Out>
	requires MemCopyable<In, Out>
Out copy(In first, In last, Out out){
	// memcpy
}

```
Here subsume relationship are hard and we can run into unexpected issues for some types.

Sutton advises not to rely too heavily on subsumption:
```cpp
template <
	IntputIterator In,
	OutputIterator<value_type_t<In>> Out
>
Out copy(In first, In last, Out out){
	if constexpr(MemCopyable<In, Out>){
		// memcpy
	} else {
		// direct loop
	}
}

```
After all, how often do we introduce new iterator categories?
# The concepts we dreamed of
In early articles about concepts, they were much more interesting (concepts initially was introduced as idea in `2003`, and they we planned to be the part of `C++11`):
```cpp
concept EqulityComparable<typename T> {
  requires constraint Equal<T>; 
  // syntactic
  
  requires axiom Equivalence_relation<Equal<T>, T>;
  // semantic
  
  template <Predicate P> axiom Equality (T x, T y, P p){
	  x == y => p(x) == p(y);
  }
  // if x == y then for any Predicate p, p(x) == p(y) 
  
  axiom Inequality(T x, T y) { (x != y) == !(x == y); }
  // inquality is the neagtion of equality
}

```
