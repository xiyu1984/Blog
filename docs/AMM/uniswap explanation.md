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

#### Accumulator and on-chain Oracle<sup>[2]</sup>
A price accumulator at time $t$ is defined as:  

$$a_t=\sum_{i=1}^{t}p_i$$  

The average price from time $t_1$ to $t_2$ is:  

$$p_{t_1, t_2}=\frac{a_{t_2}-a_{t_1}}{t_2-t_1}=\frac{\sum_{i=t_1}^{t_2}p_i}{t_2-t_1}$$  

The average prices based on the accumulation are used for other derivatives.  
Note that the mean prices from time $t_1$ to $t_2$ in a token pair are not the same, that is, the mean price of $X|Y$ is different from the mean price $Y|X$.  

#### Precision
`UQ112.112` format expresses the floating point number in a range of $[0, 2^{112}-1]$ with a precision of $\frac{1}{2^{112}}$.  

#### Trading Fees
In Uniswap v2, traders will continue to pay a 0.30% fee on all trades, $\frac{1}{6}$ of which, that is 0.05%, is paid for the protocol itself(the devopement team), and the rest is paid for liquidity providers.  
Everything starts from *Formula 1*: $x\cdot y=k$.  
Without trading fees, we can calculate $\Delta{y}$  given $x, y \text{ and } \Delta{x}$ by *Formula 2* :  
$$(x+\Delta{x})(y-\Delta{y})=k$$  
Consider trading fee is brought in, *Formula 2* changes into *Formula 3*:  
$$(x+\theta \Delta{x})(y-\Delta{y})=k\text{ where}\space \theta\space\text{is the rate}$$  
In v2 all $\Delta{x}$ will be in the liquidity pool until the provider withdrawing it, so the $k$ will change with every trading. That is $k'=(x+\Delta{x})(y-\Delta{y})$ will slightly larger than $k$ before a trading. The following *Formula 4* descripes the growth of $\sqrt{k}$ from time $t_1$ to $t_2$:  
$$f_{1,2}=1-\frac{\sqrt{k_1}}{\sqrt{k_2}}=\frac{\sqrt{k_2}-\sqrt{k_1}}{\sqrt{k_2}}$$  
Note that the denominator is $\sqrt{k_2}$, which means the growth is measured after the trading is succeded.  
New liquidity tokens with $\phi\cdot f_{1,2}$ inflation will be mint to the protocol, and the amount of the new minted tokens is solved by the following equation:  
$$\frac{s_m}{s_m+s_1}=\phi\cdot f_{1,2}=\phi\frac{\sqrt{k_2}-\sqrt{k_1}}{\sqrt{k_2}}$$  
where $s_1$ is the total liquidity tokens' quantity of outstanding shares at time $t_1$.  
$s_m$ is solved as:  
$$s_m=\frac{\sqrt{k_2}-\sqrt{k_1}}{(\frac{1}{\phi}-1)\cdot\sqrt{k_2}+\sqrt{k_1}}\cdot s_1$$  
As mentioned above, $\phi=\frac{1}{6}$ in v2. The liquidity token is `UNI` in Uniswap v2.  
In the white paper of Uniswap v2<sup>[2]</sup>, there's an example, in which $s_1$ is the total shares a liquidity pool receives when a liquedity provider deposites a trading pair. The calculation happens only once when the pool is withdrawed, then the provider gets the shares for providing the liquidity pool, and the protocol gets the "inflated" shares created every time a trading happens.  

### v3
The most important improvement of v3 is described as *concentrated liquidity*, that is, one can provide a LP with an restricted price range. This can be used to avoid very steep parts of the curve.  

Given current reserves $x_{c}$ and $y_{c}$, we have $k=x_{c}\cdot y_{c}$.  
With $k$ as a constant, once the price $p_{X}(x,y)$ is limited in a range $[p_{X}^{min}, p_{X}^{max}]$, the size of reserves $x$ and $y$ are determined:  

