На долгом пути к std::vector - проектирование реалистичного шаблонного контейнера.
# Вектора
## От ручного выделения к векторам 
Рассмотрим ручное управление памятью:
```cpp
int *n = new int[10];

n[5] = 52;

// more code

delete [] n;
```
К данному куску кода есть много вопросов:
- Тут много кода
- Какой сейчас размер у n?
- Как стереть крайний элемент?
- Пуст ли n теперь?
- Необходимо не забыть освободить память в итоге.

Сравним с вектором - классом, который сам управляет ресурсами:
```cpp
vector<int> v(10);

v[5] = 812;

// more code

size_t size = v.size();

v.pop_back();

if (v.empty()) {/* some logic */}

// cleanup by destructor
```

## Требования к элементам контейнеров
### Два непростых вопроса
**Допустим, я хочу завести в своей программе вектор из константных ссылок**:
```cpp
std::vector<const int &> v;
```
Как идея?
Работать, очевидно, это не будет..

**Теперь допустим, что я не знаю тип переменной, но я знаю, что это контейнер**:
```cpp
template <typename Cont> void(const &Cont x){
	if (x.empty()) return;
	// some logic
}
```
Могу ли я быть увереным, что это будет работать для любого контейнера?
Очевидно, только если empty() есть как метод у всех контейнеров.

Логично задаться вопросом, а так ли это?

**Из данных вопросов следует то, что на самом деле мы предъявляем некий ряд требований как к элементам контейнера, так и к интерфейсу самого контейнера**.

Общие для всех контейнеров методы 
(пункт стандарта языка container.requirements.general):
- empty - проверка пустоты
- swap - обмен контейнерных переменных содержимым
- size - действительный размер контейнера
- clear(кроме array) - очистка контейнера
- begin, end, cbegin, cend - получение итераторов

Тут вынесены не все методы!

Требования элементов зависят от конкретного вида операции над контейнером (контейнеры реализованы с помощью шаблонов, а значит неиспользуемые методы попросту не будут инстанцированы), но чаще всего это:
- DefaultConstructible - требование к наличию конструктора по умолчанию
- MoveConstructible - требование к наличию конструктора перемещения или копирования
## Гарантии непрерывности памяти
"When choosing a container, remember vector is best. Leave a comment to explain if you choose from the rest."
(Tony van Eerd)

```cpp
// функция init написана в старом стиле 
template <typename T> void init(T* arr, size_t size);

// но её можно использовать с векторами, как не странно:
vector<T> t(n);
T *start = &t[0];
init_t(start, n);
assert(t[1] == start[1]);
```
Причина в том, как располагается данный контейнер в памяти:
![[Pasted image 20251224144842.png]]
Т.е. у нас есть гарантия на то, что он не отличим от простого выделения цельного куска. Но как всегда, есть НО.
## Неприятное исключение: `std::vector<bool>`
```cpp
vector<bool> t(n);
T *start = &t[0]; // это не скомпилируется, но представим
assert(t[1] == start[1]); // fail, oops!
```
Дело в том, что `vector<bool>` хранит информацию не в каждом байте, а в каждом бите, которые в языке не являются адресуемой ячейкой. 
Т.е. у него перегружен `operator[]` так, чтобы вытаскивать необходимое значения `i-`го бита, обарачивать его в некий прокси объект и возвращает он уже его.

Важно запомнить две вещи:
- `vector<bool>` не удовлетворяет соглашениям контейнера `vector`
- `vector<bool>` не содержит элементов типа `bool`
- Не используйте `vector<bool>` для обобщенного
  ```cpp
  using std::vector_bool = vector<bool>;
  vector_bool x(10); 
  ```
  Условно ок, но тут гораздо лучше использовать `std::bitset`, если размер в битах заранее известен, что чаще всего и является правдой когда мы говорим о масках.
## Задача: что можно тут улучшить?
```cpp
vector<int> v;

for(int i = 0; i != N; ++i)
	v.push_back(i);
// тут всё плохо, так как в зависимости от N 
// могут произойте реаллокации посреди цикла
// что делает его совсем не стабильным по скорости.
```
**Это означает, что всегда полезно думать о памяти вектора не меньше, чем о памяти динамического массива**.

