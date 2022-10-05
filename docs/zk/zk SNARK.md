# zk-SNARK
## Prepare
Operation addition and scalar multiplication on an elliptic can be found [here](./Elliptic.md).  

Multiplicative homomorphic hiding based on bilinear mapping of elliptic curves.<sup>[1]</sup><sup>[2]</sup><sup>[3]</sup>

![mapping](./image/multiple%20map.png)

$$ e(\alpha \cdot G_1, \beta \cdot G_1) = \alpha\beta \cdot e(G_1, G_1) = \alpha\beta \cdot G_2 $$ 
 
$$e(G_1, G_1) = G_2$$

## Priciple
Let $f(x) = t(x)s(x)$, $t(x)$ is public, we need to prove we know a polynomial which can devide exactly $t(x)$.

### Large prime exponential modulo operation
Now suppose we have converted the information contained in a real problem into a polynomial in the form of:  

$$ f(x) = c_dx^d + c_{d-1}x^{d-1} + ... + c_1x^1 + c_0 = \sum_{i=0}^{d}c_ix^i $$  

and this polynomial is dertermined by the parameters: $\{c_i, i \in \{0, 1, ..., d\}\}$. Simplifying the problem again, let $f(x)$ be factorizable as $f(x) = h(x)t(x)$. In the zero-knowledge proof, $t(x)$ is overt, while the problem is transformed into the following: to prove to B that A knows some information, A only needs to prove to B that it knows a polynomial that can divide $t(x)$ integerly.
* Steps
1. (initialization phase) Expressions for large prime numbers $g$, $t(x)$ are disclosed.
2. (initialization phase) Verifier chooses the random number $x = s$, and the paired random number $α$. The $g^{s^0}, g^{s^1}, g^{s^2}, ..., g^{s^d}$ is public, and also the corresponding "offset" number $g^{\alpha s^0}, g^{\alpha s^1}, g^{\alpha s^2}, ..., g^{\alpha s^d}$ , the values of **$s$ and $α$ are known only to Verifier**.  
3. (Prove phase) Prover:  
    a. Generate a random $\delta $ that only the prover knows. Based on the expression for f(x), which only he knows, and h(x), calculate: 
    
    $$(\prod_{i=0}^{d}g^{c_{i}s^{i}})^{^\delta }=(g^{\sum_{i=0}^{d}{c_{i}s^{i}}})^{^\delta }=g^{\delta f(s)}$$
    
    $$(\prod_{i=0}^{m}g^{c^{'}_{i}s^{i}})^{\delta }=(g^{\sum_{i=0}^{m}{c^{'}_{i}s^{i}}})^{\delta }=g^{\delta h(s)}$$
    
    b. Calculate:
    
    $$ (\prod_{i=0}^{d}(g^{\alpha s^{i}})^{c_{i}})^{\delta }=(g^{\sum_{i=0}^{d}{\alpha c_{i}s^{i}}})^{\delta }=g^{\delta \alpha f(s)} $$
    
    c. Send $g^{\delta f(s)}, g^{\delta h(s)}, g^{\delta \alpha f(s)}$ to the verifier.  
4. (Verify phase) Verifier:  
    a. Verify: whether $(g^{\delta h(s)})^{t(s)}$ is equal to $g^{\delta f(s)}$, since Verifier has the value of s and can calculate the value of $t(s)$ for the above verification.  
    b. Verify: whether $(g^{\delta f(s)})^{\alpha}$ is equal to $g^{\delta \alpha f(s)}$, which is because Verifier holds the value of $\alpha$.

* Description
1. Since the values of **s, $\alpha$ are known only to Verifier**, Prover can construct out $g^{\delta f(s)}, g^{\delta h(s)}, g^{\delta \alpha f(s)}$ through $g^{s^{0}}, g^{s^{1}}, g^{s^{2}}, ..., g^{s^{d}}$ and $g^{\alpha s^{0}}, g^{\alpha s^{1}}, g^{\alpha s^{2}}, ..., g^{\alpha s^{d}}$ only if he really knows the expression of $f(x)$.
2. If a prover provides the values of the mathematical formulas: $g^{f(s)}, g^{h(s)}, g^{\alpha f(s)}$, The verifier may the verifier could brute-force limited range of coefficients combinations until the result is equal to the prover’s answer and get the expression of $h(x)$. **$\delta$ makes the brute-force cracking impossible**.  
3. $t(s)$ is to ensure that A provides a polynomial that can divide $t(x)$, and the $\alpha$ "offset" is to ensure that A is indeed using a polynomial in the computation of the evidence. 
4. Because the values of $s$ and $α$ cannot be exposed to the Prover, each Verifier needs to generate its own approved values independently when verifying, and each proof and verification requires a separate round of interaction between each Verifier and the Prover.

