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
Through a series of substitutions we get the principle type:
```text
A = (Z -> Y) -> [Z] -> [Y]
```