А можем ли мы зарезервировать память, если заранее знаем, сколько нам надо? 
### Ответ: вектор не терпит халатности
Ответ - да, можем:
```cpp
vector<int> v;
v.reserve(N);

for(int i = 0; i != N; ++i)
	v.push_back(i);
// теперь тут не будет перевыделений
```

## Ещё про `size` и `capacity`
size - это сколько элементов у вектора уже есть
capacity - это сколько элементов в нём может быть до первого перевыделения 
```cpp
vector<int> v(1000);
assert(v.size() == 1000)
assert(v.capacity() >= 1000)
```

Размер это что-то, чем можно в явном виде управлять, в отличии от ёмкости:
```cpp
v.resize(52);
assert (v.size() == 52);
assert (v.capacity() >= 1000);

v.shrink_to_fit();
// after C++11
assert (v.size() == 52);
assert (v.capacity() >= 52);
```
## Амортизация
При написании метода `push`, вам предлагалось оценить его алгоритмическую сложность. Проблема в том, что она очевидно `O(1)`, если реаллокация не требуется и `O(n)` если она происходит.

То есть мы платим иногда. Это примерно как купить машину и платить за бензин пока машина не износится, а потом купить новую.

В экономике распределение стоимости товара по стоимости его периода эксплуатации называется амортизацией товара.

Амортизированная алгоритмическая сложность для `O(n)` обозначается как `O(n+)`.

### Амортизированная стоимость
По определению амортизированная стоимость операзции это стоимость `N` операций, отнесённая на `N`.
Для динамического массива $c_i = 1 + [\text{realloc}] * (i -1)$.
Амортизированная стоимость одной вставки будет $\frac{\sum_{i}^{N} c_i}{N}$ для `N` вставок.
Допустим, мы, если реаллокация нужна, растим массив на 10 элементов.
Тогда чему равна: 
$$\sum_{i}^{N} c_i = ? = N + \sum_{k = 1}^{N/10} 10 * k = O(n^2)$$
Заметим, что это очень плохая стратегия.
Амортизированная сложность `push` будет $\frac{O(N^2)}{N} = O(N+)$
Но стандарт говорит, что она должна быть `O(1+)`.
Можно ли предложить стратегию получше?
#### Лучшее решение
Рассмотрим прирост вдвое, а не на 10 элементов, тогда получим сумму вида:
$$
\frac{\sum c_i}{N} = \frac{N + \sum_{j = 1}^{lg(N)} 2^j}{N} = \frac{O(N)}{N} = O(1+)
$$
Видно, что разница есть: при одной стратегии у вас в среднем линейное, а при другой в среднем постоянное время вставки.
Увы, взять такую сумму для более общих сценариев уже не так просто, а скорее мучительно. Можно ли упростить себе жизнь?
### Дополнение: метод потенциала
Выберем функцию потенциала $\Phi(n)$ так, чтобы:
$$
\Phi(0) = 0,  \Phi(n) \ge 0
$$
где `n` - это номер шага.
Тогда амортизационная стоимость это стоимость плюс изменение потенциальной функции:
$$
c_n + \Phi(n) - \Phi(n-1)
$$
Выбор потенциальной функции облегчает вычисления, так как:
$$
\sum_j (c_j + \Phi(j) - \Phi(j-1)) = \Phi(n) - \Phi(0) + \sum_j c_j \ge \sum_j c_j
$$
Удачный выбор сделает выражение $\sum_j (c_j + \Phi(j) - \Phi(j-1))$ проще, чем $\sum_j c_j$.
#### Пример для массива
Тогда возникает вопрос, допустим, мы хотим найти такую фукцию потенциала для подсчёта амортизации операций над массивом.

