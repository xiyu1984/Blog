# MPC-ECDSA : First Model
An MPC-ECDSA is a multi-party's version of the standard ECDSA, which is compatible to the standard signature verifying process, and just makes a magical mathematic variance to the standard signing process, that is, makes it as an aggregable parallel multi-parties Process.  

The references of the *first* model in this article can be found [here](#reference)<sup>[1][2]</sup>.  

## Principle
A [Standard ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) includes $(\mathbb{G}; q; g)$ denoting the group-order-generator tuple associated with the ECDSA curve, private key $x\in \mathbb{F_{q}}$ and related public key $X=g^{x}$ ( actually it's a [scalar multiplication](../zk/Elliptic.md#scalar-multiplication) $x\cdot G$ of the elliptic curve ), $\rho$ the x-projection of point $g^{k^{-1}}\in\mathbb{G}$ where $k$ is a random element of $\mathbb{F_{q}}$, $\sigma=k(m+x\rho)$ where m is the hash of the message to be signed. The signature is $(\rho,\sigma)$. The verification is whether $\rho$ is the x-projection of $g^{m\sigma^{-1}}\cdot X^{\rho\sigma^{-1}}$ ( Actually it's a [addition of two points](../zk/Elliptic.md#addition) in the elliptic curve ).  

In this ariticle, we will introduce s simple model<sup>[2]</sup> transforming the single participant signing into a multi-parties process. In this model, the security threshold is $t=n-1$, that is, all participants follow the protocol.

### Setup(Key Generation)
* For each participant $\mathcal{P_i}$, generate a random $x_i\in\mathbb{F_{q}}$ and sends $X_i=g^{x_i}$ to the public.  
* The global public key is $X=\sum{X_i}\in\mathbb{G}$. And the related global private key is $x=\sum{x_i}$, which is unknown to everyone.  
* For each $\mathcal{P_i}$ defines both the $enc_i$ and $dec_i$ algorithms that are addition homomorphic. The classic [ElGamma Encryption](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) is one of these algorithms.  
* For each $\mathcal{P_i}$ generate local a share $k_i$ and $\gamma_i$. This makes a global $k=\sum{k_i}$ and $\gamma=\sum{\gamma_i}$, but no one knows them.  
* For each pair of parties $\mathcal{P_i}$ and $\mathcal{P_j}$,  
    * $\mathcal{P_i}$ generates $K_i=enc_i(k_i)$ and sends it to $\mathcal{P_j}$. $enc_i(k_i)$ means encrypting $k_i$ with $\mathcal{P_i}$'s public key $X_i=g^{x_i}$
    * $\mathcal{P_j}$ samples a random $\beta_{j,i}\in\mathbb{F_q}$, calculate $E_{j,i}=(\gamma_j\odot K_i)\oplus enc_i(-\beta_{j,i})=enc_i(\gamma_j k_i - \beta_{j,i})$, and then sends $E_{j,i}$ to $\mathcal{P_i}$. Note that here $\mathcal{P_j}$ uses $\mathcal{P_i}$'s public key to encrypt $-\beta_{j,i}$
    * $\mathcal{P_i}$ calculates $\alpha_{i,j}=dec_i(E_{j,i})=dec_i(enc_i(\gamma_j k_i - \beta_{j,i}))=\gamma_j k_i - \beta_{j,i}$
    * As a result,  $\alpha_{i,j}+\beta_{j,i}=\gamma_j k_i$
* Fro each pair of parties $\mathcal{P_i}$ and $\mathcal{P_j}$, generate $\hat{\alpha_{i,j}}+\hat{\beta_{j,i}}=x_j k_i$ with a similar method.  
* Let's look into $\alpha_{i,j}$ and $\beta_{j,i}$ deeper, for more intuitively, we can express their relationships as matrixes:
    
    $$\left[\begin {array}{cc}
    \alpha_{1,1}&\alpha_{1,2}&...&\alpha_{1,n}\\
    \alpha_{2,1}&\alpha_{2,2}&...&\alpha_{2,n}\\
    \vdots&\vdots&\ddots&\vdots\\
    \alpha_{n,1}&\alpha_{n,2}&...&\alpha_{n,n}
    \end{array}\right]^T + \left[
    \begin{array}{cc}
    \beta_{1,1}&\beta_{1,2}&...&\beta_{1,n}\\
    \beta_{2,1}&\beta_{2,2}&...&\beta_{2,n}\\
    \vdots&\vdots&\ddots&\vdots\\
    \beta_{n,1}&\beta_{n,2}&...&\beta_{n,n}
    \end{array}
    \right] = \left[
    \begin{array}{cc}
    \gamma_1k_1&\gamma_1k_2&...&\gamma_{1}k_{n}\\
    \gamma_{2}k{1}&\gamma_{2}k{2}&...&\gamma_{2}k{n}\\
    \vdots&\vdots&\ddots&\vdots\\
    \gamma_{n}k{1}&\gamma_{n}k{2}&...&\gamma_{n}k{n}
    \end{array}
    \right]$$

    * $\mathcal{P_i}$ only knows the i-th row of matrix $\mathbf{A}$ and the i-th row of matrix $\mathbf{B}$
    * Recall that $\alpha_{i,j}+\beta_{j,i}=\gamma_j k_i$, so the pairing is the i-th row of $A$ and the i-th column of $B$
    * $\alpha_{i,i}=\gamma_{i,i}$ and $\beta_{i,i}=0$
    * $\gamma\cdot k=(\sum{\gamma_i})\cdot(\sum{k_j})=\sum_{i}\sum_{j}{\gamma_{i}k{j}}$, that is the summation of all the elements in both $A$ and $B$
* $\hat{\alpha_{i,j}}$ and $\hat{\beta_{j,i}}$ has a similiar situation, that is $x\cdot k=(\sum{x_i})\cdot(\sum{k_j})=\sum_{i}\sum_{j}{x_{i}k{j}}$ equal to the summation of all the elements in similiar matrixes $\hat{A}$ and $\hat{B}$
* For each $\mathcal{P_i}$, calculate $\delta_{i}=\gamma_{i}k_i+\sum_{j\neq i}{(\alpha_{i,j}+\beta_{i,j})}=\sum_j{(\alpha_{i,j}+\beta_{i,j})}$, that is the i-th row in both $A$ and $B$ which belongs to $\mathcal{P_i}$. And then sends $(\gamma^{i},\delta^{i})$ to public. 
* $\sum{\delta_i}$ is the summation of all elements in both $A$ and $B$. So publicly, everyone can get $g^{k^{-1}}=(\Pi g^{\gamma^{i}})^{(\sum{\delta^j})^{-1}}$, thus abtaining $\rho$  

### Sign
* Similiarly, for each $\mathcal{P_i}$, calculate $\sigma_i=k_{i}m+\rho(x_i k_i+\sum_{j\neq i}{(\hat{\alpha_{i,j}}+\hat{\beta_{i,j}})})$. And then sends to public
* Publicly, everyone can get $\sigma=\sum{\sigma_i}=m\sum{k_i}+\rho(\sum_{j}{(\hat{\alpha_{i,j}}+\hat{\beta_{i,j}})})=km+kx\rho=k(m+x\rho)$. 
* $(\rho,\sigma)$ is the standard signature for the ECDSA.
* Note that both the global random $k$ and global private key $x$ are unknown to anyone, and also the temporary variable global $\gamma$.  

### Verify
The verification is just the same as [standard ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm). That is, whether $\rho$ is the x-projection of $g^{m\sigma^{-1}}\cdot X^{\rho\sigma^{-1}}$.

## Reference
[1] [UC Non-Interactive, Proactive, Threshold ECDSA with Identifiable Aborts](https://eprint.iacr.org/2021/060.pdf)  
[2] [R. Gennaro and S. Goldfeder. Fast multiparty threshold ECDSA with fast trustless setup. In Proceedings of the 2018 ACM SIGSAC Conference on Computer and Communications Security, CCS 2018, Toronto, ON, Canada, October 15-19, 2018, pages 1179–1194, 2018. doi: 10.1145/3243734.3243859.](https://doi.org/10.1145/3243734.3243859)
