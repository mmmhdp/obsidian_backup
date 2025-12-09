# Types of type casting
## c-style
Если немного задуматься, то буквально магия, не иначе
```cpp
int x; 
double y = 1.0;

x = (double) y;

// but same syntax for this too..
const int *p = &x;
int *q = (int *) p;

// and same for that too..
long long uq = (long long) q;
```
## static_cast
Обычные безопасные преобразования

Работает static_cast по принципам семантического значения, 
угадывая верную интерпретация самостоятельно.
Ниже как раз пример такого, ведь по человечески ясно, что double 1.0, оно по смыслу ближе всего 
к int 1, но битовое представление double вообще выглядит иначе и не даёт такой уж очевидной 
картины для вычислительно машины. 
```cpp
int x;
double y = 1.0;

x = static_cast<int> (y);
```

static_cast так силён, так как является тем самым явным преобразованием типа.
От неявных мы ранее защищались с помощью explicit в конструкторах, но static_cast проламывает эту дверь:
```cpp
struct T {};
struct S {
	explicit S(T) {}
};

void foo (S s) {}

foo(T); // FAIL!
foo(static_cast<S> (T)); // Works fine!
```
То же самое касается синтаксиса копирующей инициализации
```cpp
T x;
S y = static_cast<S> (T); // Work too
```

Т.е. вообще говоря, когда у класса есть конструктор от одного аргумента, надо быть готовым, что данный конструктор может 
поучаствовать вот в таком прекрасном static_cast'е
## const_cast
Снятие константности или волатильности
```cpp
const int *p = &x;
int *q = const_cast<int *> (p);
```
## reinterpret_cast
Слабоумие и отвага.
Интерпретирует биты одного типа как другой.
```cpp
long long uq = reinterpret_cast<long long>(q);
```
Но! reinterpret_cast будет всё ещё лучше чем c-style, потому как он не может в то, что
делает static_cast и const_cast, он как бы чётко специализируется на кринжовых кастах, 
как бы давая понять, что тут происходить, ну, нечто занимательное.
### Дебри reinterpret_cast
- Побитовая интерпретация очень коварна
```cpp
float p = 1.0;
int n = *reinterpret_cast<int *>(p); // UB - stict aliasing violation
```
Тут просто проблема в том, что float* и int* - это разные типы, а в стандарте запрещеное иметь на 
одно lvalue два указателя разных типов (НО за исключением char*, byte* и ещё там что-то...).
Короче, это дебри.

Чтобы не создавать себе таких проблем, в 20 стандарте ввели std::bit_cast:
```cpp
int m = bit_cast<int> (p);
```
И делает она примерно следующее:
```cpp
std::memcpy (&m, &p, sizeof(int));
```
И не ставит вас на колени перед строгим алиасигом
# functional style cast
- Функциональный каст - это вывернутый на изнанку c-style каст
```cpp
int a = (int) y; // c-style
int b = int(y) // functional-style c-style cast
```
- Разницы между ними нет, но заметьте
```cpp
int c = int {y}; // ctor, блокирующий сужающие преобразования
int d = S(x, y); // ctor, два аргумента
```
- Неприятно иногда вместо честного конструирования объекта обнаружить c-style каст
# Итог
Итак, почти всегда ваш выбор это static_cast или нечто похожее
В частности, он является вашим выбором для явных преобразований типов
# Неявное приведение типов
## Особенности неявного приведение типов 
### C-шные правила (применять сверху вниз):

Порядок: long double, double, float
  ```
   type 'op' fptype => fptype 'op' fptype
   ```

Порядок: long long, long, int
```
	type 'op' unsigned itype => unsigned itype 'op' unsigned itype
	type 'op' itype => itype 'op' itype
```

