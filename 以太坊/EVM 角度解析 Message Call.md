
![[Message Call 的参与者与消息传递.png]]

### **Message Call 的参与者与消息传递**
##### 1. **图中主要元素**
	• EOA（Externally Owned Account）：由私钥控制的外部账户，能够发起交易或消息。
	• Contract Account：智能合约账户，包含存储和与之关联的字节码（EVM code）。

##### 2. **消息传递过程**
	•从 EOA 到 Contract Account：
		• EOA 发起一笔交易，通过网络向智能合约发送消息。
		• 消息包含调用的目标合约地址、方法签名（函数选择器）、输入数据（函数参数）、msg.value（ETH数量）等。
		• 智能合约根据 EVM 代码执行操作，更新状态或返回结果。
	• 从 Contract Account 到另一个 Contract Account：
		• 合约可以通过 call、delegatecall 或 staticcall 调用另一个合约。
		• 消息包含调用的数据（方法选择器和参数），执行上下文会被传递到目标合约。

##### 3. **重要特性**
	• Message 不等同于 Transaction：Message 是更高层的逻辑，Transaction 是更底层的具体实现。
	• 调用者和被调用者：调用者的上下文（例如 msg.sender、msg.value）可以决定逻辑的执行方式。

![[EVM 内部处理 Message Call 的工作流.png]]

### **EVM 内部处理 Message Call 的工作流**
##### 1. **图中主要元素**
	• Stack：EVM 的栈，用于存储执行指令的临时数据，例如调用参数和返回值。
	• Memory：内存用于存储调用数据和返回值，是一个暂时的存储空间。
	• CALL 指令：EVM 用 CALL 操作码触发目标合约的执行。
	• RETURN 指令：目标合约执行结束后，将返回值写回调用者的内存中。

##### 2. **Message Call 的执行流程**
	（1）. 准备输入数据：
		• 调用者将方法选择器和参数数据编码后，存储在 Memory 中。
		• 使用 CALL 指令，将这些输入数据发送到目标合约。

	（2）. 执行目标合约的逻辑：
		• 目标合约读取输入数据，从 Memory 中解析参数。
		• 根据接收到的上下文（如 msg.sender 和 msg.value）执行对应的逻辑。
		• 如果需要修改状态，则在目标合约的存储空间中完成修改。
	
	（3）. 返回结果：
		• 目标合约执行完成后，使用 RETURN 指令将结果写回调用者的内存。
		• 调用者从内存中读取返回值，供后续使用。

##### 3. **特点与优化**
	• Gas 管理：调用合约时，调用者会指定最大 Gas 限制。目标合约在 Gas 用尽时会回滚。
	• 数据传递：所有参数通过 Memory 传递，栈仅用于辅助操作。
	• 嵌套调用：合约之间可以多次嵌套调用，EVM 会逐层维护栈和内存。

### **Message Call 的总结**
	• Message Call 是合约交互的核心机制，可以从外部账户到合约，或合约到合约之间传递调用数据。
	• 第一张图展示了 Message 的参与者和传递路径。
	• 第二张图描述了 EVM 在执行 Message Call 时，如何通过栈和内存实现参数传递和逻辑执行。
	• Message Call 是基于 CALL、DELEGATECALL 等指令实现的，通过这些指令，EVM 高效实现了合约间的调用与返回逻辑。