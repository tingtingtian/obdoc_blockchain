
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

![[例子合约以及目录.png]]
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

![[新的目录.png]]

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

##### 部署合约的环境

在部署合约时，首先需要有相关的环境。以太坊区块链（通常称为 “主网”，代表 “主网络”）需要以以太币（其原生货币）的形式花费真正的金钱来使用它。这对测试是很不友好的。

为了解决这个问题，出现了很多“测试网”，其中包括 Sepolia 和 Holesky 区块链。它们的工作方式与主网非常相似，但有一个区别：您可以免费为这些网络获得以太币，因此使用它们不会花费您一分钱。然而，您仍然需要处理私钥管理、12 秒或更长时间的封锁时间，并实际获得这种免费的以太币。

在开发过程中，最好使用本地区块链。它在您的机器上运行，不需要互联网访问，为您提供所需的所有以太币，并立即挖掘区块。这些原因也使本地区块链非常适合自动化测试。

Hardhat 内置了本地区块链 Hardhat 网络。

启动后，Hardhat Network 将创建一组解锁帐户并给他们以太币。

```
npx hardhat node
```

Hardhat 网络将打印出其地址、[http://127.0.0.1:8545](http://127.0.0.1:8545/)以及可用帐户及其私钥的列表。运行结果如下：

![[npx hardhat node结果.png]]

每次运行 Hardhat Network 时，它都会创建一个全新的本地区块链 —— 不会保留以前运行的状态。这对于短期实验来说很好。

创建一个scripts目录，并在其下面创建deploy.js文件，目前项目目录如下：

![[新目录-2.png]]
deploy.js文件内容如下：
```
// scripts/deploy.js
async function main () {
  // We get the contract to deploy
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await Box.deploy();
  await box.waitForDeployment();
  console.log('Box deployed to:', await box.getAddress());
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

在我们的脚本中使用了ethers ，所以我们需要安装 [@nomicfoundation/hardhat-ethers plugin](https://hardhat.org/hardhat-runner/plugins/nomicfoundation-hardhat-ethers)

```
npm install --save-dev @nomicfoundation/hardhat-ethers ethers
```

之后，我们需要在我们的配置中添加 @nomicfoundation/hardhat-ethers 插件的相关配置，如下所示：

```
// hardhat.config.js
require("@nomicfoundation/hardhat-ethers");

...
module.exports = {
...
};
```

使用run命令，我们可以部署Box合约到本地网络中，命令和运行结果如下：
```
npx hardhat run scripts/deploy.js
Deploying Box...
Box deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

##### 通过控制台与合约交互

使用上面已部署的Box合约，我们可以用hardhat console 和它进行交互。

交互命令和运行情况如下：

```
npx hardhat console 
Welcome to Node.js v22.10.0.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = Box.attach('0x5FbDB2315678afecb367f032d93F642f64180aa3');
undefined
```

发送一个事务，调用相关函数store，运行情况如下：

```
await box.store(42)
ContractTransactionResponse {
  provider: HardhatEthersProvider {
    _hardhatProvider: LazyInitializationProviderAdapter {
      _providerFactory: [AsyncFunction (anonymous)],
      _emitter: [EventEmitter],
      _initializingPromise: [Promise],
      provider: [BackwardsCompatibilityProviderAdapter]
    },
    _networkName: 'hardhat',
    _blockListeners: [],
    _transactionHashListeners: Map(0) {},
    _eventListeners: []
  },
  blockNumber: 1,
  blockHash: '0xa819aa0887a6c632e2182f548e8ae73dc792cb620d516e91152e28164b9b2245',
  index: undefined,
  hash: '0x101bb503321f5b6ebda3ac9559c22afb805af1e279edfd0c14b3af929227b595',
  type: 2,
  to: '0x5FbDB2315678afecb367f032d93F642f64180aa3',
  from: '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  nonce: 0,
  gasLimit: 30000000n,
  gasPrice: 1875000000n,
  maxPriorityFeePerGas: 1000000000n,
  maxFeePerGas: 2750000000n,
  maxFeePerBlobGas: null,
  data: '0x6057361d000000000000000000000000000000000000000000000000000000000000002a',
  value: 0n,
  chainId: 31337n,
  signature: Signature { r: "0x6ed630a524d237ed352222a846f64755c2318241eca29c4d2391438d2410e854", s: "0x1dd4c2fc05f32f2789b5ff36a4e4e7fb46338435e64dd0cd58cf78f5bc61cd8c", yParity: 1, networkV: null },
  accessList: [],
  blobVersionedHashes: null
}
```

预期结果，如下：
```
> (await box.retrieve()).toString()
'42'
```

##### 通过脚本与合约进行交互

创建一个新的文件index.js，获取账户信息，代码如下：
```
// scripts/index.js

async function main () {

// Our code will go here

// Retrieve accounts from the local node

const accounts = (await ethers.getSigners()).map(signer => signer.address);

console.log(accounts);

}

main()

.then(() => process.exit(0))

.catch(error => {

console.error(error);

process.exit(1);

});
```

运行结果如下：
```
npx hardhat run  ./scripts/index.js
[
  '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  '0x70997970C51812dc3A010C7d01b50e0d17dc79C8',
...
]
```

获取一个合约实例
```
// Set up an ethers contract, representing our deployed Box instance
const address = '0x5FbDB2315678afecb367f032d93F642f64180aa3';
const Box = await ethers.getContractFactory('Box');
const box = Box.attach(address);
```

通过合约实例调用合约
```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

运行获取结果数据如下：
```
npx hardhat run --network localhost ./scripts/index.js
Box value is 42
```

发送一个新的事务
```
// Send a transaction to store() a new value in the Box
await box.store(23);

// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

运行结果如下：
```
npx hardhat run --network localhost ./scripts/index.js
Box value is 23
```

### 4、智能合约测试

我们将如何运行这些测试，因为智能合约是在区块链内部执行的。使用实际的以太坊网络会非常昂贵，而虽然测试网是免费的，但它们也很慢（区块时间为 12 秒或更长）。如果我们打算在每次对代码进行更改时运行数百个测试，我们需要更好的东西。  
我们将使用所谓的 “本地区块链”：它是真实区块链的精简版本，与互联网断开连接，在你的机器上运行。这将使事情简单很多：你不需要获取以太币，并且新的区块将立即被挖掘出来。

##### 搭建测试环境

我们将在单元测试中使用 Chai 断言，安装 Hardhat Toolbox 即可使用。安装hardhat-toolbox工具包命令如下：

```
npm install --save-dev @nomicfoundation/hardhat-toolbox
```

##### 单元测试

我们将把测试文件保存在一个 “test” 目录中。测试最好通过镜像 “contracts” 目录来构建：对于那里的每个 “.sol” 文件，创建一个相应的测试文件。
下面编写我们的第一个测试！这些测试将测试前面指南中的 “Box” 合约的属性：一个简单的合约，允许你检索所有者先前存储的值。
在项目根目录中创建一个 “test” 目录。我们将把测试保存为 “test/Box.test.js”。每个测试 “.js” 文件通常包含对单个合约的测试，并以该合约命名。

Box.test.js代码如下：
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Start test block
describe('Box', function () {
  before(async function () {
    this.Box = await ethers.getContractFactory('Box');
  });

  beforeEach(async function () {
    this.box = await this.Box.deploy();
    await this.box.waitForDeployment();
  });

  // Test case
  it('retrieve returns a value previously stored', async function () {
    // Store a value
    await this.box.store(42);

    // Test if the returned value is the same one
    // Note that we need to use strings to compare the 256 bit integers
    expect((await this.box.retrieve()).toString()).to.equal('42');
  });
});
```

我们现在准备好运行我们的测试了！

运行 “npx hardhat test” 将执行 “test” 目录中的所有测试。

```
npx hardhat test


  Box
    ✓ retrieve returns a value previously stored


  1 passing (578ms)
```

##### 执行复杂的断言

咱们的合约可能有许多有趣的特性难以捕捉，例如：  
	验证合约在出现错误时回退。  
	测量一个账户以太币余额的变化量。  
	检查是否发出了正确的事件。

“[**OpenZeppelin 测试助手**](https://docs.openzeppelin.com/test-helpers/0.5/)是一个旨在帮助你测试所有这些属性的库。它还将简化模拟区块链上时间流逝以及处理非常大的数字的任务。”

安装OpenZeppelin Test Helpers

```
npm install --save-dev @openzeppelin/test-helpers
```

然后，我们可以更新我们的测试，以使用 OpenZeppelin 测试助手来支持非常大的数字、检查事件是否被发出以及检查交易是否回退。但是@openzeppelin/test-helpers和truffe的框架更加兼容，在hardhat中使用不是很友好。我在测试的过程中使用的hardhat的测试工具较多。

```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

const { ethers } = require('hardhat');

// Import utilities from Test Helpers

const { BN } = require('@openzeppelin/test-helpers');

describe('Box', function () {

let box;

let owner, other;

const value = new BN('42'); // 使用普通整数测试
beforeEach(async function () {

[owner, other] = await ethers.getSigners();
// 部署 Box 合约

const BoxFactory = await ethers.getContractFactory('Box');

box = await BoxFactory.deploy();

await box.waitForDeployment();

})；
it('retrieve returns a value previously stored', async function () {

// 存储一个值

await box.store(value.toString());

// 检索存储的值

const storedValue = await box.retrieve();

expect(storedValue).to.equal(value.toString());

});
it('store emits an event', async function () {

// 存储一个值并获取交易

const tx = await box.store(value.toString());
// 验证事件是否触发

await expect(tx)

.to.emit(box, 'ValueChanged')

.withArgs(value.toString());

});
it('non owner cannot store a value', async function () {

// 尝试使用非所有者存储值，验证自定义错误
await expect(box.connect(other).store(value.toString())).to.be.revertedWithCustomError(
box,
'OwnableUnauthorizedAccount'
);

});
it('store emits an event with openzeppelin Ownable', async function () {
const tx = await box.connect(owner).store(value.toString());

await expect(tx)

.to.emit(box, 'ValueChanged')

.withArgs(value.toString());

});

it('non owner cannot store a value with openzeppelin Ownable', async function () {

// Test a transaction reverts

await expect(box.connect(other).store(value.toString())).to.be.revertedWithCustomError(box, 'OwnableUnauthorizedAccount');

});

});
```

运行结果如下：

```
npx hardhat test
......
  Box
    ✔ retrieve returns a value previously stored
    ✔ store emits an event
    ✔ non owner cannot store a value
    ✔ store emits an event with openzeppelin Ownable
    ✔ non owner cannot store a value with openzeppelin Ownable
  5 passing (528ms)
```

### 5、连接到公网做测试

合约编写完成并在本地进行全面的测试后，需要在一个持久的公共测试环境试用，让测试用户可以与应用程序进行交互。

这里我们将用到公共测试网络（又称测试网），这些网络的运行方式和以太坊主网类似，只是其中的以太币没有价值可以免费获取到，这使得测试网络非常适合测试我们自己的合约。

##### 可用的测试网络

有许多测试网络可供你选择，每个都有其自身的特点。用于测试去中心化应用程序和智能合约的推荐网络是 Sepolia（id=11155111）。

要获得可用测试网络的地址，可以注册一个Infra 相关的账号，地址：https://developer.metamask.io  进入后点击左侧菜单里的Infra RPC可以在 看到My First Key 的界面，点击 My First Key 可以看到有很多支持的Endpoints。

查询测试网络的chainId:https://chainlist.org/?search=sepolia&testnets=true

##### 连接项目到公有网络

要将我们的项目连接到公共测试网，我们需要：  
获取测试网节点
创建一个新帐户
更新我们的配置文件
资助我们的测试账户

###### 获取测试网节点

	  虽然可以自己启动服务连接到测试网的以太坊节点，但访问测试网最简单的方法是通过公共节点服务，如 Alchemy 或 Infura。Alchemy 和 Infura 通过免费和付费计划为所有测试网和主网络提供对公共节点的访问。
	  当一个节点可以被公众访问且不管理任何账户时，我们称其为公共节点。这意味着它可以回复查询并转发已签名的交易，但不能自行签署交易。
	  在本指南中，我们将使用 Infura，不过你也可以使用 Alchemy 或其他你选择的公共节点提供商。前往 Infura（包含推荐码），注册并记下分配给你的 API 密钥 —— 我们稍后将使用它连接到网络。

Infura 地址：https://www.infura.io/zh

###### 创建一个新帐户

要在测试网中发送交易，需要一个新的以太坊账户。有很多方法可以做到这一点：在这里，我们将使用助记符包，它将输出一个新的助记符（一组 12 个单词），我们将使用它来派生我们的帐户（不知道由于电脑是Mac的原因还是mnemonics 与node版本冲突或者是mnemonics不支持服务的原因，目前执行下面的命令无法安装）：

```
npx mnemonics
drama film snack motion ...
```

请确保你的助记词安全。不要将助记符提交到版本控制中。即使只是出于测试目的，仍然有恶意用户会为了好玩而对你的测试网部署造成严重破坏！

如果上面无法安装可以使用以下方式进行：
```
const { Wallet } = require("ethers");

// 生成随机钱包
const wallet = Wallet.createRandom();
console.log("Address:", wallet.address);
console.log("Mnemonic:", wallet.mnemonic.phrase);
```

###### 配置网络节点

由于我们正在使用公共节点，因此需要在本地对所有交易进行签名。我们将使用钱包和一个 Infra 端点来配置网络。

我们需要使用新的测试网网络连接来更新我们的配置文件。在这里我们将使用 Sepolia，但你可以使用任何你想要的网络。

```
// hardhat.config.js
...
  module.exports = {
+    networks: {
+     sepolia: {
+       url: "https://sepolia.infura.io/v3/<key>",
+       accounts: [privateKey1, privateKey2, ...]
+     },
+   },
...
};
```

上面的key和accounts信息可以通过配置文件设置，比如json文件。

推荐使用 dotenv（https://github.com/motdotla/dotenv），安装dotenv，将配置写到环境变量里面，便于项目的密码等信息的安全。

```
npm install --save-dev dotenv
```

之后在项目目录下创建.env文件，如下：
![[新目录-3.png]]

在hardhat.conf.js 引用，代码如下：
```
require("@nomicfoundation/hardhat-toolbox");

require('dotenv').config();

/** @type import('hardhat/config').HardhatUserConfig */

module.exports = {

networks: {

hardhat: {}, // 默认本地网络

localhost: {

url: "http://127.0.0.1:8545", // Ganache 网络

},

sepolia: {

url: process.env.SEPOLIA_URL,

accounts: [process.env.PRIVATE_KEY],

},

},

solidity: "0.8.28",

};
```

也可以说使用下面的方式，有兴趣的可以研究：

安装env-enc，将配置加密后显示在配置文件中，更加安全，保证开发者的信息安全：
```
npm install --save-dev @chainlink/env-enc
```

安装env-enc后，可以设置密码：
```
npx env-enc set-pw
```

安装env-enc后，可以设置环境变量：
```
npx env-enc set
```

现在，我们可以通过列出我们在 Sepolia 网络上可用的账户来测试此配置是否正常工作。

```
 npx hardhat console --network sepolia
Welcome to Node.js v22.13.0.
Type ".help" for more information.
> accounts = (await ethers.getSigners()).map(signer => signer.address)
[ '0x2711102CAF6b55E7b53a7591D4A201B9A75cBA30' ]
> (await ethers.provider.getBalance(accounts[0])).toString()
'0'
```

###### 测试网测试

将项目配置为在公共测试网上工作后，我们现在终于可以部署我们的 “Box” 合约了。这里的命令与你在本地开发网络上时完全相同，不过由于要挖掘新的区块，所以运行需要几秒钟的时间。

```
npx hardhat run --network sepolia scripts/deploy.js
Deploying Box...
Box deployed to: 0x1CE84314f00a7b4D7459f75f6DF5097A24169562
Box value is 0
```

这时，Box 合同实例将永远存储在测试网络中，并且任何人都可以公开访问。可以在诸如 Etherscan 这样的区块链浏览器上查看你的合同。请记住，在部署合同的测试网络上访问浏览器，例如对于 Sepolia 网络，访问 sepolia.etherscan.io。
![[Etherscan访问情况.png]]

可以在这里查看我们在上面的示例中部署的合同以及发送给它的所有交易。

也可以像平常一样与你的实例进行交互，既可以使用控制台，也可以通过编程方式进行。

```
npx hardhat console --network sepolia
Welcome to Node.js v22.13.0.
Type ".help" for more information.
> 
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x1CE84314f00a7b4D7459f75f6DF5097A24169562');
undefined
> await box.store(42);
ContractTransactionResponse {
  ......
  hash: '0x6026bb69d7bf1c56182257819195d55563130796b6c57dee1040f03fe1cfb814',
  ......
}
> (await box.retrieve()).toString()
'42'
> 
```

每笔交易都将花费一些 gas，所以你最终需要向你的账户充值更多资金。

### 6、更新智能合约

使用 OpenZeppelin 升级插件部署的智能合约可以进行升级以修改其代码，同时保留其地址、状态和余额。这使您能够迭代地为项目添加新功能，或修复在生产环境中可能发现的任何错误。

我们将学习：
- “为什么升级很重要”
- “使用升级插件升级我们的 Box”
- “了解升级在幕后是如何工作的”
- “学习如何编写可升级的合约”

##### 升级的重要性
	
  以太坊中的智能合约默认是不可变的。一旦创建，就无法更改它们，实际上在参与者之间充当了一份不可打破的合同。

   然而，在某些情况下，能够修改它们是可取的。想想双方之间的传统合同：如果他们都同意更改，他们就能够这样做。在以太坊上，他们可能希望修改智能合约以修复他们发现的漏洞（这甚至可能导致黑客窃取他们的资金！），添加额外的功能，或者只是更改由其执行的规则。

以下是在无法升级的合同中修复漏洞时需要做的事情：  
1、部署合同的新版本；  
2、手动将旧合同的所有状态迁移到新合同中（就 gas 费用而言，这可能非常昂贵！）；  
3、更新所有与旧合同交互的合同以使用新合同的地址；  
4、联系所有用户并说服他们开始使用新部署（并处理用户迁移缓慢时两个合同同时使用的情况）。

为了避免这种混乱，我们直接在插件中构建了合同升级功能。这使我们能够在保留状态、余额和地址的同时更改合同代码。让我们来看一下实际效果。

##### 使用升级插件升级

在使用 OpenZeppelin Upgrades Plugins 中的 deployProxy 部署新合同时，该合同实例稍后可以进行升级。默认情况下，只有最初部署合同的地址有权升级它。

deployProxy 会创建以下交易：
1. 部署实现合同（我们的 Box 合同）。
2. 部署代理合同并运行任何初始化函数。
在下面的场景中，代理部署会自动部署一个 ProxyAdmin 合同（我们代理的管理员）。
让我们通过部署我们 Box 合同的可升级版本来看看它是如何工作的，使用与我们之前部署时相同的设置。

安装 hardhat-upgrades 插件

```
npm install --save-dev @openzeppelin/hardhat-upgrades
```

然后，我们需要配置 Hardhat 以使用我们的`@openzeppelin/hardhat-upgrades`插件。为此，在你的`hardhat.config.js`文件中添加该插件，如下所示。

```
// hardhat.config.js 
... 
require("@nomicfoundation/hardhat-ethers"); require('@openzeppelin/hardhat-upgrades'); ... module.exports = { ... };
```

为了升级像`Box`这样的合约，我们需要先将其部署为可升级合约，这与我们目前看到的部署过程不同。我们将通过调用`store`并传入值 42 来初始化我们的 Box 合约。
使用 Hardhat 时，我们使用 scripts 来部署可升级合约。
我们将创建一个脚本来使用deployProxy  部署我们可升级的 Box 合约。我们将把这个文件保存为`scripts/deploy_upgradeable_box.js`。
```
// scripts/deploy_upgradeable_box.js 
const { ethers, upgrades } = require('hardhat');
async function main () { 
	const Box = await ethers.getContractFactory('Box');
	console.log('Deploying Box...'); 
	const box = await upgrades.deployProxy(Box, [42], { initializer: 'store' }); 
	await box.waitForDeployment(); 
	console.log('Box deployed to:', await box.getAddress()); 
} 
main();
```

然后我们可以部署我们的可升级合约。  
使用 “运行” 命令，我们可以将 “Box” 合约部署到 “开发” 网络。

```
npx hardhat run --network localhost scripts/deploy_upgradeable_box.js
Deploying Box...
Box deployed to: 0x5FC8d32690cc91D4c39d9d3abcBD16989F875707
Box value is 42
```

假设我们想要添加一个新功能：一个函数，用于增加存储在新版本的 “Box” 中的 “值”。

```
// contracts/BoxV2.sol

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract BoxV2 {

// ... code from Box.sol

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

// Increments the stored value by 1

function increment() public {

_value = _value + 1;

emit ValueChanged(_value);

}

}
```

创建了 Solidity 文件后，我们现在可以使用`upgradeProxy`函数升级我们之前部署的实例。`upgradeProxy`将创建以下交易：部署实现合约（我们的`BoxV2`合约）；调用`ProxyAdmin`以更新代理合约以使用新的实现。

我们将创建一个脚本来将我们的`Box`合约升级为使用`BoxV2`，使用[`upgradeProxy`](https://docs.openzeppelin.com/upgrades-plugins/api-hardhat-upgrades#upgrade-proxy)。我们将把这个文件保存为`scripts/upgrade_box.js`。我们需要指定我们部署`Box`合约时的代理合约的地址。
```
// scripts/upgrade_box.js

const { ethers, upgrades } = require('hardhat');

async function main () {

const BoxV2 = await ethers.getContractFactory('BoxV2');

console.log('Upgrading Box...');

await upgrades.upgradeProxy('0x5FC8d32690cc91D4c39d9d3abcBD16989F875707', BoxV2);

console.log('Box upgraded');

}

main()

.then(() => process.exit(0))

.catch(error => {

console.error(error);

process.exit(1);

});
```

```
npx hardhat run --network localhost scripts/upgrade_box.js
Compiled 1 Solidity file successfully (evm target: paris).
Upgrading Box...
Box upgraded
```

```
npx hardhat console --network localhost
Welcome to Node.js v22.13.0.
Type ".help" for more information.
> 
> const BoxV2 = await ethers.getContractFactory('BoxV2');
undefined
> const box = await BoxV2.attach('0x5FC8d32690cc91D4c39d9d3abcBD16989F875707');
undefined
> 
> await box.increment();
{......
  hash: '0x8b4ac65120bb7a448334912f711c6bea81372c882203f984553b2ba810fccd1d',
  ......
}
> 
> (await box.retrieve()).toString();
'43'
> 
```

在整个升级过程中，“Box” 的 “值” 以及其地址都是被保留下来的。并且无论你是在本地区块链、测试网还是主网上工作，这个过程都是相同的。

让我们看看 OpenZeppelin 升级插件是如何实现这一点的。

	当你创建一个新的可升级合约实例时，OpenZeppelin 升级插件实际上会部署三个合约：  
	你编写的合约，即被称为包含逻辑的 “实现合约”。  
	一个指向 “实现合约” 的代理，这是你实际与之交互的合约。  
	一个 “代理管理员”，作为代理的管理员。

	在这里，代理是一个简单的合约，它只是将所有调用 委托给一个实现合约。委托调用与常规调用类似，不同之处在于所有代码都是在调用者的上下文中执行，而不是被调用者的上下文中执行。因此，实现合约代码中的`transfer`实际上将转移代理的余额，并且对合约存储的任何读写都将从代理自己的存储中进行读写。

	  这使我们能够将合约的 状态和代码分离：代理持有状态，而实现合约提供代码。并且它还允许我们通过让代理委托给不同的实现合约来 更改 代码。

	  然后，升级涉及以下步骤：  
		部署新的实现合约。  
		向代理发送一笔交易，将其实现地址更新为新地址。
	
	智能合约的任何用户始终与代理进行交互，该代理的地址永远不会改变。这使得你可以推出升级或修复错误，而无需要求你的用户在他们那一端进行任何更改 —— 他们只需像往常一样继续与相同的地址进行交互。

##### 合约升级的限制

虽然任何智能合约都可以进行升级，但需要解决 Solidity 语言的一些限制。在编写合约的初始版本和我们要将其升级到的版本时，这些限制都会出现。

###### 初始化

	可升级合约不能有构造函数。为了帮助你运行初始化代码，OpenZeppelin 合约提供了 Initializable 基合约，允许你将一个方法标记为初始化器，确保它只能运行一次。
	例如，让我们编写一个带有初始化器的新版本的 Box 合约，存储一个管理员的地址，该管理员将是唯一被允许更改其内容的人。

	部署此合同时，我们需要指定初始化函数名称（仅当名称不是默认的 “initialize” 时），并提供我们想要使用的管理员地址。

```
// contracts/AdminBox.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract AdminBox is Initializable {
    uint256 private _value;
    address private _admin;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    function initialize(address admin) public initializer {
        _admin = admin;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    // Stores a new value in the contract
    function store(uint256 value) public {
        require(msg.sender == _admin, "AdminBox: not admin");
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

```
// scripts/deploy_upgradeable_adminbox.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const AdminBox = await ethers.getContractFactory('AdminBox');
  console.log('Deploying AdminBox...');
  const adminBox = await upgrades.deployProxy(AdminBox, ['0xACa94ef8bD5ffEE41947b4585a84BdA5a3d3DA6E'], { initializer: 'initialize' });
  await adminBox.waitForDeployment();
  console.log('AdminBox deployed to:', await adminBox.getAddress());
}

main();
```

	在所有实际用途中，初始化函数的作用类似于构造函数。但是，请记住，由于它是一个常规函数，因此你需要手动调用所有基合约（如果有）的初始化函数。

	  你可能已经注意到，我们同时包含了一个构造函数和一个初始化函数。这个构造函数的目的是使实现合约处于已初始化状态，这是对某些潜在攻击的一种缓解措施。

	由于技术限制，当你将合约升级到新版本时，你不能更改该合约的存储布局。这意味着，如果你已经在合约中声明了一个状态变量，你不能删除它、更改其类型或在它之前声明另一个变量。在我们的 Box 示例中，这意味着我们只能在 value 之后添加新的状态变量。

###### 升级

	由于技术限制，当你将合约升级到新版本时，不能更改该合约的 “存储布局”。这意味着，如果你已经在合约中声明了一个状态变量，就不能删除它、更改它的类型，或者在它之前声明另一个变量。在我们的 “Box” 示例中，这意味着我们只能在 “value” 之后添加新的状态变量。

```
// contracts/Box.sol
contract Box {
    uint256 private _value;

    // We can safely add a new variable after the ones we had declared
    address private _owner;

    // ...
}
```

幸运的是，这个限制仅影响状态变量。你可以随意更改合约的函数和事件。

  ###### 测试
```
// contracts/AdminBoxV2.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract AdminBoxV2 is Initializable {
    uint256 private _value;
    address private _admin;

    // New variable
    string private _boxName;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);
    event BoxNameChanged(string name);

    function initialize(address admin) public initializer {
        _admin = admin;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    // Stores a new value in the contract
    function store(uint256 value) public {
        require(msg.sender == _admin, "AdminBox: not admin");
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }

    // New function: set the name of the box
    function setBoxName(string memory name) public {
        require(msg.sender == _admin, "AdminBox: not admin");
        _boxName = name;
        emit BoxNameChanged(name);
    }

    // New function: get the name of the box
    function getBoxName() public view returns (string memory) {
        return _boxName;
    }
}
```

```
// scripts/upgrade_adminbox.js

const { ethers, upgrades } = require('hardhat');

async function main() {

const accounts = await ethers.getSigners(); // 获取所有 Signers

const adminSigner = accounts.find(signer => signer.address === '0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199');
const AdminBoxV2 = await ethers.getContractFactory("AdminBoxV2");

console.log("Upgrading AdminBox to AdminBoxV2...");

const adminBoxV2 = await upgrades.upgradeProxy("0x610178dA211FEF7D417bC0e6FeD39F05609AD788", AdminBoxV2);

console.log("AdminBox upgraded to AdminBoxV2 at:",await adminBoxV2.getAddress());

await adminBoxV2.connect(adminSigner).setBoxName("TinaBox");

await adminBoxV2.connect(adminSigner).store(100);

console.log("Box name set to TinaBox", await adminBoxV2.getBoxName());

console.log("Box value set to 100", await adminBoxV2.retrieve());

}

main()

.then(() => process.exit(0))

.catch((error) => {

console.error(error);

process.exit(1);

});
```

### 7、部署到主网

	在测试网上运行你的项目一段时间且没有问题后，你会希望将其部署到以太坊主网（也称为主网）。然而，进入主网的规划应该比你计划的发布日期早得多。
	在本指南中，我们将介绍将你的项目投入生产的以太坊特定注意事项，例如：
		- “审计与安全”
		- “验证源代码”
		- “安全管理密钥”
		- “处理项目治理”

	  请记住，虽然在测试网和主网中管理你的合约在技术上是相同的，但在主网上存在重要差异，因为你的项目现在为你的用户管理真实数据和财产。


##### 审计与安全

	虽然安全影响所有软件开发，但智能合约中的安全尤为重要。任何人都可以使用任何有效载荷直接向你的合约发送交易，并且你的所有合约代码和状态都是公开可访问的。更糟糕的是，如果你的项目被黑客攻击，没有办法收回被盗资金 —— 在去中心化网络中，这些资金一去不复返。

	考虑到这一点，安全应该是开发各个阶段的首要关注点。这意味着安全不是在你发布项目前一周才添加到项目中的东西，而是从项目第一天起就应遵循的指导原则。
  
	一旦完成，就是向一个或多个审计公司请求审计的好时机。

##### 验证源代码

	在将合约部署到主网后，应该立即 “验证合约源代码”。这个过程包括将 Solidity 代码提交给第三方，如 “Etherscan”（以太扫描）或 “Sourcify”，他们会编译代码并 “验证” 其与已部署的程序集相匹配。这使得任何用户都可以在区块链浏览器等地方查看你的合约代码，并知道它与该地址上实际运行的程序集相对应。

你可以在[以太坊区块链浏览器（Etherscan）](https://etherscan.io/verifyContract)网站上手动验证你的合约。

你也可以使用[hardhat-verify 插件](https://hardhat.org/hardhat-runner/plugins/nomicfoundation-hardhat-verify)。要做到这一点，请安装该插件：

```
npm install --save-dev @nomicfoundation/hardhat-verify
```

更新你的 Hardhat 配置：

```
// hardhat.config.js
const { etherscanApiKey, projectId, mnemonic } = require('./secrets.json');
require("@nomicfoundation/hardhat-verify");
...
module.exports = {
  networks: {
    mainnet: { ... }
  },
  etherscan: {
    apiKey: etherscanApiKey
  }
};
```

最后运行 “验证” 任务，传入合约地址、部署的网络以及用于部署它的构造函数参数（如果有）：

```
npx hardhat verify --network mainnet DEPLOYED_CONTRACT_ADDRESS "Constructor argument 1"
```

当你部署一个可升级的合同时，用户与之交互的合同将只是一个代理，实际的逻辑将在实现合同中。Etherscan 确实支持正确显示 OpenZeppelin 代理及其实现，但其他的区块链浏览器可能不支持。

Etherscan 的APIKey 可以在 https://etherscan.io/myapikey 中查找到，如果没有需要注册一个。


##### 安全管理密钥

在主网上工作时，你需要特别注意保护你的私钥。你用于部署和与合同交互的账户将持有真正的以太币，它具有实际价值，是黑客的诱人目标。采取一切预防措施来保护你的密钥，并在必要时考虑使用硬件钱包。

此外，可以定义某些账户在系统中拥有特殊权限 —— 并且你应该格外小心地保护它们。

###### Admin 账户

	“管理员（admin，是 administrator 的缩写）账户是在系统中具有特殊权限的账户。例如，管理员可能有权暂停合同。如果这样的账户落入恶意用户手中，他们可能会在系统中造成严重破坏。
	保护管理员账户的一个好方法是使用特殊合同，例如多重签名（multisig），而不是常规的外部拥有账户。多重签名是一种合同，只要有预定义数量的受信任成员同意，就可以执行任何操作。Safe 是一个很好的多重签名工具可以使用。”

###### 升级管理员

	在 OpenZeppelin 升级插件项目中，特殊管理员账户是有权升级其他合约的账户。默认情况下，这是用于部署合约的外部拥有账户：虽然这对于本地或测试网部署来说已经足够了，但在主网上，你需要更好地保护你的合约。攻击者如果掌握了你的升级管理员账户，就可以更改你系统中的任何合约！
	考虑到这一点，在部署后更改代理管理员的所有权是一个好主意 —— 例如，改为多重签名。要做到这一点，你可以使用 admin.transferProxyAdminOwnership 来转移我们的代理管理员合约的所有权。
	当你需要升级你的合约时，我们可以使用 prepareUpgrade 来验证并部署一个新的实现合约，以便在我们的代理更新时使用。
	
##### 项目治理

	可以说，管理员账户反映出一个项目实际上并不是完全 “去中心化” 的。毕竟，如果一个账户可以单方面更改系统中的任何合同，那么我们并没有真正创造一个无需信任的环境。
	这就是 “治理” 的用武之地。在许多情况下，项目中会有一些操作需要特殊权限，从微调系统参数到进行完整的合同升级。你需要选择这些行动将如何决定：是由一小群可信赖的开发人员决定，还是由所有项目利益相关者的 “公开投票” 决定。
	这里没有正确答案。你为项目选择哪种治理方案将在很大程度上取决于你正在构建的内容以及社区情况。











































