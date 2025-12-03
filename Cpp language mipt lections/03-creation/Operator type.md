Изначальная подводка [[Difference between coping and initialisation#Again direct vs copy initialization]].
Дело в том, что мы бы всё же хотели иметь возможность диктовать условия того, в какой тип наш класс всё же может
быть неявно преобразован. Для этого мы пишем operator type.
```cpp
struct Foo {
  int x_;
  explicit Foo(int x) : x_(x) {}
  operator int() { return x_; }
};

int main(void) {

  Foo f {2};
  int x = f;
}
// Тут тип Foo был неявно преобразован в int
```
Но мы всё ещё можем сделать явное преобразоваине из int
```cpp
struct Foo {
  int x_;
  explicit Foo(int x) : x_(x) {}
  operator int() { return x_; }
};

int main(void) {
  Foo f = static_cast<Foo>(2);
  int x = int(f);
  return 0;
}
```
На преобразовании в другой тип explicit имеет семантику запрета неявного преобразования, 
оставляя возможность только для явного перевода в другой тип.
```cpp
struct Foo {
  int x_;
  explicit Foo(int x) : x_(x) {}
  explicit operator int() { return x_; }
};

int main(void) {

  Foo f {2};
  int x = int(f);
}
// Тут тип Foo был неявно преобразован в int
```
