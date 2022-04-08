# 跨链攻击事件

## Ronin
* Time: 03-23-2022, Event link:
[1](https://learnblockchain.cn/article/3810), [2](https://news.bitcoin.com/axie-infinity-loses-620-million-after-hacker-compromised-ronin-validators/)

* Reason:
    * The hacker compromised five validators' private key;
* My Opinion:
    * The selection mechanism of off-chain validators is inadequate:
        * Not much enough;
        * Lack of randomness, which makes it easier to conspire together to commit malicious acts;

## chainswap

* 合约漏洞：（内容整理于[https://zhuanlan.zhihu.com/p/389738041](https://zhuanlan.zhihu.com/p/389738041)，侵删）；
    * 众所周知，Solidity是没有类似Python中**KeyError**的概念的，如果映射（mapping）中不存在这个键，则会返回0，**然而这里没有对返回值进行检查；**
    * 导致任意攻击者，只自己随机生成地址并提供签名，则都能冒充成验证者（mapping失败返回0，但没有进行检查）；
    * authQuotaOf在实现逻辑上的错误：




    * 总结一下，receive函数实现的整个过程，都没有对传入的Signatory的合法性进行检查。因此攻击者只需要随机生成一个地址并生成对应的签名，即可骗过ChainSwap，自己为自己提供签名。同时，由于authQuotaOf在实现逻辑上的错误，在Signatory不合法时会返回一个非常大的值，导致了这次攻击事件的发生。而本次攻击事件发生的本质，是没有对映射索引的值进行验证。由于Solidity并不会在映射的键不存在时触发任何错误（键是否存在只能靠返回值是否为0进行判断），因此这类检查就显得非常重要。正是由于（荒谬地）缺少这样的检查，导致了这次损失超过800万美元的攻击；
* 验证者是自己选的，但以上漏洞导致任意用户都可以冒充：
    * 有趣的是，在管理者Factory的合约实现中，ChainSwap设计了一个变量来保存Signatory构成的数组，代码如下图所示：

    * 有趣的是，在管理者Factory的合约实现中，ChainSwap设计了一个变量来保存Signatory构成的数组，但很遗憾的是，通过对Factory合约实现的审计，我们发现这个变量仅仅是在管理Signatory配额的时候被使用，并没有验证一个任意地址是否包含在这个数组中的实现。
* **对我们的启发：**
    * 我们合约中对验证的管理，在solidity视线中，也需要确保在代码中对每一次mapping查找结果的检查；


## Poly Network

* 问题所在（内容整理于[https://www.freebuf.com/vuls/284340.html](https://www.freebuf.com/vuls/284340.html)，侵删）：
    * 原文：
        * 经过以上分析，结果已经很明确了，攻击者只需在其他链通过 crossChain 正常发起跨链操作的交易，此交易目的是为了调用 EthCrossChainData 合约的 putCurEpochConPubKeyBytes 函数以修改 Keeper 角色。随后通过正常的跨链流程，Keeper 会解析用户请求的目标合约以及调用参数，构造出一个新的交易提交到以太坊上。这本质上也只是一笔正常的跨链操作，因此可以直接通过 Keeper 检查与默克尔根检查。最后成功执行修改 Keeper 的操作。
        * 但我们注意到 putCurEpochConPubKeyBytes 函数定义为
```plain
function putCurEpochConPubKeyBytes(bytes calldata curEpochPkBytes) external returns (bool);
```
        * 而_executeCrossChainTx 函数执行的定义为
```plain
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

* 私钥被破解
    * 原因是R签名算法，两次签名使用了相同的随机数；
* 1.18再次被攻击：（内容整理于[https://medium.com/zengo/without-permit-multichains-exploit-explained-8417e8c1639b](https://medium.com/zengo/without-permit-multichains-exploit-explained-8417e8c1639b)，侵删）
    * **该攻击发生在源链；**


    * “1”中没有对token地址进行合法性验证，只是生成相关的“any”Token的调用接口，underlying直接映射成了真正的Token地址（如案发现场的WETH）；
    * “2”中，实际上相关的真实Token ERC20没有实现permit（**相当于做外包验证**），执行fallback后也没有错误提示，导致后续代码正常执行，**这一步是所有问题的关键**；（这是solidity不严谨导致的问题）
    * “3”中没有对token地址进行验证，按理说，“any”Token是anyswap自己生成的，但这里却没有去进行验证是否是自己生成的那个地址。在这次攻击中，该合约是攻击者伪造的，那么这里实际上把WETH转到了攻击者控制的合约中；
    * “4”的问题实际上这次攻击中没有真正发生，因为在“3”执行后攻击者已经拿走token。“4”的问题是，如果真实Token ERC20没有实现permit，那么“2”验证会假通过，攻击者依然可以通过“4”完成正常跨链，然后在目的链中将资产取走。
## WormHole

[https://www.jinse.com/news/blockchain/1178392.html](https://www.jinse.com/news/blockchain/1178392.html)

[https://blog.csdn.net/mutourend/article/details/122807577](https://blog.csdn.net/mutourend/article/details/122807577)

[https://twitter.com/kelvinfichter/status/1489041221947375616](https://twitter.com/kelvinfichter/status/1489041221947375616)

* **事件根本原因**
    * 合约依赖了外部函数，这个外部函数是调用合约方法时通过参数传入的，合约并没有对该外部函数地址做校验（即校验被调用的方法是否属于系统合约）；
    * 使用了solana被遗弃的接口，在正准备升级的时候被攻击了；
    * 内部合约方法调用好像没有使用权限控制，导致整个流程可以构造假消息后，从中间开始调用；
    * 这一点和solana的开发机制有关；


## V神提出的跨链的问题

* 源于PoW公链可能存在的短暂分叉情况；


## THORChain

* 假充值（内容整理于[https://www.freebuf.com/articles/blockchain-articles/279120.html](https://www.freebuf.com/articles/blockchain-articles/279120.html)，侵删）
    * 代码漏洞，
        * 由于错误的定义（对于代币符号的默认值设成了ETH），如果跨链充值的 ERC20 代币符号为 **ETH**，那么将会出现逻辑错误，导致充值的代币被识别为真正的以太币 **ETH**。（该漏洞已经被修复）
* 非法转账（内容整理于[https://www.tuoniaox.com/news/p-509320.html](https://www.tuoniaox.com/news/p-509320.html)，侵删）
    * 原因：
        * THORChain Router合约的TransferOut函数的漏洞（没有对账户金额进行判断）；
        * Token合约的接口缺陷；
    * 攻击者之所以可以实现这样的攻击，是因为THORChain Router合约的TransferOut函数漏洞导致--使用asset.call(abi.encodeWithSignature("transfer(address,uint256)" , to, amount))语句进行转账;
    * 管理Token的合约，其 `transfer` 方法，为"transfer(address,uint256)" 形式，address为to地址，uint256为金额：
        * 这样的transfer只能在内部以 `msg.sender` 来获取到"from"地址，那么此时就变成了"THORChain Router"合约的地址；
        * 但是THORChain Router的TransferOut中在调用asset.call之前并没有判断调用者是否有足够的金额进行转账；
* **对我们的启发**
    * 虽然在我们的场景中，跨链Token方面的操作在应用层合约中完成（如locker），但权限校验机制需要由底层跨链合约来提供；
    * 跨链协议层的SQOS确实非常重要；
# 