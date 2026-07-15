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
![[Pasted image 20260715075717.png]]
