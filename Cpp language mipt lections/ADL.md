Argument dependent lookup
Данная техника связана с разрешением имён. 
Суть её в том, что имя функции может быть найдёно не только в текущем `namespace`е, но и в `namespase`ах его аргументов.

Пример:
```cpp
#include <iostream>

namespace Zoo {
	struct Foo {
	  int x = 10;
	};
	
	void boo(Foo) { std::cout << "boo" << std::endl; }
	// since Foo is and argrument, ADL will
	// provide boo in the namespace, where Foo exists
} // namespace Zoo

int main() {
  Zoo::Foo f;

  boo(f); // ADL kicks here 
}

```