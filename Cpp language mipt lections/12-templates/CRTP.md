`Curiously Recurring Template Pattern`

Template technique where a class inherits from a template instantiated with the derived class itself.
```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() {
        std::cout << "Derived implementation\n";
    }
};
```
# `Why`?
It enables:
- `**Static polymorphism** (compile-time polymorphism)`
- `Avoids virtual function overhead`
- `Code reuse`
- `Mixins`
- `Static interface enforcement`