Любые комбинации (unsigned) short и (unsigned) char
```
	(itype less than int) 'op' (itype less than int) => int 'op' int
```
### Касты на инициализации
- Неявные касты на инициализации
```cpp
widetype x; narrowtype y;

[decayed] widetype z = y; // ok
[decayed] narrowtype v = x; // ok, если v вмещает значение x
```
- Понятно, что параментры функции - это тоже инициализация
```cpp
void foo(double);

foo(5); // ok, int implicitely promoted to double
```
### Унарный плюс (positive hack)
Оператор унарного плюса интересен тем, что для почти всех встроенных типов он не значит ничего.
Например, (2 == +2) is true

Но при этом, **даже если он не перегружен**, предоставляет легальный способ приведения к встроеному типу:
```cpp
struct Foo {
	operator long () {return 42;}
};

void foo (int x);
void foo (Foo x);

Foo f;
foo(f); // вызовет foo (Foo)
foo(+f); // вызовет foo (int)
```

## Еще про неявное приведение типов
Продолжение разговора о [[Operators and chains#Цепочечные операторы]]

Часто мы хотим, чтобы работали неявные преобразования
```cpp
Quat::Quat(int x);
Quat Quat::operator+(const Quat& rhs);

Quat t = x + 2; /*
	ok, int -> Quat;
	а тут все есть, так как Quat имеет 
	конструктор для 1 аргумента, 
	в котором и произойдёт этот implicit cast
*/  
Quat t = 2 + x; // FAIL!!!
// у int нету оператора +, который бы брал Quat
```
Увы, но метод класса не преобразует свой неявный аргумент

Отсюда вытекает, что единственный вариант сделать настоящие бинарные операторы - это делать их вне класса, вот так:
```cpp
// уже вне класса
Quat operator+ (Quat lhs, Quat rhs)
// и нам даже не нужно делать его friend,
// ведь есть += 
{
	Quat tmp {lhs};
	tmp += rhs;
	// опять же в терминах +=
	// но вот += всегда внутри класса
	return tmp;
}
```
### Призыв к осторожности
Одновременное наличие implicit ctors и 
внешних операторов может вызвать странные эффекты
```cpp
struct S {
	S(std::string) {}
	S(std::wstring) {}
};

bool operator==(S lhs, S rhs){return true;}

/* 
	для string и wstring нет оператора ==,
	но есть наш)
	Выберет компилятор его потому, что
	из-за наличия неоходимых конструкторов произойдёт 
	implicit cast из string -> S и wstring -> S,
	а потом и вызовется наше сравнение 
*/
assert (std::string{"foo"} == std::wstring{L"bar"}); // WAT?
```
В таких случаях стоит таки рассмотреть возможность занести сравнение
внутрь класса и сделать его friend.

Либо сделать конструкторы explicit, тогда тоже всё будет как надо
#### Одна небольшая проблема
Увы, для шаблонов данная техника с вынесенным за определение 
класса оператором работать не будет
```cpp
template <typename T>
Quat<T> operator+ (const Quat<T>& x, const Quat<T>& y)
{
	Quat<T> tmp {x};
	tmp += y;
	return tmp;
}
```
Такой оператор скорее всего будет иметь проблемы с подстановкой типов.
А происходит это потому, что неявное преобразование не используется
во время шаблонной подстановки по банальной причине -
она происходит после шаблонной подстановки.

Т.е. сначала у нас идёт процесс шаблонной подстановки, а уже 
потом разрешение перегрузки (overload resolution) с implicit кастами.

НО выход есть и тут))
```cpp
template <typename T>
Quat<T> operator+ (const Quat<T>& x, const Quat<T>& y)
{
	Quat<T> tmp {x};
	tmp += y;
	return tmp;
}

template <typename T>
Quat<T> operator+ (T& x, const Quat<T>& y)
{
	Quat<T> tmp {x};
	tmp += y;
	return tmp;
}

template <typename T>
Quat<T> operator+ (const Quat<T>& x, T& y)
{
	Quat<T> tmp {x};
	tmp += y;
	return tmp;
}
```
И это не прикол, так делают в стандартной библиотеке языка:
![[Pasted image 20251204182114.png]]