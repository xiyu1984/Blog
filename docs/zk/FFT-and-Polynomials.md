# Fast Fourier Transform and Polynomials

Polynomials have two representations, one is the coefficient, and the other is the point-value. DFT (Discrete Fourier Transform) and inverse DFT can transform the two representations.  

## Principle

### Intuitive Explanation

Suppose a polynomial $A(x)=\sum_{j=0}^{n-1}{a_j\cdot x^j}$. The $\vec{a}=\{a_0, a_2, ..., a_{n-1}\}$ is the `coefficients vector`.  
Construct a matrix:  

$$
M=
\left[\begin {array}{cc}
x_{0}^0&x_{0}^1&...&x_{0}^{n-1}\\
x_{1}^0&x_{1}^1&...&x_{1}^{n-1}\\
\vdots&\vdots&\ddots&\vdots\\
x_{n-1}^0&x_{n-1}^1&...&x_{n-1}^{n-1}
\end{array}\right]
$$

Let $\{x_0, A(x_0)\}, \{x_1, A(x_1)\}, ..., \{x_{n-1}, A(x_{n-1})\}$ be the $n$ points on polynomial $A(x)$, and we have:  

$$
\vec{A_{x_0, x_1, ..., x_{n-1}}} = 
\left[\begin {array}{cc}
A(x_0)\\
A(x_1)\\
\vdots\\
A(x_{n-1})
\end{array}\right] =
\left[\begin {array}{cc}
x_{0}^0&x_{0}^2&...&x_{0}^{n-1}\\
x_{1}^0&x_{1}^2&...&x_{1}^{n-1}\\
\vdots&\vdots&\ddots&\vdots\\
x_{n-1}^0&x_{n-1}^2&...&x_{n-1}^{n-1}
\end{array}\right] \times 
\left[\begin {array}{cc}
a_0\\
a_1\\
\vdots\\
a_{n-1}
\end{array}\right] = 
M \times \vec{a}
$$

As a result, matrix $M$ makes a transformation from coefficients vector $\vec{a}$ to the point-value vector $\vec{A_{x_0, x_1, ..., x_{n-1}}}$.  

Now, if we choose a proper points vector $\vec{x}$ to let there exist an inverse matrix of $M$, we can get:  

$$
M^{-1} \times 
\left[\begin {array}{cc}
A(x_0)\\
A(x_1)\\
\vdots\\
A(x_{n-1})
\end{array}\right] = 
M^{-1} \times M \times \vec{a} = \vec{a}
$$

As a result, matrix $M^{-1}$ makes a transformation from the values vector $\vec{A_{x_0, x_1, ..., x_{n-1}}}$ to the coefficients vector $\vec{a}$.  

### DFT (Discrete Fourier Transform)

A proper points vector $\vec{x}$ is $\{\omega_n^0, \omega_n^1, ..., \omega_n^{n-1}\}$, where $\omega_n=e^{-2\pi i / n} = \cos(\frac{-2\pi}{n})+i\cdot\sin(\frac{-2\pi}{n})$. Conventionally, we will subsequently use the $F_n$ to denote the matrix:  

$$
F_n = \frac{1}{\sqrt{n}}
\left[\begin {array}{cc}
(\omega_{n}^0)^0&(\omega_{n}^0)^1&...&(\omega_{n}^0)^{n-1}\\
(\omega_{n}^1)^0&(\omega_{n}^1)^1&...&(\omega_{n}^1)^{n-1}\\
\vdots&\vdots&\ddots&\vdots\\
(\omega_{n}^{n-1})^0&(\omega_{n}^{n-1})^1&...&(\omega_{n}^{n-1})^{n-1}
\end{array}\right]
= \frac{1}{\sqrt{n}}
\left[\begin {array}{cc}
\omega_{n}^0&\omega_{n}^0&...&\omega_{n}^0\\
\omega_{n}^0&\omega_{n}^1&...&\omega_{n}^{n-1}\\
\vdots&\vdots&\ddots&\vdots\\
\omega_{n}^0&\omega_{n}^{n-1}&...&\omega_{n}^{(n-1)^2}
\end{array}\right]
$$

