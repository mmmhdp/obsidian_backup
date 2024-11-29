# <mark class="hltr-coler"> Базовые элементарные функции</mark>
## <mark class="hltr-coler">Степенная</mark>
$$
f(x) = x^\alpha, \alpha \in \mathbb{R}
$$
### <mark class="hltr-yellow">Замечание</mark>
$$
\begin{align}
\textit{Пусть }\alpha > 0,\; \textit{тогда } x^\alpha \Rightarrow x\geq 0 \\
\textit{Пусть }\alpha < 0,\; \textit{тогда } x^\alpha \Rightarrow x > 0 
\end{align}
$$
## <mark class="hltr-coler">Показательная</mark>
$$
\begin{align}
f(x)= a^{x}\\
a>0,\; a\neq 1
\end{align}
$$
### <mark class="hltr-yellow">Замечание 1</mark>
$$
\begin{align}
\textit{Пусть } x>0,   \textit{тогда } a^x \text{определена для } \forall a\geq 0\\
\textit{Пусть } x\leq0,\textit{тогда } a^x \text{определена для } \forall a > 0
\end{align}
$$
### <mark class="hltr-yellow">Замечание 2</mark>
$0^0$ - не определён
## <mark class="hltr-coler">Логарифмическая</mark>
$$
\begin{align}
f(x) = \log_{a} x \\
a>0,a\neq 1, x>0 
\end{align}
$$
### <mark class="hltr-yellow"> Свойства логарифмов</mark>
$$
\begin{align}
\log_{a}1 = 0 \\
\log_{a}a = 1 \\
\log_{a}a^b = b \\
\log_{a} (bc) = \log_{a} (|b|) + \log_{a} (|c|), (bc)>0 \\
a^{\log_{a}bc} = bc \\ 
\log_{a} \left(\frac{b}{c}\right)= \log_{a} (|b|)- \log_{a} (|c|)),\left(\frac{b}{c}\right)>0  \\
\log_{a^n} b^m = \frac{m}{n}\log_{a} |b|, \text{с условием}: \\
\text{при } m \text{ нечетном } \Rightarrow b>0\\
\text{при } m \text{ четном } b\neq 0 \\
\log_{a}b  = \frac{\log_c{b}}{\log_{c}{a}}, c>0, c\neq1 \\
a^{\log_{c}b} = b^{\log_{c}a}
\end{align}
$$

# <mark class="hltr-coler">Тригонометрические и обратные к ним</mark>

$$
\begin{align}
\arcsin{x}+\arccos{x} = \frac{\pi}{2}, \forall x\in [-1;1]\\
\text{arctg }x + \text{arcctg }x = \frac{\pi}{2}, \forall x\in \mathbb{R}
\end{align}
$$
# <mark class="hltr-coler">Элементарные функции</mark>
## <mark class="hltr-yellow">Определение</mark>
Любые функции, полученные из основных элементарных функций с помощью
арифметических операций ($\pm,\cdot$) и операции композиции:
$$
\begin{align}
f(x),g(x) \to f(x)\pm g(x),\;f(x)g(x),\; \frac{f(x)}{g(x)}:g(x)\neq 0\\
f(g(x)) = f\circ g(x)
\end{align}
$$
## <mark class="hltr-yellow">Свойство</mark>
Все элементарные функции непрерывны на своей области определения.
