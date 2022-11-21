# AMM of Uniswap
## Principle
### v1-v2
We all know that Uniswap use $x\cdot y=k$<sup>[1]</sup> to express their liquidity pools' relationship, which is really succinct and make sense that one may think that how to conclude out this.  

Let $x$ and $y$ denote the reserves of token X and token Y. An intuitive idea is that the price of the pair $X|Y$, which means how many tokens of $Y$ need to cost to get a token $X$, is determined by the size of their reserves.  

The pirce of pair $X|Y$ can be expressed as:  
$$p_{X}(x,y)=\frac{-dy}{dx}$$  
Note that there is a minus sign($-$) because the moving direction of token $Y$ and token $X$ is just the opposite.  
Next let's make a reasonable assumption:  
***Assumption 1**: Given the size of the adversary's reserves, the larger the size of the reserves the lower of the price.*   
And a simple expression of *Assumption 1* is:  
$$p_{X}(x,y)=\frac{y}{x}$$  
So we get:  
$$-\frac{dy}{dx}=\frac{y}{x}$$  
i.e.  
$$xdy+ydx=0$$  
This ordinary differential equation is solved as $x\cdot y=C$, where $C$ is an constant.  

Note that other expressions of $p_{X}(x,y)$ based on *Assumption 1* can be used to make AMMs and get similar effectives, e.g., $p_{X}(x,y)=\frac{y^2}{x^2}$. **The key point is to solve the related ordinary differential equations**.  

### v3
The most important improvement of v3 is described as *concentrated liquidity*, that is, one can provide a LP with an restricted price range. This can be used to avoid very steep parts of the curve.  

Given current reserves $x_{c}$ and $y_{c}$, we have $k=x_{c}\cdot y_{c}$.  
With $k$ as a constant, once the price $p_{X}(x,y)$ is limited in a range $[p_{X}^{min}, p_{X}^{max}]$, the size of reserves $x$ and $y$ are determined:  
$$\left\{ \begin {array}{cc}
p_{X}^{min}=\frac{y_{min}}{x_{max}} & \text{where} & x_{max}\cdot y_{min}=k \\
p_{X}^{max}=\frac{y_{max}}{x_{min}} & \text{where} & x_{min}\cdot y_{max}=k 
\end {array}\right.
$$  
we can get $x_{min}=\sqrt{\frac{k}{p_{X}^{max}}}$, and $y_{min}=\sqrt{k\cdot p_{X}^{min}}$. We can also get $x_{max}$ and $y_{max}$, but they may not be so important.  

As a result, uniswap v3 changes the equation<sup>[1]</sup> to:  
$$(x+\frac{L}{\sqrt{p_X^{max}}})(y+L\cdot\sqrt{p_X^{min}})=L^2 \space\space\space\space \text{where} \space\space\space\space L^2=k$$  
which is a movation of the curve $x\cdot y=k$.  

Besides, given current reserves $x_{c}$ and $y_{c}$, one can calculate the amount of liquidity he needs to provide.

## Reference
[1] [white paper-v3](https://uniswap.org/whitepaper-v3.pdf)
