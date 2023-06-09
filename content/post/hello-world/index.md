---
title: Hello World
description: Welcome to Hugo Theme Stack
slug: hello-world
date: 2022-03-06 00:00:00+0000
image: cover.jpg
categories:
    - Example Category
tags:
    - Example Tag
---
# 实现一个基本的区块链
## 创造一个区块
```javascript
const SHA256 = require("crypto-js/sha256");
class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index;
    this.previousHash = previousHash;
    this.timestamp = timestamp;
    this.data = data;
    this.hash = this.calculateHash();
  }

  calculateHash() {
    return SHA256(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data)).toString();
  }
}

```
定义了一个构造函数来初始化我区块的属性。每一个区块上都被赋予了`index`属性来告知我们这个区块在整个链上的位置。我们同时也生成了一个时间戳，以及需要在区块里存储的一些数据。最后是前一个区块的hash。
## 创造一个链
```javascript
class Blockchain{
  constructor() {
    this.chain = [this.createGenesisBlock()];
  }

  createGenesisBlock() {
    return new Block(0, "01/01/2017", "Genesis block", "0");
  }

  getLatestBlock() {
    return this.chain[this.chain.length - 1];
  }

  addBlock(newBlock) {
    newBlock.previousHash = this.getLatestBlock().hash;
    newBlock.hash = newBlock.calculateHash();
    this.chain.push(newBlock);
  }

  isChainValid() {
    for (let i = 1; i < this.chain.length; i++){
      const currentBlock = this.chain[i];
      const previousBlock = this.chain[i - 1];

      if (currentBlock.hash !== currentBlock.calculateHash()) {
        return false;
      }

      if (currentBlock.previousHash !== previousBlock.hash) {
        return false;
      }
    }
    return true;
  }
}

```
在构造函数里，我通过创建一个包含创世块的数组来初始化整个链。第一个区块是特殊的，因为它不能指向前一个区块。我还添加了下面两个方法：

- `getLatestBlock()`返回我们区块链上最新的区块。
- `addBlock()`负责将新的区块添加到我们的链上。为此，我们将前一个区块的hash添加到我们新的区块中。这样我们就可以保持整个链的完整性。因为只要我们变更了最新区块的内容，我们就需要重新计算它的hash。当计算完成后，我将把这个区块推进链里（一个数组）。

最后，我创建一个`isChainValid()`来确保没有人篡改过区块链。它会遍历所有的区块来检查每个区块的hash是否正确。它会通过比较`previousHash`来检查每个区块是否指向正确的上一个区块。如果一切都没有问题它会返回`true`否则会返回`false`。

## 
```javascript
let savjeeCoin = new Blockchain();
savjeeCoin.addBlock(new Block(1, "20/07/2017", { amount: 4 }));
savjeeCoin.addBlock(new Block(2, "20/07/2017", { amount: 8 }));

```

## 试用
```javascript
// 检查是否有效(将会返回true)
console.log('Blockchain valid? ' + savjeeCoin.isChainValid());

// 现在尝试操作变更数据
savjeeCoin.chain[1].data = { amount: 100 };

// 再次检查是否有效 (将会返回false)
console.log("Blockchain valid? " + savjeeCoin.isChainValid());

```
# 实现POW（proof-of-work工作量证明）

