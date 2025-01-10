#以太坊 

	在以太坊（包括以太坊1.0和2.0）中，区块头、区块体和Merkle树是构成区块链数据结构的核心组件。它们通过巧妙的设计确保了区块链的安全性、透明性和可验证性。以下是它们的详细描述：

### **1. 区块头（Block Header）**

   区块头是区块的元数据，包含了当前区块的关键信息，用于验证区块的合法性、计算区块链的哈希值，并关联到链中的其他区块。

```
type Header struct {
	ParentHash  common.Hash    `json:"parentHash"       gencodec:"required"`
	UncleHash   common.Hash    `json:"sha3Uncles"       gencodec:"required"`
	Coinbase    common.Address `json:"miner"`
	Root        common.Hash    `json:"stateRoot"        gencodec:"required"`
	TxHash      common.Hash    `json:"transactionsRoot" gencodec:"required"`
	ReceiptHash common.Hash    `json:"receiptsRoot"     gencodec:"required"`
	Bloom       Bloom          `json:"logsBloom"        gencodec:"required"`
	Difficulty  *big.Int       `json:"difficulty"       gencodec:"required"`
	Number      *big.Int       `json:"number"           gencodec:"required"`
	GasLimit    uint64         `json:"gasLimit"         gencodec:"required"`
	GasUsed     uint64         `json:"gasUsed"          gencodec:"required"`
	Time        uint64         `json:"timestamp"        gencodec:"required"`
	Extra       []byte         `json:"extraData"        gencodec:"required"`
	MixDigest   common.Hash    `json:"mixHash"`
	Nonce       BlockNonce     `json:"nonce"`

	// BaseFee was added by EIP-1559 and is ignored in legacy headers.
	BaseFee *big.Int `json:"baseFeePerGas" rlp:"optional"`

	// WithdrawalsHash was added by EIP-4895 and is ignored in legacy headers.
	WithdrawalsHash *common.Hash `json:"withdrawalsRoot" rlp:"optional"`

	// BlobGasUsed was added by EIP-4844 and is ignored in legacy headers.
	BlobGasUsed *uint64 `json:"blobGasUsed" rlp:"optional"`

	// ExcessBlobGas was added by EIP-4844 and is ignored in legacy headers.
	ExcessBlobGas *uint64 `json:"excessBlobGas" rlp:"optional"`

	// ParentBeaconRoot was added by EIP-4788 and is ignored in legacy headers.
	ParentBeaconRoot *common.Hash `json:"parentBeaconBlockRoot" rlp:"optional"`

	// RequestsHash was added by EIP-7685 and is ignored in legacy headers.
	RequestsHash *common.Hash `json:"requestsHash" rlp:"optional"`
}
```

![[区块信息图.png]]

 
   **区块头的主要字段：**
#### 1. **parentHash**

	• 类型：bytes32

	• 含义：当前区块的父区块哈希。每个区块都通过parentHash链接到前一个区块，形成一个链式结构。

#### 2. **sha3Uncles**

	• 类型：bytes32

	• 含义：当前区块的叔块的哈希。以太坊1.0允许多个“并行”区块（叔块），这些区块未被主链采纳，但仍然包含在当前区块中。

#### 3. **miner**

	• 类型：address

	• 含义：矿工（或验证者）地址，表示挖掘当前区块的节点。

#### 4. **stateRoot**

	• 类型：bytes32

	• 含义：当前区块的状态根哈希，代表了当前区块的所有账户、余额和合约状态的Merkle树的根。它是通过Trie（Merkle Patricia树）计算得到的。

#### 5. **transactionsRoot**

	• 类型：bytes32

	• 含义：当前区块内所有交易的Merkle树根哈希。它将所有交易通过Merkle树结构进行哈希，并用一个根值表示所有交易的集合。

#### 6. **receiptsRoot**

	• 类型：bytes32

	• 含义：当前区块内所有交易的回执（Receipts）的Merkle树根哈希。交易回执包含交易的执行结果、日志等信息。

#### 7. **logsBloom**

	• 类型：bytes256

	• 含义：布隆过滤器，用于快速检查区块内是否存在某些类型的日志。这个结构可以高效地查询交易日志。

#### 8. **difficulty**

	• 类型：uint256

	• 含义：当前区块的工作量证明（PoW）难度，在以太坊2.0（PoS）中，此字段将被替代为 权益证明 相关的字段。

#### 9. **number**

	• 类型：uint64

	• 含义：区块的编号或高度，区块链中区块的唯一标识符。

#### 10. **gasLimit** 和 **gasUsed**

	• 类型：uint64

	• 含义：区块可以使用的最大Gas量和实际使用的Gas量。Gas是以太坊用来衡量计算量和交易费用的单位。

#### 11. **timestamp**

	• 类型：uint64

	• 含义：区块生成的时间戳（自1970年1月1日以来的秒数）。

#### 12. **extraData**

	• 类型：bytes

	• 含义：矿工可用于存储额外信息的可选字段。

#### 13. **mixHash** 和 **nonce**

	• 类型：bytes32 和 uint64

	• 含义：在PoW机制下，mixHash是挖矿结果的一部分，nonce是一个随机数，用于在挖矿过程中找到符合要求的区块哈希。
  

**存储方式：**

	区块头的字段通常存储在以太坊的块链数据结构中，每个区块头的哈希值可以通过Merkle树来验证。区块头的数据是不可篡改的，因此它提供了区块链的不可变性和安全性。

