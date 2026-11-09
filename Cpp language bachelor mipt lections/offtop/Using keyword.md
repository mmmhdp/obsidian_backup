# Продвинутая альтернатива typedef
Одна из возможностей бороться со сложность cdecl в c++ - использование using для создания алиасов на типы. Лучше чем typedef, так как есть возможность создавать параметризарованные алиасы, что помогает при метапрограммировании.

```cpp
ptr_to_fref = void (*) (int&); 
ptr_to_fref bar(int x, ptr_to_fref func);
```
Или вот 
```cpp
template <typename T> 
using ptr_to_fref = void (*) (T&);
```

# Втаскивает новое имя в пространство имён
```cpp
namespace X {
	int foo(); // so we have to use X::foo() for call
}

using namespace X;
/* now X is included in global namespace :: and in reality its 
 not good idea, since we will do a space back in terms of sence 
 of namespaces if we bring all names from X to ::
*/

/* but this could work fine
 and in some inheritance cases we will bring names to 
 derived class from his ancestor with that mechanics
*/
using namespace vector;

vector<int> v;
v.push_back(foo()); 
```

Также есть ещё один трюк, позволяющий избежать использования статических функций - это 
анонимные пространства имён
```cpp

// HERE ALL IN COMMENTS ARE DONE BY COMPILER

namespace /*GENERIC_NAME_FOR_NAMESPACE */{
	int foo();
}
/*using namespace GENERIC_NAME_FOR_NAMESPACE*/

int bar() {return foo();}
```
