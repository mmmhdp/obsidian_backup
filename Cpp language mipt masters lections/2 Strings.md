String manipulation as a motivating example for generic programming.
# Hello, world!
The `C++` language is currently undergoing some active changes.
```cpp
import std;

int main () {
	std::println("{0}, {1}!", "Hello", "world");
}
```

Officially, this program has been valid since `2023`.
In reality, it barely started working in `2025` with some effort.
We will be learning `C++23`, occasionally touching on `C++26`.
# Hello, simplified world!
We can use print the old way through a header.
```cpp
#include <print>

int main () {
	std::println("{0}, {1}!", "Hello", "world");
}
```
Or we can use a very classic `C++98`:
```cpp
#include <iostream>

int main () {
	std::cout << "Hello, world!" << std::endl;
}
```
# Hello, really old-school world!
If we don't have `std::print`, we can emulate it via `std::format`:
```cpp
#include <format>
int main () {
	std::cout << std::format("{}, {}!\n", "Hello", "world");
}
```
And of course, we also have access to the old APIs inherited from `C` language:
```cpp
#include <cstdio>
int main () {
	std::printf("%s\n", "Hello, world!");
}
```
# What is a string?
```cpp
foo("Hello, world!");
```
- How to encode the literal "Hello, world!" to use it in a program?
- Let's say, we agreed on a length-prefixed string.
- What could the argument type for `foo` look like? 
![[Pasted image 20260521210225.png]]
# What is a string? (ans)
```cpp
foo("Hello, world!"); // -> foo(const char *);
```
It' much easier to think about strings if they are null-terminated.
it this case, it's just a pointer to the first element.
```cpp
auto t = "Hello world!"; // -> const char *
std::cout << sizeof("Hello, world!") << std::endl; // -> 14
std::cout << sizeof(t) << std::endl; // -> 8
```
![[Pasted image 20260521211242.png]]
# Working with `C-strings`: `<cstring>` header
```cpp
strlen
strcpy, strcat
strcmp
strchr, strstr
strspn, strcspn
strtok
strpbrk
strerror

#include <cstring>
#include <cassert>

char astr[] = "hello";
char bstr[15];

int alen = std::strlen(astr);
assert(alen == 5);
std::strcpy(bstr, astr);
std::strcat(bstr, ", world!");
int res = std::strcmp(astr, bstr);
assert(res < 0);
```
# Discussion
When passing a pointer to incorrect data to a function like `strcpy`, we will get in return all the data up to the nearest null character.

A solution in the `C-style`:function with a character limit.
```cpp
char * strncpy(char *dst, const char *src, size_t n);
char * strncat(char *dst, const char *src, size_t n);
int strncmp(const char *s1, const char *s2, size_t n);
```
Does this option work?
Old joke - "Server, can u say `Hat`, with `500` symbols."

The real reason for the problems it that for `C` strings, the length is not an `invariant`.
# The idea: Let's Write a String Class
Again, the real reason for the problems it that for `C` strings, the length is not an `invariant`.

To preserve the invariants of the object such as strings, private state that is inaccessible for modification is needed, i.e., `encapsulation` is necessary.

