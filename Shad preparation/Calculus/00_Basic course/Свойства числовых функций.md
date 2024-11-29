# <mark class="hltr-coler">Монотонность</mark>

Функция $f(x)$ <mark class="hltr-yellow">возрастает</mark> на $(a,b)$, если:
$$
\forall x_1,x_2\in(a,b): x_1>x_2 \Rightarrow f(x_1) > f(x_2)
$$
Для <mark class="hltr-yellow">убывающей</mark> сменим знаки.
Если знак строгий - <mark class="hltr-yellow">убывание строгое</mark>, верно и обратное.

## <mark class="hltr-yellow">Применение монотонности к решение уравненией и неравенств</mark>
### <mark class="hltr-yellow">П(о количестве корней)</mark>
Пусть $f\uparrow,g\downarrow$ на $(a,b)$.
Тогда уравнение $f(x) = g(x)$ имеет на $(a,b)$ не более 1 корня.

### <mark class="hltr-yellow">П(о неравенствах)</mark>
Пусть $f\uparrow,g\downarrow$ на $(a,b), f(x_0) = g(x_0)$, тогда:
$$
\begin{align}
x > x_0 : f(x) > g(x) \\
x < x_0 : f(x) < g(x) \\
\end{align}
$$
# <mark class="hltr-coler">Чётность</mark>

Функция $f(x)$ <mark class="hltr-yellow">чётная</mark>, если:
$$
\forall x \in D(f) : f(-x) = f(x)
$$
Функция $f(x)$ <mark class="hltr-yellow">НЕчётная</mark>, если:
$$
\forall x \in D(f) : f(-x) = -f(x)
$$
Иначе - назовём функцией <mark class="hltr-yellow">общего вида</mark>.
## <mark class="hltr-yellow">Следствие 1</mark>
![[Pasted image 20241128121131.png]]
## <mark class="hltr-yellow">Следствие 2</mark>
Пусть $0 \in D(f) \wedge f(x) -$ нечётная $\Rightarrow f(0) = 0$

## <mark class="hltr-yellow">Cледствие(о представлении любой функции)</mark>
Любую функцию всегда можно представить в виде суммы некоторой чётной и нечётной функции:
$$
\begin{align}
\begin{cases}
f(x) = \underbrace{u(x)}_{чётная} + \underbrace{v(x)}_{нечётная} \\
f(-x) = u(-x) + v(-x)
\end{cases} \Rightarrow
\begin{cases}
u(x) = \frac{f(x) + f(-x)}{2}\\
v(x) = \frac{f(x) - f(-x)}{2}
\end{cases}
\end{align}
$$

# <mark class="hltr-coler">Переодичность</mark>

Функция $f(x)$ - <mark class="hltr-yellow">переодическая</mark>, если:
$$
\exists T : \forall x\in D(f), x+T \in D(f): f(x) = f(x+T) 
$$
## <mark class="hltr-yellow">Л(о наименьшем периоде)</mark>
Пусть $T_0>0$ - наименьший период $y= f(x)$. Тогда любой другой период имеет вид:
$$
T = nT_0, \; n\in \mathbb{N}
$$
# <mark class="hltr-coler">Выпуклость</mark>
Функция $y=f(x)$ - <mark class="hltr-yellow">выпуклая вверх</mark> на $[a,b]$, если:
$$
\begin{align}
\forall x_1,x_2\in [a,b], \forall \alpha_1, \alpha_2\geq 0 :\alpha_1 + \alpha_2 = 1:\\
f(\alpha_1x_1 + \alpha_2x_2) > f(\alpha_1x_1) + (\alpha_2x_2)
\end{align}
$$
Получено через параметризацию отрезка $[a,b]$:
$$
\begin{align}
\alpha = \alpha_1 \\
\alpha_2 = 1 - \alpha \\
x =  \alpha x_1 + (1-\alpha)x_2, \; 0\leq\alpha\leq 1
\end{align}
$$
Для <mark class="hltr-yellow">выпуклости вниз</mark> - противоположный знак.
<mark class="hltr-yellow">Нестрогая выпуклость</mark> - нестрогий знак.

## <mark class="hltr-yellow">Замечание</mark>
Для непрерывной функции достаточно:
$$
f(\frac{x_1 + x_2}{2}) < \frac{f(x_1) + f(x_2)}{2}
$$
Т.е. $\alpha = \frac{1}{2}$
## <mark class="hltr-yellow">Неравенство Йенсена</mark>

Пусть $y = f(x)$ <mark class="hltr-yellow">выпуклая вниз</mark> на $(a,b)$, тогда:
$$
\begin{align}
\forall x_1,\ldots,x_n \in(a,b):\\
\forall \alpha_1,\ldots,\alpha_n, \sum_{i=1}^n \alpha_i = 1:\\
f(\alpha_1x_1 + \ldots + \alpha_nx_n) \leq \sum_{i=1}^n f(\alpha_ix_i)
\end{align}
$$
Для выпуклой вниз - знак противоположный.
Знак сторогий тогда, когда все $x_i$ различны между собой.
# <mark class="hltr-coler">Монотонность композиции</mark>
![[Pasted image 20241128124431.png]]

 