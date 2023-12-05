> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.liaoxuefeng.com](https://www.liaoxuefeng.com/article/1569797590482976)

> 一文搞懂 NFT 分类与实现原理

NFT 即 Non-Fungible Token，与 ERC-20 不同，NFT 是一种不可分割的 Token，每个 NFT 都是唯一的。

### ERC-721

最早的 NFT 就是由加密猫（Crypto Kitties）搞出来的，一种 NFT 就是一个合约，它通过一个唯一 ID 标识持有人：

```
contract ERC721 {

    mapping (uint256 => address) public tokenIdToOwnerAddress;

    function tokenURI(uint256 tokenId) public view returns (string memory) {
        return "https://example.com/" + tokenId;
    }
}


```

ERC-721 合约的核心存储就是一个 ID 到用户地址的映射表，根据一个 NFT ID 查找持有人地址。如果要查看某个 ID 的 NFT 具体信息，则通过`tokenURI()`获得一个 URI 地址，访问该 URI 地址获得类似如下的 JSON 信息：

```
{
    "name": "Crypto Kitties",
    "image": "https://example.com/12345.png"
}


```

因此，一个 ERC-721 的 NFT 由合约地址和一个整数 ID 唯一确定。

### ERC-1155

像加密猫这种使用 ERC-721 作为 NFT，每个 NFT 都是唯一的，因此，与之对应的每只猫都是唯一的，不存在两只完全相同的猫。

如果用 NFT 对应游戏的道具呢？有些道具可能是唯一的，但有些道具可能有多个，这时，用 ERC-721 就不好表示，这种情况就要用 ERC-1155 实现。

```
contract ERC1155 {

    mapping (uint256 => mapping(address => uint256)) public balances;

}


```

ERC-1155 合约的核心存储由两级映射表组成：一个 ID 对应一个地址: 数量的映射表，这样，一个 ID 可对应多个完全相同的 NFT，分别由一个或多个地址持有，每个地址至少持有 1 个。

ERC-1155 介于全部唯一的 ERC-721 和全部相同的 ERC-20 之间。如果限定每个 ID 的数量只有一个，那么 ERC-1155 就退化为 ERC-721，如果限定只存在一个 ID，那么 ERC-1155 就退化为 ERC-20。

### ERC-6551

像加密猫这种使用 ERC-721 作为 NFT，每只猫都有唯一的所有者，也就是钱包地址的控制人。但是我们想想，如果加密猫这个 NFT 本身也能持有资产呢？比如，一只加密猫可以持有衣服、帽子等 NFT，甚至还可以持有 ETH 给自己买猫粮，是不是格局一下子就打开了？

但是 ERC-721 的 NFT 本质上就是一个整数 ID，不能作为地址，也没有私钥，自然无法持有其他 NFT 或代币，怎么破？

ERC-6551 提出了一种方法，它可以让一个 ERC-721 的 NFT 持有资产，具体方式如下：

首先，一个 ERC-721 的 NFT 可以由合约地址与 ID 作为唯一标识，然后，通过一个注册合约，动态部署一个新的合约，这个合约只做一件事，就是持有资产，并且支持任何转移资产的方法调用：

```
contract ERC6551Account {
    address erc721Contract;
    uint256 erc721TokenId;

    function execute(address to, bytes calldata data) {
        
        require(isNftOwner(erc721Contract, erc721TokenId, msg.sender));
        
        to.call(data);
    }
}


```

也就是说，一个 ERC-721 的 NFT 通过注册合约动态部署了一个新的合约，这个新合约记录了该 NFT 的地址和 ID，由于合约可持有资产，因此，新合约可正常接收其他 NFT 和代币。 如果想要从新合约把资产转移给 NFT 的所有者怎么办？观察`execute()`方法，实际上是先检查调用方是不是 NFT 的持有者，如果是，就执行转移指令。因此，新合约的控制权实际上是在 NFT 的持有人手里。

可以看出，对 ERC-721 来说，每个 NFT 都需要由持有人手动部署一个新合约才能充当 NFT 的 “资产持有人”。上述代码是一个简化版本，为了节省 GAS 费用，ERC-6551 实际上动态部署的是一个最小化的代理合约，再指向一个固定的`ERC6551Account`合约。

因此，ERC-6551 以一种 “外挂” 的方式来解决 ERC-721 无法充当资产持有人的问题。它解决的另一个问题是，如果一个 ERC-721 的所有权发生了转移，那么该 NFT 对应的代理合约持有的所有资产都归新的控制人所有，这样就极大地方便了交易，即买一只加密猫同时买到了这只猫持有的衣服、帽子甚至 ETH 等全部资产。