This naturally leads to the idea: write a string class.
# A creative challenge
Draw me a bicycle (and let's see if you can reinvent the wheel).

Here are just a few of the many 'reinvented wheels' for strings that are actively used today:
- CString (MFC / ATL)
- QString (QT framework)
- CComBSTR (ATL)
- FBString (Facebook Folly)
- GString (GTK Glib)
- EASTL::string (EASTL)

Since you're unlikely to do any better, let's first take a look at how `std::string` built.
# How `std::string` is structured in principle
![[Pasted image 20260524104526.png]]
This picture lies in one very significant detail.
The detail will be taught at the end of the class.
But it is good as a fundamental diagram.

The strangest thing in this picture seems to be the trailing null.
Why is it needed, if we **already** store the size?
# String as legacy string
The `c_str()` method converts the string to const char pointer.
The `data()` method converts the string to char pointer.
```cpp
std::string s = "Hello, world!";
std::cout << s.c_str() << std::endl;
std::cout << s.data() << std::endl;
```
By passing anything to a legacy API, you are somewhat compromising security.
# `std::string`: A Net Positive
Automatic memory management:
```cpp
std::string s = "Hello, world!"; 
// memory allocated and owned
```
Size is a string invariant.
A rich set of built-in methods and even its own literals.
```cpp
auto sz = "Hello"s.find_first_of("klms"s); // sz equals 2
```
Compatibility with legacy `C` API.

Have I sold you on standard strings?
Any clouds on the horizon?
# Static stings
What do you think about using constant static strings? 
```cpp
static const std::string s = "Foo";
// ....
int foo(const std::string &arg);
// ....
foo(s);
```
The idea looks bad: we are adding heap indirection.
`"Foo"` is a literal. When the program is loaded, it will be copied on the heap.
# Replacement with a pointer
What do you think about replacing a static string with a pointer? 
```cpp
static const char* s = "Foo";
// ....
int foo(const std::string &arg);
// ....
foo(s);
```
But it got even worse! 
Now we hit the creation of a temporary string object on every call to the function foo.
# Solution: `string_view` (`C++17`)
`std::string_view` is a non-owning pointer to a string:
```cpp
static constinit const std::string_view s= "Foo";
// ....
int foo(const std::string_view arg);
// ....
foo(s);
```
There is neither heap indirection nor temporary object creation here.
# How `std::string_view` is structured `**in principle**`
![[Pasted image 20260524111953.png]]
Same lies as before about representation here. 
# Let's give `std::string_view` a try
```cpp
static constinit const std::string_view s = "  FOO  ";

std::string_view trim(std::string_view s){
	auto start = s.find_first_not_of(" \t\n\r\f\v");
	if (start == std::string_view::npos){
		return "";
	}
	auto end = s.find_last_not_of(" \t\n\r\f\v");
	return s.substr(start, end - start + 1);
}

std::println("[{}]", trim(s));
```
# Value-semantics
What would you say about the following uses of string and string view?
```cpp
const std::string& s1 = "hello world"; // 1 - ok
const std::string& s2 = std::string("hello world"); // 2- ok
std::string_view sv1 = "hello world"; // 3 - ok
std::string_view sv2 = std::string("hello world"); // 4- dangle pointer...
```
The main problem with this kind of classes: they pretend to be values, but in reality they are not. 
# One more dead parrot example
```cpp
auto identity(std::string_view sv) { return sv; } 

std::string s = "Hello";
auto sv1 = identity(s); // 1 - ok
auto sv2 = identity(s + " world"); // 2 - dangling pointer
```
Correct use of reference types: only as temporary values.
They should not be stored.
# A Rule of Thumb
Entities with reference semantics should be used in two cases:
1. If function parameters:
```cpp
std::string identity(std::string_view sv) { return sv; }   
```
2. in for-loop initializers:
```cpp
std::vector<std::string> elements;

// ....

for (std::string_view elt : elements){
	do_smth(elt);
}
```
# Do see any other problems?
```cpp
int alen = std::strlen(astr);
assert(alen == 5);
std::strcpy(bstr, astr);
std::strcat(bstr, ", world!");
int res = std::strcmp(astr, bstr);
assert(res < 0);

// or in string terms
std::string astr = "hello";
std::string bstr;

bstr.reserve(15);

int alen = astr.length();
assert(alen == 5);

bstr = astr;
bstr += ", world!";
int res = astr.compare(bstr);
assert(res < 0);
```
# Memory allocations
Be aware on that lines:
```cpp
std::string astr = "hello";
bstr.reserve(15);
bstr += ", world!";
```
Here we could have a possible memory allocations.
# Forming strings
`Direct concatenation`:
```cpp
std::string result, proto = ssl ? "https" : "http";
result = proto + "://" + path + "/" + query;
```
`Input-output streams`:
```cpp
std::stringstream ss;
ss << proto << "://" << path << "/" << query;
result = ss.str();
```
`Formatting`:
```cpp
result = std::format("{}://{}/{}", proto, path, query);
```
`fmt` and `direct` are much faster than `streams` approach.
But why direct is so fast - maybe some of compilations here were able to proceed during compilation time somehow?)
# Not All Benchmarks Are Created Equal
Before:
```cpp
for (auto _ : state){
	std::stringstream ss;
	ss << (ssl ? "https" : "http") << "://"
		<< path << "/" << query;
	auto s = ss.c_str();
	benchmark::DoNotOptimize(s);
}
```
After:
```cpp
std::stringstream ss;
for (auto _ : state)
{
	if (ss.rdbuf())
	{
		ss.rdbuf()->pubseekpos(0);
	}
	ss << (ssl ? "https" : "http") << "://"
		<< path << "/" << query;
	auto s = ss.c_str();
	benchmark::DoNotOptimize(s);
}
```
And now we get real picture in term of time of execution:
`direct` < `stream` < `fmt`

