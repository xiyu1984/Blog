# AMM of Uniswap
## Principle
### v1-v2
We all know that Uniswap use $x\cdot y=k$<sup>[1]</sup> to express their liquidity pools' relationship, which is really succinct and make sense that one may think that how to conclude out this.  

Let $x$ and $y$ denote the reserves of token X and token Y. An intuitive idea is that the price of the pair $X|Y$, which means how many tokens of $Y$ need to cost to get a token $X$, is determined by the size of their reserves.  

The pirce of pair $X|Y$ can be expressed as:  
$$p_{X}(x,y)=\frac{-dy}{dx}$$  
Note that there is a minus sign($-$) because the moving direction of token $Y$ and token $X$ is just the opposite.  
Next let's make a reasonable suppose:  
***Suppose 1**: Given the size of the adversary's reserves, the larger the size of the reserves the lower of the price.*   
And a simple expression of *Suppose 1* is:  
$$p_{X}(x,y)=\frac{y}{x}$$  
So we get:  
$$-\frac{dy}{dx}=\frac{y}{x}$$  
i.e.  
$$xdy+ydx=0$$  
This ordinary differential equation is solved as $x\cdot y=C$, where $C$ is an constant.  

Note that other expressions of $p_{X}(x,y)$ based on *Suppose 1* can be used to make AMMs and get similar effectives, e.g., $p_{X}(x,y)=\frac{y^2}{x^2}$. The key point is to solve the related ordinary differential equations.  

## Reference
[1] [white paper-v3](https://uniswap.org/whitepaper-v3.pdf)
