`Empty Base Class Optimization`
Класс без полей, выступая в качестве родительского класса, 
не вносит свой размер в размер наследующегося класса.

Заметье, что класс с хотя бы одним виртуальным методом пустым не является, там же `vtable` под ним тогда сидит.
```cpp
class EmptyClass {};

class AnInt  : public EmptyClass 
{
	int data;
};   // size = sizeof(int)

class AnotherEmpty : public EmptyClass {};  // size = 1
```

Интересно эту идею использует `unique_ptr` [[PImpl#Unique_ptr и как он реально выглядит изнутри]] для сохранения размера, равным обычному указателю:
```cpp
template <typename T, typename Deleter = default_delete<T>>
class unique_ptr : public Deleter {
	T *ptr_;
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) :
		ptr_(ptr), Deleter(del) {}
		
	~unique_ptr (){
		Deleter::operator()(ptr_);
	}
	// и так далее
};
```

Увы, это невозможно реализовать прям так, так как если `Delete` функция, то всё перестаёт работать. А вот как отличить функцию от класса, об этом позже.

# Private inheritance
[[Наследование#Отношение part-of]] исходя из этого, логично что на самом деле мы хотим тут закрыто отнаследоваться от `Deleter`:
```cpp
template <typename T, typename Deleter = default_delete<T>>
class unique_ptr : private Deleter {
	T *ptr_;
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) :
		ptr_(ptr), Deleter(del) {}
		
	~unique_ptr (){
		Deleter::operator()(ptr_);
	}
	// и так далее
};
```
И теперь тут нет опасности приведения к базовому классу:
```cpp
DeleterTy *pD = new unique_ptr<int, DeleterTy>{};
// FAIL now!!! 
```