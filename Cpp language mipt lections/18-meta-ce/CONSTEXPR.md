Программы времени компиляции и метапрограммы.
# Идея "рекурсивного" раскрытия
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
# Обсуждение
На самом деле никакой рекурсии тут нет:
цепочка инстанцирований порождает разные инстанцирования.

Но сама схожесть процессов наводит на некоторые мысли...

Первым, кого она навела на мысли был Эрвин Анрух в далёком 1994-ом.
Без малого, эти мысли **обусловили успех `C++` как языка!**
# Открытие метапрограммирования
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
# Факториал
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
# Числа Фибоначчи
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
# Две модели вычислений
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
# Целочисленный квадратный корень: рантайм вычисление
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
## Условный тип
Вспомним уже известный нам условный тип [[Итераторы#Интерлюдия conditional_type]].
Это отображение из `{true, false} -> {T, F}`
# Целочисленный квадратный корень: компайл тайм вычисление
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
# Квадранты вычислений
- `Runtime computations`
- `Compile-time computations`
- `Type-level computations`
```cpp
template <typename T>
struct add_const_pointer {
	using type = const T*;
};

using types = mpl::vector<int, char, float, void>;
using pointers = mpl::transform<types,
	ass_const_pointer<mpl::_1>>::type;
```
- `Heterogenious computations`
```cpp
auto to_string = [](auto t) {
	std::stringstream ss; ss << t; return ss.str();
};

fusion::vector<int, std::string, float> seq {
	1, "abc", 3.4f
};

auto strings = fusion::transform(seq, to_string);
```
# Обсуждение
