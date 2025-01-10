
### 1、创建Node项目
Node下载地址如下：https://nodejs.org/en/about/previous-releases
可以通过下载msi的包直接安装。

安装完成后，通过如下方式验证：
```
node --version
v20.17.0
```

创建一个项目
```
mkdir learn && cd learn
```

初始化项目
```
npm init -y
```

npm 注册表中存储有两种主要类型的包：库和可执行文件。已安装的库与其他任何 JavaScript 代码一样被使用，但可执行文件是特殊的。
安装 Node 时包含了第三方二进制文件：npx。它用于运行在你的项目中本地安装的可执行文件。
虽然 Hardhat 可以全局安装，但我们建议在每个项目中本地安装，这样你就可以逐个项目地控制版本。
```
hardhat init
hardhat: command not found
npx hardhat init
👷 Welcome to Hardhat v2.22.12 👷‍
? What do you want to do? …
```

### 2、开发智能合约
##### 安装开发工具

以太坊最流行的开发框架是 Hardhat 和 Foundry。每个都有自己的优势，合适地使用这些是非常有用的。

下面将展示如何使用 Hardhat 开发、测试和部署智能合约，并介绍它在 ethers. js 中最常见的用途。

要开始使用 Hardhat，将把它安装在项目目录中。

```
npm install --save-dev hardhat
```

一旦安装，我们就可以运行 npx  hardhat 来创建一个空的hardhat.config.js 文件
```
npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.22.12 👷‍

✔ What do you want to do? · Create an empty hardhat.config.js
Config file created
```

##### 创建第一个智能合约

编写我们的第一个简单的智能合约，称为 Box：它将让人们存储一个以后可以检索的值。

![[截屏2025-01-09 15.43.25.png]]
现在项目目录如上左侧展示，合约代码如下：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

##### 编译智能合约

以太坊虚拟机（EVM）不能直接执行 Solidity 代码：首先需要将其编译成 EVM 字节码。
 Box. sol 合约使用 Solidity 0.8，因此需要首先配置 Hardhat 以使用适当的 solc 版本。
在 hardhat. config.js 中指定 Solidity 0.8 solc 版本。

```
// hardhat.config.js

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
 module.exports = {
  solidity: "0.8.24",
};
```

然后可以通过运行单个编译命令来实现编译：
```
npx hardhat compile
Compiled 1 Solidity file successfully (evm target: paris).
```

编译内置任务将自动查找合约目录中的所有合约，并使用 Solidity 编译器使用 hardhat. config.js 中的配置对其进行编译。
会注意到创建了一个工件目录：它包含已编译的文件（字节码和元数据），它们是. json 文件。将此目录添加到您的.gitignore 是必要的。

##### 添加其它智能合约

合约代码如下：
```
// contracts/access-control/Auth.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Auth {
    address private _administrator;

    constructor(address deployer) {
        // Make the deployer of the contract the administrator
        _administrator = deployer;
    }

    function isAdministrator(address user) public view returns (bool) {
        return user == _administrator;
    }
}
```

要在Box合约中使用Auth合约，需要把Auth合约的相对地址引入，如下：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Auth from the access-control subdirectory
import "./access-control/Auth.sol";

contract Box {
    uint256 private _value;
    Auth private _auth;

    event ValueChanged(uint256 value);

    constructor() {
        _auth = new Auth(msg.sender);
    }

    function store(uint256 value) public {
        // Require that the caller is registered as an administrator in Auth
        require(_auth.isAdministrator(msg.sender), "Unauthorized");

        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

现在合约目录如下：

![[截屏2025-01-09 16.22.50.png]]

##### 使用 OpenZeppelin 智能合约

导入OpenZeppelin 智能合约
```
npm install @openzeppelin/contracts
```

为了替换上面自己写的Auth合约，可以引入import `@openzeppelin/contracts/access/Ownable.sol`做权限控制
代码如下：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private _value;

    event ValueChanged(uint256 value);

    constructor() Ownable(msg.sender) {}

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

### 3、部署并且和智能合约交互