* Problems  
However, this approach (which is an interactive zero-knowledge proof) suffers from the following problems.
1.  Since multiplicative homomorphism hiding is not supported, the values of $α$ as well as $t(s)$ are required to be kept by the verifier at all times, which then creates the possibility of attacks on the verifier to obtain $α$, $t(s)$ in order to falsify the proof.
2. Verifier and Prover can collude to falsify proofs.
3. As $α$, $t(s)$ is only known to one verifier, so a prover needs to interact with each verifier to make a proof. **It is `not Non-Interactivity` at this stage**.

### Elliptic curve
To completely hide the values of $α$ and $s$ above and make it `Non-Interactivity`, the process of zero-knowledge proof requires the multiplicative homomorphism hiding mentioned above, i.e., the mapping (i.e., bilinear mapping) method mentioned [above](#prepare). With such a "magic tool", the design idea of zero-knowledge proof can be evolved into the following form (we adopt a more standardized logo for the following expression, i.e., $g^{\alpha}$ represents the elliptic curve with $g$ as the base point hitting alpha times, $e(g^{\alpha},g^{\beta})$ represents "mapping").  
* Steps:
1. Suppose $s$ and $\alpha$ is unkown to everyone, and Provers and Verifiers only know the following CRS(Common Reference String). (I will show you how to get it without knowing the concrete $s$ and $\alpha$). The `CRS` is:  
    a. Prover key: $(g_{1}^{s^{i}}, g_{1}^{\alpha s^{i}})$  
    b. Verifier key: $(g_{1}^{t(s)}, g_{1}^{\alpha})$  
    c. $g_1$ is the base point in elliptic curve $E_1$      
2. Prover:  
    a. Calculate 
    
    $$ g_{1}^{\sum_{i=0}^{d}{c_{i}s^{i}}} = g_{1}^{f(s)} $$
    
    $$ g_{1}^{\sum_{i=0}^{m}{c_{i}^{'}s^{i}}} = g_{1}^{h( s)} $$
    
    according to he point $g_{1}^{s^{i}}$ on the elliptic curve $E_1$  
    b. Calculate 
    
    $$ g_{1}^{\sum_{i=0}^{d}{c_{i}\alpha s^{i}}}=g_{1}^{\alpha f(s)} $$
    
    according to the point $g_{1}^{\alpha s^{i}}$ on the elliptic curve $E_1$  
    c. sampling $\delta$ and calculate a proof:  
$$\pi = (g_1^{\delta f(s)}, g_1^{\delta h(s)}, g_1^{\delta \alpha f(s)})$$

3. Verifier:  
    a. Verify that the point $e(g_{1}^{t(s)}, g_{1}^{\delta h(s)})$ on elliptic curve $E_2$ is equal to point $e(g_{1}^{\delta f(s)}, g_{1})$ on $E_2$ according to points $g_{1}^{t(s)}, g_{1}^{\delta h(s)}, g_{1}^{\delta f(s)}$ on $E_1$  
    b. Verify that the point $e(g_{1}^{\delta f(s)}, g_{1}^{\alpha})$ on $E_2$ is equal to the point $e(g_{1}^{\delta \alpha f(s)}, g_{1})$ on $E_2$ according to points $g_{1}^{\delta f(s)}, g_{1}^{\alpha}, g_{1}^{\delta \alpha f(s)}$ on $E_1$  

However, the CRS parameters need to be generated and disclosed in a trusted way, and the traditional approach requires a trusted "third party", which poses the risk of centralization. We will explain in the [next section](#generate-crs-in-a-decentralized-manner) a typical solution to avoid centralization. With these parameters, the prover provides the points $p, h, p^{'}$ on the elliptic curve $E_1$ and the verifier can complete the verification, and the `CRS` need to be disclosed only once, which is the reason why this approach is called non-interactive.  

### Generate CRS in a decentralized manner
`CRS` can be generated in a decentralized manner.  

In simple terms, the values of $s, α$ are generated by multiple participants, but each person does not know what the "part" of the others is, for example, suppose there are three participants, A, B and C:
1. Participant `A` Compute and publish the CRS values: $(g_{1}^{s_{A}^{i}}, g_{1}^{\alpha_{A}}, g_{1}^{\alpha_{A} s_{A}^{i}})$;
2. Participant `B` compute and publish on the basis of A: $(g_{1}^{s_{A}^{i}s_{B}^{i}}, g_{1}^{\alpha_{A}\alpha_{B}}, g_{1}^{\alpha_{A}\alpha_{B} s_{A}^{i}s_{B}^{i}})$.
3. Participant `C` Compute and publish on the basis of B: $(g_{1}^{s_{A}^{i}s_{B}^{i}s_{C}^{i}}, g_{1}^{\alpha_{A}\alpha_{B}\alpha_{C}}, g_{1}^{\alpha_{A}\alpha_{B}\alpha_{C} s_{A}^{i}s_{B}^{i}s_{C} ^{i}})$.  

The final value published by `C` serves as the public `CRS`. Since `A`, `B`, and `C` all publish only values of the form $g^{s^{i}}$, the $s, α$ values generated by each participant are agnostic to everyone else.   
But in this model, it is also the agnosticism of $s, α$ that leads to a problem, that the last participant can ignore the data provided by all the previous participants and provide the `CRS` only by himself. therefore, a validation mechanism is needed to guarantee that the `CRS` made public by each participant (except the first one), is calculated based on the results of the previous ones. In fact, the validation mechanism is relatively simple, and only requires that others, except the first one, disclose some information about themselves on top of the published `CRS`, e.g.
1. Participant `A` Compute and publish the CRS values: $(g_{1}^{s_{A}^{i}}, g_{1}^{\alpha_{A}}, g_{1}^{\alpha_{A} s_{A}^{i}})$
2. Participant `B` compute and publish on the basis of A: $(g_{1}^{s_{A}^{i}s_{B}^{i}}, g_{1}^{\alpha_{A}\alpha_{B}}, g_{1}^{\alpha_{A}\alpha_{B} s_{A}^{i}s_{B}^{i}})$ and $(g_{1}^{s_{B}^{i}}, g_{1}^{\alpha_{B}}, g_{1}^{\alpha_{B} s_{B}^{i}})$
3. Participant `C` Compute and publish on the basis of B: $(g_{1}^{s_{A}^{i}s_{B}^{i}s_{C}^{i}}, g_{1}^{\alpha_{A}\alpha_{B}\alpha_{C}}, g_{1}^{\alpha_{A}\alpha_{B}\alpha_{C} s_{A}^{i}s_{B}^{i}s_{C} ^{i}})$ and $(g_{1}^{s_{C}^{i}}, g_{1}^{\alpha_{C}}, g_{1}^{\alpha_{C} s_{C}^{i}})$   

For example, for a message posted by B, we can confirm that $g_{1}^{s_{A}^{i}s_{B}^{i}}$ is based on the information posted by `A` through verifying that $e(g_{1}^{s_{A}^{i}}, g_{1}^{s_{B}^{i}})$ is equal to $e(g_{1}^{s_{A}^{i}s_{B}^{i}}, g_{1})$. The other two are similar.  

## Example
A non-interactive ZK (NIZK) proof system includes algorithms $(Setup_{nizk}, Prove, Verify_{nizk})$, where $Setup_{nizk}$ outputs some public parameters, $Prove$ generates a proof for a statement given a witness, and $Verify_{nizk}$ checks if the proof is valid w.r.t the statement.  
Some features are a) correct, an honest prover can produce a valid proof; b) zero-knowledge, a verifier learns nothing from the proof but the validity of the statement, and c) sound, a computationally bounded prover cannot convince a verifier of a false statement. 
`Σ protocols`, which is used in [zether](https://crypto.stanford.edu/~buenz/papers/zether.pdf), with the Fiat-Shamir transform applied, have all these properties. 

## Reference
* [1] P.S.L.M. Barreto, H.Y. Kim, B.Lynn, and M.Scott, Efficient algorithms for
pairing-based cryptosystems, Crypto 2002, LNCS 2442, pp.354-368, SpringerVerlag, 2002.
* [2] D. Boneh, B. Lynn, and H. Shacham, Short signatures from the Weil pairing,
Asiacrypt 2001, LNCS 2248, pp.514-532, Springer-Verlag, 2001.
* [3] S. D. Galbraith, K. Harrison, and D. Soldera, Implementing the Tate pairing,
ANTS 2002, LNCS 2369, pp.324-337, Springer-Verlag, 2002.
