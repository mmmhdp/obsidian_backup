Empty Base Class Optimization
Класс без полей, выступая в качестве родительского класса, 
не вносит свой размер в размер наследующегося класса.
```cpp
class EmptyClass {};

class AnInt  : public EmptyClass 
{
	int data;
};   // size = sizeof(int)

class AnotherEmpty : public EmptyClass {};  // size = 1
```