## 什么是POW
POW是在第一个区块链被创造之前就已经存在的一种机制。这是一项简单的技术，通过一定数量的计算来防止滥用。工作量是防止垃圾填充和篡改的关键。如果它需要大量的算力，那么填充垃圾就不再值得。
比特币通过要求hash以特定0的数目来实现POW。这也被称之为**难度**
不过等一下！一个区块的hash怎么可以改变呢？在比特币的场景下，一个区块包含有各种金融交易信息。我们肯定不希望为了获取正确的hash而混淆了那些数据。
为了解决这个问题，区块链添加了一个`nonce`值。Nonce是用来查找一个有效Hash的次数。而且，因为无法预测hash函数的输出，因此在获得满足难度条件的hash之前，只能大量组合尝试。寻找到一个有效的hash（创建一个新的区块）在圈内称之为挖矿。
在比特币的场景下，POW确保每10分钟只能添加一个区块。你可以想象垃圾填充者需要多大的算力来创造一个新区块，他们很难欺骗网络，更不要说篡改整个链。
## 实现POW
```javascript
constructor(index, timestamp, data, previousHash = '') {
  this.index = index;
  this.previousHash = previousHash;
  this.timestamp = timestamp;
  this.data = data;
  this.hash = this.calculateHash();
  this.nonce = 0;
}

```
我们还需要一个新的方法来增加Nonce，直到我们获得一个有效hash。强调一下，这是由难度决定的。所以我们会收到作为参数的难度。
```javascript
mineBlock(difficulty) {
    while (this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")) {
        this.nonce++;
        this.hash = this.calculateHash();
    }
    console.log("BLOCK MINED: " + this.hash);
}

```
最后，我们还需要更改一下`calculateHash()`函数。因为目前他还没有使用Nonce来计算hash。
```javascript
calculateHash() {
  return SHA256(this.index +
    this.previousHash +
    this.timestamp +
    JSON.stringify(this.data) +
    this.nonce
  ).toString();
}

```
将他们结合在一起，你会得到如下所示的区块类
```javascript
class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index;
    this.previousHash = previousHash;
    this.timestamp = timestamp;
    this.data = data;
    this.hash = this.calculateHash();
    this.nonce = 0;
  }

  calculateHash() {
    return SHA256(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data) + this.nonce).toString();
  }

  mineBlock(difficulty) {
    while (this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")) {
      this.nonce++;
      this.hash = this.calculateHash();
    }
    console.log("BLOCK MINED: " + this.hash);
  }
}

```
## 修改区块链
现在，我们的区块已经拥有Nonce并且可以被开采了，我们还需要确保我们的区块链支持这种新的行为。让我们先在区块链中添加一个新的属性来跟踪整条链的难度。我会将它设置为2（这意味着区块的hash必须以2个0开头）
```javascript
constructor() {
  this.chain = [this.createGenesisBlock()];
  this.difficulty = 2;
}

```
现在剩下要做的就是改变`addBlock()`方法，以便在将其添加到链中之前确保实际挖到该区块。下面我们将难度传给区块。
```javascript
addBlock(newBlock) {
  newBlock.previousHash = this.getLatestBlock().hash;
  newBlock.mineBlock(this.difficulty);
  this.chain.push(newBlock);
}

```
## 测试
现在让我们来测试一下我们的区块链，看看在POW下添加一个新区块会有什么效果。我将会使用之前的代码。我们将创建一个新的区块链实例然后往里添加2个区块。
```javascript
let savjeeCoin = new Blockchain();
console.log('Mining block 1');
savjeeCoin.addBlock(new Block(1, "20/07/2017", { amount: 4 }));
console.log('Mining block 2');
savjeeCoin.addBlock(new Block(2, "20/07/2017", { amount: 8 }));
```
如果你运行了上面的代码，你会发现添加新区块依旧非常快。这是因为目前的难度只有2（或者你的电脑性能非常好）。
如果你创建了一个难度为5的区块链实例，你会发现你的电脑会花费大概十秒钟来挖矿。随着难度的提升，你的防御攻击的保护程度越高。

