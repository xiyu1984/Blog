# Series ZK - Introduction

## Principle
* Transform normal problems into a polynomial problems.    
    * zk-SNARK: Convert the problem of proving that a CI statement holds into a problem of proving that a polynomial **equation** holds.
    * zk-STARK: Converts the problem of proving that a CI statement holds into a problem of proving that a polynomial is **less than** a certain degree.
* Transfor normal problems into a set problems.
    * $\sum$-Bullets: range proofs, settings, etc.

### Statement
* A standard ZK-proof for the statement:  
$$st: \{(a, b, c, ...; x, y, z, ...) : f(a, b, c, ...; x, y, z, ...)\}$$    
    means that the prover shows knowledge of x, y, z, . . . such that f(a, b, c, . . . , x, y, z, . . .) is true, where a, b, c, . . . are public variables. We use st[a, b, c, . . .] to denote an instance of st where the variables a, b, c, . . . have some fixed values.
* [Example](https://crypto.stanford.edu/~buenz/papers/zether.pdf)  
![example of statement](./image/example%20statement.png)

### Circle
* [R1CS](https://www.zeroknowledgeblog.com/index.php/the-pinocchio-protocol/r1cs)
* [Rank-1 Constraint System](https://tlu.tarilabs.com/cryptography/rank-1)  
![Circle](./image/polynomial-eg-ac.png)  

Easy for algebraic operations, but hard for logical operations.

## Sulotuion
`zk-SNARK` and `zk-STARK` permeate almost everything zero-knowledge in blockchain.

* [Differences between zk-SNARK and zk-STARK](https://blog.pantherprotocol.io/zk-snarks-vs-zk-starks-differences-in-zero-knowledge-technologies/): 
    * [Trusted setup](./zk%20SNARK.md#generate-crs-in-a-decentralized-manner): SNARK
    * Size(gas cost): SNARK < STARK
    * Quantum resistant: STARK
* [Details of zk-SNARK](./zk%20SNARK.md)
* [Details of zk-STARK](./zk%20STARK.md)

## Engineering
* [Circom](./zk%20Circom.md): solidity
* [Bellman](https://github.com/zkcrypto/bellman): Rust
* [zkMove](https://github.com/young-rocks/zkmove): Move

## Application
Note that what ZK could do currently can all be done engineeringly better by hardware based TEE, e.g., SGX. But ZK seems to be still believed as the future by most Web3ers. We are not sure whether it's promoted by technology trend or just by risk ventures, but anyway, the technology part is still beautiful.    

### Privacy
* [ private payment system](https://github.com/ConsenSys/anonymous-zether)
* [Privacy-Cross-Chain-Demo](https://github.com/dantenetwork/Privacy-Cross-Chain-Demo/tree/main/Anonymous)

### Rollup
Note that the `Rollup` implemented based on zk can be considered as an instance of VC(verifiable computation), which is general capacity expansion. Another instance of VC is TEE(Trusted Execution Environment), one implementation of which is [Intel SGX](https://medium.com/@integritee/tee-101-how-intel-sgx-works-and-why-we-use-it-at-integritee-5cb2957c050f). Intel SGX is based on hardware and has a high performance in engineering, but it depends on the centralized certificate of Intel company. Actually the certificate can be decentralized by a MPC method.  

## Projects
* [scroll](https://hackmd.io/@yezhang/S1sJ2cEWY): Rollup, GPU
* [StarkNet(StarkWare)](https://starkware.co/starknet/): Rollup
* [Aleo](https://www.aleo.org/): Privacy
* [zkEVM(zksync)](https://docs.zksync.io/zkevm/): Rollup
* [Zether](https://crypto.stanford.edu/~buenz/papers/zether.pdf): Smart contract based privacy trading 

## Reference
* [Why And How ZkSnark Works](https://arxiv.org/abs/1906.07221)
