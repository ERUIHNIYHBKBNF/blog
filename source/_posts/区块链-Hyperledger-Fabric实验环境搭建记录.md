---
title: '[区块链] Hyperledger Fabric实验环境搭建记录'
date: 2022-11-28 22:22:56
tags: [区块链, web3]
---

Hyperledger Fabric安装：[安装示例、二进制和 Docker 镜像 ‒ hyperledger-fabricdocs master 文档](https://hyperledger-fabric.readthedocs.io/zh_CN/latest/install.html)

文档地址：[编写你的第一个应用 ‒ hyperledger-fabricdocs master 文档](https://hyperledger-fabric.readthedocs.io/zh_CN/latest/write_first_app.html)

> 注意这边中文文档这里是js的fabCar例子，英文文档给的是另一个ts的例子，作者因为课程实验要求按中文文档来的(2022.11.28)

Github例子地址：https://github.com/hyperledger/fabric-samples

## 这是什么

粘贴自 [Hyperledger Fabric是什么_开源区块链框架介绍-AWS云服务](https://aws.amazon.com/cn/blockchain/what-is-hyperledger-fabric/)

> Hyperledger Fabric 是一种开源许可区块链框架，由 The Linux Foundation 在 2015 年开始提供。它采用模块化通用型框架，所具有的独一无二的身份管理及访问控制功能使其非常适用于各种行业应用，例如，供应链的追踪和跟踪、贸易融资、忠诚度和奖励，以及金融资产的清算和结算等。

总之这篇文章就是自己用Hyperledger Fabric搭建一套实验性联盟链网络的记录。

## Hyperledger Fabric 相关概念

区块链，一句话就是分布式数据库，相关内容可以看看咱之前的（虽然写得自己也看不懂就是了）[链接戳这里](https://eruihniyhbkbnf.github.io/blog/2022/07/11/区块链简单了解一下/)

这里概念看不懂似乎没什么关系（反正咱一开始也没看懂），手动搭一下网络看看交易流程就懂了qwq

Hyperledger Fabric网络架构：

![img](https://tefoq5o3f4.feishu.cn/space/api/box/stream/download/asynccode/?code=MDI5MzAzNWM0ZjY2NWI0MDdiM2UxNzUxZDI5NzhhMjBfQTRSaExtSFpad2tLNHdRNVVNSzI1WnB4UzFoelRNdHZfVG9rZW46Ym94Y255SjJLMmpRcVhNN2k2eXJCbjBuSExiXzE2Njk2NDU0NTU6MTY2OTY0OTA1NV9WNA)

每个颜色代表一个组织（Organization），组织由Peer/Order节点构成

组织通过信道（Channel）组成联盟（Consortium），同一个联盟可以共享一些数据。

相关图例解释：A: Application, S: Smart Contract, P: Peer, L: Ledger, O: Order, NC: Network Configuration, CA: Certificate Authentication, R: Administrator of a network, CC: Chain Code

智能合约：相当于接口，提供给区块链应用程序，告诉应用程序可以进行什么操作

补充一个World State的概念：区块链可能存在记录没有及时同步的情况，但**主要起到的作用是保持历史操作记录不可篡改性**。**World State内存储最新数据，Ledger内存储操作记录。**也就是说，数据未及时同步时，新传来的事务仍然要写入区块链（那当然要写总不能不写叭qwq），由此产生的最新数据存到一个传统的数据库（例如Mysql）内，称之为世界状态。

Peer节点的类型：

![img](https://tefoq5o3f4.feishu.cn/space/api/box/stream/download/asynccode/?code=NGNjNjQyOWNjYjM2MjMyN2Q3YWY2MTIwODA3ZWMxOGZfbnFxUHFXNnB2aGgxYkM3WjdoUDZWWU5TUElPMlc0dlFfVG9rZW46Ym94Y24zSEpuQVg4RXJFb2FkekRIdEFjbm1lXzE2Njk2NDU0NTU6MTY2OTY0OTA1NV9WNA)

![img](https://tefoq5o3f4.feishu.cn/space/api/box/stream/download/asynccode/?code=YTAxZGFlNjNmNjI5OGNjN2RmZTNiNGNmOTZkZWMwM2JfaElLT081VGZEMGZBekNRRzdTYmdkaWVRZW5Qb0l4bk5fVG9rZW46Ym94Y25ZU2JSV0NvODN0NFg2WjVKTUZjY1FlXzE2Njk2NDU0NTU6MTY2OTY0OTA1NV9WNA)

## Hyperledger Fabric 事务流程

![img](https://tefoq5o3f4.feishu.cn/space/api/box/stream/download/asynccode/?code=YmEzOTJiYTY1NGQxMDNjYzZlMmQ4YWY4YWY2M2QxZmFfSkZRNlhDNHRDdG04QnRnU0VEN1ZvdVNWM1RFeWh5TDZfVG9rZW46Ym94Y245cGJtN05HbEc4andWVTNLVmZ2bnJkXzE2Njk2NDU0NTU6MTY2OTY0OTA1NV9WNA)

继续从Amazon粘：

> 1.当客户端应用程序向每个组织中的对等节点发送事务提议以寻求获得背书时，事务处理流程就会启动。
>
> 2.对等节点会验证提交客户端的身份与提交该事务的权限。接下来，它们会模拟提议事务的结果，如果与预期相符，则将背书签字发回到客户端。
>
> 3.客户端会从对等节点收集背书，一旦收到背书策略中定义的适当数量背书，它会将事务发到排序服务。
>
> 4.最后，排序服务会检查该事务是否具有适当数量的背书，从而满足背书策略的要求。然后，它会以时间顺序对事务进行排序，将已获得批准的事务打包置于区块中，并且发送此类区块到每个组织的对等节点。对等节点从排序服务收取新的事务区块，接着对该区块中的事务进行最后验证。一旦验证完成，新区块会被添加到分类账，分类账的状态也会得到更新。新事务便提交完毕了。

## Hyperledger Fabric 安装

按照这个来：[安装示例、二进制和 Docker 镜像 ‒ hyperledger-fabricdocs master 文档](https://hyperledger-fabric.readthedocs.io/zh_CN/latest/install.html)

MacOS m1芯片用户先自己滚了QAQ：

![img](https://tefoq5o3f4.feishu.cn/space/api/box/stream/download/asynccode/?code=MDFlMzk4ODJlZmZkMjg3NmI2ZDdjYzA3MWU5ZjM0MGFfMlBrM3dhWml3Vmd3TWhZbE5PSEtXQUkxWHNld0ZtVXFfVG9rZW46Ym94Y25nNXZ2bnJSRDRJUE1ieGlPWFc5dzRlXzE2Njk2NDU0NTU6MTY2OTY0OTA1NV9WNA)

整了台服务器弄这个qwq

安装前置条件：

```Bash
docker
docker-compose
go >= 1.13 # hyperledger fabric需要用go来编译，如果手动安装记得配环境变量
node >= 8 # 非必需，但官网文档例子都是用node的
```

之后执行文档里的命令：

```Bash
curl -sSL https://bit.ly/2ysbOFE | bash -s
```

网络问题可以下载脚本看里面内容手动执行，脚本主要做了这些事情：

- 克隆fabric-samples仓库

- 将 hyperledger fabric 平台特定的二进制文件和配置文件安装 到 fabric -samples的 /bin 和 /config 目录中（就是上图的压缩包，和另一个ca的压缩包，脚本里都能找到链接。下载后都解压到fabric-samples的bin和config文件夹即可）

- 下载指定版本的 hyperledger fabric docker 镜像

## FabCar 示例网络搭建

之后可能看起来就比较顺利了：[编写你的第一个应用 ‒ hyperledger-fabricdocs master 文档](https://hyperledger-fabric.readthedocs.io/zh_CN/latest/write_first_app.html)

多翻翻源码可能好理解一些，源码也不算复杂qwq

首先安装build-essential：`sudo apt install build-essential`

另外还需要jq，可能一些linux是默认不自带的：`sudo apt install jq`

运行脚本，这边使用typescript javascript：

> 尝试使用ts报错：Error: endorsement failure during invoke. response: status:500 message:"error in simulation: failed to execute transaction 73087c3304017e5862b1d1021304f30fdf15882ebc615b6663aae8cc119ce709: could not launch chaincode fabcar_1:2e32781f4c99094f69aba39d0179fb8997c5e1e79f65e342f370427a4cb62846: chaincode registration failed: container exited with 1"
>
> 搜到的Ref: https://stackoverflow.com/questions/60619391/hl-fabric-2-0-external-chaincode-launcher-error-in-simulation-failed-to-execu
>
> 目前原因不明，遂采用js

```Bash
cd fabcar
./startFabric.sh javascript
```

报错信息藏的很深仔细找找（指跟日志一起打印，不换号，没颜色QAQ）

网络创建成功后，可以看到如下消息：

```Bash
JavaScript:

  Start by changing into the "javascript" directory:
    cd javascript

  Next, install all required packages:
    npm install

  Then run the following applications to enroll the admin user, and register a new user
  called appUser which will be used by the other applications to interact with the deployed
  FabCar contract:
    node enrollAdmin
    node registerUser

  You can run the invoke application as follows. By default, the invoke application will
  create a new car, but you can update the application to submit other transactions:
    node invoke

  You can run the query application as follows. By default, the query application will
  return all cars, but you can update the application to evaluate other transactions:
    node query
```

之后可以按照官网文档愉快的先玩耍一波啦，顺着官方文档还可以看到智能合约里有什么，就很像各种接口叭qwq

## Transaction Control Flow 详细解析

引用官网图片：

![img](https://tefoq5o3f4.feishu.cn/space/api/box/stream/download/asynccode/?code=OTI5OGRkNGQ2ZGM1ZDZiM2NkNTE2NWVkYTg3MTUxOTBfNWFZQmttbFZJeHdqQmU5RFR4SmdVODNhNFhxd0FLT3hfVG9rZW46Ym94Y251TVJueW9BTXFqd245TDFiSEZHZ0xjXzE2Njk2NDU0NTU6MTY2OTY0OTA1NV9WNA)

我们以`createCar`方法为例详细解释调用智能合约的事务流程：

调用`invoke.js`，使用`submitTransaction`新建汽车，代码执行过程（内容来自源码）：

1. 创建Transaction，进行提交：

fabcar/javascript/node_modules/fabric-network/lib/contract.js:195：

```JavaScript
createTransaction(name) {
    verifyTransactionName(name);
    const qualifiedName = this._getQualifiedName(name);
    const transaction = new transaction_1.Transaction(this, qualifiedName);
    return transaction;
}
deserializeTransaction(data) {
    const state = JSON.parse(data.toString());
    return new transaction_1.Transaction(this, state.name, state);
}
async submitTransaction(name, ...args) {
    return this.createTransaction(name).submit(...args);
}
```

1. 从contract获取channel，创建新的endorcement，通过endorcement向endorser提交proposal（由endorsement模拟执行），并获取response：

fabcar/javascript/node_modules/fabric-network/lib/transaction.js:208：

```JavaScript
const method = `submit[${this.name}]`;
logger.debug('%s - start', method);
const channel = this.contract.network.getChannel();
const transactionOptions = this.gatewayOptions.eventHandlerOptions;
// This is the object that will centralize this endorsement activities
// with the fabric network
const endorsement = channel.newEndorsement(this.contract.chaincodeId);
const proposalBuildRequest = this.newBuildProposalRequest(args);
logger.debug('%s - build and send the endorsement', method);
// build the outbound request along with getting a new transactionId
// from the identity context
endorsement.build(this.identityContext, proposalBuildRequest);
endorsement.sign(this.identityContext);
// ------- S E N D   P R O P O S A L
// This is where the request gets sent to the peers
const proposalSendRequest = {};
// ......
const proposalResponse = await endorsement.send(proposalSendRequest);
```

fabcar/javascript/node_modules/fabric-common/lib/index.s.ts:238：

```TypeScript
export interface SendProposalRequest {
  targets?: Endorser[];
  handler?: ServiceHandler;
  requestTimeout?: number;
  requiredOrgs?: string[];
}
```

1. 获取endorsement response后，交给committer，由commit提交给order：

fabcar/javascript/node_modules/fabric-network/lib/transaction.js:262：

```JavaScript
// -----  C O M M I T   E N D O R S E M E N T
// this is where the endorsement results are sent to the orderer
const commitSendRequest = {};
if (isInteger(transactionOptions.commitTimeout)) {
    commitSendRequest.requestTimeout = transactionOptions.commitTimeout * 1000; // in ms;
}
if (proposalSendRequest.handler) {
    logger.debug('%s - use discovery to commit', method);
    commitSendRequest.handler = proposalSendRequest.handler;
}
else {
    logger.debug('%s - use the orderers assigned to the channel', method);
    commitSendRequest.targets = channel.getCommitters();
}
// by now we should have a discovery handler or use the target orderers
// that have been assigned from the channel to perform the commit
const commitResponse = await commit.send(commitSendRequest);
```

fabcar/javascript/node_modules/fabric-common/lib/index.s.ts:209：

```TypeScript
export interface CommitSendRequest {
  targets?: Committer[];
  handler?: ServiceHandler;
  requestTimeout?: number;
}
```

1. 获取commit response，判断是否成功写入：

fabcar/javascript/node_modules/fabric-network/lib/transaction.js:278：

```JavaScript
const commitResponse = await commit.send(commitSendRequest);
logger.debug('%s - commit response %j', method, commitResponse);
if (commitResponse.status !== 'SUCCESS') {
    const msg = `Failed to commit transaction %${endorsement.getTransactionId()}, orderer response status: ${commitResponse.status}`;
    logger.error('%s - %s', method, msg);
    eventHandler.cancelListening();
    throw new Error(msg);
}
else {
    logger.debug('%s - successful commit', method);
}
logger.debug('%s - wait for the transaction to be committed on the peer', method);
await eventHandler.waitForEvents();
return result;
```

基本跟上面流程一致啦，详细内容可以看看咱的实验报告：[Hyperledger Fabric 2.0 分布式实验环境搭建记录](https://tefoq5o3f4.feishu.cn/docx/FfkgdwEC1oJgB2x1YaZc2Gtbn1e) 
