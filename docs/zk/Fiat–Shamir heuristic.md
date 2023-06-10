# Fiat–Shamir heuristic

## Details

The details of the `Fiat–Shamir heuristic` can be found [here](https://en.wikipedia.org/wiki/Fiat%E2%80%93Shamir_heuristic#cite_note-8), and it is recommended to read it first.  

Here we just want to explain why ***If the hash value used below does not depend on the (public) value of y, the security of the scheme is weakened, as a malicious prover can then select a certain value y so that the product cx is known***.  

In the step 3 of `Fiat–Shamir heuristic`, it calculate $c$ as:  

$$c=\mathcal{H}(g,y,t)$$  

where $\mathcal{H}$ is a cryptographic hash function.   

If $y=g^{x}$ is selected first (maybe a third party) and given as an input to the adversary who tries to produce a forgery, it is safe ***even if $y$ is not used as a parameter in the hash function***.  

But if $y$ is provided by the prover as part of the proof (now the proof is $(y,t,r)$ ), the adversary can produce a forgery.  

### How Can an Adversary Cheat without $y$ Participating in Hash Calculation

The adversary can:  

- Randomly choose $t\in\mathbb{Z^{\ast}}_ {q}$ and $r\in\mathbb{Z^{\ast}}_ {q}$
- Provide $y$ from $y\equiv(\frac{t}{g^{r}})^{\frac{1}{c}}$, where $c=\mathcal{H}(g,t)$  
- The the verification passes as:  

    $$g^{r}y^{c}\equiv g^{r}((\frac{t}{g^{r}})^{\frac{1}{c}})^{c}\equiv t$$

    - $\frac{1}{c}=c^{-1}$ and $c \cdot c^{-1}\equiv1 \ mod \ \varphi(q)$  

### Let $y$ Participate in Hash Calculation

The key point is that if $c=\mathcal{H}(g,y,t)$, the adversary cannot calculate $c$ before $y=g^{x}$ is generated, and if the adversary generates a $y$ not the form $y=g^{x}$ the verification fails.  

## Reference

- [Euler's theorem](https://en.wikipedia.org/wiki/Euler%27s_theorem)
- [Euler's totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function)
