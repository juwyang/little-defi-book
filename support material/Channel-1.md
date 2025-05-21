> Q:介绍payment channel, 如何开启，需要谁的签名，谁抵押一定的资产，这个资产会被怎么消耗，是交易的双方都会支付手续费吗？每一笔链下的交易的手续费怎么计算，最后通道可以怎么关闭，可以由谁关闭，关闭的时候为开启通道而抵押的资产还剩多少，减少的那部分钱被谁赚了？payment channel的技术细节，它是一个智能合约吗？

*Thinking...*

> Examine: Payment Channels
> 
> Okay, I need to get a handle on payment channels. First, I'll find a solid, high-level explanation of the core concepts. This will be my foundation. Then I can start building from there.
> 
> 
> ### Examining Payment Channels - Initial Steps
> 
> Alright, I need a solid base. I'm kicking off by hunting for a clear, concise overview. Defining the core concept is priority one. From there, I'll delve deeper into opening, closing, and the mechanics of these payment channels.
> 
> 
支付通道 (Payment Channel) 是一种旨在解决区块链可扩展性问题的链下技术方案，它能让交易更快、更高效，并降低交易费用。简单来说，支付通道允许两个参与方在主区块链之外进行多次交易，只有在通道开启和最终关闭时才与主区块链交互。

**如何开启支付通道？**

