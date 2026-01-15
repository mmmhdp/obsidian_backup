Программы времени компиляции и метапрограммы.
# Рекурсивное инстанцирование
## Идея "рекурсивного" раскрытия
Вспомним функцию `print_all`, которая была написана ранее:
```cpp
void print_all() {}

template <typename T, typename... Arg>
void print_all(T first, Args... args){
	std::cout << first << " ";
	print_all(args...);
}
```
Здесь порождается цепочка экземпляров шаблонной функции:
```cpp
print_all(1, 1.0, 1u);

// -> print_all <double, unsigned>(1.0, 1u)
// -> print_all <unsigned> (1u) -> print_all();
```
## Обсуждение
На самом деле никакой рекурсии тут нет:
цепочка инстанцирований порождает разные инстанцирования.

Но сама схожесть процессов наводит на некоторые мысли...

Первым, кого она навела на мысли был Эрвин Анрух в далёком 1994-ом.
Без малого, эти мысли **обусловили успех `C++` как языка!**
## Открытие метапрограммирования
```cpp
template <int i> struct D {
	D(void *); // 0 is ok, but not 1
	operator(int);
};

template <int p, int i> struct is_prime {
	enum {
		prim = (p % i) 
		&& is_prime<(i > 2 ? p : 0), i - 1>::prim
	};
};

template <int i> struct Prime_print {
	Prime_print<i - 1> a;
	enum {
		prim = is_prime<i, i - 1>::prim
	};
	void f() { D<i> d = prim;} // error:
	// Type enum can't be converted to type D<3>
};

struct is_prime<0, 0> { enum{ prim = 1}; }
struct is_prime<0, 1> { enum{ prim = 1}; }
struct Prime_print<2> { 
	enum{ prim = 1}; 
	void f() {
		D<2> d = prim;	
	}
};

int main () {
	Prime_print<30> a;
}
```
Этот код н рабочий на текущих компиляторах, чтобы запустить его сейчас - его необходимо обновить под текущие требования компиляторов.
## Факториал
Идея лежит на поверхности:
что если развернуть систематическое `SFINAE` от типов на целые числа?

```cpp
#include <iostream>

template <size_t N>
struct fact : 
	std::integral_constant<size_t, N * fact<N - 1>{}> {};

template <>
struct fact<0> : std::integral_constant<size_t, 1> {};

int main() { std::cout << fact<40>::value << std::endl; }
```

