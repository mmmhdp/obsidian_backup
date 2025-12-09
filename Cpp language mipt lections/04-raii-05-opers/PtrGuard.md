# Раз
```cpp
#include <iostream>

template <typename T> class PtrGuard {
  T *ptr_;

public:
  PtrGuard(T *ptr) : ptr_(ptr) {
    std::cout << "ctor" << std::endl;
  }

  T &operator*() { 
    std::cout << "(*) op" << std::endl;
    return *ptr_; 
  }
  const T &operator*() const {
    std::cout << "const (*) op" << std::endl;
    return *ptr_;
  }

  T *operator->() {
    std::cout << "(->) op" << std::endl;
    return ptr_;
  }
  const T *operator->() const {
    std::cout << "const (->) op" << std::endl;
    return ptr_; 
  }

  // DEEP COPY
  PtrGuard(const PtrGuard &rhs) {
    std::cout << "copy ctor" << std::endl;
    ptr_ = new T{*rhs.ptr_};
    /*
     * strong assumption on T that
     * it should have implicit copy
     * constructor a.k.a T should
     * be CopyConstuctible
     * */
  }

  // HERE WE ARE
  // SHALLOW COPY
  PtrGuard(PtrGuard &&rhs) : ptr_(rhs.ptr_) {
    std::cout << "move ctor" << std::endl;
    rhs.ptr_ = nullptr;
  }

  PtrGuard &operator=(const PtrGuard &rhs) {
    std::cout << "(=) op" << std::endl;
    if (this == &rhs) {
      return *this;
    }
    ptr_ = new T{*rhs.ptr_};
    return *this;
  }

  // HERE WE ARE
  PtrGuard &operator=(PtrGuard &&rhs) {
    std::cout << "move (=) op" << std::endl;
    if (this == &rhs) {
      return *this;
    }

#ifdef WECLEAN
    // вариант #1: оставляем пустое состояние
    delete ptr_;
    ptr_ = rhs.ptr_;
    rhs.ptr_ = nullptr;
    return *this;
#else
    // вариант #2: делаем обмен, а удаляет уже пусть деструктор
    // в том объекте класса, который как раз уже собирается
    // отъехать
    std::swap(ptr_, rhs.ptr_);
    return *this;
#endif
  }

  ~PtrGuard() {
    std::cout << "dtor" << std::endl;
    delete ptr_;
  }
};

struct Foo {
  int x;
  int y;
};

int main() {
  int *p = new int(42);
  const PtrGuard<int> pg {p};

  PtrGuard<int> ag = std::move(pg);

  std::cout << *ag << std::endl;

  PtrGuard<int> &rag = ag;
  PtrGuard<int> bg = nullptr;
  bg = ag;

  std::cout << *bg << std::endl;
}
```
# Два
```cpp
#include <iostream>

template <typename T> class PtrGuard {
  T *ptr_;

public:
  PtrGuard(T *ptr) : ptr_(ptr) {
    std::cout << "ctor" << std::endl;
  }

  T &operator*() { 
    std::cout << "(*) op" << std::endl;
    return *ptr_; 
  }
  const T &operator*() const {
    std::cout << "const (*) op" << std::endl;
    return *ptr_;
  }

  T *operator->() {
    std::cout << "(->) op" << std::endl;
    return ptr_;
  }
  const T *operator->() const {
    std::cout << "const (->) op" << std::endl;
    return ptr_; 
  }

  // DEEP COPY
  PtrGuard(const PtrGuard &rhs) {
    std::cout << "copy ctor" << std::endl;
    ptr_ = new T{*rhs.ptr_};
    /*
     * strong assumption on T that
     * it should have implicit copy
     * constructor a.k.a T should
     * be CopyConstuctible
     * */
  }

  // HERE WE ARE
  // SHALLOW COPY
  PtrGuard(PtrGuard &&rhs) : ptr_(rhs.ptr_) {
    std::cout << "move ctor" << std::endl;
    rhs.ptr_ = nullptr;
  }

  PtrGuard &operator=(const PtrGuard &rhs) {
    std::cout << "(=) op" << std::endl;
    if (this == &rhs) {
      return *this;
    }
    ptr_ = new T{*rhs.ptr_};
    return *this;
  }

  // HERE WE ARE
  PtrGuard &operator=(PtrGuard &&rhs) {
    std::cout << "move (=) op" << std::endl;
    if (this == &rhs) {
      return *this;
    }

#ifdef WECLEAN
    // вариант #1: оставляем пустое состояние
    delete ptr_;
    ptr_ = rhs.ptr_;
    rhs.ptr_ = nullptr;
    return *this;
#else
    // вариант #2: делаем обмен, а удаляет уже пусть деструктор
    // в том объекте класса, который как раз уже собирается
    // отъехать
    std::swap(ptr_, rhs.ptr_);
    return *this;
#endif
  }

  ~PtrGuard() {
    std::cout << "dtor" << std::endl;
    delete ptr_;
  }
};

struct Foo {
  int x;
  int y;
};

int main() {
  int *p = new int(42);
  const PtrGuard<int> pg {p};

  PtrGuard<int> ag = std::move(pg);

  std::cout << *ag << std::endl;

  PtrGuard<int> &rag = ag;
  PtrGuard<int> bg = nullptr;
  bg = ag;

  std::cout << *bg << std::endl;
}
```