1.  **创建多签名地址/智能合约**：支付通道的开启首先需要在区块链上创建一个特殊类型的地址或智能合约。在比特币中，这通常是一个2-of-2的多签名地址，意味着从该地址花费资金需要通道双方的签名。在以太坊等支持智能合约的平台上，会部署一个特定的智能合约来管理通道。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)
2.  **资金抵押（注资交易）**：通道的参与方（一方或双方）将一定数量的加密资产发送到这个多签名地址或智能合约中。这笔交易被称为“充值交易”或“锚点交易”，它会被广播到网络并记录在区块链上，标志着通道的正式开启和初始资金的锁定。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[3]](https://imnisen.github.io/lightning-network-payment-channel.html)
3.  **需要谁的签名**：开启通道的初始注资交易需要抵押资产一方的签名。在创建多签名地址或智能合约时，通常也需要双方的某种形式的同意或参与。例如，在2-of-2多签中，双方都需要参与到地址的创建过程中。 [[3]](https://imnisen.github.io/lightning-network-payment-channel.html)

**谁抵押一定的资产？**

通常情况下，参与支付通道的一方或双方都可以抵押资产。 [[3]](https://imnisen.github.io/lightning-network-payment-channel.html) 例如，如果Alice想频繁地向Bob支付，Alice可以单方面抵押资金开启一个单向通道；或者，Alice和Bob可以共同抵押资金，创建一个双向通道，允许资金在两者之间双向流动。 [[3]](https://imnisen.github.io/lightning-network-payment-channel.html)[[4]](https://www.btcstudy.org/2022/11/14/understanding-payment-channels/)

**这个资产会被怎么消耗？**

抵押在通道中的资产并不会在传统意义上“消耗”掉，而是其在通道内的分配状态会随着链下交易的发生而改变。 [[5]](https://aandds.com/blog/bitcoin-ln-basic.html)

*   **链下交易与承诺交易**：通道开启后，双方之间可以进行任意多次的链下交易。每一次交易都会生成一个新的“承诺交易 (Commitment Transaction)”。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[5]](https://aandds.com/blog/bitcoin-ln-basic.html) 这个承诺交易本身是一个有效的链上交易，它会重新分配通道内锁定的资金，反映双方最新的余额。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/) 例如，如果Alice最初抵押了10个币，她向Bob支付了1个币，那么新的承诺交易会表明Alice拥有9个币，Bob拥有1个币。
*   **不广播中间状态**：这些承诺交易在通道保持开放期间并不会立即广播到区块链上，双方只是互相交换签名并保存最新的承诺交易。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/) 这使得交易可以即时发生，几乎没有延迟。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)

**是交易的双方都会支付手续费吗？每一笔链下的交易的手续费怎么计算？**

*   **链上交易手续费**：开启和关闭支付通道时，由于需要在主区块链上进行交易（注资交易和结算交易），这两笔交易需要支付正常的链上交易手续费。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels) 这些费用由发起交易的一方或双方共同承担，具体取决于实现方式。
*   **链下交易手续费**：在支付通道内部进行的链下交易，其手续费非常低，甚至可以忽略不计。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)[[6]](http://pdf.dfcfw.com/pdf/H3_AP201906121334822764_1.pdf) 这是因为这些交易不直接与主区块链交互，避免了矿工费。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels) 有些支付通道网络（如闪电网络）中的中介节点可能会为路由支付收取少量费用。 [[7]](https://www.hanjiaming.com.cn/note/cs/cn/bc/channel.html)
*   **链下交易手续费计算**：由于链下交易不直接上链，其“手续费”更多的是通道参与者之间或通过网络中介节点路由支付时的一种协议。在简单的两方通道中，可能没有明确的每笔链下交易手续费。在更复杂的支付通道网络中，路由节点可能会根据交易金额或跳数等因素收取少量费用。 [[8]](https://www.themoonlight.io/zh/review/an-analysis-of-the-correctness-and-computational-complexity-of-path-planning-in-payment-channel-networks)

**最后通道可以怎么关闭？**

支付通道可以通过以下几种方式关闭：

1.  **双方协商关闭 (Cooperative Close)**：这是最理想的关闭方式。通道双方达成一致，共同签署一个最终的结算交易 (Settlement Transaction)，该交易将通道内锁定的资金按照最新的余额分配给双方。这个结算交易会被广播到区块链上，通道随之关闭。 [[5]](https://aandds.com/blog/bitcoin-ln-basic.html)[[9]](https://www.btcstudy.org/2020/08/24/understanding-the-lightning-network-part-completing-the-puzzle-and-closing-the-channel/)
2.  **单方面关闭 (Unilateral Close)**：如果一方不同意关闭或者失联，另一方可以单方面关闭通道。 [[5]](https://aandds.com/blog/bitcoin-ln-basic.html)[[9]](https://www.btcstudy.org/2020/08/24/understanding-the-lightning-network-part-completing-the-puzzle-and-closing-the-channel/) 这时，单方面关闭的一方会将他们持有的最新的、经过双方签名的承诺交易广播到区块链上。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)
    *   **争议期/挑战期**：为了防止一方广播一个过时的、对自己有利的承诺交易来进行欺诈，通常会有一个“争议期”或“挑战期”。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels) 在此期间，另一方如果拥有更新的承诺交易（通常通过某种机制如惩罚机制来证明其更新性），可以提交该交易来纠正最终的资金分配，并可能惩罚欺诈方。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)[[4]](https://www.btcstudy.org/2022/11/14/understanding-payment-channels/)

**可以由谁关闭？**

通常情况下，支付通道的任何一方都可以发起关闭请求。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[9]](https://www.btcstudy.org/2020/08/24/understanding-the-lightning-network-part-completing-the-puzzle-and-closing-the-channel/)

**关闭的时候为开启通道而抵押的资产还剩多少？减少的那部分钱被谁赚了？**

*   **剩余资产**：关闭通道时，最初抵押的资产总额减去在通道内发生的净支付额，就是剩余的资产。这些剩余资产会根据最新的承诺交易或最终的结算交易分配给通道双方。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[5]](https://aandds.com/blog/bitcoin-ln-basic.html) 例如，如果Alice最初存入10 BTC，并通过通道向Bob支付了总计3 BTC，那么在通道关闭时，Alice将获得7 BTC，Bob将获得3 BTC（假设Bob最初未存入资金）。
*   **减少的钱被谁赚了**：
    *   **交易对手方**：减少的那部分钱实际上是支付给了通道的另一方。 [[5]](https://aandds.com/blog/bitcoin-ln-basic.html)
    *   **链上交易费**：开启和关闭通道时的链上交易费用会被矿工赚取。 [[10]](https://cryptomus.com/zh/blog/what-is-the-blockchain-network-commission)
    *   **中介节点（支付通道网络中）**：如果在支付通道网络中通过中介节点路由支付，这些中介节点可能会收取少量路由费用。 [[7]](https://www.hanjiaming.com.cn/note/cs/cn/bc/channel.html)
    *   **惩罚机制**：在某些设计中，如果一方试图欺诈（例如广播旧的承诺交易），另一方成功挑战后，欺诈方可能会损失其部分甚至全部抵押资产，这部分资产可能会被诚实的一方获得。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)[[11]](https://www.btcstudy.org/2020/08/23/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel/)

**Payment channel的技术细节，它是一个智能合约吗？**

*   **技术细节**：支付通道的核心技术依赖于密码学原语（如哈希锁、时间锁）和区块链的特性（如多重签名、交易可替换性）。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[4]](https://www.btcstudy.org/2022/11/14/understanding-payment-channels/)
    *   **承诺交易 (Commitment Transactions)**：双方不断签署新的承诺交易来更新通道状态，旧的承诺交易通过某种机制失效（例如，通过交换秘密值来作废旧承诺，或者通过递减的时间锁使得新交易优先）。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[3]](https://imnisen.github.io/lightning-network-payment-channel.html)
    *   **哈希时间锁合约 (HTLC - Hashed Time-Lock Contract)**：这是实现更复杂支付通道网络（如闪电网络）的关键技术。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[5]](https://aandds.com/blog/bitcoin-ln-basic.html) HTLC允许在不信任中介的情况下进行多跳支付，并确保支付的原子性（要么全部成功，要么全部失败）。 [[5]](https://aandds.com/blog/bitcoin-ln-basic.html)[[7]](https://www.hanjiaming.com.cn/note/cs/cn/bc/channel.html) 它结合了哈希锁（知道秘密值才能解锁资金）和时间锁（超过一定时间资金可以退回）。 [[5]](https://aandds.com/blog/bitcoin-ln-basic.html)
    *   **惩罚机制**：为防止欺诈，许多支付通道设计了惩罚机制。如果一方试图广播一个过时的状态，另一方可以提交证据（例如，一个特定的秘密值），从而获得通道内更多的资金，甚至对方的全部抵押。 [[4]](https://www.btcstudy.org/2022/11/14/understanding-payment-channels/)[[11]](https://www.btcstudy.org/2020/08/23/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel/)
*   **是否是智能合约**：
    *   在像比特币这样的区块链上，支付通道可以通过其脚本语言（一种非图灵完备的智能合约）来实现，例如利用多重签名和时间锁操作码（如`OP_CHECKLOCKTIMEVERIFY` - CLTV, `OP_CHECKSEQUENCEVERIFY` - CSV）。 [[3]](https://imnisen.github.io/lightning-network-payment-channel.html)[[4]](https://www.btcstudy.org/2022/11/14/understanding-payment-channels/)
    *   在像以太坊这样支持图灵完备智能合约的区块链上，支付通道通常是通过部署专门的智能合约来实现的。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)[[12]](https://patents.google.com/patent/CN111325628A/zh) 这些智能合约负责管理资金的锁定、状态更新的验证以及争议的解决。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)
    *   因此，可以说支付通道的实现可以基于智能合约，尤其是在功能更强大的区块链平台上。 [[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels) 状态通道是支付通道的更通用形式，它可以处理任意的状态更新，而不仅仅是支付，并且也依赖于智能合约。 [[6]](http://pdf.dfcfw.com/pdf/H3_AP201906121334822764_1.pdf)[[13]](https://www.bcskill.com/index.php/archives/965.html)

总而言之，支付通道是一种强大的二层扩容技术，通过将大部分交易移至链下处理，显著提高了交易速度和效率，降低了成本，同时利用底层区块链的安全性来保证最终结算的可靠性。 [[1]](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)[[2]](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)

---
Learn more:
1. [闪电网络深入解读（上）：支付通道 - BTC study](https://www.btcstudy.org/2021/09/15/lightning-network-in-depth-part-1-payment-channels/)
2. [什么是支付通道？](https://www.nervos.org/zh/knowledge-base/what_are_payment_channels)
3. [理解闪电网络1——支付通道(payment channel)](https://imnisen.github.io/lightning-network-payment-channel.html)
4. [支付通道演变史 - BTC study](https://www.btcstudy.org/2022/11/14/understanding-payment-channels/)
5. [Lightning Network](https://aandds.com/blog/bitcoin-ln-basic.html)
6. [状态通道：链下交互链上清算，两条腿走路](http://pdf.dfcfw.com/pdf/H3_AP201906121334822764_1.pdf)
7. [Payment Channel | 韩佳明& Hirsun - 韩佳明的博客](https://www.hanjiaming.com.cn/note/cs/cn/bc/channel.html)
8. [\[论文审查\] An Analysis of the Correctness and Computational Complexity of Path Planning in Payment Channel Networks](https://www.themoonlight.io/zh/review/an-analysis-of-the-correctness-and-computational-complexity-of-path-planning-in-payment-channel-networks)
9. [理解闪电网络，Part-3：结算并关闭支付通道 - BTC study](https://www.btcstudy.org/2020/08/24/understanding-the-lightning-network-part-completing-the-puzzle-and-closing-the-channel/)
10. [什么是区块链网络手续费• 博客Cryptomus](https://cryptomus.com/zh/blog/what-is-the-blockchain-network-commission)
11. [理解闪电网络，Part-1：构建比特币的双向支付通道 - BTC study](https://www.btcstudy.org/2020/08/23/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel/)
12. [CN111325628A - 一种基于区块链的多方支付通道交易方法 - Google Patents](https://patents.google.com/patent/CN111325628A/zh)
13. [理解以太坊的Layer 2扩容解决方案：状态通道（State Channels）、Plasma 和Truebit](https://www.bcskill.com/index.php/archives/965.html)