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