$$\left\{ 
\begin {array}{cc}
p_{X}^{min}=\frac{y_{min}}{x_{max}}&\text{where}&x_{max}\cdot y_{min}=k\\
p_{X}^{max}=\frac{y_{max}}{x_{min}}&\text{where}&x_{min}\cdot y_{max}=k
\end{array}
\right.$$

we can get $x_{min}=\sqrt{\frac{k}{p_{X}^{max}}}$, and $y_{min}=\sqrt{k\cdot p_{X}^{min}}$. We can also get $x_{max}$ and $y_{max}$, but they may not be so important.  

As a result, uniswap v3 changes the equation<sup>[1]</sup> to:  
$$(x+\frac{L}{\sqrt{p_X^{max}}})(y+L\cdot\sqrt{p_X^{min}})=L^2 \space\space\space\space \text{where} \space\space\space\space L^2=k$$  
which is a movation of the curve $x\cdot y=k$.  

Besides, given current reserves $x_{c}$ and $y_{c}$, one can calculate the amount of liquidity he needs to provide according to his price range.  

#### Geometric Mean Price Oracle
Besides the arithmetic mean of the prices based on liner accumulator in v2, v3 provides another accumulator based on the geometric mean, in which the mean prices of $X|Y$ and $Y|X$ are the same.  
Take `1.0001` as the base of the logarithm, the accumulated price at time $t$ is:  
$$a_t=\sum_{i=1}^{t}\log_{1.0001}{P_i}$$  
As:  
$$P_{t_1, t_2}=(\prod_{i=t_1}^{t_2}{P_i})^{\frac{1}{t_2-t_1}}$$  
We have:  
$$\log_{1.0001}P_{t_1, t_2}=\frac{\sum_{i=t_1}^{t_2}(\log_{1.0001}P_i)}{t_2-t_1}=\frac{a_{t_2}-a_{t_1}}{t_2-t_1}$$  

#### Implementation
A concept of virtual reserves is brought in, that is, instead of tracking reserves $X$ and $Y$ as in v2, `liquidity` $L=\sqrt{x\cdot y}$ and `sqrtPrice` $\sqrt{P}=\sqrt{\frac{y}{x}}$ are tracked. The main advantage presented in the white paper is that only one of $L$ and $\sqrt{P}$ changes at a time.  
As a result, virtual reserves of $X$ and $Y$ are presented as:  
$$x=\frac{L}{\sqrt{P}}$$  
$$y=L\sqrt{P}$$  
Besides, given current `sqrtPrice` $\sqrt{P_c}$ and a user chooses a `liquidityDelta` $\sqrt{\Delta{L_c}}$, if someone wants to add a new liquidity with a price range $[P_a, P_b]$ into the pool, the reserves of $X$ and $Y$ he needs to provide can be calculated as(suppose $P_c \in [P_a, P_b]$):  
$$\Delta{x}=\Delta{L_c}\cdot (\frac{1}{\sqrt{P_c}}-\frac{1}{\sqrt{P_b}})$$  
$$\Delta{y}=\Delta{L_c}\cdot (\sqrt{P_c}-\sqrt{P_a})$$  
The above formulas can be derived from the following:  
$$
\left\{ 
\begin {array}{cc}
\frac{y_c}{x_c}=\frac{\Delta{y}}{\Delta{x}} \\
x_a = x_c+\Delta{x} \\
y_b = y_c+\Delta{y}
\end{array}
\right.
$$  
That is, after the addition or remove of the `liquidity`, the price won't change.  
The situation when $P_c \notin [P_a, P_b]$ is very simple, check the white paper<sup>[1]</sup> for details.  

## Implementation
In this hackathon, we have designed and made out a [prototype](../src/O-AMM/) of the Omniverse AMM algorithm. As we decided to participate in this hackathon on about December 8, the time is very limited and until now the on-chain version of O20k is the simplest, that is, we temporarily use $x\cdot y=k$ as the AMM mechanism to **prove the Omniverse Protocols**.  
* [Detailed explanation of Simple AMM](https://github.com/xiyu1984/Blog/blob/main/docs/AMM/uniswap%20explanation.md)

## Key points
### Price
Suppose there are token X and token Y, and the price of X means how many Ys of a token X deserves. Suppose the reserve of X is x, and the reserve of Y is y.  

$$P_{X|Y}(x,y)=\frac{y}{x}$$

### Liquidity Deposit
Keep the price not change.  
$$\frac{y+\Delta{y}}{x+\Delta{x}}=\frac{y}{x}$$  

The key point of the implementation of the interface `add pool` is that given $x$, $y$ and $\Delta x$, the rest $\Delta y$ can be determined.  

#### Working Flow
* The provider calls `get expected pool` to get $\Delta y$ if input $\Delta x$, and vice versa.
* The provider encapsulates Omniverse Transfer of **both** $X$ and $Y$ from provider's account to swap account(contract MPC wallet account).
* The on-chain swap makes the on-chain checking for the validation of amount $\Delta x$ and $\Delta y$ according to the fomula above.
* The on-chain Omniverse Swap updates and records the new state of the pool.
* The on-chain swap sends the two encapsulated Omniverse Transactions to the related on-chain Omniverse token.
* The Omniverse tokens of $X$ and $Y$ processes the transfers.

#### Note
We can also provide a liquidity value $\ell$ to calculate $\Delta{x}$ and $\Delta{y}$:  

$$\left \{ \begin {array}{lcl} 
\frac{y+\Delta{y}}{x+\Delta{x}}=\frac{y}{x} \\  
(x+\Delta{x})(y+\Delta{y})=xy+\ell  
\end{array}\right.$$  

Solving this system of equations yields:  

$$\left \{ \begin {array}{lcl}
\Delta{x}=\sqrt{\frac{x}{y}(xy+\ell)}-x\\
\Delta{y}=\sqrt{\frac{y}{x}(xy+\ell)}-y
\end{array}\right.$$

###  Liquidity Withdraw
Given the withdraw amount of the liquidity $\ell$, solve the following system of equations to get $\Delta{x}$ and $\Delta{y}$:  

$$\left \{ \begin {array}{lcl}
\frac{y-\Delta{y}}{x-\Delta{x}}=\frac{y}{x}\\
(x-\Delta{x})(y-\Delta{y})=xy-\ell
\end{array}\right.$$  

The result is:  

$$\left \{ \begin {array}{lcl}
\Delta{x}=x-\sqrt{\frac{x}{y}(xy-\ell)}\\
\Delta{y}=y-\sqrt{\frac{y}{x}(xy-\ell)}
\end{array}\right.$$  


### Swap
Keep the liquidity $xy=k$ not change.  
Input $\Delta x$, solve the output of $\Delta y$.  
$$(x+\Delta x)(y-\Delta y)=xy$$  

For instance, given $x$, $y$ and $\Delta x$, the rest $\Delta y$ can be solved as:  
$$\Delta y = y-\frac{xy}{x+\Delta x}=\frac{y\Delta x}{x+\Delta x}$$  

#### Working Flow
* A normal user claims his tokens of $X$ or $Y$ from related on-chain Omniverse tokens.
* A normal user encapsulates an Omniverse trasfer of token $X$ or $Y$ and calls `make swap` on Omniverse Swap.
* The Omniverse Swap calculates the output amount of the opponent token, and generate an on-chain transfer in its pre-transfer cache.
* The off-chain Contract Wallet Account(MPC) makes a signature to the swap transferring out on the Omniverse Swap cache.
* The Omniverse Swap call on-chain Omniverse token to make the transfer.
 
### Half V3
* The initial propotion of $x$ and $y$ is $1:2$, that is, 10,000 $X$ and 20,000 $Y$. So the initial price is $P_{X|Y}=2$. 
* We limit the range of $x:y$ as $[1000:1, 1:1000]$. As a result, the range price of $X$ is $[0.001, 1000]$. If the price exceeds the range, the pool is resetted as $(x=10,000, y=20,000)$.  

## Reference
[1] [white paper-v3](https://uniswap.org/whitepaper-v3.pdf)  
[2] [white paper-v2](https://uniswap.org/whitepaper.pdf)
