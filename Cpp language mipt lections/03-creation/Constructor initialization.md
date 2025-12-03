# Правила инициализации
Два правила инициализации конструкторов:
1. Список инициализации выполняется строго в том порядке, в каком поля определены
  в классе (а не в том, в котором они записаны в списке)
  ```cpp
  class A {
	  int x_;
	  int y_;
	  int z_;
  public:
	  A (int x, int y, int z): z_(z), x_(x), y_(y) {}
	  // first will be x_, then y_, then z_
	  // and NOT z_, x_, y_
  };
    ```
2. Инициализация в теле класса незримо входит в список иницилизации
  ```cpp
  class A {
	  int x_;
	  int y_ = 32;
	  int z_;
  public:
	  A (int x, int z): z_(z), x_(x) {}
	  // anyway first will be x_, then y_, then z_
	  // and NOT some weird stuff that could u pretend to 
	  // be possible here
  };
    ```
# Параметры по умолчанию
Дабы не триггерить инициализацию преждевременно, лучше использовать не параметры, инициализированные в теле класса, а параметры по умолчанию для конструктора:
  ```cpp
  class A {
	  int x_;
	  int y_;
	  int z_;
  public:
	  A (int x, int y = 32, int z): z_(z), x_(x), y_(y) {}
	  // anyway first will be x_, then y_, then z_
	  // and NOT some weird stuff that could u pretend to 
	  // be possible here
  };
    ```
# Делегирование конструктора
```cpp
struct Foo{
	int max_ = 0; int min_ = 0;
	Foo(int x, int y): max_((x > y) ? x : y) {}
	Foo(int x, int y): Foo(x, y) {
		min_((x < y) ? x : y);
	}
} 
```

