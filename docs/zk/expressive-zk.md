# Expressive zk

This article aims to make an easy and concrete expression to help understand the main principle of the ZKP technology underground.  

Discussions here are more about the theoretical pieces of knowledges including both `SNARK` and `STARK`, based on implementations including [starknet](https://medium.com/starkware/a-framework-for-efficient-starks-19608ba06fbe), [plonky2](https://github.com/0xPolygonZero/plonky2), and [halo2](git@github.com:zcash/halo2.git).  

Note that this is just a personal comprehension of the knowledge context of the Zero Knowledge Proofs from the technical view.  

## Framework

`ZKP` is proof, not calculation. The highly generalized framework of `SNARK`s and `STARK`s can be both described as follows:   

- Arithmetization: translates a computation into a polynomial
- Polynomial commitment scheme: determines how to generate a proof from the polynomial to prove the computation is right, and how it is verified based on the polynomial

## Arithmetization

`Arithmetization`, more comprehensively, `Arithmetization and Arithmetic Constraint System`, generates a polynomial from a computation.  

A computation can be expressed as:

- [*execution trace*](https://medium.com/starkware/arithmetization-i-15c046390862): a series of values from the start to the end of the computation. (note that in the language of `zk` , both the *inputs* and *outputs* of the computation are *inputs in the `zk` statement*)
    - *inputs* of the computation(start)
    - *outputs* of the computation(end)
    - *intermidiate values* along with the process of the computation
- [*Arithmetic Constraint*](https://aszepieniec.github.io/stark-anatomy/overview#arithmetization-and-arithmetic-constraint-system): in another word, [*polynomial constraint*](https://medium.com/starkware/arithmetization-ii-403c3b3f4355). Simply, this is the relationship between the values in the *execution trace*. In most cases, the range of one relationship is limited only in the neighboring rows of the *execution trace*. Normally, there are at least two types of constraints:  
    - *boundary constraints*: the start and the end values of the *execution trace*. Normally and simplified, the *inputs* and *outputs* of the computation.  
    - *transition constraints*: intuitively, it's the calculation steps in the computation itself, as the 'relationships'(formally constraint) between consecutive states tuples in the *execution trace* are established by the calculation steps.

Now let's dissect the composition of the computation in another view, the varible part and the unvariable part:  
- **variable part**: the values in the *execution trace*. These parts are variable accroding to the concrete values of the *inputs* and *outputs* of the computation.
- **unvariable part**: the *transition constraints*. Intuitively, this part can be considered as the fixed structure of the computation.  

From this perspective, the mapping by the `Arithmetization` from a computation to a polynomial will be more intuitive.  
Now it's time for the polynomial. Simply there are two key steps for the generation:  
- build a basic polynomial, say $f_{j}(x)$, by interpolation form the nodes($\{g^{i}, t_{ij}\}^{n}_{i=0}$) in the *execution trace*, where $g$ is the generator of a choosing finite field, $t_{ij}$ is the value in the *execution trace* of row $i$ and column $j$. Namely, $f_{j}(g^{i})=t_{ij}$, and $f_{j}(x)$ is interpolated by the $j$ th column of the *execution trace*. Combine ${f_{j}(x)}^{m}_{j=0}$ in all columns with random scalars.  
- Encode $f(x)$ and *transition constraints* together to build the final polynimal, say $F(x)$, of the computation.  
A comprehensive implementation of the `Arithmetization` mentioned above can be found in [PLONK](https://eprint.iacr.org/2019/953), derived from which [plonky2](https://github.com/0xPolygonZero/plonky2) and [halo2](git@github.com:zcash/halo2.git) made out a `PLONKish` version. To understand it eaier and more intuitively, check the examples in [Arithmetization I](https://medium.com/starkware/arithmetization-i-15c046390862) and [Arithmetization II](https://medium.com/starkware/arithmetization-ii-403c3b3f4355).    


Let the final polynomial $F(x)$ be with degree `d`, namely:  

$$F(x)=\sum^{d}_{i=0}\alpha^{i}x^{i}$$  

Then, $F(x)$ will be used to prove the original computation is right, why and how? The practical significance of $f(x)$ makes sense.  

Diving into the processing steps of the `Arithmetization`, it's not very hard to find the related parts in the original computation being encoded in $F(x)$:     
- variable part: the $f(x)$, which is generated from the values in the *execution trace*. 
- unvariable part: the *transition constraints* which transforms $f(x)$ to $F(x)$. The transformation is not so intuitive, but at a high level, it restricts $F(x)$ to be evaluated to be `0` in a series of $x$-points known to both the prover and verifier. In the [simplied examples](https://medium.com/starkware/arithmetization-ii-403c3b3f4355), the $x$-points are just the ${g^i}^{n}_{i=0}$, and more complex in `in PLONKish Arithmetizations`.  
In other words, we can conclude that if $F(x)$ has a factor polynomial $Z(x)=\prod^{l}_{i=0}(x-g^{i})$, all the *transition constraints* are met no matter what the *inputs in the `zk` statement* are, then the original computation is right under some concrete *inputs* and *outputs*.   

Another interesting and convenient thing is that the values of the $x$-points are just determined by $g$ and $i$, exposing nothing about the values of the *execution trace* of the original computation so that zero-knowledge could be kept.  

Until now, there's no difference between `SNARK` and `STARK`, that is, if $Z(x)$ is the factor of $F(x)$, both `SNARK` and `STARK` can make a proof that the original computation is right.  
Essentially, the difference between `SNARK` and `STARK` is how the proof and the related verifying are made, which is the main content in the next chapter, namely, `Polynomial commitment scheme`.  

## Polynomial commitment scheme