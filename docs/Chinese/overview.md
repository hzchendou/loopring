## Loopring 是什么
Loopring是一个去中心化的代币交换协议，在以太坊公链上，使用智能合约实现Loorping去中心化交易系统中的核心功能。与传统中心化交易所相比，Loopring带来了一系列提升，包括如下几个重要方面：

  - 降低交易风险：不需要将资产交由第三方管理，所有的操作都掌握在交易者手中，保障了数字资产安全。
  - 高流动性：Loopring独创的环路撮合技术使得任何资产交易对都可以参与订单撮合，提高了资产间的流动性
  - 公平性：手续费以及优惠机制能够保障各个参与方的利益均衡（这里的利益均衡并不是利益平分，而是各个参与方都可以制定符合自己利益的规则，在保障规则的前提下执行），这些参与方包括：订单创建者、订单购买者以及订单撮合矿工。

如果你想了解更多关于Loopring协议研发背景、设计思想信息，建议参考Loopring官方[白皮书](http://hzchendou-loopring.oss-cn-hangzhou.aliyuncs.com/zh_whitepaper.pdf)，需要指出的是，官方团队在实际开发实现时，实现方式与当前白皮书描述的方式有出入，如果有不同之处，一切以官方网站信息为准。

## 为什么需要开发Loopring协议

Loopring协议用于解决一系列中心化交易所机制带来的不可避免的问题，这些问题在本文[最后小节](overview.md#_2)中有相关阐述。

## Loopring生态系统

本节将介绍Loopring生态中的重要组件，简要介绍它们的功能以及组件之间的联系。这些组件组合在一起完成了中心化交易所提供的功能，各个组件的详细内容将在接下来章节进行详细描述。

**[钱包：Wallets](./projects/wallet.md)**: 钱包提供了基础钱包服务，使得用户可以通过钱包管理代币，连接到Loopring网络，发送订单交易信息。

**[中继器：Relays](./projects/relay.md)**: 中继器维护公开订单账本信息以及交易历史信息，广播新订单，通知其他中继器以及环路撮合矿工。环路撮合矿工通常作为中继器的一个功能，环路撮合矿工承担了大量计算任务，这些计算任务是在链下完成的，这样可以节省链上交易成本，这个计算任务至少包含两种代币信息，完成的撮合订单称之为订单环（order ring）

**[Loopring智能合约：Loopring Protocol Smart Contracts](./projects/protocol.md)**:  Loopring 包含了一系列智能合约，对矿工提交的环路订单进行校验，校验合格的订单将按照用户提交的订单信息进行代币转账，同时合约包含了激励机制，可以激励撮合成功的矿工，触发事件。中继器、订单浏览器监听到相关事件后更新订单账本信息并将订单添加到交易历史信息。

**[资产代币化服务：Asset Tokenization Services](./projects/tokenization.md)**: 资产代币化服务为为不可直接在Loopring上交易的资产提供了转换桥梁，这是一种中心化服务，由可信任的第三方公司或组织提供。用户通过资产代币服务商抵押自己的资产，得到发行的代币。Loopring不具备跨链交易能力，但是通过资产代币化服务，让Loopring可以与现实资产以及其它的链上资产进行交易。

资产代币化服务并不包含在当前的Loopring项目规划内，资产代币化服务的出现将进一步完善Loopring生态

## 预览

下图描述了一个正常的订单在Loopring网络中的生命周期信息，包含了一系列步骤。

![](/img/diagrams/loopring-overview.png)


**用户发起一笔交易**: 用户先要用`Y`个数量的`TokenB`换取`X`个数量的`TokenA`，当前Loopring网络中存在多个未成交订单满足用户交易条件（例如：订单浏览器）。一旦用户准备交易，填写完整的订单信息，利用钱包接口提交这笔订单，发起交易。一定数量的LRC（Loopring生态中的一种ERC20代币，由loopring官方发布）可以作为矿工手续费添加到订单信息中，当然手续费用越高，矿工将优先处理。

**ERC20代币交易授权**:  钱包授权Loopring智能合约可以处理用户想要出售的数量`x`个`TokenA`。用户的授权操作并将代币锁定在智能合约中，用户随时可以对代币进行转账操作，直到订单在区块链上被确认。在交易时刻，矿工或者是Loopring将会检查用户账户余额信息，当余额不满足当前交易金额时，成交数量将会按比例进行缩减，这种按比例缩减的订单操作与订单取消操作是两种不同的机制：当满足条件的金额充值到当前地址时，按比例缩减订单将会自动按比例扩大到原先交易金额，而被取消的订单不能再次恢复到取消前的状态。

**发送订单信息到网络节点**: 一旦订单中的信息验证通过，钱包将会把已签名的订单信息发送到网络中的节点（中继器）

**广播订单信息**: 一旦中继器接受到订单信息，将会立即更新订单账本信息，然后将订单广播到其他中继器节点或者是矿工节点，让它们开始处理订单。

**订单环路撮合（订单撮合匹配）**: 环路撮合矿工接收到订单信息，首先将订单信息更新到订单账本中， 矿工将会按照订单中的兑换比例（或者是更划算的比例）使用环路撮合技术与其它订单进行匹配尽可能地完成订单部分或者是全部交易金额。这种环路撮合技术是提高资产流动性的主要原因，因为这种技术能够使得不同的交易对参与撮合，而不仅仅是特定的交易对。如果订单实际兑换比例比原先更加优惠，那么在满足订单交易的基础上剩余一部分资产，这部分资产被称作利润。矿工可以选择自己的报酬，矿工可以选择保留利润，但是必须将订单中的LRC返还给用户，已可以选择保留LRC作为报酬。

**验证&结算**:当Loopring智能合约接收到订单环路信息时，将会使用多种方式对订单环路信息进行校验，校验通过后，将按照环路信息决定环路中的订单是否能够进行完整结算或者是部分结算（这依赖于订单兑换比例，以及用户账户中的代币余额信息）。智能合约将按照订单信息进行转账操作，并且支付矿工报酬，这一步骤是自动的。

为了便于理解，本文省略了订单在Loopring协议网络中的部分细节信息。如果想进一步理解协议（订单取消、环路撮合、订单账本以及交易历史信息等等）建议读者浏览本文列举的Loopring生态中的[关键组件信息](overview.md#loopring_2).。



## 中心化交易所弊端

为了更好地理解像Loopring这样交易所的优势，首先介绍一下中心化交易所系统运行原理，下图简要地展示了用户发送代币到中心化交易系统所发生的事情

![](/img/diagrams/centralized-model.png)

为了使用交易系统进行交易，首先使用者必须发送自己的数字资产（Token）到交易所，当交易系统钱包接受到你的代币时，将会给你一张拮据，这张拮据信息描述了你的资产，然后你可以使用这张拮据与其他用户的拮据进行交易。交易完成后，你依据拮据信息要求交易系统将相应数量代币返还到你的钱包。

**缺乏安全**: 在这种模型中，你无法控制你的代币，你可以使用IOU（拮据）进行即时交易，但同时已产生了一定风险。在多种情况下，你将失去自己的资产（冻结账户，交易所关闭，黑客攻击，系统缺陷等等）。

**缺乏透明度**: 当你将自己的资产充值到交易所钱包时，你将失去控制权，任何事情都可能发生，然后你将对此一无所知。用户永远无法预料交易所运营情况，当交易所宣布破产时，对用户来说已为时已晚。

**缺乏流动性**: 你只允许在交易所特定的订单池中进行交易，只支持特定交易对。如果没有足够的交易量，你只能尝试其它交易所，如果你想要的兑换对不存在时，只能先将代币转换成其它代币，然后将代币兑换成想要的代币，或者是将代币转账到其它交易所，然后这将导致你耗费而外的时间以及手续费，甚至是丢失资产。

上述只是中心化交易所包含的部分弊端，我们将不会在此进一步讨论其它弊端，如果你对此感兴趣可以在网络上查找相关内容。