We can prove that:  

$$
F_n^{-1} = \frac{1}{\sqrt{n}}
\left[\begin {array}{cc}
\omega_{n}^0&\omega_{n}^0&...&\omega_{n}^0\\
\omega_{n}^0&\omega_{n}^{-1}&...&\omega_{n}^{-(n-1)}\\
\vdots&\vdots&\ddots&\vdots\\
\omega_{n}^0&\omega_{n}^{-(n-1)}&...&\omega_{n}^{-(n-1)^2}
\end{array}\right]
= \bar{F_n}
$$

Acturally, $F_n$ is an [Unitary](https://en.wikipedia.org/wiki/Unitary_matrix), whichs is the complex form of an [Orthogonal matrix](https://en.wikipedia.org/wiki/Orthogonal_matrix). The elements in $F_n^{-1}$ is the [complex conjugate](https://en.wikipedia.org/wiki/Complex_conjugate) of the elements in $F_n$.  

Discrete Fourier Transform (DFT) use $F_n$ and $F_n^{-1}=\bar{F_n}$ to make transformations between point-value and coefficients of a polynomial.  

### FFT (Fast Fourier Transform)

The DFT and IDFT can be computed in time $\Theta(n^2)$. FFT can reduce it to $\Theta(n\log{n})$.  

Suppose $n=2^l$, we have:  

$$\omega_n^2=(e^{-2\pi i / n})^2=e^{-2\pi i / \frac{n}{2}}=\omega_{\frac{n}{2}}$$  

For $A(x)$, we can represent is as follows:  

$$A(x)=\sum_{j=0}^{n-1}{a_j\cdot x^j}=(a_0\cdot x^0+a_2\cdot x^2+...+a_{n-2}\cdot x^{n-2})+x\cdot(a_1\cdot x^0+a_3\cdot x^2+...+a_{n-1}\cdot x^{n-2})$$  

Let  

$$
\left\{\begin {array}{cc}
A_0(x^2)=a_0\cdot x^0+a_2\cdot x^2+...+a_{n-2}\cdot x^{n-2} \\
A_1(x^2)=a_1\cdot x^0+a_3\cdot x^2+...+a_{n-1}\cdot x^{n-2}
\end{array}\right.
$$  

we have  

$$A(x)=A_0(x^2)+x\cdot A_1(x^2)$$  

and substitute the $n^{th}$ root of unity $\omega_n$ into $A_0(x^2)$ and $A_1(x^2)$, we get  

$$
\left\{\begin {array}{cc}
A_0(\omega_n^2)=A_0(\omega_{\frac{n}{2}})=a_0\cdot \omega_{\frac{n}{2}}^0+a_2\cdot \omega_{\frac{n}{2}}^1+...+a_{n-2}\cdot \omega_{\frac{n}{2}}^{\frac{n}{2}-1}\\
A_1(\omega_n^2)=A_1(\omega_{\frac{n}{2}})=a_1\cdot \omega_{\frac{n}{2}}^0+a_3\cdot \omega_{\frac{n}{2}}^1+...+a_{n-1}\cdot \omega_{\frac{n}{2}}^{\frac{n}{2}-1}
\end{array}\right.
$$  

and  

$$A(\omega_n^k)=A_0(\omega_{\frac{n}{2}}^{k\mod\frac{n}{2}})+\omega_n^k\cdot A_1(\omega_{\frac{n}{2}}^{k\mod\frac{n}{2}}), \space \text{where}\space k\in\{0,1,2,...,n-1\}$$  

Now the calculation of $DFT_n$ is reduced to two calculations of $DFT_{\frac{n}{2}}$ along with $2n-1$ other operations including $n$ operation $+$ and $n-1$ operation $\times$.  