So, note, that every time a word about benchmark of things arises, be sure, that you've spent time on the understanding of the process behind bench, not only on output chars and results.
# Structure of `<format>`
Starting from `C++20`, `std::format` is defined as:
```cpp
template <typename... Args>
std::string format(std::format_string<Args...> fmt,
				   Args&& ... args
);
```
Here `std::format_string` is a wrapper on top of `std::string_view`:
```cpp
std::string vformat(std::string_view sfmt,
					std::format_args fargs);
```
It is possible to rewrite `format` as `vformat`:
```cpp
std::vformat(fmt.get(), std::make_format_args(args,,,));
```
# Discussion
Using `std::print` and `std::format` instead of `I/O streams` is still not an obvious solution.

At a very least, you again have to parse the format string:
```cpp
std::println("{} {:7} {}", j, i, j);
std::println("{} {:<7} {}", j, i, j);
std::println("{} {:_>7} {}", j, i, j);
std::println("{} {:_^7} {}", j, i, j);
```

On the other hand, it works for `constexpr` context.
# A bit more about performance
Very often, dozens of copies of the same string live simultaneously in a program.
```cpp
void foo(string s);

std::string s1 = "Hello";

foo(s1);

std::string s2 = s1;

foo(s2);
```
![[Pasted image 20260524170338.png]]
# Copy On Write (идиома `COW`)
What if we try to count references in a string?
```cpp
class stringbuf {
	char *data;
	size_t size;
	size_t capacity;
	int refcount;
	// .... etc
};

class string {
	stringbuf *buf;
	// .... etc
};
```
And thus we get:
```cpp
string s1 = "Hello";
string s2 = s1;
```
![[Pasted image 20260524170632.png]]
![[Pasted image 20260524170802.png]]
And then if we change `s2`:
```cpp
s2[1] = 'a';
```
We get:
![[Pasted image 20260524170909.png]]
# GCC string (version < 5), `libstdc++`
![[Pasted image 20260524171006.png]]
The reference count is stored as -1, so it is zero in the picture.
COW is actively used.
# Discussion
From the very beginning, the COW idiom had its supporters and opponents.

Which side are you on?
# Advantages and disadvantages
Positives:
- Memory saving
- Cheap coping (just incrementing the reference count)
- Fewer allocations and deletions in the heap, thus performance gain
Negatives:
- Extra level of indirection
- The copy operation virally spreads into all modifying operations
- Thread safety issues (Multithread COW disease)
  
However, there is a consideration that breaks the balance.
This is **pointer invalidation**.
# Pointer invalidation
Operations on a string can invalidate pointers into the string.
For example:
```cpp
std::string a = "Hello";
const char* p = &a[3];
a += "world"; // beyond this point, p cannot be used
```

Here, there is no problem.
The problem is that with COW, pointer invalidated by seemingly harmless operations.

For example:
```cpp
std::string s("str");
const char* p = s.data();

{
	std::string s2(s);
	// here s2 and s have same buffer
	s[0] = 'S';
	// here s allocates new buffer
}

// but here s2 is destroyed, so string with
// 's' is actually not existing by now

// SO HERE p IS ACTUALLY DANGLING!

std::cout << *p << '\n';
```
For non-COW strings, p is still valid, but for COW in 
may no longer be valid in that scenario.

In `2011`, it was officially forbidden to invalidate pointers when executing `operator[]`(C++11, {string.require}).

