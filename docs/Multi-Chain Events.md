# The Attack Vector for Multi-Chain interoperation
## **Index**
* [BNB](#bnb)
* [Nomad](#nomad)
* [Harmony](#harmony)
* [Anyswap](#anyswap)
* [Chainswap](#chainswap)
* [Poly Network](#poly-network)
* [Ronin](#ronin)
* [THORChain](#thorchain)
* [Wormhole](#wormhole)


## BNB

* Analysis
  * [root := op.Proof.ComputeRootHash()](https://github.com/bnb-chain/bsc/blob/46d185b4cfed54436f526b24c47b15ed58a5e1bb/core/vm/lightclient/multistoreproof.go#L112)
  * [ComputeRootHash](https://github.com/bnb-chain/bsc/blob/46d185b4cfed54436f526b24c47b15ed58a5e1bb/core/vm/lightclient/multistoreproof.go#L22)
  * [Hash](https://github.com/bnb-chain/bsc/blob/46d185b4cfed54436f526b24c47b15ed58a5e1bb/core/vm/lightclient/rootmultistore.go#L41)
  * [return merkle.SimpleHashFromMap(m)](https://github.com/bnb-chain/bsc/blob/46d185b4cfed54436f526b24c47b15ed58a5e1bb/core/vm/lightclient/rootmultistore.go#L47). `SimpleHashFromMap` cannot be found in the related import ["github.com/tendermint/tendermint/crypto/merkle"](https://github.com/bnb-chain/bsc/blob/46d185b4cfed54436f526b24c47b15ed58a5e1bb/core/vm/lightclient/rootmultistore.go#L6). Instead, we can found it [here](https://pkg.go.dev/github.com/bluele/tendermint/crypto/merkle#SimpleHashFromMap)
  * They are said to eventually use `iavl prove` to make the verification, details are as below:
    * [DefaultProofRuntime](https://github.com/bnb-chain/bsc/blob/46d185b4cfed54436f526b24c47b15ed58a5e1bb/core/vm/lightclient/multistoreproof.go#L131)
    * [iavl: prove.go](https://github.com/cosmos/iavl/blob/de0740903a67b624d887f9055d4c60175dcfa758/proof.go#L53). The right child is mistakenly ignored when a left child exists.  

## Nomad
* Time: 02-08-2022
* Event summary
Nomad bridge has been attacked when they are making an update. The attack happened on target chain and the main reason is the code vulnerability which somehow relates to the uncrigorous of solidity. That is the default value of any undefined element of `mapping`.
* Analysis
   * Technical principle of Nomad
      * The key Certification: The signature of an `Updater` governed by Nomad. The updater is selected by Nomad itself, and any chain engaged with Nomad trust the updater's signature of the message root(a Merkle tree root constructed by cross-chain messages). The root is composited with messages to be sent as the Merkle leaf on the source chain, and the `Updater` is only signing to let target chains trust the root. So actually, it sacrifices decentralization.
      * It's easy to prove mistack messages(messages are different between source chain and target chain) on the target chain as `Wacher` can submit the real signed message to a target chain.
      * But it's very hard to prove messages that are created out of nothing.
   * Reason of the attack:
      * Insecure access control to `process`, which can invoke assets transferring. And abusing of undefined values in Solidity `mapping`:  
![1659428386465](https://user-images.githubusercontent.com/83746881/182327510-16d760be-79c2-453d-8c9b-dd40fc5352bc.png)
      * Invalid initiallizing value:  
![1659428574365](https://user-images.githubusercontent.com/83746881/182328203-a8f94689-44f7-4ca0-95bf-16688e428062.png)


## Harmony
* Time: 06-24-2022
* Event summary
   Two owner accounts are abused resulting in a loss of about $100 million. The related private keys are suspected to be leaked.
* Analysis
   * Harmony bridge authurizes 5 specified off-chain nodes to make the cross-chain transactions, and adopts a 2/5 multi-signature to authorize the permission of assets pool.
   * It's a centralized way and two of the working nodes leak their private key, or even worse they make collussion to do evil.

## Ronin
* Time: 03-23-2022
* Event details:
[1](https://learnblockchain.cn/article/3810), [2](https://news.bitcoin.com/axie-infinity-loses-620-million-after-hacker-compromised-ronin-validators/)

* Event Summary
    * The hacker compromised five validators' private key;
* My Opinion:
    * The selection mechanism of off-chain validators is inadequate:
        * Not much enough;
        * Lack of randomness, which makes it easier to conspire together to commit malicious acts;

## chainswap

* 合约漏洞：（内容整理于[https://zhuanlan.zhihu.com/p/389738041](https://zhuanlan.zhihu.com/p/389738041)，侵删）；
* Event Summary
    * 众所周知，Solidity是没有类似Python中**KeyError**的概念的，如果映射（mapping）中不存在这个键，则会返回0，**然而这里没有对返回值进行检查；**
    * 导致任意攻击者，只自己随机生成地址并提供签名，则都能冒充成验证者（mapping失败返回0，但没有进行检查）；
    * authQuotaOf在实现逻辑上的错误：
![1](https://user-images.githubusercontent.com/83746881/162397382-1443cfa8-3c1e-4fe4-ae94-79003b47beaf.png)
![2](https://user-images.githubusercontent.com/83746881/162397409-14f39bc3-41d8-46f0-9869-9409808e9fd8.png)
![3](https://user-images.githubusercontent.com/83746881/162397423-0d67a77d-c852-4baf-96bc-a6fef91aab22.png)
![4](https://user-images.githubusercontent.com/83746881/162397434-959e3b6a-792b-4052-9e0e-f81abefcbbd5.png)

    * 总结一下，receive函数实现的整个过程，都没有对传入的Signatory的合法性进行检查。因此攻击者只需要随机生成一个地址并生成对应的签名，即可骗过ChainSwap，自己为自己提供签名。同时，由于authQuotaOf在实现逻辑上的错误，在Signatory不合法时会返回一个非常大的值，导致了这次攻击事件的发生。而本次攻击事件发生的本质，是没有对映射索引的值进行验证。由于Solidity并不会在映射的键不存在时触发任何错误（键是否存在只能靠返回值是否为0进行判断），因此这类检查就显得非常重要。正是由于（荒谬地）缺少这样的检查，导致了这次损失超过800万美元的攻击；
* 验证者是自己选的，但以上漏洞导致任意用户都可以冒充：
    * 有趣的是，在管理者Factory的合约实现中，ChainSwap设计了一个变量来保存Signatory构成的数组，代码如下图所示：
![5](https://user-images.githubusercontent.com/83746881/162397457-c2a13374-fec0-4f82-aabb-df4e829353d1.png)
    * 有趣的是，在管理者Factory的合约实现中，ChainSwap设计了一个变量来保存Signatory构成的数组，但很遗憾的是，通过对Factory合约实现的审计，我们发现这个变量仅仅是在管理Signatory配额的时候被使用，并没有验证一个任意地址是否包含在这个数组中的实现。
* **对我们的启发：**
    * 我们合约中对验证的管理，在solidity视线中，也需要确保在代码中对每一次mapping查找结果的检查；


## Poly Network

* Event details（内容整理于[https://www.freebuf.com/vuls/284340.html](https://www.freebuf.com/vuls/284340.html)，侵删）：
* Event Summary
    * 原文：
        * 经过以上分析，结果已经很明确了，攻击者只需在其他链通过 crossChain 正常发起跨链操作的交易，此交易目的是为了调用 EthCrossChainData 合约的 putCurEpochConPubKeyBytes 函数以修改 Keeper 角色。随后通过正常的跨链流程，Keeper 会解析用户请求的目标合约以及调用参数，构造出一个新的交易提交到以太坊上。这本质上也只是一笔正常的跨链操作，因此可以直接通过 Keeper 检查与默克尔根检查。最后成功执行修改 Keeper 的操作。
        * 但我们注意到 putCurEpochConPubKeyBytes 函数定义为
        ```
        function putCurEpochConPubKeyBytes(bytes calldata curEpochPkBytes) external returns (bool);
        ```
        * 而_executeCrossChainTx 函数执行的定义为
        ```
        abi.encodePacked(bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)")))
        ```
        * 我们可以知道这两个函数的函数签名在正常情况下传入的_method 为 putCurEpochConPubKeyBytes 肯定是完全不同的，因此通过_toContract.call 理论上是无法调用到 putCurEpochConPubKeyBytes 函数的。但_method 是攻击者可以控制的，其完全可以通过枚举各个字符组合以获得与调用 putCurEpochConPubKeyBytes 函数相同的函数签名，这要求其只需枚举前 4 个字节符合即可。我们也可以自己尝试枚举验证；
        * 系统由中继链keeper来负责验证并签名；
        * 由于没有对跨链合约调用的合约方法名进行判断，那么跨链系统合约的接口也可能被调用；
        * 攻击者使用了哈希碰撞来调用系统方法 `putCurEpochConPubKeyBytes` ，且没有看到对调用源sender进行验证的地方；
        * 如果目的链被调用的合约，无法检查源链调用者的身份信息，则都可能发生Poly所遭遇的问题；

* **对我们的启发：**
    * 我们目前已经有相关机制来应对这个情况
        * SQOS中的权限验证机制（已经实现），使目的链被调用时，可以先去检查源链的调用者、以及相关调用接口；
        * 目的链所检查的这些相关权限的事项，不能由用户自己打包，需要由底层的跨链合约来打包；
    

## Anyswap

* 首次被攻击，私钥被破解
    * 原因是R签名算法，两次签名使用了相同的随机数；
* 1.18再次被攻击：（内容整理于[https://medium.com/zengo/without-permit-multichains-exploit-explained-8417e8c1639b](https://medium.com/zengo/without-permit-multichains-exploit-explained-8417e8c1639b)，侵删）
    * **Event Summary**
        * **该攻击发生在源链；**
![6](https://user-images.githubusercontent.com/83746881/162397513-3c61bd82-659e-4d00-8008-08b0436c252e.png)
        * “1”中没有对token地址进行合法性验证，只是生成相关的“any”Token的调用接口，underlying直接映射成了真正的Token地址（如案发现场的WETH）；
        * “2”中，实际上相关的真实Token ERC20没有实现permit（**相当于做外包验证**），执行fallback后也没有错误提示，导致后续代码正常执行，**这一步是所有问题的关键**；（这是solidity不严谨导致的问题）
        * “3”中没有对token地址进行验证，按理说，“any”Token是anyswap自己生成的，但这里却没有去进行验证是否是自己生成的那个地址。在这次攻击中，该合约是攻击者伪造的，那么这里实际上把WETH转到了攻击者控制的合约中；
        * “4”的问题实际上这次攻击中没有真正发生，因为在“3”执行后攻击者已经拿走token。“4”的问题是，如果真实Token ERC20没有实现permit，那么“2”验证会假通过，攻击者依然可以通过“4”完成正常跨链，然后在目的链中将资产取走。
## WormHole
* Event details

[https://www.jinse.com/news/blockchain/1178392.html](https://www.jinse.com/news/blockchain/1178392.html)
[https://blog.csdn.net/mutourend/article/details/122807577](https://blog.csdn.net/mutourend/article/details/122807577)
[https://twitter.com/kelvinfichter/status/1489041221947375616](https://twitter.com/kelvinfichter/status/1489041221947375616)

* **Event Summary**
    * 合约依赖了外部函数，这个外部函数是调用合约方法时通过参数传入的，合约并没有对该外部函数地址做校验（即校验被调用的方法是否属于系统合约）；
    * 使用了solana被遗弃的接口，在正准备升级的时候被攻击了；
    * 内部合约方法调用好像没有使用权限控制，导致整个流程可以构造假消息后，从中间开始调用；
    * 这一点和solana的开发机制有关；


## V神提出的跨链的问题

* 源于PoW公链可能存在的短暂分叉情况；


## THORChain

* 事件1详情：假充值（内容整理于[https://www.freebuf.com/articles/blockchain-articles/279120.html](https://www.freebuf.com/articles/blockchain-articles/279120.html)，侵删）
    * 事件摘要：代码漏洞，
        * 由于错误的定义（对于代币符号的默认值设成了ETH），如果跨链充值的 ERC20 代币符号为 **ETH**，那么将会出现逻辑错误，导致充值的代币被识别为真正的以太币 **ETH**。（该漏洞已经被修复）
* 事件2详情：[参考](https://thearchitect.notion.site/THORChain-Incident-07-15-7d205f91924e44a5b6499b6df5f6c210)
    * 时间：2021-07-15
    * 事件摘要：
        * 攻击者构造一个Wrapped Router合约C
        * 攻击者调用合约C，并且传入value为200，即200个ETH
        * C内部调用官方Router合约的deposit方法，传入value改为0
        * Router合约抛出事件Deposit，amount为0
        * 节点从Deposit事件取出的amount为0，但是又用tx.value方法获取到值200，从而攻击成功
    * 损失：约8M
    * 总结：
        * 需要对EVM的系统方法非常熟悉才去使用，否则可能踩坑
        * 尽量减少逻辑，使得跨链功能尽可能简单，减少出错概率
        * 逻辑主要放在合约中，节点只负责搬运
* 事件3详情：非法转账（内容整理于[https://www.tuoniaox.com/news/p-509320.html](https://www.tuoniaox.com/news/p-509320.html)，侵删）,[参考](https://thearchitect.notion.site/THORChain-Incident-07-22-874a06db7bf8466caf240e1823697e35)
    * 时间： 2021-07-22
    * 事件摘要：
        * 攻击者构造一个Wrapped Router合约C
        * 攻击者调用Router方法returnVaultAssets，设置asgard（接收者）参数为C
        * returnVaultAssets方法中调用`asgard.call{value:msg.value}("")`方法将msg.value转给C
        * C接收到ETH会触发onReceive或者fallback方法，C抛出Deposit事件，参数为想要获得的token信息
        * 节点处理Deposit事件，发现memo参数无法解析，会进入refund流程
        * 攻击者调用transferOut进行退款，将之前Deposit中设置的token退回到指定地址，攻击发生
    * 总结：
        * EVM的低级调用有一些副作用，需要妥善处理
        * 节点不应该承担过多的任务
* **对我们的启发**
    * 虽然在我们的场景中，跨链Token方面的操作在应用层合约中完成（如locker），但权限校验机制需要由底层跨链合约来提供；
    * 跨链协议层的SQOS确实非常重要；
