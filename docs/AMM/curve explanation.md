# AMM of Curve
Curve<sup>[1]</sup> is an AMM for stable coins.  

## Principle
We mentioned the price determined by AMM can be described as $p_{X}(x,y)=\frac{-dy}{dx}$ in [Uniswap explanation](uniswap%20explanation.md). Intuitively, the price in stable coin swap should be stable too, that is $P_{X}(x,y)=\text{const}$. So we have $\frac{-dy}{dx}=C_0$, which can be solved as $x+y=C\space\text{where}\space C\space\text{is constant}$.  
This is exactly where `Curve` starts, but as the pool of coin $X$ and coin $Y$ id not infinite, there needs to be some adaptability in the engineering implementation.  
In general, the fomula used for swap in `Curve` is as follow:  
$$\lambda D^{n-1}\sum{x_i}+\prod{x_i}=\lambda D^n+(\frac{D}{n})^n$$  
It seems to be puzzling, but it's actual succinct and ingenous. I will explain it step by step.  


## Reference
[1] [white paper](https://classic.curve.fi/files/stableswap-paper.pdf)  