This excludes COW implementation of `std::string`.

Desired outcome: COW is (almost) dead.

In reality, removal of COW strings from the standard without introducing a worthy replacement spawned a bunch of bicycle (e.g. QString).
# COW is (almost) dead
![[Pasted image 20260524173448.png]]

But does this mean that we can't do anything at the class design level?
# Discussion: let's return to the picture
![[Pasted image 20260524173553.png]]
And here we are, part about significant missing detail, which we start from.

So, the problem is real size of things in memory.
Real memory layout looks approximately like that:
![[Pasted image 20260524173803.png]]
Same picture, just correct scale.

We may easily fit small data in the control block.
# Small string optimization (SSO)
We don't need allocations for small strings.

Below is the worst way to implement `SSO`:
```cpp
class string {
	size_type size_;
	union {
		struct {
			char* data_;
			size_type capacity_;
		} large_;
		char small_[sizeof(large_)];
	}
	
	// and so on
};
```

What problems with this approach?
# Yes, there are some problems
Main one - move constructor. With such approach it's not clear at how to write one.

Coping complexity also increasing a lot.

Time of every execution increasing also because of time on making a choose on every w/r with size check:
```cpp
this->small_[i];
this->large_.data[i];
```

The last one is much complex. How to avoid that kind of problems? What can we do about it?
# GCC string(version >= 5), `libstdc++`
![[Pasted image 20260524174815.png]]
We traded off SSO buffer size for faster string operations.

# Problem: now let's consider `UTF32`
In the case where one character does not occupy one byte (but, for example, four), SSO has problems.

But first of all, we have problems.
How to generalize the developed string for characters of different sizes?

First idea: let's write three separate classes for `utf8string, utf16string, utf32string`.

I'd like to get feedback on this idea.
And of course, we want to use templates as a solution here.
# String class template
How basic_string is structured in **principle**.
```cpp
template <typename CharT>
class basic_string {
	CharT *data;
	size_t size;
	
	union {
		size_t capacity;
		enum {
			SZ =
			(sizeof(data) + 2 * sizeof(size_t) + 31) / 32
		};
		CharT small_str[SZ];
	} sso;
	
public:
	// all 89 methods
};
```
# Using for convenience
```cpp
using string = basic_string<char>;
using u16string = basic_string<u16char_t>;
using u32string = basic_string<u32char_t>;
using wstring = basic_string<wchar_t>;
```
Now we have separate typedefs and single generic class.
# Type traits
There are many questions, the answers to which are different for different strings with different character types.

It is reasonable to combine all this into a class:
```cpp
template <typename CharT> class char_traits;
```

With methods like `assign`,`eq`,`lt`,`move`,`compare`,`find`,`eof` ....
```cpp
template <typename CharT
          typename Traits = std::char_traits<CharT>>
// but this not all params of the class,
// here is one more...
class basic_string {
	....
// all here we are doing via traits
};
```
# Allocators
Abstract away memory allocation (where to go to get some memory).
```cpp
template <typename CharT,
		  typename Traits = std::char_traits<CharT>,
		  typename Allocator = std::allocator<CharT>>
class basic_string {
....
// and this if final state,
// all here we are doing via traits and allocator
}
```
# Strings not only for symbols
The following code is ill-formed:
```cpp
void toggle(std::vector<bool>& bits){
	for (auto &b : bits){
		b = !b;
	}
}
```

What can we do?

Let's use basic string!
```cpp
void toggle(std::basic_string<bool>& bits){
	for (auto &b : bits){
		b = !b;
	}
}
```
We would prefer basic_string_view but it is immutable.
# Span approach
```cpp
void toggle(std::span<bool>& bits){
	for (auto &b : bits){
		b = !b;
	}
}

int main () {
	auto osit = std::ostream_iterator<bool>(std::cout, " ");
	std::basic_string<bool> v = {1, 0, 0, 1, 1};
	toggle(v);
	std::copy(v.begin(), b.end(), osit);
	std::cout << std::endl;
}
```
# Discussion
Are you ready to invent some wheels now?
![[Pasted image 20260524210702.png]]