# 交易与挖矿奖励
在前面两部分我们创建了一个简单的区块链，并且加入了POW来抵御攻击。然而我们在途中也偷了懒：我们的区块链只能在一个区块中存储一笔交易，而且矿工没有奖励。现在，让我们解决这个问题！
## 重构区块类
现在一个区块拥有`index`,`previousHash`,`timestamp`,`data`,`hash`和`nonce`属性。这个`index`属性并不是很有用，事实上我甚至不知道为什么开始我要将它添加进去。所以我把它移除了，同时将`data`改名为`transactions`来更语义化。
```javascript
class Block{
  constructor(timestamp, transactions, previousHash = '') {
    this.previousHash = previousHash;
    this.timestamp = timestamp;
    this.transactions = transactions;
    this.hash = this.calculateHash();
    this.nonce = 0;
  }
}
```
当我们改变区块类时，我们也必须更改`calculateHash()`函数。现在它还在使用老旧的`index`和`data`属性。
```javascript
calculateHash() {
  return SHA256(this.previousHash + this.timestamp + JSON.stringify(this.transactions) + this.nonce).toString();
}
```
## 交易类
在区块内，我们将可以存储多笔交易。因此我们还需要定义一个交易类，一边我们可以锁定交易应当具有的属性：
```javascript
class Transaction{
  constructor(fromAddress, toAddress, amount){
    this.fromAddress = fromAddress;
    this.toAddress = toAddress;
    this.amount = amount;
  }
}
```
这个交易例子非常的简单，仅仅包含了发起方（`fromAddress`）和接受方（`toAddress`）以及数量。如果有需求，你也可以在里面加入更多字段，不过这个只是为了最小实现。
## 调整我们的区块链
当前的最大任务：调整我们的区块链来适应这些新变化。我们需要做的第一件事就是存储待处理交易的地方。
正如你所知道的，由于POW，区块链可以稳定的创建区块。在比特币的场景下，难度被设置成大约每10分钟创建一个新区块。但是，是可以在创造两个区块之间提交新的交易。
为了做到这一点，首先需要改变我们区块链的构造函数，以便他可以存储待处理的交易。我们还将创造一个新的属性，用于定义矿工获得多少钱作为奖励：
```javascript
class Blockchain{
  constructor() {
    this.chain = [this.createGenesisBlock()];
    this.difficulty = 5;
    // 在区块产生之间存储交易的地方
    this.pendingTransactions = [];
    // 挖矿回报
    this.miningReward = 100;
  }
}
```
下一步，我们将调整我们的`addBlock()`方法。不过我的调整是指删掉并重写它！我们将不再允许人们直接为链上添加区块。相反，他们必须将交易添加至下一个区块中。而且我们将`addBlock()`更名为`createTransaction()`，这看起来更语义化：
```javascript
createTransaction(transaction) {
  // 这里应该有一些校验!
  // 推入待处理交易数组
  this.pendingTransactions.push(transaction);
}
```
## 挖矿
人们现在可以将新的交易添加到待处理交易的列表中。但无论如何，我们需要将他们清理掉并移入实际的区块中。为此，我们来创建一个`minePendingTransactions()`方法。这个方法不仅会挖掘所有待交易的新区块，而且还会向采矿者发送奖励。
```javascript
minePendingTransactions(miningRewardAddress) {
  // 用所有待交易来创建新的区块并且开挖..
  let block = new Block(Date.now(), this.pendingTransactions);
  block.mineBlock(this.difficulty);
  // 将新挖的看矿加入到链上
  this.chain.push(block);
  // 重置待处理交易列表并且发送奖励
  this.pendingTransactions = [
      new Transaction(null, miningRewardAddress, this.miningReward)
  ];
}
```
请注意，该方法采用了参数`miningRewardAddress`。如果你开始挖矿，你可以将你的钱包地址传递给此方法。一旦成功挖到矿，系统将创建一个新的交易来给你挖矿奖励（在这个栗子里是100枚币）。
有一点需要注意的是，在这个栗子中，我们将所有待处理交易一并添加到一个区块中。但实际上，由于区块的大小是有限制的，所以这是行不通的。在比特币里，一个区块的大小大概是2Mb。如果有更多的交易能够挤进一个区块，那么矿工可以选择哪些交易达成哪些交易不达成（通常情况下费用更高的交易容易获胜）。
## 地址的余额
在测试我们的代码钱让我们再做一件事！如果能够检查我们区块链上地址的余额将会更好。
```javascript
getBalanceOfAddress(address){
  let balance = 0; // you start at zero!
  // 遍历每个区块以及每个区块内的交易
  for(const block of this.chain){
    for(const trans of block.transactions){
      // 如果地址是发起方 -> 减少余额
      if(trans.fromAddress === address){
        balance -= trans.amount;
      }
      // 如果地址是接收方 -> 增加余额
      if(trans.toAddress === address){
        balance += trans.amount;
      }
    }
  }
  return balance;
}
```

## 测试
好吧，我们已经完成并可以最终一切是否可以正常工作！为此，我们创建了一些交易：
```javascript
let savjeeCoin = new Blockchain();
console.log('Creating some transactions...');
savjeeCoin.createTransaction(new Transaction('address1', 'address2', 100));
savjeeCoin.createTransaction(new Transaction('address2', 'address1', 50));
```
这些交易目前都处于等待状态，为了让他们得到证实，我们必须开始挖矿：
```javascript
console.log('Starting the miner...');
savjeeCoin.minePendingTransactions('xaviers-address');
```
当我们开始挖矿，我们也会传递一个我们想要获得挖矿奖励的地址。在这种情况下，我的地址是`xaviers-address`（非常复杂！）。
之后，让我们检查一下`xaviers-address`的账户余额：
```javascript
console.log('Balance of Xaviers address is', savjeeCoin.getBalanceOfAddress('xaviers-address'));
// 输出: 0
```
我的账户输出竟然是0？！等等，为什么？难道我不应该得到我的挖矿奖励么？那么，如果你仔细观察代码，你会看到系统会创建一个交易，然后将您的挖矿奖励添加为新的待处理交易。这笔交易将会包含在下一个区块中。所以如果我们再次开始挖矿，我们将收到我们的100枚硬币奖励！
```javascript
console.log('Starting the miner again!');
savjeeCoin.minePendingTransactions("xaviers-address");
console.log('Balance of Xaviers address is', savjeeCoin.getBalanceOfAddress('xaviers-address'));
// 输出: 100
```

# 附录
比特币：一种点对点电子货币系统
[https://bitcoin.org/bitcoin.pdf](https://bitcoin.org/bitcoin.pdf)
[https://bitcoin.org/files/bitcoin-paper/bitcoin_zh_cn.pdf](https://bitcoin.org/files/bitcoin-paper/bitcoin_zh_cn.pdf)

