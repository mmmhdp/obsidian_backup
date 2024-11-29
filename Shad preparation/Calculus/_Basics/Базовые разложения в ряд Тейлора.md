# <mark class="hltr-coler">Ряд Тейлора</mark> 
Разложение в общем виде разложение функции в ряд Тейлора вокруг точки $x =a$:
$$
\sum_{n=0}^\infty \frac{f^{(n)}(a)}{n!}(x-a)^n
$$
# <mark class="hltr-coler">Ряд Маклорена</mark>
Часто необходимо рассмотреть разложение вокруг $x=0$. 
Такое разложение называется рядом Маклорена для функции $f$.
Полезно помнить некоторые разложение в данный ряд:
$$
e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \dots + \frac{x^n}{n!} + \dots = \sum_{n=0}^\infty \frac{x^n}{n!}, \quad |x| < \infty \\
$$
$$
\sin x = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \dots + (-1)^n \frac{x^{2n+1}}{(2n+1)!} + \dots = \sum_{n=0}^\infty (-1)^n \frac{x^{2n+1}}{(2n+1)!}, \quad |x| < \infty \\
$$
$$
\cos x  = 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \dots + (-1)^n \frac{x^{2n}}{(2n)!} + \dots = \sum_{n=0}^\infty (-1)^n \frac{x^{2n}}{(2n)!}, \quad |x| < \infty \\
$$
$$
\ln(1 + x) = x - \frac{x^2}{2} + \frac{x^3}{3} - \dots + (-1)^{n+1} \frac{x^n}{n} + \dots = \sum_{n=1}^\infty (-1)^{n+1} \frac{x^n}{n}, \quad x \in (-1, 1] \\
$$
$$
\begin{align}
(1 + x)^\alpha = 1 + \alpha x + \frac{\alpha (\alpha - 1) x^2}{2!} + \frac{\alpha (\alpha - 1)(\alpha - 2) x^3}{3!} + \dots \\
= \sum_{n=0}^\infty \frac{\alpha (\alpha - 1)(\alpha - 2) \dots (\alpha - n + 1) x^n}{n!}, \quad |x| < 1 \\
\end{align}
$$
$$
\frac{1}{1 - x} = 1 + x + x^2 + \dots + x^n + \dots = \sum_{n=0}^\infty x^n, \quad |x| < 1 \\
$$
$$
\text{arctg } x = x - \frac{x^3}{3} + \frac{x^5}{5} - \frac{x^7}{7} + \dots + (-1)^n \frac{x^{2n+1}}{2n+1} + \dots = \sum_{n=0}^\infty (-1)^n \frac{x^{2n+1}}{2n+1}, \quad |x| \leq 1
$$