### **2. 区块体（Block Body）**

	区块体包含了具体的交易数据和其他与区块相关的信息。区块体的核心部分是交易和交易回执（Receipts）。区块体的结构决定了如何存储和验证区块中的所有交易。

```
type Body struct {
	Transactions []*Transaction
	Uncles       []*Header
	Withdrawals  []*Withdrawal `rlp:"optional"`
}
```

```
type Transaction struct {
   From       string `json:"from"`
   To         string `json:"to"`
   Value      uint64 `json:"value"`
   Gas        uint64 `json:"gas"`
   GasPrice   uint64 `json:"gasPrice"`
   Nonce      uint64 `json:"nonce"`
   Data       string `json:"data"`
   Signature  string `json:"signature"`
}
```

![[交易信息图.png]]

**区块体的主要字段：**

#### 1. **transactions**

	• 类型：Transaction[]

	• 含义：包含在当前区块内的所有交易。每笔交易包含发送者、接收者、金额、数据和其他重要的字段。

	以太坊区块体中的 Transaction（交易） 是区块体的核心部分之一，它包含了所有由用户或智能合约发起的交易信息。每笔交易的参数都可以用来描述该交易的执行、来源、目的、费用等。以下是以太坊交易（Transaction）中包含的 关键参数 和它们的含义：
		1. from：交易发起者的地址。
		2. to：交易接收者的地址（对于合约部署，to为null）。
		3. value：转账金额，单位为wei。
		4. gas：交易执行的Gas限制。
		5. gasPrice：每单位Gas的价格，决定交易的手续费。
		6. nonce：发送者账户的交易计数器，防止重放攻击。
		7. data：附加数据，通常用于智能合约调用。
#### 2. **uncles**

	• 类型：BlockHeader[]

	• 含义：当前区块的叔块（即同时间生成的未被包含在主链中的区块）列表。

#### **3. Withdrawals（提款数据）**

	• 类型：[]*Withdrawal（带有 rlp:"optional" 标签）

	• 含义：该字段是可选的，它包含一个指向 Withdrawal 结构体的切片。在以太坊2.0中，提款（Withdrawal）通常与**以太坊2.0权益证明**（PoS）相关。Withdrawal表示验证者从以太坊2.0网络中提取资金或奖励的交易数据。
	
	• RLP（Recursive Length Prefix）标签：在Withdrawals字段中，rlp:"optional"是Go语言中的RLP序列化标签，表示这个字段是可选的。这意味着在某些情况下，区块中可能不包含提款数据，这取决于网络状态和区块链的具体情况。
	

**存储方式：**

	• 交易数据（transactions）通常以数组的形式存储。每个交易是一个复杂的结构体，包含多个字段，如发送者、接收者、金额、数据、Gas费用等。

	• 交易回执数据（receipts）存储了交易执行后的状态和日志，可以用来验证交易是否成功，消耗的Gas等。

	• 每个交易通过Merkle树计算其哈希，transactionsRoot字段指向所有交易数据的Merkle树根。

### **3. Merkle树（Merkle Tree）**

	Merkle树是以太坊中用于保证数据完整性和可验证性的核心数据结构。它以树的形式将大量数据合并，通过不断哈希组合生成一个根哈希值，用来快速验证数据的一致性。

#### **Merkle树的基本结构**

##### 1. **Merkle树的叶子节点**

	• 每个叶子节点代表一个单独的数据元素（如交易或区块内的日志）。在以太坊中，交易和回执是Merkle树的叶子节点。

##### 2. **中间节点**

	• 中间节点是父节点，通过将子节点的数据（如哈希值）合并计算出来的。每个中间节点都是它的子节点哈希值的组合。

##### 3. **Merkle根**

	• Merkle树的根（stateRoot、transactionsRoot、receiptsRoot）是树的最上层节点的哈希值。它代表了整棵树的唯一标识，确保树中所有数据的一致性和完整性。

#### **Merkle树的存储和验证**

	• 存储：Merkle树的叶子节点通常存储在链上，而每个区块的根哈希（stateRoot、transactionsRoot、receiptsRoot）则是将所有数据压缩成一个固定长度的哈希值，用来验证数据的一致性。通过Merkle树根，可以验证区块中的数据是否被篡改。

	• 验证：通过Merkle树，用户可以快速地验证某笔交易是否包含在某个区块中。只需要提供一部分哈希路径（称为Merkle证明），就可以通过哈希链式结构验证数据的存在性和有效性。

#### **Merkle树的优势**

	  • 高效性：Merkle树大大提高了数据验证的效率。即使是一个包含成千上万交易的区块，也可以通过Merkle树验证某个交易是否包含在区块内。

	• 数据完整性：Merkle树确保了数据的完整性。如果区块中的任何数据（如交易或回执）被篡改，Merkle树的根哈希也会改变，从而使得区块的哈希值不再有效。


**总结**

	• 区块头：包含区块的元数据和关键参数，如父区块、矿工、状态树根等，它们通过哈希值确保区块链的安全性和一致性。

	• 区块体：包含具体的交易数据和交易回执，交易通过Merkle树进行哈希，以保证数据的完整性。

	• Merkle树：用于将多个数据元素（如交易、回执等）结合成一个根哈希，用于高效地验证数据的完整性和一致性。

	  通过这些数据结构，区块链能够保证交易的不可篡改性和数据的高效验证，同时为去中心化网络中的参与者提供透明性和安全性。




