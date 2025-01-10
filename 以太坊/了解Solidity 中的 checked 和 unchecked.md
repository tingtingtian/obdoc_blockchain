
	在 Solidity 智能合约开发中，数值运算是非常常见的操作，而溢出（overflow）和下溢（underflow）问题是开发中必须面对的挑战。从 Solidity 0.8.0 开始，编译器对算术运算默认启用了溢出和下溢检查（checked mode）。然而，为了在某些场景下提升效率，Solidity 还引入了 unchecked 关键字，可以跳过这些检查。本文将深入探讨 checked 和 unchecked 的机制、使用方式、以及适用场景。

### **1. Checked 模式**

##### **什么是 Checked 模式？**

	Checked 模式是 Solidity 默认的数值运算模式。它对加法 (+)、减法 (-)、乘法 (*) 等算术操作进行溢出和下溢检查。如果运算结果超出数值类型的范围，会直接抛出错误，导致交易回滚。

##### **工作机制**

	  • 对于无符号整数（uint），溢出是指运算结果超过了其最大值（2^n - 1），下溢是指结果低于 0。
	  • 对于有符号整数（int），溢出是指结果超过了其范围（-2^(n-1) 到 2^(n-1) - 1），下溢是指低于最小值。

##### **示例代码**

```
pragma solidity ^0.8.0;

contract CheckedExample {
    function safeAdd(uint256 a, uint256 b) public pure returns (uint256) {
        return a + b; // 默认会检查溢出
    }

    function safeSub(uint256 a, uint256 b) public pure returns (uint256) {
        return a - b; // 默认会检查下溢
    }
}
```

##### **优点**

1. **安全性高**：避免了因溢出或下溢导致的意外错误。

2. **调试方便**：当发生溢出时，直接抛出异常并回滚交易，便于定位问题。

##### **缺点**

1. **Gas 成本略高**：由于增加了检查逻辑，会略微提高执行成本。
### **2. Unchecked 模式**

##### **什么是 Unchecked 模式？**

	  Unchecked 模式是一种通过 unchecked 关键字显式跳过溢出和下溢检查的运算方式。在这种模式下，如果发生溢出或下溢，数值会 自动环绕，而不会抛出异常。

##### **工作机制**

• 自动环绕是指：

	• 对于无符号整数：从最大值溢出时变为 0，下溢时从 0 倒退至最大值。
	• 对于有符号整数：从最大值溢出时变为最小值，下溢时从最小值变为最大值。

##### **示例代码**
```
pragma solidity ^0.8.0;

contract UncheckedExample {
    function uncheckedAdd(uint256 a, uint256 b) public pure returns (uint256) {
        unchecked {
            return a + b; // 不检查溢出
        }
    }

    function uncheckedSub(uint256 a, uint256 b) public pure returns (uint256) {
        unchecked {
            return a - b; // 不检查下溢
        }
    }
}
```

##### **优点**

1. **性能更高**：跳过了溢出检查，节省了 gas。

2. **灵活性更强**：在某些需要环绕行为的特定场景下，能直接利用其特性。

##### **缺点**

1. **安全性低**：如果开发者没有预期到溢出行为，可能会导致逻辑错误甚至合约漏洞。

2. **调试困难**：溢出行为不会引发异常，可能掩盖代码中的潜在问题。

### **3. Checked 和 Unchecked 的对比**

![[checked模式和unchecked模式的区别.png]]

### **4. 使用场景分析**
##### **Checked 模式适用场景**

1. **用户输入相关的计算**

	• 外部用户输入的数据可能不可靠，需要检查溢出。
	• 示例：处理用户的代币余额转账。
	
```
function transfer(address to, uint256 amount) public {
    require(balances[msg.sender] >= amount, "Insufficient balance");
    balances[msg.sender] -= amount; // 默认检查溢出
    balances[to] += amount;        // 默认检查溢出
}
```

2. **关键逻辑中的数值运算**

	• 合约的核心逻辑通常需要更高的安全性，建议使用 checked 模式。

##### **Unchecked 模式适用场景**

  1. **性能优化**
	  • 当溢出不可能发生或可以接受时，可以使用 unchecked 优化 gas。
	  • 示例：循环累加的总和。

```
function sumArray(uint256[] memory arr) public pure returns (uint256) {
    uint256 total = 0;
    for (uint256 i = 0; i < arr.length; i++) {
        unchecked {
            total += arr[i]; // 假设总和永远不会溢出
        }
    }
    return total;
}
```

2. **加密算法**

	• 某些加密算法或数学运算需要利用整数的环绕特性。
	• 示例：哈希函数实现。

3. **受信任的数据**

	• 当输入数据来自合约内部，且开发者完全控制其值时，可以使用 unchecked。

### **5. 代码示例：综合使用 Checked 和 Unchecked**

  以下代码展示了如何根据场景合理选择 checked 和 unchecked 模式：

```
pragma solidity ^0.8.0;

contract CheckedAndUnchecked {
    uint256 public constant MAX_SUPPLY = 1e6; // 最大供应量
    uint256 public totalSupply;

    // 使用 Checked 模式进行用户操作
    function mint(uint256 amount) public {
        require(totalSupply + amount <= MAX_SUPPLY, "Exceeds max supply");
        totalSupply += amount; // 默认检查溢出
    }

    // 使用 Unchecked 模式优化内部循环
    function calculateSum(uint256[] memory numbers) public pure returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < numbers.length; i++) {
            unchecked {
                sum += numbers[i]; // 假设总和永远不会溢出
            }
        }
        return sum;
    }
}
```

### **6. 开发建议**

1. **默认使用 Checked 模式**：

	• 安全性是智能合约的第一优先级。
	• 如果对溢出行为没有明确预期，应使用 checked 模式。

2. **谨慎使用 Unchecked 模式**：

	• 只有在经过验证的代码中、特定场景下（如性能优化）才使用 unchecked。
	• 确保开发和测试阶段已完全覆盖潜在问题。

3. **测试覆盖**：
	
	• 无论使用哪种模式，确保测试覆盖了所有可能的溢出和下溢情况。

### **7. 总结**

checked 和 unchecked 模式为 Solidity 提供了灵活的数值运算机制。checked 模式适合需要安全性和鲁棒性的场景，而 unchecked 模式则为优化性能和特殊场景提供了工具。在实际开发中，开发者需要根据具体需求权衡安全性和性能，选择合适的模式。

