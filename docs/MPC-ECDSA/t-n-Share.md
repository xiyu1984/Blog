# t-n Sharing

## Introduction
From a high level, `t-n-sharing` is for n participants sharing a secrete $s$. Arbitrary $t+1(t < n)$ participants can work together to recover the secret, but with negligible probability for any $t$ participants.  

## Priciple
### [Lagrange interpolation theorem](https://en.wikipedia.org/wiki/Lagrange_polynomial)  
Given a $t+1$ nodes: $\{(\xi_0, f(\xi_0)), (\xi_1, f(\xi_1)),...,(\xi_t, f(\xi_t)); \xi_i \neq \xi_j(i \neq j)\}$, we can uniquely define a polynomial with degree $\leq t$ by:    

$$\mathcal{l}_ j(\xi)=\prod_{\begin{array}{cl} 0\leq m \leq t\\
m\neq j \end{array}}^{}{\frac{\xi-\xi_m}{\xi_j-\xi_m}}$$  

So the polynomial about $\xi$ is as follow:

$$f(\xi)=\sum_{j=0}^{t}{f(\xi_j)\mathcal{l}_j(\xi)}$$  
 
### [Shamir’s scheme](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)  
Shamir’s scheme for sharing secret is based on [Lagrange interpolation theorem](#lagrange-interpolation-theorem).  
Let $\{(1,f(1)), (2,f(2)),..., (t,f(t)), (t+1,f(t+1),...,(n,f(n))\}$ be the n nodes owned by $n$ participants, in which $f(j)$ are only known by participant $\mathcal{P}_j$.  
A polynomial with degree $t$ has the following form:  

$$f(\xi)=\sum_{j=0}^{t}{a_j\cdot\xi^j}$$

and $f(0)=a_0$.  
Also, suppose we select $t+1$ nodes $N_{s} = \{\xi_i; i\in [0, t]\}$, so we have:  

$$f(0)=\sum_{j=0}^{t}{f(\xi_j)\mathcal{l}_j(0)}$$  

and  

$$\mathcal{l}_ j(0)=\prod_ {\begin{array}{cl} \xi_m\in N_{s} \\
m\neq j \end{array}}{\frac{0-\xi_m}{\xi_j-\xi_m}}=\prod_ {\begin{array}{cl} \xi_m\in N_{s} \\
m\neq j \end{array}}{\frac{\xi_m}{\xi_m-\xi_j}}$$

so with arbitrary $t+1$ out of $n$ participants, we can get: 

$$a_0=\sum_{j=0}^{t}{f(\xi_j)\mathcal{l}_j(0)}=\sum_{j=0}^{t}{[f(\xi_j)\cdot\prod_{\begin{array}{cl} \xi_m\in N_{s}\\
m\neq j \end{array}}{\frac{\xi_m}{\xi_m-\xi_j}}]}$$  

We can let $a_0$, the coefficient of $\xi^0$, in a t-degree polynomial be the secret, and let  

$$x_j=f(j)\cdot\prod_{\begin{array}{cl} \xi_m\in N_{s}\\
m\neq j \end{array}}{\frac{\xi_m}{\xi_m-\xi_j}}$$  

be the secret slice of $\mathcal{P}_ j$, and let 

$$x=a_{0}=\sum_{j=0}^{t}{x_{j}}$$

be the global secret. Every participant only knows their own secret slice and any $t$ participants cannot recover the global secret.  

## Application to MPC-ECDSA  
For building a ECDSA signature, things are the same as [First Model](./First%20Model.md) except that $\mathcal{P}_ j$ he uses $x_ j$ as his secret slice. So the global secret is 

$$x=\sum_{i=0}^{t}{x_i}=a_{0}$$    

Besides, an auxiliary information $\{v_j=g^{a_j}\in\mathbb{G};j\in[0,t]\}$ is published, where $a_j$ is the coefficient of $\xi^j$ of a t-degree polynomial $f(\xi)$.  
$\mathcal{P}_ j$ verifies 

$$g^{f(j)}=?\prod_{i=0}^{t}{v_i^{j^i}}=\prod_{i=0}^{t}{g^{j^i \cdot a_i}}$$

in $\mathbb{G}$ to check that whether his share $f(j)$ is valid on the polynomial.  

The rest of the are just the same as [First Model](./First%20Model.md). As the summation of arbitrary $t+1$ shares of secret slices is equal to $a_0$, what we only need to verify the signature is a unique global public key.  