Для массива у нас есть решение, опирающееся на удачную реаллокацию, которую мы рассмотрели ранее.
Тогда получим:
$$
\begin{align}
\text{since:}\\
2\cdot s_n \ge c_n\\
\text{thus:}\\
\Phi(n) = 2 \cdot s_n - c_n\\
s_n - \text{size of n'th step} \\
c_n - \text{capacity of n'th step} \\
\end{align}
$$
Тогда без реаллокации:
$$
\begin{align}
c_j + \Phi(j) - \Phi(j-1) = \\
c_j + 2\cdot s_j - c_j - (2\cdot s_{j-1} - c_{j-1}) = \\
C + 2\cdot s_j - C - (2\cdot s_{j-1} - C) = \\
2(s_j - s_{j-1}) - C = 2 - C = C'\\
\end{align}
$$
А с реаллокацией:
$$
\begin{align}
\Phi(j) = 2(k+1) - 2k = 2\\
\Phi(j - 1) = 2k - k = k\\
\Rightarrow c_j + \Phi(j) - \Phi(j-1) = \\
(k + 1) + 2 - k = 3 = C'
\end{align}
$$
В итоге в любом случае амортизация даст асимптотику `O(1)+`, ч.т.д

### Обсужение: реализация в std
Выбор простого роста в двое не всегда лучшая стратегия.
Реальная реализация из `stdlibc++` несколько сложнее и обладает рядом приятных теоретических свойств:
```cpp
const size_type __len = size() + std::max(size(), __n);
```

## Два механизма инициализации
- Расширенный синтакс с фигурными скобками.
- Явный конструктор из списка инициализации:
```cpp
class B{
	int a_;
public:
	B(int a) : a_(a){}
	B(std::initializer_list<int> il);
};

B b(1), c{1};
// теперь тут вызываются два разных конструктора
```
### Списочная инициализация: вектора
```cpp
vector<int> v1(3, 14);
// это вектор [14, 14, 14]

vector<int> v1{3, 14};
// это вектор [3, 14]
```

Это связано с наличием у вектора **нескольких** конструкторов:
```cpp
v(10); // размер 10, инициализация по умолчанию
v(10, 1); // размер 10, инициализация единицами
v{10, 1}; // размер = размеру списка, инициализация списком
```
### То же для ваших контейнеров 
Хорошая новость: initializer_list это тоже разновидность **последовательного контейнера** и его можно обходить итераторами:
```cpp
template <typename T> class Tree {
	// some specific to Tree code
	bool add_node(T &data);
public:
	Tree(initialize_list<T> il){
	for(auto ilit = il.begin(),
		ilend = il.end(); ilit != ilend; ++ilit){
			add_node(*ilit);
		}
	}
};
```
Плохая новость: теперь вам надо следить есть ли он в классе.
### Простое правило для `{}`
Называется этот поведение скобок - uniform initialization (C++11).
Интересное свойство - такая инициализация блокирует сужающие преобразования.

Порядок выбора при конструировании:
- Если в классе совсем нет конструкторов, это агрегат как в C.
  Но такое поведение доступно только если все поля класса имеют один и тот же модификатор доступа public! Иначе это перестаёт работать
- Иначе, если есть конструктор от initialize_list, будет рассмотрен он.
- Иначе, если есть любой другой конструктор, будет рассмотрен этот любой другой конструктор.
## Первое представление об итератораx
![[Pasted image 20251225192742.png]]
### Итератора - абстракция указателя
Важно, что итераторы не являются указателями, они абстрагируют их.
В итоге любой контейнер может быть сконструирован из любого диапазона.
```cpp
std::list<int> l {1,2,3};
std::vector<int> (l.begin(), l.end());
```
Это потрясающе удобно чтобы перекидывать один контейнер в другой.

А теперь наконец вернёмся к вопросу, который был задан ранее, а как написать конструктор из двух итераторов? 
## Конструирование из итераторов
Наивная попытка выглядит как:
```cpp
template <typename T> class MyVector {
	// ....
public:
	MyVector(size_t nelts, T value); // 1
	template <typename Iter>
	MyVector(Iter fst, Iter lst); // 2
	
	// ....
};

MyVector<int> v (2, 2); 
// пупупу, тут ошибка, выбран второй конструктор
```

# SFINAE
## Обсуждение: провал подстановки
Что если подстановка в шаблон некотором контексте не может быть выполнена?
```cpp
template <typename T>
typename T::ElementT at(T const &a, int i);

int *p = new int[30];
auto a = at<int *>(p, i); // substitution failure
```
Что если вывод типов в некотором контексте провален?
```cpp
template <typename T> T max(T a, T b);

int g = max(2, 1.0); // deduction failure
```

Правильный ответ - искать дальше.
## Идиома SFINAE
**Substitution failure is not an error** 
(Провал подстановки не является ошибкой)
```cpp
template <typename T> T max(T a, T b);
template <typename T, typename U> auto max(T a, U b);

int g = max(1, 1.0):
// подстановка в 1 провалена
// подстановка в 2 успешна! 
```

Если в результате подстановки в **непосредственном контексте** класса (функции, алиаса, переменной) возникает **невалидная конструкция**, эта подстановка неуспешна, но не ошибочна!

В этом случае вторая фаза инстанцирования, т.е. поиск имён, просто не выполняется.
## SFINAE и ошибки
Не любая ошибочная конструкция это SFINAE.
Важен контекст подстановки.
```cpp
int negate(int i){
	return -i;
}

template <typename T> T negate(const &T t){
// успешная подстановка в непосредственном контексте - 
// в данном случае, подстановка в определение функции
	typename T::value_type n = -t();
	// тут используем n
}


negate(2.0);
// ошибка второй фазы, т.е. ошибка компиляции
// ошибка не потому, что подставить не получилось, а 
// потому, что нет у double члена double::value_type
```
Здесь в контексте сигнатуры и шаблонных параметров нет никакой невалидности. 

```cpp
int negate(int i){
	return -i;
}

template <typename T> 
typename T::value_type negate(const T &t){
// подстановка уже не выйдет в непосредственном контексте
// так как опять же double::value_type не существует
}


negate(2.0);
// а вот теперь substitution failure
```
## Обсуждение
Техника SFINAE кажется очень простой, но вообще-то её приложения многочисленны и часто очень нетривиальны.

Рассмотрим такую задачу: у вас есть два типа и вам нужно определить, равны они или нет.

```cpp
template <typename T, typename U>
int foo{
	// как вернуть 1, если T == U и 0 если нет?
}
```
Обратим внимание, что это задача отображения из типов на числа.
Прежде, чем её решать, решим обратную к ней.

## Интегральные константы
Отображение из чисел на типы называется интегральной константой:
```cpp
template <typename T, T v> struct integral_constant {
	static const T value = v;
	using value_type = T;
	using type = integral_constant;
	operator value_type() const {return value;}
	// это оператор implicit каста пользовательский 
	// в тип value_type = T;
};
```
Возможна даже арифметика:
```cpp
using ic6 = integral_constant<int, 6>;
auto n = 7 * ic6{};
// и вот тут мы вызываем этот оператор 
```
### Истина и ложь для типов
Самые полезные из интегральных констант - самые простые:
```cpp
using true_type = std::integral_constant<bool, true>;
using false_type = std::integral_constant<bool, false>;
```
Всё это есть в стандарте: `std::integral_constant` и т.д.

Попробуем написать простой определитель, чтобы проверить одинаковые ли два типа:
```cpp
template <typename T, typename U>
struct is_same : std::false_type {};
```
По умолчанию разные, а что дальше?
Правильно, делаем частичную специализацию:
```cpp
#include <cassert>

template <typename T, typename U>
struct is_same : std::false_type {};

template <typename T>
struct is_same<T, T> : std::true_type {};

template <typename T, typename U>
using is_same_t = typename is_same<T, U>::type;

int main() {
  assert(
    (is_same<int, int>::value &&
     !is_same<char, int>::value)
  );
    assert(
    (is_same_t<int, int>::value &&
     !is_same_t<char, int>::value)
    );
}
```

Тут замечаем данную триаду, которая образует так называемые SFINAE триады:
```cpp
// primary template
template <typename T, typename U>
struct is_same : std::false_type {};

// specialization
template <typename T>
struct is_same<T, T> : std::true_type {};

// convinient alias
template <typename T, typename U>
using is_same_t = typename is_same<T, U>::type;
```

### Определители и модификаторы
Определитель: является ли тип ссылкой
```cpp
template <typename T> 
struct is_reference : std: false_type {};

template <typename T> 
struct is_reference<T&>: std: true_type {};

template <typename T>
struct is_reference<T&&>: std: true_type {};
```
Модификатор: убираем ссылку с типа, если ссылки не было, оставляем тип
```cpp
template <typename T> 
struct remove_reference {using type = T};

template <typename T> 
struct remove_reference<T&> {using type = T};

template <typename T>
struct remove_reference<T&&> {using type = T};

// для модификатора полезен алиас
template <typename T>
using remove_reference_t = typename remove_reference<T>::type;
```
## Категории объектов в языке
Любой объект в языке относится к одной из 14 категорий:
```cpp
is_void;
is_null_pointer;
is_integral, is_floating_point; 
// для T и для cv T& транзитивно

is_array;
// только встроенные, не std::array тут имеется ввиду

is_pointer;
// включая указатели на обычные функции

is_lvalue_reference, is rvalue_reference;
is_member_object_pointer, is member_function_pointer;
is_enum, is_union, is_class;

is function;
// обычные функции
```
Использование довольно тривиально:
```cpp
std::cout << std::boolalpha << 
	std::is_void<T>::value << std::endl;
```

## Свойства типов в языке
Также очень полезны определители свойств типов:
```cpp
is_trivially_copyable;
// побайтово копируемый, memcpy

is_standard_layout;
// можно адресовать поля указателем

is_aggregate;
// доступна агрегатная инициализация как в C

is_default_constructible;
// есть default ctor

is_copy_constructible; is_copy_assignable;
is_move_constructible; is_nothrow_move_constructible;
is_move_assignable;

is_base_of;
// B является базой (транзитивно, включая сам тип)

is_convertible;
// есть преобразования из A к B
```
И многие другие(их реально десятки).
И все они написаны на таких же SFINAE триадах, которые мы обсудили выше. Поэтому всегда стоит заглянуть, возможно, триада, которую тянутся руки написать самому уже является частью стандартной библиотеки.
## Обсуждение: std::copy
Рассмотрим наивное копирование, чем-то похожее на алгоритм std::copy
```cpp
template <typename InpIter, typename OutIter>
OutIter cross_copy(InpIter fst, InpIter lst, OutIter dst){
	while (fst != lst) {
		*dst = *fst; ++fst; ++dst;	
	}
	return dst;
}
```
Увы, по сравнению с настоящим std::copy у него есть проблемы.
Он тупо медленнее.

Можем ли мы решить с помощью SFINAE?
### Решение проблемы std::copy
Заведём хелпер и его специализацию для true:
```cpp
template <bool Triv, typename In, typename Out>
struct CpSel {
	static Out select(In begin, In end, Out out){
		return CopyNormal(begin, end, out);
	}
};

template <typename In, typename Out>
struct CpSel<true, In, Out>{
	static Out select(In begin, In end, Out out){
		return CopyFast(begin, end, out); 
		// для простых типов
	}
};

// теперь сам алгоритм копирования будет просто решать
// кого он вызывает
```

Также тривиально мы решаем проблему с копированием:
```cpp
template <typename It, typename Out>
// конечно, мы ещё пока не гарантировали
// что тут именно итераторы, но до этого ещё дойдём
// но уже лезем в iterator_traits...)
Out realistic_copy(It begin, It end, Out out){
#if NO_CLUE
	using in_type = // pointee type(In); 
	using out_type = // pointee type(Out); 
	// а как это написать?
	// ответ - с помощью iterator_traits
#else 
	using in_type = typename 
		std::iterator_traits<In>::value_type;
	using out_type = typename 
		std::iterator_traits<Out>::value_type;

#endif

#if PRE_C++11_ERA
// enum's value is garanteed to be 
// compile time constant, so 
// Sel here becomes 1 or 0
// so it's more like type-level if semantics
	enum {
		Sel = std::is_trivially_copyable<in_type>::value && 
		std::is_trivially_copyable<out_type>::value &&
		std::is_same<in_type, out_type>::value
	};
#else
	static constexpr bool Sel =
    std::is_trivially_copyable_v<in_type> &&
    std::is_trivially_copyable_v<out_type> &&
    std::is_same_v<in_type, out_type>;
#endif
	
	return CpSel<Sel, In, Out>::select(begin, end, out);
}
```

# Вариабельные шаблоны
Мы всё ещё на пути к реализации вектора и его инициализации через два итератора, которая не будет ловить два int как валидные аргументы.
## Обсуждение: emplace
Теперь единственным облачком на горизонте остался emplace
```cpp
struct S {
	S();
	S(int, double, int);
};

std::vector<S> v;

v.emplace_back(1, 1.0, 52); // создали на месте

S s;
v.emplate_back(s);
// к сожалению, работает, но такая конструкция - это буквально вызов копирования и ничем не отличается от push_back
```
Но вот вопрос, а как это может работать для любого типа, если мы в общем случае не знаем количество аргументов конструктора?

## Вариабельные шаблоны
Пример вариабильной шаблонной функции:
```cpp
<typename ... Args> 
void f(Args ... args);

// Способы вызова:
f(); // ок, пачка не содержит аргументов
f(1); // ок, пачка содержит 1 аргумент: int
f(1, 52.2); // ок, пачка содержит 2 аргумента: int, double
```
И есть специальная конструкция, которая возвращает размер пачки в штуках:
```cpp
sizeof...(Args);
// либо
sizeof...(args);
```
### Паттерн раскрытия
Говорят, что пачка параметров "раскрывается" в теле функции или класса:
```cpp
template <typename ... Types> void f(Types ... args);

template <typename ... Types> void g(Types ... args){
	f(args ...);    // -> f(x, y);
	f(&args ...);   // -> f(&x, &y);
	f(h(args) ...); // -> f(h(x), h(y));
	f(const_cast<const Types *>(&args) ...); // ->
	// -> f(
	// const_cast<const int *>(&x),
	// const_cast<const int *>(&y),
	// );
}

g(1, 1.0); // g(int x, double y);
```
Т.е. паттерн `...` применяется к максимально длинному синтаксически корректному выражению слева от себя, и делается это жадно.

#### Задача: раскрытие пачек
Допустим, `args` это пачка параметров `x,y,z`.
Тогда во что раскроются следующие выражения:
```cpp
f(h(args ...) + h(args)...);
f(
	h(x, y, z) + h(x), 
	h(x, y, z) + h(y), 
	h(x, y, z) + h(z)
);

А(h(args, args...)...)
f(
	h(x, x,y,z),
	h(y, x,y,z),
	h(z, x,y,z)
);
```
Т.е. раскрытие мы начинаем с самой вложенной конструкции `...`.

## Снова прозрачная оболочка
На лекции по `rvalue refs` была написана почти идеальная прозрачная оболочка для одного аргумента:
```cpp
template <typename Fun, typename Arg>
decltype(auto) transparent(Fun fun, Arg&& arg){
	return fun(std::forward<Arg>(arg));
}
```
Как её расширить для того, чтобы количество аргументов не было ограничено? Это очень простое и чисто техническое изменение:
```cpp
template <typename Fun, typename ... Args>
decltype(auto) transparent(Fun fun, Args&& ... args){
	return fun(std::forward<Args>(args) ...);
}
```
Следует обратить особое внимание на паттерн совместного раскрытия при пробросе.
### Обсуждение: пробросим функцию?
В нашу оболочку может прийти как функция, так и функцие-подобный объект (наш или лямбда). Но тут возникает проблема с тем, что у этого объекта будет вызван конструктор копирования или `&&` аннотированна фунция для проброса (перегруженный operator()). 

В фунцие-подобном объекте оператор вызова может быть `&&` аннотирован, тогда он не пролезет в наш прозрачный шаблон. Выход есть, собственно, такой же, как и для проброса аргументов:
```cpp
template <typename Fun, typename ... Args>
decltype(auto) transparent(Fun &&fun, Args&& ... args){
	return std::forward<Fun>(fun)
		(std::forward<Args>(args)...);
}
```
Теперь и фукции тоже не потребуют копирования в обозначенном кейсе.

ВОТ ТЕПЕРЬ НАША ОБОЛОЧКА ДЕЙСТВИТЕЛЬНО НЕ ДЕЛАЕТ ВООБЩЕ НИЧЕГО, ПРЯМ СОВСЕМ...

Казалось бы, зачем? Но вообще-то это и есть фукция emplace.

## Контейнеры тяжёлых классов
Мы уже говорили о хранении тяжёлых классов в контейнерах:
```cpp
template <typename T> class Stack {
	struct StackNode {
		T elem;
		StackNode * next;
		StackNode(T e, StackNode *next) : 
			elem(e), next(next) {}
	};
public:
	void push(const T& elem){
		top_ = new StackNode(elem, top_);
	}
	// и т.д.
};
```
А теперь подумаем о следующем коде:
```cpp
s.push(Heavy(100, 200, 300));
// очень плохо, объект дополнительно скопируется
// и более того - не один раз, так как например StackNode 
// принимает элемент также по значению, а значит, чтобы он 
// появился на фрейме этой функции, тоже будет вызвано 
// копирование(допущение - move ctor делегирует copy ctor`у)
```
### Emplace
Обычно, метод контейнера, который **размещает** объект, а не пробрасывает его, называется `emplace`.
```cpp
template <typename T> class Stack {
// детали реализации
public:
	void push(const T& elem){
		top_ = new StackNode(elem, top_);
	}
	
	template <typename U> void emplace(U&& ... args){
		top_ = new StackNode(top_,
			std::forward<U>(args)...);
	}
};
```
В стандартной библиотеке размещение поддерживают все последовательные контейнеры.

## Интерлюдия: шаблонные методы
Шаблонный метод вне класса определяется с двумя наборами параметров - 
параметрами класса и своими параметрами:
```cpp
template <typename T>
template <typename ... Args>
void Stack<T>::emplace_back(Args&& ... args) {
	top_ = new StackElem(top_, std::forward<Args>(args)...)
}
```
Т.е. тут нет опечатки, каждый набор идёт отдельно, один за маму...
Все наборы совокупно участвуют в `template-id` метода и это важно для специализации
### Специализация шаблонных методов
При специализации шаблонных методов важно понимать, что их необходимо специализировать их по всем аргументам:
```cpp
template <typename T> /* раз пачка */
struct Foo {
	template <typename U> /* два пачка */
	void foo { .... }
};

template <> /* раз пачка */
template <> /* два пачка */
void Foo<int>::foo<double>() { .... }
```
Иначе это будет частичная специализация, которая запрещена как для функций, так и для методов.

## Шаблонные методы против ООП
Тезис - шаблонные методы это неизбежное, но зло.
Обычно мы хотим делать конструктор шаблонным, но не остальные метод.
Пример `emplace` - это пример неизбежного зла.

Вы должны понимать, что любой открытый шаблонный метод в вашем классе обнуляет инкапсуляцию:
```cpp
class Foo {
	int donottouch_ = 52;
public:
	template <typename T> foo () { .... }
};


struct MyTag{};

template <> void Foo::foo<MyTag>() {donottouch_ = 3};
// ooups...
```
## Обсуждение
Тем не менее, мы пока не понимаем, как использовать SFINAE для решения проблемы с конструктором из пары итераторов, пусть даже с вариабельными шаблонами. 

Решим наконец этот вопрос.

## void_t
Появился в C++17 как std::void_t но вообще-то довольно прост:
```cpp
template <typename ...> using void_t = void;
```
Интуитивно `void_t <T, U, V>` означает `void`, если все указанные в пачке типы легальны, и нелегален, если нелегален хотя бы один из этих типов.

Думайте о нём как о логической конъюнкции("И") SFINAE характеристик.

### Задача: зависимый тип
С ранних пор была замечена полезность техники SFINAE для трюков и хаков. Классический пример: определить наличие зависимого типа в классе:
```cpp
struct Foo { using foobar = float};
struct Bar {};

std::cout << std::booalpha << 
 /*тут true надо */ ??? Foo << " " <<
 /*а тут false*/ ??? Bar;
```
Это снова отображение из типов в целые числа и без SFINAE задача опять выглядит нерешаемой.

#### Решение: void_t
Решение использует SFINAE и void_t:
```cpp
template <typename, typename = void>
struct has_using_foobar: std::false_type{};

template <typename T>
struct has_using_foobar<T, std::void_t<typename T::foobar>> : std::true_type;
```

И теперь мы можем определить вещи на этапе компиляции:
```cpp
struct Foo {using foobar = float;};
std::cout << std::boolalpha << has_using_foobar<Foo> {};
```
## Конструирование из итераторов
Можно попытаться решить задачу с итераторами вот так:
```cpp
MyVector(size_t nelts, T value);

template <
	typename Iter,
	typename = void_t<
		decltype(*Iter{}),
		decltype(++Iter{}),
		// временные объекты, которые мы тут создаём
		// это печально, нам бы этого не хотелось
		
		// а вдруг их вообще создать нельзя так, вдруг 
		// нет у такого итератора конструктора по умолчанию,
		// а ++ и * определены, что тогда?
		
		// т.е. тут мы неявно ещё потребовали наличие
		// конструктора по умолчанию у итератора
	>	
>
MyVector(Iter fst, Iter lst);
```
Увы, но это не слишком изящно.
Дело в том, что инкремент требует lvalue.

Но его-то мы пока как раз и не можем создать.
Хотя, иногда везёт.

## Абстракция значения
В некоторых случаях (например, для использования внутри decltype) хочется получить значение некоего типа.

Часто для этого используется конструктор по умолчанию:
```cpp
template <typename T> struct Tricky {
	Tricky() = delete;
	const volatile T foo();
};

decltype(Tricky<int>().foo()) t; // ошибка
```
А что тогда делать, если конструктора нет?
Что такое "значение вообще" для такого типа?
### Абстракция значение: declval
Интересный способ решить эти проблемы это ввести шаблон функции (который выводит типы) без тела (чтобы его нельзя было по ошибке вызвать):
```cpp
template <typename T>
add_rvalue_reference_t <T> declval();
```
А теперь всё просто:
```cpp
template <typename T> struct Tricky {
	Tricky() = delete;
	const volatile T foo();
};

decltype(declval<Tricky<int>>().foo()) t; // ok
/*
	decltype не вычисляет выражение, а только оценивает.
	тут мы оцениваем выражение, которое как бы возвращает 
	функция declval, но у неё нет тела, т.е. её буквально
	нельзя вызвать, а только оценить, что и решает
	нашу проблему с созданием временного объекта для оценки,
	теперь в нём нет необходимости.
	
	также мы отмели старый добрый С-шный вариант
	кастануть nullptr к Tricky<int> для решения нашей 
	проблемы, так как такой каст - это UB
*/
```
Но какова природа этого значения?
### Обсуждение: возвращение && value из функции
Пожалуй, есть всего три функции, для которых имеет смысл возвращать правую ссылку (то есть производить xvalue):
1. std::move
2. std::forward
3. std::declval
(Это не все конечно, но о других пока не надо)

Если вы хотите написать свою функцию, которая возвращает `&&` это значит:
 - Вы что-то делаете не так
 - Или вы хотите написать одну из функций выше 
 - Или вы пишите `&&`-аннотированную функцию

## Конструирование из итераторов 2
Теперь мы видим совсем изящное решение:
```cpp
MyVector(size_t nelts, T value);

template <
	typename Iter,
	typename = void_t<
		decltype(*std::declval(Iter&)()),
		decltype(++std::declval(Iter&)()),
	>	
>
MyVector(Iter fst, Iter lst);
....

MyVector v1 (10 ,3); // 1, так как 2 провалилось
MyVector v2 (v1.begin(), v1.end()); // 2
```
# Свёртки 

| Паттерн свёртки    | Результирующее выражение                  |
| ------------------ | ----------------------------------------- |
| `... op pack`      | `( ... (p1 op p2) op p3) ... op pN)`      |
| `init ... op pack` | `( ... (init op p1) op p2) ... op pN)`    |
| `pack op ...`      | `(p1 op (p2 op ( ... (pN-1 op pN) ...)`   |
| `pack op ... fini` | `(p1 op (p2 op ( ... (pN-1 op fini) ...)` |
```cpp
template <typename ... T>
auto sum_all (T ... args) {return (args + ...);}

template <typename ... T>
auto print_all (T ... args) {
	(std::cout << ... << args) << std::endl;
}
```
## Проблемы print_all
```cpp
template <typename ... T>
auto print_all (T ... args) {
	(std::cout << ... << args) << std::endl;
}
```
Очевидно, что `print_all` записанный как есть не вставляет между выводимыми числами пробельные символы:
```cpp
print_all(1,1.2,3); // -> 11.23
```
Как заставить его работать? 
С учётом того, что после последнего аргумента нам не нужен пробел, 
как и в случае отсутствия аргументов.
### Решение: функтор AddSpace
Вандерворд и Йосьюттис предлагают следующее решение (правда с тем самым лишним пробелом):
```cpp
template <typename T> class AddSpace {
	const T& ref;
public:
	AddSpace(const T& r) : ref(r) {}
	ostream& operator<< (ostream &os, AddSpace s){
		return os << s.ref << ' ';	
	})
};

template <typename ... T>
void print_all (T ... args) {
	(std::cout << ... << AddSpace(args)) << std::endl;
}
```
### Экзотические свёртки
![[Pasted image 20251229134311.png]]