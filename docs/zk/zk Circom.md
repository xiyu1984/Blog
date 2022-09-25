# Circom

## Official
* [Circom](https://docs.circom.io/) is a circle compiler whose ecosystem of tools allows you to create, test and create zero knowledge proofs for your circuits.
* [GitHub](https://github.com/iden3)

## Technology Stack
* solidity smart contract 
    * Output
    * On-chain verification
    * Generate automatically according to an [*.circom](https://github.com/xiyu1984/ZK-Cook/blob/main/simpleZKP/simpleCircle.circom)
    * [Contract example](https://github.com/xiyu1984/ZK-Cook/blob/main/simpleZKP/simpleCircle_js/verifier.sol)
* Rust
    * [Core](https://github.com/iden3/circom)
* js
    * [Old core](https://github.com/iden3/circom_old)
* circomlib
    * [lib](https://github.com/iden3/circomlib)
    * [docs](https://docs.circom.io/circom-language/signals/)

## Tools
* [circomlibjs](https://github.com/iden3/circomlibjs): witness
* [circom_tester](https://github.com/iden3/circom_tester)
* [ffjavascript](https://github.com/iden3/ffjavascript): Finite Field Operation
* [snarkjs](https://github.com/iden3/snarkjs): a npm package that contains code to generate and validate ZK proofs from the artifacts produced by circom.

## Operation
The [official tutorial](https://docs.circom.io/getting-started/installation/) includes the details of the steps to create and execute a user-difined zkp process.  

Here is a simple [use case](https://github.com/xiyu1984/ZK-Cook/tree/main/simpleZKP).

