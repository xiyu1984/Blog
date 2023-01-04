# AMM of Curve
Curve<sup>[1]</sup> is an AMM for stable coins.  

## Principle
### Curve V1
We mentioned the price determined by AMM can be described as $p_{X}(x,y)=\frac{-dy}{dx}$ in [Uniswap explanation](uniswap%20explanation.md). Intuitively, the price in stable coin swap should be stable too, that is $P_{X}(x,y)=\text{const}$. So we have $\frac{-dy}{dx}=C_0$, which can be solved as $x+y=C\space\text{where}\space C\space\text{is constant}$.  
This is exactly where `Curve` starts, but as the pool of coin $X$ and coin $Y$ id not infinite, there needs to be some adaptability in the engineering implementation.  
In general, the fomula used for swap in `Curve` is as follow:  
$$\lambda D^{n-1}\sum{x_i}+\prod{x_i}=\lambda D^n+(\frac{D}{n})^n$$  
It seems to be puzzling, but it's actual succinct and ingenous. I will explain it step by step.  

### Curve V2
Curve V2 can be used for swaps of unstable coins.  

The initial $D$ is equal to the summation of the transformed amount of all reserves, $D=Nx_{eq}$. As the equation of `V2` is derived from `V1`, the amount of each reserve is transformed by a coefficient so that they act as a *1:1:1:...* stable coins portfolio.   

$$K_0=\frac{\prod{x_{i}N^{N}}}{D^{N}}$$  

$$K=AK_{0}\frac{\gamma^{2}}{(\gamma+1-K_-0)^2}$$
where $A$(amplification coefficient as in `V1`) and $\gamma$(meaning distance of two constant-product curves) is artificially specified coefficients.  

The equation is as below:  
$$F(\mathbf{x}, D)=K(\mathbf{x}, D)D^{N-1}\sum{x_i}+\prod{x_i}-K(\mathbf{x}, D)D^N-(\frac{D}{N})^N$$  

It seems the equation if `V2` just substitues the $\lambda$ in `V1` with $K(\mathbf{x}, D)$, but $D$ is treated as a variable in this function.  

#### Swap
When a swap happens, $D$ needs to be calculated iteratively by Newton's method first, that is, $D_{k+1}=D_k-\frac{F(\mathbf{x}, D_k)}{F'_{D}(\mathbf{x}, D_k)}$. And then calculate the $x_i$ in the swap with the same method.  

#### Reflections
* According to the defination of $D$, it is not independent to $\mathbf{x}$ as it is actually the the summation of $\mathbf{x}$. But $D$ is used as a independent variable after initialized, in which way $D$ is more like a free coefficient.  




## Reference
[1] [white paper](https://classic.curve.fi/files/stableswap-paper.pdf)  