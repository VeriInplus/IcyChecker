# IcyChecker
A Intention-to-Code consistency checker for  Smart Contracts

> 一个智能合约代码意图一致性检测工具

## 项目概述

本项目是InplusLab与Webank关于智能合约意图一致性检测算法的研究内容，旨在利用形式化验证等方法验证编写合约与意图是否一致，这其中包含了功能性与非功能性的需求。该项目旨在追求最大程度的用户友好和自动化，能够让没有过多形式化验证经验的使用者也能顺利使用该工具。理想情况下，用户仅需要编写简单的规约即可自动完成智能合约的一致性验证。

### 依赖工具

本项目基于现有的两个工具进行研究和开发，目前主要是第二个：

* [VeriSol](https://github.com/utopia-group/verisol)
* [SmartPulseTool](https://github.com/utopia-group/SmartPulseTool/tree/master)

## 快速上手

目前，本仓库除了提供部分新案例和可直接使用运行的docker外，暂未加入太多修改。

#### 1 直接使用

由于目前使用的两个工具有较多的依赖项，暂时还无法将其简单进行合并，实现工具的即插即用。因此，我们提供了一个已经配置好的Ubuntu容器（3.49G），它已经配置好了所有内容，工具位于`/tool/smartpulse/SmartPulse`目录下，点击此处下载。你需要做的仅仅是`docker import`和`docker run`，使用交互式命令`/bin/bash`启动，并首先运行`source /etc/profile`。

#### 2 逐步配置

直接参考依赖工具构建方法，点击[原始构建](originBuild.md)查看如何构建。

如要了解当前工具的细节，建议采用此方法。除本文档以外，也可以参考原仓库。

## 使用方法

在使用之前，[了解规约的含义是有必要的](spec.md)。

本节将列举两个极其常见的案例，众筹和竞拍，两个案例都涉及到了自由变量，是较为复杂的规约形式，在大多数的情况下，只需要属性就足够了。我们提供了一个优化版的测试数据集，你可以克隆或下载本仓库内容，所有测试用例位于`/data`文件夹下。在使用时，可以将文件夹中内容置于`/SmartPulse`根目录下。你也可以从依赖工具原始仓库中获取论文相关的所有benchmark和运行结果。

**众筹**

如何出资人没有取走钱，那么合约中保留了出资人的出资记录。

```
// #LTLVariables: b:Ref,v:int
// #LTLFairness: [](!started(Crowdfunding.Claim, msg.sender == b))
// #LTLProperty: [](finished(Crowdfunding.Donate, msg.sender == b && msg.value == v && v != 0) ==> [](!finished(*, this.backers[b] != v))) 
```

进入到`/SmartPulse`目录下运行

```shell
./SmartPulse.py Crowdfunding.sol Crowdfunding CrowdfundingSpec.spec
```

**竞拍**

> 长合约示例，可能需要较长时间。

竞拍完全结束之后，用户可以取回他之前竞拍时投入的所有资金。	

```
// #LTLVariables: user:Ref
// #LTLFairness: (<>(finished(ValidatorAuction.withdraw, (user == msg.sender))))
// #LTLProperty: []((finished(ValidatorAuction.closeAuction)) ==> (<>(finished(send(from, to, amt), (to == user && amt == fsum(ValidatorAuction.bid, 2, (user == msg.sender)))))))
```

运行

```shell
./SmartPulse.py ValidatorAuction.sol ValidatorAuction ValidatorAuctionSpec.spec 
```

说明：

- 更多的例子在`/data`下，该目录下还包含了VeriSol工具提供的示例，以及工具[Verx](https://github.com/eth-sri/verx-benchmarks)的数据集。你可以在其中找到更多更简单或复杂的用例。
- 值得说明的是，当前工具所能识别的规约一般以`.spec`为后缀，但也可以不是，并且一个合约可以验证的规约一般来说有多个。
- 可以将结果重定向输出为`.log`文件，如果你想直接运行`.bpl`文件，可通过使用`SmartPulse.sh xxxx.bpl`来实现。

## 参考资料

* [Formal Verification of Workflow Policies for Smart Contracts in Azure Blockchain.](https://doi.org/10.1007/978-3-030-41600-3_7)
* [SmartPulse: Automated Checking of Temporal Properties in Smart Contracts.](https://doi.org/10.1109/SP40001.2021.00085)