Вычисление итогового значения выполняется на этапе компиляции.
Наследование играет роль рекурсивного вызова.
## Числа Фибоначчи
С той же легкостью можно вычислить на этапе компиляции и числа Фибоначчи:
```cpp
#include <iostream>

template <int N>
struct fibonacci : std::integral_constant<
	int,
	fibonacci<N - 1>{} + fibonacci<N - 2>{}> {};

template <> struct fibonacci<0> : std::integral_constant<int, 0> {};

template <> struct fibonacci<1> : std::integral_constant<int, 1> {};

int main() { 
    std::cout << fibonacci<0>{} << std::endl;
    std::cout << fibonacci<1>{} << std::endl;
    std::cout << fibonacci<2>{} << std::endl;
    std::cout << fibonacci<3>{} << std::endl;
    std::cout << fibonacci<4>{} << std::endl;
    std::cout << fibonacci<5>{} << std::endl;
    std::cout << fibonacci<6>{} << std::endl;
    std::cout << fibonacci<7>{} << std::endl;
}
```
Не смущает ли нас тут двойная "рекурсия"?
Нет, так как её тут просто нет, инстанциации шаблона с одинаковой сигнатурой происходить не будет, после создания типа он кэшируется и компилятор помнит об его существовании в дальнейшем.
## Две модели вычислений
- "Императивная"
```cpp
int fact (int x) {
	int i = 2, res = 1;
	for (; i <= x; ++i)
		res *= i;
	return res;
}
```
1. Временные переменные 
2. Циклы
3. Изменяемая память
- "Функциональная"
```cpp
int fact (const int x) {
	if (x < 2)
		return x;
	else 
		return x * fact(x - 1);
}
```
1. Вызовы функций
2. Рекурсия
3. "Чистые" вычисления
## Целочисленный квадратный корень: рантайм вычисление
Чтобы сделать такие сложные вещи на шаблонах, полезно сначала просто написать программу в функциональном стиле и тогда мы уже заметим базис для шаблонной реализации:
```cpp
int isqrt (int N, int lo = 1, int hi = N){
	// тут mid - это не переменная
	// а алиас для выражения слева - такие переменные
	// мы можем иметь в своем функциональном коде
	int mid = (lo + hi + 1) / 2;
	//
	
	// а вот и наш "конец рекурсии"
	// т.е. специализация шаблона
	if (lo == hi)
		return lo;
	//

	// а вот, собственно, и основной шаблон
	// но внутри у него есть проблема
	else {
		// как организовать этот if?
		// на уровне шаблонов
		if (N < mid * mid)	
			return isqrt (N, lo, mid - 1);
		else
			return isqrt (N, mid, hi);
	}
	//
	//
}
```
Решение есть.
### Условный тип
Вспомним уже известный нам условный тип [[Итераторы#Интерлюдия conditional_type]].
Это отображение из `{true, false} -> {T, F}`
## Целочисленный квадратный корень: компайл тайм вычисление
Здесь `std::conditional` вполне сработает в качестве `meta-if`:
```cpp
template 
<int N, int L = 1, int H = N, int mid = (L + H + 1) / 2>
struct Sqrt : std::integral_constant<
	int,
    std::conditional_t<
	    (N < mid * mid),
	    Sqrt<N, L, mid - 1>,
	    Sqrt<N, mid, H>
    > {}
>{};

template 
<int N, int S> struct Sqrt <N, S, S, S> :
	std::integral_constant<int, S> {};
```
## Квадранты вычислений
1. `Runtime computations`
2. `Compile-time computations`
3. `Type-level computations`
```cpp
template <typename T>
struct add_const_pointer {
	using type = const T*;
};

using types = mpl::vector<int, char, float, void>;
using pointers = mpl::transform<types,
	ass_const_pointer<mpl::_1>>::type;
```
4. `Heterogenious computations`
```cpp
auto to_string = [](auto t) {
	std::stringstream ss; ss << t; return ss.str();
};

fusion::vector<int, std::string, float> seq {
	1, "abc", 3.4f
};

auto strings = fusion::transform(seq, to_string);
```
## Обсуждение
Поговорим о вычислениях времени компиляции.
Допустим, я хочу предвычислить на этапе компиляции первые двадцать чисел Фибоначчи и использовать их на этапе исполнения.  

Неожиданно - становится сложно. Допустим, мы даже будем записывать в массив вычисленные в шаблоне значения, но опять же, вычислим мы на этапе компиляции - а запишем всё равно в рантайме.

Вспоминая о квадрантах, на самом деле то, чего мы в итоге действительно хотим, и чем разработчики языка `C++` занимаются всё это время, это как раз сближение квадрантов, сближения формы работы с разными квадрантами.

Именно в контексте сближения формы вычислений на этапе компиляции и формы вычислений в рантайме появляются `constexpr`.
# Constexpr функции
## Константность
В чём смысл следующей конструкции и где она может быть применима?
```cpp
uint8_t const volatile * const p_latch_reg = (uint8_t *) 0x42;
```
Это неизменяемый указатель на неизменяемую переменную, которая таки может измениться, но не из нашей программы - проще говоря, это хендл для работы с сигналом на резисторе (или проводок из платы).

Т.е. мы можем с него считать данные, но не изменить их.
При этом сами данные могут непредсказуемо меняться, так что доступ к ним нельзя соптимизировать:
```cpp
data = *p_latch_reg; // считали данные
....
data = *p_latch_reg; // ЕЩЁ РАЗ считали данные
```

Этот пример показывает, что на самом деле смысл `const` правильно толковать как `readonly`.

Хотя нативно могло показаться, что `const` - если прочитать как "неизменяемое" - это вещи, которые известны на этапе компиляции.
К сожалению, это не так.

Спойлер - для выражения именно этой мысле - что нам известно нечто на этапе компиляции, и появится `constexpr`.
## Что известно на этапе компиляции
Обсудим, а что нам известно на этапе компиляции:
- Литералы `(1, "hello", 'c', 1.0, 1ull)` и члены `enum`
- Параметры шаблонов и результаты `sizeof` над типами
- `constexpr` переменные:
  ```cpp
  template <typename T> struct my_numeric_limits;
  
  template <> struct my_numeric_limits<char> {
	 static constexpr size_t max() { return CHAR_MAX;}  
  };
  
  constexpr size_t arrsz = my_numeric_limits<char>::max();
  int arr[arrsz]; // OK
  ```
## Ограничения на `constexpr` переменные
- `constexpr` переменная должна иметь **литеральный тип**
- Использовать `constexprs` с плавающей точкой можно, но не рекомендуется:
  ```cpp
  constexpr float ct = 1.0f / 3.0f;
  
  assert (x == 1.0f && y = 3.0f);
  float (rt = x / y);
  
  assert (rt == ct); // ORLY?
  ```
  Потенциальная проблема раз - компилируем с одной точностью, а исполнение на машине с другой точностью. Но допустим, что и там и там IEEE floats.
  
  Но если так, то они всё равно могут разойтись - в рантайме можно поменять округление, что всё также может разойтись с округлением на этапе компиляции.
## `constexpr` означает `const`??
Следующий случай может быть несколько неочевиден:
```cpp
constexpr int arr[] = {2, 3, 5, 7, 11};

constexpr int *x = &arr[3]; // а всё ли тут хорошо?
```
Тут зависит от того, к чему относится `constexpr` во второй строчке.
Варианта, собственно, два:
1. `constexpr int * x -> const int * x`
2. `constexpr int * x -> int * const x` (предположу, это верный)

И кажется, я прав.
Указатель в массив известен на этапе компиляции - это же `arr + offset`, а и имя и `offset` известны на этом этапе.

Далее, для `constexpr` не работает `cdecl` по простой причине - `constexpr` не является частью типа переменной, а является аннотацией для этой переменной.

А значит, правильный вариант:
```cpp
constexpr int arr[] = {2, 3, 5, 7, 11};

constexpr const int *x = &arr[3]; // а всё ли тут хорошо?
```
Т.е. теперь второй вариант семантически верен: мы объявили `constexpr pointer`.

А теперь, раз у нас есть известный на этапе компиляции указатель, мы знаем, куда мы можем писать наши числа Фибоначчи и прочие вещи, которые мы были способны вычислить на этапе компиляции. 

## `C++17`: `constexpr` control flow
Возможность использования выражений времени компиляции делает интересным вопрос переключения по ним:
```cpp
if constexpr(b) {
	// тут много кода
} else {
	// эта ветка не участвует в инстанцировании
}
```
Начиная с `C++17` такое ленивое поведение предоставляет `if constexpr`.

Обратите внимание: основное использование этой конструкции это выбрасывание веток инстанцирований. И по стандарту это именно что его главная идейное предназначение.

В примере выше, если условие `b == true`, тогда код в ветке `else` буквально не будет скомпилирован в итоговый модуль. 

## Некоторые альтернативы `SFINAE`
```cpp
template <typename T> enable_if_t<(sizeof(T) > 4)>
foo(T x) { 
	//  сделать что-то с x
} 

template <typename T> enable_if_t<(sizeof(T) <= 4)>
foo(T x) { 
	//  сделать что-то ещё с x
} 
```

Кажется, у нас появился другой вариант:
```cpp
template <typename T> void
foo(T x) {
	if constexpr (sizeof(T) > 4) {
		//  сделать что-то с x
	} else {
		//  сделать что-то ещё с x
	}
}
```

Но выглядит это слишком интрузивно (связанно).
Скоро мы увидим более качественные опции.

## `if constexpr` для вариабельных шаблонов
В случае вариабельных шаблонов тоже можно избежать специализации:
```cpp
template <typename Head, typename... Tail>
void print(Head head, Tail... tail){
	std::cout << head;
	if constexpr (sizeof...(tail) > 0) {
		std::cout << ", ";
		print(tail...);
	}
}
```
## Снова о метапрограммах
Простая задача: возведение в квадрат времени компиляции:
```cpp
template <size_t n> struct square :
	std::integral_constant<size_t, n * n>;

int arr[square<5>{}]; // ok arr[25]
```
Тут угадать, что `square` на самом деле функтор - довольно сложно.

```cpp
constexpr int square(int x) { return x * x; }

int arr[square(5)]; // ok arr[25]
```
Теперь очевидно, что мы вызываем функцию времени компиляции.
Стандарт накладывает некоторые ограничения на тела таких фунций.
## Список ограничений на `constexpr`-функции для `C++14` 
Важно понимать, что данный список кажется куда более натуральным, если принять то, что для вычислений на этапе компиляции не существует ещё никакой памяти у программы. Скажем, вычисления на этапе компиляции происходят в своего рода вакууме, Платоновском пространстве. 

Именно поэтому, если у нас есть массив `int a [4]`, а мы пытаемся обратиться к `a[5]` это уже не UB, как бы это было для рантайма, а ошибка компиляции - у нас в воздухе массив из 4 элементов, пятый там откуда?

Исходя из отсутствия памяти, данный список вполне понятен:
- `new` и `delete`
- Генерация исключений через `throw`
- Вызов не-`constexpr` функций
- Использование `goto`(запретили, так как control flow граф программы из дерева превразается в граф - а такое исполнить компилятору исключительно сложнее)
- Лямбда выражения (можно в `C++17`)
- Преобразования `const_cast` и `reinterpret_cast`(памяти нет, какие тогда биты нужно "реинтерпертировать"?)
- Преобразования `void *` в `object *`
- Модификация нелокальных объектов
- Неинициализированные данные
- Сравнение с `unspecified` результатом
- Вызов `type_id` для полиморфных классов и `dynamic_cast`
- Блоки `try` для обработки исключений
- Операции с `undefined behavior`(любое `constexpr`-функции является ошибкой компиляции)
- Инлайн ассемблер во всех разновидностях(как знать, а на каком ассемблере писать, если у нас разные архитектуры)
- Большая часть операций с `this`

Важно понимать, что в дальнейших стандартах список ослабляли, так что многие вещи теперь доступны.
Однако, этот список выглядит наиболее разумным, так как в дальнейшем его послабления происходят за счёт исключительно нетривиальных техник на грани с фокусами.

## Пример: целочисленный логарифм
```cpp
#include <climits>
#include <iostream>

constexpr size_t int_log(size_t N) {
  size_t pos = sizeof(size_t) * CHAR_BIT, mask = 0;

  // throw idiom
  if (N == 0)
    throw "N == 0 not supported";
  // since throw is not allowed
  // we will see compilation error here
  // so it's a way to tell about exceptions 
  // during compile time.
  // but this will work like that only if
  // there is an attempt to evaluate it at compile time
  // on other hand - it will throw at runtime
  
  // but if no throw (N != 0)- "if" here will work like 
  // "constexpr if", and code inside it will not be
  // instanciated 

  do {
    pos -= 1;
    mask = 1ull << pos;
  } while ((N & mask) != mask);

  if (N != mask)
    pos += 1;

  return pos;
}

int main() {
#if COMPILATION_ERROR_WITH_THROW_IDIOM_TRIGGER
  constexpr size_t log = int_log(0);
#endif
  
  std::cout << int_log(1024) << std::endl; 
  // 10
}
```

## Не всегда `constexpr`
Логичный вопрос: можно ли перегрузить функцию по `constexpr`, чтобы иметь и статический и нестатический вариант `int_log`?

И что теперь, нам теперь ещё писать перегрузку для рантайма отдельно?
Нет, `constexpr` - аннотация, а не часть типа, по нему нельзя перегрузить.

Оказывается, что в этом даже нет необходимости - `constexpr` функция может быть вызвана и на этапе компиляции и на этапе исполнения - никаких проблем, всё зависит от контекста её вызова, статический вариант уже может быть использован с неизвестными на этапе компиляции аргументом.
```cpp
std::cin >> x;

std::cout << int_log(x) << std::endl;
```

Поэтому `constexpr` и не входит в тип функции и не может аннотировать параметры.
## Обсуждение
Можем ли мы каким-то образом гарантировать, что `constexpr` функция выполнилась на этапе компиляции?
```cpp
int t = int_log(5);
```
Законных оснований надеяться на это у нас нет.
Компилятор не хочет считать, всё что можно отложить на этап исполнения - скорее всего будет отложено. Но мы можем его заставить, если надо.

Решение: использовать в `compile-time` контексте (положить в `constexpr` переменную, сделать размером массива, параметризовать шаблон):
```cpp
constexpr int logval = int_log(5);
int t = logval;
```
Вот теперь мы уверены, что вызов состоялся на этапе компиляции.

## `C++20`, введение `consteval` и `constinit`
Функции, помеченные `consteval` обязаны быть выполнены именно и конкретно на этапе компиляции:
```cpp
consteval int ctsqr(int n) {return n*n;}

constexpr int r = ctsqr(100); // OK

int x = 100;
int r2 = ctsqr(x); // Ошибка: не ct const
```

Для того, чтобы гарантировать только константную инициализацию `constexpr` наоборот слишком сильная гарантия и достаточно `constinit`:
```cpp
constinit int x = 1000;
// запрещено для локальных переменных
/*
	иначе сложно понять, что такое начальное значение 
	переменной на стеке, которое известно на этапе 
	компиляции.
	стека то нет никакого на этапе компиляции.
*/

++x; // ок
```
Т.е. значение такой переменной известно на этапе компиляции, но только её начальное значение.
## Не везде `constexpr`
Двойная природа `constexpr` функций имеет обратную сторону:
```cpp
template <typename T>
constexpr size_t ilist_sz (std::initializer_list<T> list){
	constexpr size_t init_sz = init.size();
	return init_sz;
}
```
И это ошибка.
Компилятор тут не может дать **гарантию** константности для переменной (хотя сама функция и `constexpr`).

Как вы думаете, измениться ли ситуация, если заменть `constexpr` на `consteval`?
Нет, не будет, так как проблема не в этом.
Дело в `std::initializer_list<T> list` - это не `core constant expression`, что и вызывает проблемы. Даже если будет подано в качестве параметра при вызове `constexpr` выражение, компилятор действует формально и смотрит на аргумент - он не `constexpr`(а аннотировать его мы не можем - мы же не можем аннотировать `constexpr`
аргументы).

А если убрать `constexpr` из определения `init_sz`?
На удивление, тогда всё будет работать:
```cpp
template <typename T>
consteval size_t ilist_sz (std::initializer_list<T> list){
	size_t init_sz = init.size();
	
	// и более того
	init_sz += 2; // ок
	return init_sz;
}
```
Т.е. тут мы получаем забавный парадокс - мы точно знаем, что `init_sz`- будет вычисленна на этапе компиляции, но аннотировать её таким образом мы её не можем!
И более того, в данном случае мы получили не просто переменную, а 
**не константную переменную времени компиляции**. 
## Обсудение
Статические `constexpr` метод в классе - это просто `constexpr` функция.

Имеют ли смысл нестатические `constexpr` методы в классах?
Имеют.

Дело в том, что у нас есть особый метод у класса - конструктор.
Секрет в том, что если сделать конструктор у класса `constexpr` - то у нас появится возможность создать данный объект на этапе компиляции.
Такие объекты - которые имеют репрезентацию на этапе компиляции - называются литералами. Часть из них нам хорошо знакома, остаётся разобраться, а как создать свои)

# Meta-OOP
## Пользовательские литеральные типы
Чтобы сделать пользовательский тип литеральным, ему нужен **`constexpr` конструктор**:
```cpp
struct Complex {
	constexpr Complex (double r, double i) : re(r), im(i) {}
	constexpr double real() const { return re; }
	constexpr double imag() const { return im;}
private:
	double re, im;
};

constexpr Complex c{0.0, 1.0}; // это литеральное значение
```
## Арифметика
Для таких объектов становится возможной арифметика времени компиляции:
```cpp
constexpr Complex Complex::operator+= (Complex rhs) {
	re += rhs.re, im += rhs.im;
	return *this;
}

constexpr Complex operator+ (Complex lhs, Complex rhs){
	lhs += rhs;
	return lhs;
}

// использование

constexpr Complex c {0.0, 1.0}, d {1.0, 2.0};
constexpr Complex e = c + d;
```
## Обсуждение
Литералы такого класса выглядят как `Complex{1.0, 2.0}`, но нам бы хотелось более привычной формы `1.0 + 1.0_i`.

Для сложения у нас есть выход, но как приделать суффикс?
Удивительно, но для этого мы тоже используем перегрузку очень специального оператора.
## Пользовательский суффикс
И этот оператор - это **оператор кавычки**.
```cpp
constexpr Complex Complex::operator+= (Complex rhs) {
	re += rhs.re, im += rhs.im;
	return *this;
}

constexpr Complex operator+ (Complex lhs, Complex rhs){
	lhs += rhs;
	return lhs;
}


constexpr Complex operator"" _i (long double arg){
	return Complex{0.0, arg};
}

// использование

constexpr Complex c {0.0, 1.0}, d {1.0, 2.0};
constexpr Complex e = c + d;

constexpr Complex a = 0.0 + 1.0_i; 
// ok, arg_i -> ""_i(arg) 
//     => Complex(0.0, 0.0) + Complex(0.0, 1.0);
```
Здесь суффикс определён с параметром типа `double`.

## Внезапная проблема
Допустим, хочется переопределить суффикс`_binary` для бинарных констант.

Но уже даже довольно маленькая константа:
`1010101010101_binary` не влазит в `unsigned long long` параметр.

Решение есть - синтаксис с вариабельным суффиксом:
```cpp
template <char... Chars>
constexpr unsigned long long operator "" _binary(){
	// и что тут?
}_
```
## Небольшая метапрограмма
```cpp
#include <iostream>

template <int Sum, char... Chars> struct binparser;

template <int Sum, char... Rest> struct binparser<Sum, '0', Rest...> {
  static constexpr int value = binparser<Sum * 2, Rest...>::value;
};

template <int Sum, char... Rest> struct binparser<Sum, '1', Rest...> {
  static constexpr int value = binparser<Sum * 2 + 1, Rest...>::value;
};

template <int Sum> struct binparser<Sum> {
  static constexpr int value = Sum;
};

template <char... Chars> constexpr int operator""_binary() {
  return binparser<0, Chars...>::value;
}

int main() {
  constexpr auto x = 1001_binary;
  std::cout << x << std::endl;
  // 9
}
```
Но это всё - привет из 98 года.
## Ладно, это была шутка
Тоже, но современное:
```cpp
template<char... Chars>
constexpr int operator""_binary() {
	std::array<int, sizeof...(Chars)> arr { Chars... };
	int sum = 0;
	for (auto c : arr)
		switch(c) {
			case '0': sum = sum * 2; break;
			case '1': sum = sum * 2 + 1; break;
			default : throw "Unexpected symbol";
		}
		
	return sum;
}
```
Но откуда в программе времени компиляции взялся `std::array`?
А ответ простой - у `std::array` тоже есть `constexpr` конструктор.

Когда люди поняли, что `constexpr` мало к чему обязывает, но при этом даёт новые возможности для времени компиляции, тут началось...

Более того, уже можно даже вот так:
```cpp
#include <iostream>
#include <vector>

template <char... Chars> constexpr int operator""_binary() {
  std::vector<char> arr{Chars...};
  int sum = 0;
  for (auto c : arr)
    switch (c) {
    case '0':
      sum = sum * 2;
      break;
    case '1':
      sum = sum * 2 + 1;
      break;
    default:
      throw "Unexpected symbol";
    }

  return sum;
}

int main() {
  constexpr auto x = 1001_binary;
  std::cout << x << std::endl;
}
```
## `Constexpr all the things!`
После их появления, `constexpr-ctors` начали торжественно расползаться по стандартной библиотеке.

Очевидно сразу появились `constexpr`-контейнеры `std::array` и `std::bitset`.

Точно также появились `constexpr`-алгоритмы.

Постепенно, таких контейнеро и алгоритмов становится (с некоторыми ограничениями) всё больше и больше.

Первоначально написание дуального кода было связано с некоторыми проблемами.
## Case study: замена `vector` на `array`
Да, можно оствить и вектор, но это, скажем, теперь можно.

Попробуем перейти от:
```cpp
template <typename T> class PermLoop {
	std::vector<T> loop_;
	....	
	
	PremLoop(std::initializer_list<T> ls) : 
		loop_(ls) {
			reroll(); // rearange to meet normal perm form
		}
};
```
К чему-то вроде (в таком виде работать не будет):
```cpp
template <typename T, size_t N> class PermLoop {
	std::array<T> loop_;
	....	
	constexpr PremLoop(std::initializer_list<T> ls) : 
		loop_(ls)
		// не будет работать 
		// у array нет инициализации из
		// initializer_list, это именно что 
		// нативный агрегат, но у него нет 
		// конструктора из initializer_list
		{
			reroll(); 
		}
};
```
И что с этим делать? 
Кажется, нам бы хотелось тогда поэлементно разобрать этот лист и уже так запихнуть всё в массив. И всё это на этапе компиляции.
Выход есть.
## Index sequences
Удивительно полезный класс `interget_sequence`;
```cpp
template <typename T, T... Ints>
class integet_sequence;
```
Его синоним, если нам нужны индексы:
```cpp
template <size_t... Ints>
using index_sequence = 
	std::integer_sequence<size_t, Ints...>;
```
Мы можем писать `std::make_index_sequence<3>`.
Типом этого выражения является `integer_sequence<size_t, 0, 1, 2>`.
Теперь у нас есть инструменты, чтобы подступиться к созданию `array`.
## Переход от вектора к массиву
```cpp
template <typename T, size_t N, size_t... Ns>
constexpr std::array<T, N>
make_array_impl(
	std::initializer_list<T> t,
	std::index_sequence<Ns...>){
	return std::array<T, N>{ *(t.begin() + Ns)...};
}

template <typename T, size_t N>
constexpr std::array<T, N>
make_array(std::initializer_list<T> t) {
	return make_array_impl<T, N>(
		t,
		std::make_index_sequence<N>{}
	);
}
```
На самом деле, подобный паттерн - некое действие к элементу во время свёртки - частый паттерн, когда речь заходит о коде программ, который работает с 3 и 4 квадрантом [[#Квадранты вычислений]].
## `C++20`: `constexpr` `vector` и `string`
Казалось бы, мучений с заменой на `array` больше не надо?
```cpp
struct S {
	std::vector<int> arr;
	constexpr S(std::initializer_list<int> il) : arr(il) {}
};
```

Интересно, что это УЖЕ работает, но всё ещё конечно интересно, а как оно работает? Отсылка к магистерскому курсу.
## Core constant expression...
Интересно, что идея того, что на самом деле компилятор считает известными на этапе компиляции не совсем те вещи, которые нативно попадают в эту категорию, а скорее он дейсвует формально - всё, что является `core constant expression` - то и известно.

Так что всё, что касается `constexpr`, полно сложных и странных сюрпризов.

Это в свою очередь приводит нас к таком вот:
```cpp
struct S {
	int n_;
	S(int n) : n_(n) {} // non-constexpr ctor
	constexpr int get() { return 42;}
};

int main (){
	S s{2};
	// это объект на стеке
	constexpr int k = s.get();
	// но по ряду формальных признаков, 
	// компилятор разрешает такой вызов и 
	// формально является правым
	
}
// это довольно хрупкий пример, его легко сломать
// продолжение этого разговора нужно искать в 
// углублённом курсе по стандарту
```
## Обсуждение 
Если у нас есть