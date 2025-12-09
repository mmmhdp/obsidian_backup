Идиома PImpl (pointer to implementation) предполагает единичное владение реализацией:
```cpp
class Ifacade {
	CImpl *impl_;
public:
	Ifacade () : impl_(new CImpl) {}
	// методы
};
```
Данная идиома полезна тем, что несмотря на то, что методы в классе Ifacade будут опредлены 
в териминах impl_, что добавляет степень косвенности, но при этом даёт в замен возможность 
сохранить размер объекта класса (Ifacade) неизменным, что критически важно для сохранения ABI.

Но дело в том, что мы бы хотели делигировать управление ресурса, указателя на объект класса
с реализацией, в RAII класс, например, unique_ptr.
```cpp
class MyClass; // предварительное объявление

struct MyWrapper {
	MyClass *impl_; // это ок
	MyWrapper () impl_(nullptr) {}
}

struct MySafeWrapper {
	unique_ptr<MyClass> impl_;
	/*
	 увы, но не компилируется, так как unique_ptr
	 требует наличие реализации, для того, чтобы понимать
	 как полноценно управлять данным ресурсом.
	 Деструктор он хочет видеть в момент объявления,
	 проще говоря. 
	*/
	MySafeWrapper() : impl_(nullptr);
} 
```

# Unique_ptr и как он реально выглядит изнутри
Оказывется, что тип - это не единственный шаблонный параметр в unique_ptr,
там же отдельным параметром вынесена стратегия удаление объекта, т.е. deleter
```cpp
template <typename T, typename Deleter = default_delete<T>>
class unique_ptr{
	T *ptr_;
	Deleter del_;
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) :
		ptr_(ptr), del_(del) {}
	~unique_ptr (){
		del_(ptr_);
	}
	// и так далее
};
```
## Дефолтный удалитель (Default deleter)
Разумеется, по дефолту это пустой класс с перегруженным оператором ()
```cpp
template <typename T> struct default_delete {
	void operator () (T *ptr) { delete ptr;}
};
```
А теперь мы можем решить проблему с объявлением выше:
```cpp
class MyClass; // предварительное объявление

struct MyClassDeleter {
	void operator () (MyClass *); // определен где-то ещё
};

struct MySafeWrapper {
	unique_ptr<MyClass, MyClassDeleter> impl_;
	MySafeWrapper() : impl_(nullptr); // теперь ок
} 
```
Повторюсь, что в данном случае проблема была в том, что delete 
проникает в хедер при использовании стандартного удалителя.