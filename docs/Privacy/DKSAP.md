# Dual-Key Stealthy Addresses Protocol

## Principle
### Algoritm Work Flow
$(\mathbb{G}; q; G)$ denotes a group-order-generator tuple associated with the an elliptic curve, with which we can generate a private key $x\in \mathbb{F_{q}}$ and related public key $X=xG\in \mathbb{G}$.  

The work flow of transactions driven by DKSAP is as follows:    
1. All the participants generete their own *view key pairs* $(v_i, V_i=v_{i}G)$ and *spend key pairs* $(s_i, S_i=s_{i}G)$. $V_i$ and $S_i$ could be published to the public.  
2. Suppose a sender $A$ is sending a transaction to a receiver $B$.
3. $A$ generates an ephemeral key pair $(r_{A}\in \mathbb{F}_ q,R_A=r_{A}G)$ , $R_A$ is published out.  
4. ECDH protocol for $A$:
    * $A$ calculate: $c_{AB}=Hash(r_{A}V_B)=Hash(r_{A}v_{B}G)$
5. $A$ sends the transaction by setting the destination address as $T_A=c_{AB}G+S_B$, where $S_B$ is the public *spend key* of the receiver $B$. The one who has the private key of $T_A$, can operate on address $T_A$
6. ECDH protocol for $B$:  
    * $B$ calculate: $c^{,}_ {AB}=Hash(v_{B}R_{A})=Hash(v_{B}r_{A}G)$
    * Obviously $c_{AB}==c^{,}_{AB}$
7. As $T_A=c_{AB}G+S_B=c_{AB}G+s_{B}G$, so $B$ can operate on $T_A$ by $t_{A}=c_{AB}+s_{B}$, which could be treated as the private key of $T_A$

### Details
* $V_i$, $S_i$ are public
* When participant $\mathcal{P}_i$ makes a transaction, he will publish an ephemeral destination address with a temporary $R$ with it
* Everyone need to check if he has the right to operate on the new address
* It is able to achieve the scenario that the sender knows whom he sends to, but the receiver cannot publicly know where a transaction is from  
* Except the sender, other participants can only see ephemeral addresses and related temporary numbers


## Reference
[1] The Shadow Project. \Dual-key Stealth Addresses", in part of Shadow Documentation.
