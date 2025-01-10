简易版：
```
pragma solidity ^0.5.10;

contract Token {
    // 一个 `address` 类比于邮件地址 - 它用来识别以太坊的一个帐户。
    // 地址可以代表一个智能合约或一个外部（用户）帐户。
    address public owner;

    //  `mapping` 是一个哈希表数据结构。
    // 此 `mapping` 将一个无符号整数（代币余额）分配给地址（代币持有者）。
    mapping (address => uint) public balances;

    // 事件允许在区块链上记录活动。
    // 以太坊客户端可以监听事件，以便对合约状态更改作出反应。
    event Transfer(address from, address to, uint amount);

    // 初始化合约数据，设置 `owner`为合约创建者的地址。
    constructor() public {
        // 所有智能合约依赖外部交易来触发其函数。
        // `msg` 是一个全局变量，包含了给定交易的相关数据，
        // 例如发送者的地址和包含在交易中的 ETH 数量。
        owner = msg.sender;
    }

    // 创建一些新代币并发送给一个地址。
    function mint(address receiver, uint amount) public {
        // `require` 是一个用于强制执行某些条件的控制结构。
        // 如果 `require` 的条件为 `false`，则异常被触发，
        // 所有在当前调用中对状态的更改将被还原。
        // 只有合约创建人可以调用这个函数
        require(msg.sender == owner, "You are not the owner.");

        // 强制执行代币的最大数量
        require(amount < 1e60, "Maximum issuance exceeded");

        // 将 "收款人"的余额增加"金额"
        balances[receiver] += amount;
    }

    // 从任何调用者那里发送一定数量的代币到一个地址。
    function transfer(address receiver, uint amount) public {
        // 发送者必须有足够数量的代币用于发送
        require(amount <= balances[msg.sender], "Insufficient balance.");

        // 调整两个帐户的余额
        balances[msg.sender] -= amount;
        balances[receiver] += amount;

        // 触发之前定义的事件。
        emit Transfer(msg.sender, receiver, amount);
    }
}

```

标准版：

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/// @title 简单的 ERC20 代币实现
/// @dev 此合约实现了符合 ERC20 标准的所有必要方法。
contract MyERC20Token {
    // ---------------------------
    // ERC20 状态变量
    // ---------------------------

    // @dev 用于存储每个地址余额的映射
    mapping(address => uint256) private balances;

    // @dev 用于存储所有者为花费者设置的授权额度的映射
    mapping(address => mapping(address => uint256)) private allowances;

    // @dev 代币的总供应量
    uint256 private _totalSupply;

    // @dev 代币的基本信息
    string public name; // 代币名称
    string public symbol; // 代币符号
    uint8 public decimals; // 代币的小数位数

    // ---------------------------
    // 事件
    // ---------------------------

    // @dev 当代币被转移时触发的事件
    event Transfer(address indexed from, address indexed to, uint256 value);

    // @dev 当代币所有者为某个花费者设置授权额度时触发的事件
    event Approval(address indexed owner, address indexed spender, uint256 value);

    // ---------------------------
    // 构造函数
    // ---------------------------

    /// @dev 构造函数，用于初始化代币信息并铸造初始供应量
    /// @param _name 代币名称
    /// @param _symbol 代币符号
    /// @param _decimals 小数位数
    /// @param initialSupply 代币的初始供应量（以最小单位表示）
    constructor(
        string memory _name,
        string memory _symbol,
        uint8 _decimals,
        uint256 initialSupply
    ) {
        name = _name;
        symbol = _symbol;
        decimals = _decimals;
        _mint(msg.sender, initialSupply); // 将初始供应量铸造给合约部署者
    }

    // ---------------------------
    // ERC20 标准函数
    // ---------------------------

    /// @notice 返回代币的总供应量
    function totalSupply() public view returns (uint256) {
        return _totalSupply;
    }

    /// @notice 返回某个地址的代币余额
    /// @param account 要查询的地址
    function balanceOf(address account) public view returns (uint256) {
        return balances[account];
    }

    /// @notice 从调用者账户向指定地址转移代币
    /// @param recipient 接收代币的地址
    /// @param amount 要转移的代币数量
    function transfer(address recipient, uint256 amount) public returns (bool) {
        _transfer(msg.sender, recipient, amount);
        return true;
    }

    /// @notice 返回某个花费者被授权的额度
    /// @param owner 代币所有者的地址
    /// @param spender 被授权花费代币的地址
    function allowance(address owner, address spender) public view returns (uint256) {
        return allowances[owner][spender];
    }

    /// @notice 授权某个地址为调用者花费指定数量的代币
    /// @param spender 被授权的地址
    /// @param amount 被授权的代币数量
    function approve(address spender, uint256 amount) public returns (bool) {
        _approve(msg.sender, spender, amount);
        return true;
    }

    /// @notice 从一个地址向另一个地址转移代币（基于授权）
    /// @param sender 转出代币的地址
    /// @param recipient 接收代币的地址
    /// @param amount 转移的代币数量
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public returns (bool) {
        _transfer(sender, recipient, amount);

        uint256 currentAllowance = allowances[sender][msg.sender];
        require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
        unchecked {
            _approve(sender, msg.sender, currentAllowance - amount);
        }

        return true;
    }

    // ---------------------------
    // 内部辅助函数
    // ---------------------------

    /// @dev 内部函数，用于在两个地址之间转移代币
    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(balances[sender] >= amount, "ERC20: transfer amount exceeds balance");

        unchecked {
            balances[sender] -= amount;
            balances[recipient] += amount;
        }

        emit Transfer(sender, recipient, amount);
    }

    /// @dev 内部函数，用于批准花费者的额度
    function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");

        allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }

    /// @dev 内部函数，用于铸造新的代币
    function _mint(address account, uint256 amount) internal {
        require(account != address(0), "ERC20: mint to the zero address");

        _totalSupply += amount;
        balances[account] += amount;

        emit Transfer(address(0), account, amount);
    }

    /// @dev 内部函数，用于销毁代币
    function _burn(address account, uint256 amount) internal {
        require(account != address(0), "ERC20: burn from the zero address");
        require(balances[account] >= amount, "ERC20: burn amount exceeds balance");

        unchecked {
            balances[account] -= amount;
            _totalSupply -= amount;
        }

        emit Transfer(account, address(0), amount);
    }
}
```

**功能说明**

1. **核心功能**

	• 提供 transfer、approve、transferFrom、balanceOf 和 allowance，符合 ERC20 标准要求。
	• 实现了 totalSupply，跟踪代币总供应量。

2. **初始化**

	• 在部署合约时通过构造函数设置代币名称、符号、小数位数以及初始供应量。

3. **事件**

	• Transfer 和 Approval 事件用于记录转账和授权操作。

4. **额外功能**

	• 添加了 _mint 和 _burn 函数，方便在扩展中实现代币的铸造和销毁逻辑。


**使用示例**

  • 部署时传入代币名称（如 “MyToken”）、符号（如 “MTK”）、小数位数（如 18）和初始供应量（如 1,000,000）。
  • 使用 transfer 和 transferFrom 实现代币转账功能。
  • 使用 approve 和 allowance 实现代币的委托转账。

  这份代码既是一个 ERC20 标准的完整实现，也可以作为扩展的基础，例如创建可销毁代币、铸币功能等。


