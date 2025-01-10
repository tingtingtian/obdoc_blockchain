
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/// @title ERC721 NFT 实现
/// @dev 满足 ERC721 标准，并实现基本的 NFT 功能
contract MyERC721Token {
    // ------------------------------
    // ERC721 标准的状态变量
    // ------------------------------

    // @dev 映射：tokenId => 拥有者地址
    mapping(uint256 => address) private _owners;

    // @dev 映射：地址 => 其拥有的 NFT 数量
    mapping(address => uint256) private _balances;

    // @dev 映射：tokenId => 授权地址
    mapping(uint256 => address) private _tokenApprovals;

    // @dev 映射：地址 => （操作员地址 => 是否被授权）
    mapping(address => mapping(address => bool)) private _operatorApprovals;

    // @dev NFT 名称
    string public name;

    // @dev NFT 符号
    string public symbol;

    // ------------------------------
    // ERC721 事件
    // ------------------------------

    // @dev 当 token 转移时触发事件
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);

    // @dev 当 token 授权时触发事件
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);

    // @dev 当操作员被设置或移除时触发事件
    event ApprovalForAll(address indexed owner, address indexed operator, bool approved);

    // ------------------------------
    // 构造函数
    // ------------------------------

    /// @dev 初始化 NFT 名称和符号
    /// @param _name NFT 名称
    /// @param _symbol NFT 符号
    constructor(string memory _name, string memory _symbol) {
        name = _name;
        symbol = _symbol;
    }

    // ------------------------------
    // ERC721 标准函数实现
    // ------------------------------

    /// @notice 查询地址拥有的 NFT 数量
    /// @param owner 查询的地址
    /// @return balance NFT 数量
    function balanceOf(address owner) public view returns (uint256) {
        require(owner != address(0), "ERC721: address zero is not a valid owner");
        return _balances[owner];
    }

    /// @notice 查询 tokenId 的拥有者
    /// @param tokenId 查询的 tokenId
    /// @return owner 拥有者地址
    function ownerOf(uint256 tokenId) public view returns (address) {
        address owner = _owners[tokenId];
        require(owner != address(0), "ERC721: invalid token ID");
        return owner;
    }

    /// @notice 安全转移 NFT（需要目标地址支持 ERC721 接口）
    /// @param from 当前拥有者地址
    /// @param to 接收者地址
    /// @param tokenId 转移的 tokenId
    function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId
    ) public {
        require(_isApprovedOrOwner(msg.sender, tokenId), "ERC721: caller is not token owner or approved");
        _safeTransfer(from, to, tokenId);
    }

    /// @notice 转移 NFT（不检查目标地址是否支持 ERC721）
    /// @param from 当前拥有者地址
    /// @param to 接收者地址
    /// @param tokenId 转移的 tokenId
    function transferFrom(
        address from,
        address to,
        uint256 tokenId
    ) public {
        require(_isApprovedOrOwner(msg.sender, tokenId), "ERC721: caller is not token owner or approved");
        _transfer(from, to, tokenId);
    }

    /// @notice 授权地址可以管理指定的 tokenId
    /// @param to 授权地址
    /// @param tokenId 授权的 tokenId
    function approve(address to, uint256 tokenId) public {
        address owner = ownerOf(tokenId);
        require(to != owner, "ERC721: approval to current owner");
        require(
            msg.sender == owner || isApprovedForAll(owner, msg.sender),
            "ERC721: caller is not token owner or approved for all"
        );

        _approve(to, tokenId);
    }

    /// @notice 查询 tokenId 的授权地址
    /// @param tokenId 查询的 tokenId
    /// @return operator 授权地址
    function getApproved(uint256 tokenId) public view returns (address) {
        require(_exists(tokenId), "ERC721: invalid token ID");
        return _tokenApprovals[tokenId];
    }

    /// @notice 设置或移除操作员
    /// @param operator 操作员地址
    /// @param approved 是否授权
    function setApprovalForAll(address operator, bool approved) public {
        require(operator != msg.sender, "ERC721: approve to caller");
        _operatorApprovals[msg.sender][operator] = approved;
        emit ApprovalForAll(msg.sender, operator, approved);
    }

    /// @notice 查询是否已授权操作员
    /// @param owner 拥有者地址
    /// @param operator 操作员地址
    /// @return 是否授权
    function isApprovedForAll(address owner, address operator) public view returns (bool) {
        return _operatorApprovals[owner][operator];
    }

    // ------------------------------
    // 内部和私有函数
    // ------------------------------

    /// @dev 转移 NFT 的内部实现
    function _transfer(
        address from,
        address to,
        uint256 tokenId
    ) internal {
        require(ownerOf(tokenId) == from, "ERC721: transfer from incorrect owner");
        require(to != address(0), "ERC721: transfer to the zero address");

        // 清除之前的授权
        _approve(address(0), tokenId);

        // 更新持有数量
        _balances[from] -= 1;
        _balances[to] += 1;

        // 更新 tokenId 拥有者
        _owners[tokenId] = to;

        emit Transfer(from, to, tokenId);
    }

    /// @dev 授权地址的内部实现
    function _approve(address to, uint256 tokenId) internal {
        _tokenApprovals[tokenId] = to;
        emit Approval(ownerOf(tokenId), to, tokenId);
    }

    /// @dev 判断调用者是否是拥有者或被授权的地址
    function _isApprovedOrOwner(address spender, uint256 tokenId) internal view returns (bool) {
        require(_exists(tokenId), "ERC721: operator query for nonexistent token");
        address owner = ownerOf(tokenId);
        return (spender == owner || getApproved(tokenId) == spender || isApprovedForAll(owner, spender));
    }

    /// @dev 查询 tokenId 是否存在
    function _exists(uint256 tokenId) internal view returns (bool) {
        return _owners[tokenId] != address(0);
    }

    /// @dev 安全转移的内部实现
    function _safeTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal {
        _transfer(from, to, tokenId);
        // 此处可以扩展，调用目标地址的 onERC721Received 检查
    }

    /// @dev 铸造新 NFT
    function _mint(address to, uint256 tokenId) internal {
        require(to != address(0), "ERC721: mint to the zero address");
        require(!_exists(tokenId), "ERC721: token already minted");

        _balances[to] += 1;
        _owners[tokenId] = to;

        emit Transfer(address(0), to, tokenId);
    }
}
```

**业务解析**

1. **状态变量**

	• _owners 映射记录了每个 tokenId 的拥有者。
	• _balances 映射记录了每个地址持有的 NFT 数量。
	• _tokenApprovals 和 _operatorApprovals 用于实现授权逻辑。

2. **基本功能**
	• balanceOf 和 ownerOf 是核心查询功能。
	• transferFrom 和 safeTransferFrom 实现了 NFT 的转移功能。
	• _mint 方法用于创建新的 NFT，_transfer 用于完成转移的核心逻辑。

3. **安全性**
	• 每个转移都需要验证 from 是实际拥有者。
	• 所有授权和转移都确保目标地址不为零地址。

4. **扩展**
	• 此基础实现未包含元数据（如 tokenURI）或 onERC721Received 回调函数，可以在实际应用中扩展。


