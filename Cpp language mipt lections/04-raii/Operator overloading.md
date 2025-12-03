# Dereference operator (\*) and arrow operator (->)
## Drill down
Оператор стрелочки имеет спецсемантику:
- Вызов p->x эквивалентен (p.operator->())->x и так сколько угодно раз
- Стрелочка как бы зарывается в глубину на столько уровней, на сколько сможет
## Implementation
```cpp
/*
 Тут ещё и семантика копирования реализована
 с помощью конструктора копирования и наличия 
 перегрузки оператора =
*/
#include <iostream>

template <typename T>
class PtrGuard {
  T *ptr_;
public:
  PtrGuard (T* ptr) : ptr_(ptr) {}

  T& operator*() {
    return *ptr_;
  }
  const T& operator*() const {
    return *ptr_;
  }

  T* operator->() {
    return ptr_;
  }
  const T* operator->() const {
    return ptr_;
  }

  PtrGuard (const PtrGuard& rhs) {
    ptr_ = new T{*rhs.ptr_};
    /*
     * strong assumption on T that 
     * it should have implicit copy 
     * constructor a.k.a T should 
     * be CopyConstuctible 
     * */
  }
  PtrGuard & operator=(const PtrGuard& rhs) {
    if (this == &rhs){
      return this;
    }
    delete ptr_;

    ptr_ = rhs.ptr_;
    return *this;
  }


  ~PtrGuard () {
    delete ptr_;
  }
};

struct Foo {
  int x;
  int y;
};

int main() {
  int *p = new int(42);
  const PtrGuard<int> pg (p);

  std::cout << *pg << std::endl;
  std::cout << std::endl;

  Foo *fp = new Foo{1,2};
  const PtrGuard<Foo> pgf (fp);

  std::cout << fp->x << std::endl;
  std::cout << fp->y << std::endl;
  std::cout << std::endl;

  PtrGuard<Foo> xg {pgf};
  std::cout << xg->x << std::endl;
  std::cout << xg->y << std::endl;
  std::cout << std::endl;

  PtrGuard<Foo> yg = pgf;
  std::cout << yg->x << std::endl;
  std::cout << yg->y << std::endl;
  std::cout << std::endl;
}
```