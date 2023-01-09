# Ethereum Improvement Proposals (EIP) 以太坊改进协议


ERC 肯定是 EIP， 反之不是


# Ethereum Request for Comment (以太坊版的意见征求稿)

用以记录以太坊上应用级的各种开发标准和协议(application-level standards and conventions).



## 参考

https://eips.ethereum.org/erc

https://github.com/ethereum/EIPs


## ERC-20


```
								
function name() public view returns (string)      																【可选】 返回令牌的名称 - 例如"MyToken"
																
function symbol() public view returns (string)    																【可选】 返回令牌的符号。例如"HIX"
																
function decimals() public view returns (uint8)   																【可选】 返回令牌使用的小数位数 - 例如8，表示将令牌数量除以100000000以获得其用户表示
								
function totalSupply() public view returns (uint256) 															返回总代币供应量
								
function balanceOf(address _owner) public view returns (uint256 balance)										返回另一个具有 address 的帐户的帐户余额_owner
								
function transfer(address _to, uint256 _value) public returns (bool success)    								转账
	
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)  				转被授权的帐

function approve(address _spender, uint256 _value) public returns (bool success) 								批准

function allowance(address _owner, address _spender) public view returns (uint256 remaining)					津贴 (查看被批准的数值)


事件:

event Transfer(address indexed _from, address indexed _to, uint256 _value)

event Approval(address indexed _owner, address indexed _spender, uint256 _value)


```

## ERC 721 (非同质代币)


```

event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);
event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);




必备函数

function balanceOf(address _owner) external view returns (uint256);
function ownerOf(uint256 _tokenId) external view returns (address);
function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes calldata data) external payable;
function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
function approve(address _approved, uint256 _tokenId) external payable;
function setApprovalForAll(address _operator, bool _approved) external;
function getApproved(uint256 _tokenId) external view returns (address);
function isApprovedForAll(address _owner, address _operator) external view returns(bool);
```


```
// 例子

pragma solidity ^0.4.20;


interface ERC721 {
    /// @dev 当任何NFT的所有权更改时（不管哪种方式），就会触发此事件。
    ///  包括在创建时（`from` == 0）和销毁时(`to` == 0), 合约创建时除外。
    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);

    /// @dev 当更改或确认NFT的授权地址时触发。
    ///  零地址表示没有授权的地址。
    ///  发生 `Transfer` 事件时，同样表示该NFT的授权地址（如果有）被重置为“无”（零地址）。
    event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

    /// @dev 所有者启用或禁用操作员时触发。（操作员可管理所有者所持有的NFTs）
    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);

    /// @notice 统计所持有的NFTs数量
    /// @dev NFT 不能分配给零地址，查询零地址同样会异常
    /// @param _owner ： 待查地址
    /// @return 返回数量，也许是0
    function balanceOf(address _owner) external view returns (uint256);

    /// @notice 返回所有者
    /// @dev NFT 不能分配给零地址，查询零地址抛出异常
    /// @param _tokenId NFT 的id
    /// @return 返回所有者地址
    function ownerOf(uint256 _tokenId) external view returns (address);

    /// @notice 将NFT的所有权从一个地址转移到另一个地址
    /// @dev 如果`msg.sender` 不是当前的所有者（或授权者）抛出异常
    /// 如果 `_from` 不是所有者、`_to` 是零地址、`_tokenId` 不是有效id 均抛出异常。
    ///  当转移完成时，函数检查  `_to` 是否是合约，如果是，调用 `_to`的 `onERC721Received` 并且检查返回值是否是 `0x150b7a02` (即：`bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`)  如果不是抛出异常。
    /// @param _from ：当前的所有者
    /// @param _to ：新的所有者
    /// @param _tokenId ：要转移的token id.
    /// @param data : 附加额外的参数（没有指定格式），传递给接收者。
    function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable;

    /// @notice 将NFT的所有权从一个地址转移到另一个地址，功能同上，不带data参数。
    function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;

    /// @notice 转移所有权 -- 调用者负责确认`_to`是否有能力接收NFTs，否则可能永久丢失。
    /// @dev 如果`msg.sender` 不是当前的所有者（或授权者、操作员）抛出异常
    /// 如果 `_from` 不是所有者、`_to` 是零地址、`_tokenId` 不是有效id 均抛出异常。
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;

    /// @notice 更改或确认NFT的授权地址
    /// @dev 零地址表示没有授权的地址。
    ///  如果`msg.sender` 不是当前的所有者或操作员
    /// @param _approved 新授权的控制者
    /// @param _tokenId ： token id
    function approve(address _approved, uint256 _tokenId) external payable;

    /// @notice 启用或禁用第三方（操作员）管理 `msg.sender` 所有资产
    /// @dev 触发 ApprovalForAll 事件，合约必须允许每个所有者可以有多个操作员。
    /// @param _operator 要添加到授权操作员列表中的地址
    /// @param _approved True 表示授权, false 表示撤销
    function setApprovalForAll(address _operator, bool _approved) external;

    /// @notice 获取单个NFT的授权地址
    /// @dev 如果 `_tokenId` 无效，抛出异常。
    /// @param _tokenId ：  token id
    /// @return 返回授权地址， 零地址表示没有。
    function getApproved(uint256 _tokenId) external view returns (address);

    /// @notice 查询一个地址是否是另一个地址的授权操作员
    /// @param _owner 所有者
    /// @param _operator 代表所有者的授权操作员
    function isApprovedForAll(address _owner, address _operator) external view returns (bool);
}


interface ERC165 {
    /// @notice 是否合约实现了接口
    /// @param interfaceID  ERC-165定义的接口id
    /// @dev 函数要少于  30,000 gas.
    /// @return 合约实现了 `interfaceID`（不为  0xffffffff）返回`true` ， 否则false.
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}



interface ERC721TokenReceiver {
    /// @notice 处理接收NFT
    /// @dev ERC721智能合约在`transfer`完成后，在接收者地址上调用这个函数。
    /// 函数可以通过revert 拒绝接收。返回非`0x150b7a02` 也同样是拒绝接收。
    /// 注意: 调用这个函数的 msg.sender 是ERC721的合约地址
    /// @param _operator ：调用 `safeTransferFrom` 函数的地址。
    /// @param _from ：之前的NFT拥有者
    /// @param _tokenId ： NFT token id
    /// @param _data ： 附加信息
    /// @return 正确处理时返回 `bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`
    function onERC721Received(address _operator, address _from, uint256 _tokenId, bytes _data) external returns(bytes4);
}



interface ERC721Metadata {
    /// @notice NFTs 集合的名字
    function name() external view returns (string _name);

    /// @notice NFTs 缩写代号
    function symbol() external view returns (string _symbol);

    /// @notice 一个给定资产的唯一的统一资源标识符(URI)
    /// @dev 如果 `_tokenId` 无效，抛出异常. URIs在 RFC 3986 定义，
    /// URI 也许指向一个 符合 "ERC721 元数据 JSON Schema" 的 JSON 文件
    function tokenURI(uint256 _tokenId) external view returns (string);
}



interface ERC721Enumerable {
    /// @notice  NFTs 计数
    /// @return  返回合约有效跟踪（所有者不为零地址）的 NFT数量
    function totalSupply() external view returns (uint256);

    /// @notice 枚举索引NFT
    /// @dev 如果 `_index` >= `totalSupply()` 则抛出异常
    /// @param _index 小于 `totalSupply()`的索引号
    /// @return 对应的token id（标准不指定排序方式)
    function tokenByIndex(uint256 _index) external view returns (uint256);

    /// @notice 枚举索引某个所有者的 NFTs
    /// @dev  如果 `_index` >= `balanceOf(_owner)` 或 `_owner` 是零地址，抛出异常
    /// @param _owner 查询的所有者地址
    /// @param _index 小于 `balanceOf(_owner)` 的索引号
    /// @return 对应的token id （标准不指定排序方式)
    function tokenOfOwnerByIndex(address _owner, uint256 _index) external view returns (uint256);
}



```

## ERC-1820 (伪自省注册表合约)  比对 ERC-165 和 ERC-672

ERC1820标准向后兼容 ERC165, ERC1820标准定义了一个通用注册表合约，任何地址（合约或普通用户帐户）都可以注册它支持的接口以及哪个智能合约负责接口实现。

ERC1820标准定义智能合约和普通用户帐户可以向注册表发布其实现了哪些功能（普通用户帐户通过代理合约实现）

任何人都可以查询此注册表，询问哪个地址是否实现了给定的接口以及哪个智能合约处理实现逻辑。

ERC1820注册表合约可以部署在任何链上，并在所有链上的地址是相同的。

接口的后28个字节都为0的话，会认为是 ERC-165 接口，并且注册表将转发到合约以查看是否实现了接口。

此合约还充当 ERC165 缓存，以减少 gas 消耗。




在以太坊上有很多方法定义伪自省，ERC165不能由普通用户帐户使用。 ERC672 则使用了反向 ENS，反向 ENS 有两个问题：增加了不必要的复杂度，其次，ENS 是由多签控制的中心化合约。 从理论上讲，这种多签能够修改系统。

ERC1820标准比 ERC-672 简单得多，并且完全去中心化。

此标准还为所有链提供一个唯一（相同的）地址。从而解决了解决不同链的查找注册表地址的问题。




```

pragma solidity 0.5.3;
// IV is value needed to have a vanity address starting with '0x1820'.
// IV: 53759

/// @dev 如果合约为其他的地址实现了接口， 则必须实现这个接口。
interface ERC1820ImplementerInterface {
    /// @notice 指示合约是否为地址 “addr” 实现接口 “interfaceHash”。
    /// @param interfaceHash 接口名称的 keccak256 哈希值
    /// @param addr 为哪一个地址实现接口
    /// @return 只有当合约为地址'addr'实现'interfaceHash'时返回 ERC1820_ACCEPT_MAGIC
    function canImplementInterfaceForAddress(bytes32 interfaceHash, address addr) external view returns(bytes32);
}


/// @title ERC1820 伪自省注册表合约
/// @notice 该合约是ERC1820注册表的官方实现。
contract ERC1820Registry {
    
    /// @notice ERC165 无效 ID.
    bytes4 constant internal INVALID_ID = 0xffffffff;
    /// @notice ERC165 的 supportsInterface 接口ID (= `bytes4(keccak256('supportsInterface(bytes4)'))`).
    bytes4 constant internal ERC165ID = 0x01ffc9a7;
    /// @notice 如果合约代表某个其他地址实现接口，则返回Magic值。
    bytes32 constant internal ERC1820_ACCEPT_MAGIC = keccak256(abi.encodePacked("ERC1820_ACCEPT_MAGIC"));

    /// @notice 映射地址 及 接口 到对应的实现合约地址
    mapping(address => mapping(bytes32 => address)) internal interfaces;
   
    /// @notice 映射地址 到 管理者
    mapping(address => address) internal managers;
    
    /// @notice 每个地址 和 ERC-165 接口的flag，指示是否被缓存。
    mapping(address => mapping(bytes4 => bool)) internal erc165Cached;

    /// @notice 表示合约implementer是'addr'的'interfaceHash'的'实现者'。
    event InterfaceImplementerSet(address indexed addr, bytes32 indexed interfaceHash, address indexed implementer);
    
    /// @notice 表示'newManager'是'addr'的新管理者的地址。
    event ManagerChanged(address indexed addr, address indexed newManager);

    /// @notice 查询地址是否实现了接口以及通过哪个合约实现的。
    /// @param _addr 查询地址（如果'_addr'是零地址，则假定为'msg.sender'）。
    /// @param _interfaceHash 查询接口，它是接口名称字符串的 keccak256 哈希值
    /// 例如: 'web3.utils.keccak256("ERC777TokensRecipient")' 表示 'ERC777TokensRecipient' 接口.
    /// @return 返回实现者的地址，没有实现返回 ‘0’
    function getInterfaceImplementer(address _addr, bytes32 _interfaceHash) external view returns (address) {
        address addr = _addr == address(0) ? msg.sender : _addr;
        if (isERC165Interface(_interfaceHash)) {
            bytes4 erc165InterfaceHash = bytes4(_interfaceHash);
            return implementsERC165Interface(addr, erc165InterfaceHash) ? addr : address(0);
        }
        return interfaces[addr][_interfaceHash];
    }

    /// @notice 设置某个地址的接口由哪个合约实现，需要由管理员来设置。（每个地址是他自己的管理员，直到设置了一个新的地址）。
    /// @param _addr 待设置的关联接口的地址（如果'_addr'是零地址，则假定为'msg.sender'）
    /// @param _interfaceHash 接口，它是接口名称字符串的 keccak256 哈希值
    /// 例如: 'web3.utils.keccak256("ERC777TokensRecipient")' 表示 'ERC777TokensRecipient' 接口。
    /// @param _implementer 为地址'_addr'实现了 '_interfaceHash'接口的合约地址
    function setInterfaceImplementer(address _addr, bytes32 _interfaceHash, address _implementer) external {
        address addr = _addr == address(0) ? msg.sender : _addr;
        require(getManager(addr) == msg.sender, "Not the manager");

        require(!isERC165Interface(_interfaceHash), "Must not be an ERC165 hash");
        if (_implementer != address(0) && _implementer != msg.sender) {
            require(
                ERC1820ImplementerInterface(_implementer)
                    .canImplementInterfaceForAddress(_interfaceHash, addr) == ERC1820_ACCEPT_MAGIC,
                "Does not implement the interface"
            );
        }
        interfaces[addr][_interfaceHash] = _implementer;
        emit InterfaceImplementerSet(addr, _interfaceHash, _implementer);
    }

    /// @notice 为地址_addr 设置新的管理员地址_newManager， 新的管理员能给'_addr' 调用 'setInterfaceImplementer' 设置是实现者。
    ///  (传 '0x0' 为地址_addr 重置管理员)

    function setManager(address _addr, address _newManager) external {
        require(getManager(_addr) == msg.sender, "Not the manager");
        managers[_addr] = _newManager == _addr ? address(0) : _newManager;
        emit ManagerChanged(_addr, _newManager);
    }

    /// @notice 获取地址 _addr的管理员
    function getManager(address _addr) public view returns(address) {
        // By default the manager of an address is the same address
        if (managers[_addr] == address(0)) {
            return _addr;
        } else {
            return managers[_addr];
        }
    }

    /// @notice 计算给定名称的接口的keccak256哈希值。
    function interfaceHash(string calldata _interfaceName) external pure returns(bytes32) {
        return keccak256(abi.encodePacked(_interfaceName));
    }

    /* --- ERC165 相关方法 --- */

    /// @notice 更新合约是否实现了ERC165接口的缓存。
    function updateERC165Cache(address _contract, bytes4 _interfaceId) external {
        interfaces[_contract][_interfaceId] = implementsERC165InterfaceNoCache(
            _contract, _interfaceId) ? _contract : address(0);
        erc165Cached[_contract][_interfaceId] = true;
    }

    /// @notice 检查合约是否实现ERC165接口。
    //  如果未缓存结果，则对合约地址进行查找。 如果结果未缓存或缓存已过期，则必须通过使用合约地址调用“updateERC165Cache”手动更新缓存。
    /// @param _contract 要检查的合约地址。
    /// @param _interfaceId 要检查ERC165接口。
    /// @return True 如果合约实现了接口返回 true, 否则false.
    function implementsERC165Interface(address _contract, bytes4 _interfaceId) public view returns (bool) {
        if (!erc165Cached[_contract][_interfaceId]) {
            return implementsERC165InterfaceNoCache(_contract, _interfaceId);
        }
        return interfaces[_contract][_interfaceId] == _contract;
    }

    /// @notice 在不使用或更新缓存的情况下检查合约是否实现ERC165接口。
    /// @param _contract 要检查的合约地址。
    /// @param _interfaceId 要检查ERC165接口。
    /// @return True 如果合约实现了接口返回 true, 否则false.
    function implementsERC165InterfaceNoCache(address _contract, bytes4 _interfaceId) public view returns (bool) {
        uint256 success;
        uint256 result;

        (success, result) = noThrowCall(_contract, ERC165ID);
        if (success == 0 || result == 0) {
            return false;
        }

        (success, result) = noThrowCall(_contract, INVALID_ID);
        if (success == 0 || result != 0) {
            return false;
        }

        (success, result) = noThrowCall(_contract, _interfaceId);
        if (success == 1 && result == 1) {
            return true;
        }
        return false;
    }

    /// @notice 检查_interfaceHash 是否是ERC165接口（以28个零结尾）。
    /// @param _interfaceHash 要检查接口 hash。
    /// @return  如果 '_interfaceHash'是ERC165接口返回 True, 否则返回false
    function isERC165Interface(bytes32 _interfaceHash) internal pure returns (bool) {
        return _interfaceHash & 0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF == 0;
    }

    /// @dev 调用合约接口，如果函数不存在也不抛出异常。
    function noThrowCall(address _contract, bytes4 _interfaceId)
        internal view returns (uint256 success, uint256 result)
    {
        bytes4 erc165ID = ERC165ID;

        assembly {
            let x := mload(0x40)               // Find empty storage location using "free memory pointer"
            mstore(x, erc165ID)                // Place signature at beginning of empty storage
            mstore(add(x, 0x04), _interfaceId) // Place first argument directly next to signature

            success := staticcall(
                30000,                         // 30k gas
                _contract,                     // To addr
                x,                             // Inputs are stored at location x
                0x24,                          // Inputs are 36 (4 + 32) bytes long
                x,                             // Store output over input (saves space)
                0x20                           // Outputs are 32 bytes long
            )

            result := mload(x)                 // Load the result
        }
    }
}
```


## ERC-777 (ERC-20 的高级拓展)

如：操作员（operators） 可以代表另一个地址（合约或普通账户）发送代币， 以及 send/receive 加入了钩子函数（hooks ）让代币持有者可以有更多的控制。



使用和发送以太相同的理念发送token，方法为：send(dest, value, data).

合约和普通地址都可以通过注册tokensToSend hook函数来控制和拒绝发送哪些token（拒绝发送通过在hook函数tokensToSend 里 revert 来实现）。

合约和普通地址都可以通过注册tokensReceived hook函数来控制和拒绝接受哪些token（拒绝接受通过在hook函数tokensReceived 里 revert 来实现）。

tokensReceived 可以通过hook函数可以做到在一个交易里完成发送代币和通知合约接受代币，而不像 ERC20 必须通过两次调用（approve/transferFrom）来完成。

持有者可以"授权"和"撤销"操作员（operators: 可以代表持有者发送代币）。 这些操作员通常是（去中心化）交易所、支票处理机或自动支付系统。

每个代币交易都包含 data 和 operatorData 字段， 可以分别传递来自持有者和操作员的数据。

可以通过部署实现 tokensReceived 的代理合约来兼容没有实现tokensReceived 函数的地址。


```
interface ERC777Token {
    function name() external view returns (string memory);
    function symbol() external view returns (string memory);
    function totalSupply() external view returns (uint256);
    function balanceOf(address holder) external view returns (uint256);
    function granularity() external view returns (uint256);


    // 获取代币合约默认的操作员列表。
    //
	//注意: 如果代币合约没有默认操作员, 必须返回空列表。
    function defaultOperators() external view returns (address[] memory);
    function isOperatorFor(
        address operator,
        address holder
    ) external view returns (bool);
    function authorizeOperator(address operator) external;
    function revokeOperator(address operator) external;

    // 持有者自己转账
    function send(address to, uint256 amount, bytes calldata data) external;

    // 操作员 (被授权的) 代替持有者转账
    function operatorSend(
        address from,
        address to,
        uint256 amount,
        bytes calldata data,
        bytes calldata operatorData
    ) external;

    // ##########################################################################
	// ##########################################################################
	// ERC-777 中故意没有定义 mint 函数，用意是不希望限制ERC777标准的使用，因为铸币通常特定于特定的代币。
	//

    function burn(uint256 amount, bytes calldata data) external;
    function operatorBurn(
        address from,
        uint256 amount,
        bytes calldata data,
        bytes calldata operatorData
    ) external;

    event Sent(
        address indexed operator,
        address indexed from,
        address indexed to,
        uint256 amount,
        bytes data,
        bytes operatorData
    );

    // 虽然 没定义 mint 函数，但是还是支持 用户需要 [铸币] 需求的
    event Minted(
        address indexed operator,
        address indexed to,
        uint256 amount,
        bytes data,
        bytes operatorData
    );
    event Burned(
        address indexed operator,
        address indexed from,
        uint256 amount,
        bytes data,
        bytes operatorData
    );

    // 操作员是可以代表持有者发送和销毁代币的账号地址。
    //
	// 当地址成为持有者的操作员时，需要触发 AuthorizedOperator 事件。触发事件时操作员是第 一个参数，持有者是第二个参数。
    event AuthorizedOperator(
        address indexed operator,
        address indexed holder
    );

    // 撤销操作员时需要触发 RevokedOperator 事件。触发事件时操作员是第一个参数，持有者是第二个参数。
    event RevokedOperator(address indexed operator, address indexed holder);

    // 注意: 持有者可以有多个操作员。
}



合约需要用自己的地址通过 ERC1820 标准注册 ERC777Token 接口。

注册方法是调用ERC1820 注册表合约的 setInterfaceImplementer 方法，参数 _addr 及 _implementer 均是合约的地址，_interfaceHash 是 ERC777Token 的 keccak256 哈希值， 即0xac7fbab5f54a3ca8194167523c6753bfeb96a445279294b6125b68cce2177054

如果合约有一个开关来启用或禁用ERC777功能，每次触发开关时，代币合约必须相应地通过ERC1820注册或取消注册ERC777Token接口。

取消注册使用代币合约地址作为参数 _addr 、ERC777Token的keccak256哈希作为接口哈希及'0x0作为实现者参数_implementer调用函数setInterfaceImplementer`， 有关详细信息，请参阅ERC1820中的为接口设置实现地址

当和代币合约进行交互时，所有的数量和余额都是无符号整型 uint256 类型 。总是以18次方存储（ decimals 只能是 18），0.5个代币存储为 500,000,000,000,000,000 (0.5×1018) ERC20内部处理也是一样（不过decimals可为其他值），最小单位相当于 wei, 用户看见的币相当于 ether。









interface ERC777TokensSender {

	// 通知（或请求）从持有人地址发送或销毁 amount 数量的代币。
	//
	// 注意: 请勿在发送（或 ERC20 transfer）或 销毁 之外调用。
    function tokensToSend(
        address operator,
        address from,
        address to,
        uint256 amount,
        bytes calldata userData,
        bytes calldata operatorData
    ) external;
}

调用tokensToSend钩子函数用于通知持有者余额减少（如发送和销毁）。

任何希望收到代币通知的地址（普通地址或合约）都会从需要按ERC1820注册及实现 ERC777TokensSender 接口，描述如下：

通过调用 ERC1820 注册表合约上的 setInterfaceImplementer 函数来完成的，其中持有者地址为地址参数，ERC777TokensSender 的 keccak256哈希值（0x29ddb589b1fb5fc7cf394961c1adf5f8c6454761adf795e67fe149f658abe895）作为接口哈希参数，以及实现ERC777TokensSender的合约作为实现者参数。

注意: 普通地址可以用一个实现的合约（代表普通地址来执行）地址注册，合约可以用自己的地址也可以用另一个地址注册，只要其实现了对应的接口。





interface ERC777TokensRecipient {

	// 用于通知接受代币。
	//
	// 注意: 请勿在发送（或 ERC20 transfer）或 铸币 之外调用。
    function tokensReceived(
        address operator,
        address from,
        address to,
        uint256 amount,
        bytes calldata data,
        bytes calldata operatorData
    ) external;
}





调用tokensReceived钩子函数用于通知接收者余额增加了（如发送和铸币）。

任何希望收到代币通知的地址（普通地址或合约）都会从需要按ERC1820注册及实现 ERC777TokensRecipient 接口，描述如下：

通过调用 ERC1820 注册表合约上的 setInterfaceImplementer 函数来完成的，其中接收者地址为地址参数，ERC777TokensRecipient 的 keccak256哈希值（0xb281fc8c12954d22544db45de3159a39272895b169a852b314f9cc762e44c53b）作为接口哈希参数，以及实现ERC777TokensRecipient的合约作为实现者参数。

注意: 普通地址可以用一个实现的合约（代表普通地址来执行）地址注册，合约可以用自己的地址也可以用另一个地址注册，只要其实现了对应的接口。





与 ERC-20 相比，ERC-777 提供了以下改进。

钩子
钩子是智能合约代码中描述的一个函数。 钩子将会在代币通过合约发送或者接收时调用。 这将允许智能合约对进出的通证做出互动。

** 钩子是使用 ERC-1820 标准注册及发现利用的 **





```



## ERC-55 (混合大小写校验和地址编码)


## ERC-1167  (最小代理合约： 节省Gas部署合约)


```

pragma solidity ^0.8.0;
library Clones {
   

    // @implementation: 被用来做复制 (未使用构造函数做初始化)的合约实例地址
    // @instance: 被复制部署的 新合约实例地址
    function clone(address implementation) internal returns (address instance) {
        <!-- assembly {
            let ptr := mload(0x40)
            mstore(ptr, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
            mstore(add(ptr, 0x14), shl(0x60, implementation))
            mstore(add(ptr, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
            instance := create(0, ptr, 0x37)
        } -->
        assembly {
          let clone := mload(0x40)
          mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
          mstore(add(clone, 0x14), bytes20(implementation))
          mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
          instance := create(0, clone, 0x37)
        }
        require(instance != address(0), "ERC1167: create failed");
    }
    ///....
}

```


## ERC-165  (标准接口检测)


这个提案创建一个标准方法以发布和检测智能合约实现了哪些接口。

接口如何识别。
合约如何发布实现的接口。
如何检测合约是否实现了 ERC-165。
如何检测合约是否实现了某个接口。



```
以下Solidity 代码示例演示如何计算接口标识符：

pragma solidity ^0.4.20;

interface Solidity101 {
    function hello() external pure;
    function world(int) external pure;
}

contract Selector {
    function calculateSelector() public pure returns (bytes4) {
        Solidity101 i;
        return i.hello.selector ^ i.world.selector;
    }
}

注意: 接口不允许可选函数，因此接口标识符不包含它们。






兼容 ERC-165的合约应该实现以下接口（ ERC165.sol）：

pragma solidity ^0.4.20;

interface ERC165 {
    /// @notice 查询一个合约时候实现了一个接口
    /// @param interfaceID  参数：接口ID, 参考上面的定义
    /// @return true 如果函数实现了 interfaceID (interfaceID 不为 0xffffffff )返回true, 否则为 false
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}


这个接口的接口ID 为 0x01ffc9a7， 可以使用 bytes4(keccak256('supportsInterface(bytes4)')); 或者 i.supportsInterface.selector 得到；


因此，合约实现 supportsInterface 函数将返回：

true ：当接口ID interfaceID 是 0x01ffc9a7 (EIP165 标准接口)返回 true
false ：当 interfaceID 是 0xffffffff 返回 false
true ：任何合约实现了接口的 interfaceID 都返回 true
false ：其他的都返回 false。





检测合约是否实现了 ERC-165

在合约地址上使用附加数据（input data）0x01ffc9a701ffc9a700000000000000000000000000000000000000000000000000000000 和 gas 30,000 进行STATICCALL调用，相当于 contract.supportsInterface(0x01ffc9a7)。

如果调用失败或返回false , 说明合约不兼容ERC-165标准
如果返回true，则使用输入数据0x01ffc9a7ffffffff000000000000000000000000000000000000000000000000000000000000进行第二次调用，相当于 contract.supportsInterface(0xffffffff)。
如果第二次调用失败或返回true，则目标合约不会实现ERC-165。
否则它实现了ERC-165。





示例：



pragma solidity ^0.4.20;

import "./ERC165.sol";

interface Simpson {
    function is2D() external returns (bool);
    function skinColor() external returns (string);
}

contract Homer is ERC165, Simpson {
    function supportsInterface(bytes4 interfaceID) external view returns (bool) {
        return
          interfaceID == this.supportsInterface.selector || // 证明实现了 ERC165
          interfaceID == this.is2D.selector
                         ^ this.skinColor.selector; // 证明实现了 Simpson
    }

    function is2D() external returns (bool){}
    function skinColor() external returns (string){}
}

```


## ERC-4626 (代币化资金库标准)


## ERC-1155 (多代币标准)



```
pragma solidity ^0.5.9;

/**
    @title ERC-1155 Multi Token Standard
    @dev See https://learnblockchain.cn/docs/eips/eip-1155.html
    Note: The ERC-165 identifier for this interface is 0xd9b67a26.
 */
interface ERC1155 /* is ERC165 */ {
    /**
        @dev Either `TransferSingle` or `TransferBatch` MUST emit when tokens are transferred, including zero value transfers as well as minting or burning (see "Safe Transfer Rules" section of the standard).
        The `_operator` argument MUST be the address of an account/contract that is approved to make the transfer (SHOULD be msg.sender).
        The `_from` argument MUST be the address of the holder whose balance is decreased.
        The `_to` argument MUST be the address of the recipient whose balance is increased.
        The `_id` argument MUST be the token type being transferred.
        The `_value` argument MUST be the number of tokens the holder balance is decreased by and match what the recipient balance is increased by.
        When minting/creating tokens, the `_from` argument MUST be set to `0x0` (i.e. zero address).
        When burning/destroying tokens, the `_to` argument MUST be set to `0x0` (i.e. zero address).
    */
    event TransferSingle(address indexed _operator, address indexed _from, address indexed _to, uint256 _id, uint256 _value);

    /**
        @dev Either `TransferSingle` or `TransferBatch` MUST emit when tokens are transferred, including zero value transfers as well as minting or burning (see "Safe Transfer Rules" section of the standard).
        The `_operator` argument MUST be the address of an account/contract that is approved to make the transfer (SHOULD be msg.sender).
        The `_from` argument MUST be the address of the holder whose balance is decreased.
        The `_to` argument MUST be the address of the recipient whose balance is increased.
        The `_ids` argument MUST be the list of tokens being transferred.
        The `_values` argument MUST be the list of number of tokens (matching the list and order of tokens specified in _ids) the holder balance is decreased by and match what the recipient balance is increased by.
        When minting/creating tokens, the `_from` argument MUST be set to `0x0` (i.e. zero address).
        When burning/destroying tokens, the `_to` argument MUST be set to `0x0` (i.e. zero address).
    */
    event TransferBatch(address indexed _operator, address indexed _from, address indexed _to, uint256[] _ids, uint256[] _values);

    /**
        @dev MUST emit when approval for a second party/operator address to manage all tokens for an owner address is enabled or disabled (absence of an event assumes disabled).
    */
    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);

    /**
        @dev MUST emit when the URI is updated for a token ID.
        URIs are defined in RFC 3986.
        The URI MUST point to a JSON file that conforms to the "ERC-1155 Metadata URI JSON Schema".
    */
    event URI(string _value, uint256 indexed _id);

    /**
        @notice Transfers `_value` amount of an `_id` from the `_from` address to the `_to` address specified (with safety call).
        @dev Caller must be approved to manage the tokens being transferred out of the `_from` account (see "Approval" section of the standard).
        MUST revert if `_to` is the zero address.
        MUST revert if balance of holder for token `_id` is lower than the `_value` sent.
        MUST revert on any other error.
        MUST emit the `TransferSingle` event to reflect the balance change (see "Safe Transfer Rules" section of the standard).
        After the above conditions are met, this function MUST check if `_to` is a smart contract (e.g. code size > 0). If so, it MUST call `onERC1155Received` on `_to` and act appropriately (see "Safe Transfer Rules" section of the standard).
        @param _from    Source address
        @param _to      Target address
        @param _id      ID of the token type
        @param _value   Transfer amount
        @param _data    Additional data with no specified format, MUST be sent unaltered in call to `onERC1155Received` on `_to`
    */
    function safeTransferFrom(address _from, address _to, uint256 _id, uint256 _value, bytes calldata _data) external;

    /**
        @notice Transfers `_values` amount(s) of `_ids` from the `_from` address to the `_to` address specified (with safety call).
        @dev Caller must be approved to manage the tokens being transferred out of the `_from` account (see "Approval" section of the standard).
        MUST revert if `_to` is the zero address.
        MUST revert if length of `_ids` is not the same as length of `_values`.
        MUST revert if any of the balance(s) of the holder(s) for token(s) in `_ids` is lower than the respective amount(s) in `_values` sent to the recipient.
        MUST revert on any other error.
        MUST emit `TransferSingle` or `TransferBatch` event(s) such that all the balance changes are reflected (see "Safe Transfer Rules" section of the standard).
        Balance changes and events MUST follow the ordering of the arrays (_ids[0]/_values[0] before _ids[1]/_values[1], etc).
        After the above conditions for the transfer(s) in the batch are met, this function MUST check if `_to` is a smart contract (e.g. code size > 0). If so, it MUST call the relevant `ERC1155TokenReceiver` hook(s) on `_to` and act appropriately (see "Safe Transfer Rules" section of the standard).
        @param _from    Source address
        @param _to      Target address
        @param _ids     IDs of each token type (order and length must match _values array)
        @param _values  Transfer amounts per token type (order and length must match _ids array)
        @param _data    Additional data with no specified format, MUST be sent unaltered in call to the `ERC1155TokenReceiver` hook(s) on `_to`
    */
    function safeBatchTransferFrom(address _from, address _to, uint256[] calldata _ids, uint256[] calldata _values, bytes calldata _data) external;

    /**
        @notice Get the balance of an account's tokens.
        @param _owner  The address of the token holder
        @param _id     ID of the token
        @return        The _owner's balance of the token type requested
     */
    function balanceOf(address _owner, uint256 _id) external view returns (uint256);

    /**
        @notice Get the balance of multiple account/token pairs
        @param _owners The addresses of the token holders
        @param _ids    ID of the tokens
        @return        The _owner's balance of the token types requested (i.e. balance for each (owner, id) pair)
     */
    function balanceOfBatch(address[] calldata _owners, uint256[] calldata _ids) external view returns (uint256[] memory);

    /**
        @notice Enable or disable approval for a third party ("operator") to manage all of the caller's tokens.
        @dev MUST emit the ApprovalForAll event on success.
        @param _operator  Address to add to the set of authorized operators
        @param _approved  True if the operator is approved, false to revoke approval
    */
    function setApprovalForAll(address _operator, bool _approved) external;

    /**
        @notice Queries the approval status of an operator for a given owner.
        @param _owner     The owner of the tokens
        @param _operator  Address of authorized operator
        @return           True if the operator is approved, false if not
    */
    function isApprovedForAll(address _owner, address _operator) external view returns (bool);
}



pragma solidity ^0.5.9;

/**
    Note: The ERC-165 identifier for this interface is 0x4e2312e0.
*/
interface ERC1155TokenReceiver {
    /**
        @notice Handle the receipt of a single ERC1155 token type.
        @dev An ERC1155-compliant smart contract MUST call this function on the token recipient contract, at the end of a `safeTransferFrom` after the balance has been updated.
        This function MUST return `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))` (i.e. 0xf23a6e61) if it accepts the transfer.
        This function MUST revert if it rejects the transfer.
        Return of any other value than the prescribed keccak256 generated value MUST result in the transaction being reverted by the caller.
        @param _operator  The address which initiated the transfer (i.e. msg.sender)
        @param _from      The address which previously owned the token
        @param _id        The ID of the token being transferred
        @param _value     The amount of tokens being transferred
        @param _data      Additional data with no specified format
        @return           `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))`
    */
    function onERC1155Received(address _operator, address _from, uint256 _id, uint256 _value, bytes calldata _data) external returns(bytes4);

    /**
        @notice Handle the receipt of multiple ERC1155 token types.
        @dev An ERC1155-compliant smart contract MUST call this function on the token recipient contract, at the end of a `safeBatchTransferFrom` after the balances have been updated.
        This function MUST return `bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))` (i.e. 0xbc197c81) if it accepts the transfer(s).
        This function MUST revert if it rejects the transfer(s).
        Return of any other value than the prescribed keccak256 generated value MUST result in the transaction being reverted by the caller.
        @param _operator  The address which initiated the batch transfer (i.e. msg.sender)
        @param _from      The address which previously owned the token
        @param _ids       An array containing ids of each token being transferred (order and length must match _values array)
        @param _values    An array containing amounts of each token being transferred (order and length must match _ids array)
        @param _data      Additional data with no specified format
        @return           `bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))`
    */
    function onERC1155BatchReceived(address _operator, address _from, uint256[] calldata _ids, uint256[] calldata _values, bytes calldata _data) external returns(bytes4);
}



// ERC1155TokenReceiver ERC-165 rules


function supportsInterface(bytes4 interfaceID) external view returns (bool) {
    return  interfaceID == 0x01ffc9a7 ||    // ERC-165 support (i.e. `bytes4(keccak256('supportsInterface(bytes4)'))`).
            interfaceID == 0x4e2312e0;      // ERC-1155 `ERC1155TokenReceiver` support (i.e. `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)")) ^ bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))`).
}


bytes4 constant public ERC1155_ERC165 = 0xd9b67a26; // ERC-165 identifier for the main token standard.
bytes4 constant public ERC1155_ERC165_TOKENRECEIVER = 0x4e2312e0; // ERC-165 identifier for the `ERC1155TokenReceiver` support (i.e. `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)")) ^ bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))`).
bytes4 constant public ERC1155_ACCEPTED = 0xf23a6e61; // Return value from `onERC1155Received` call if a contract accepts receipt (i.e `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))`).
bytes4 constant public ERC1155_BATCH_ACCEPTED = 0xbc197c81; // Return value from `onERC1155BatchReceived` call if a contract accepts receipt (i.e `bytes4(keccak256("onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)"))`).




pragma solidity ^0.5.9;

/**
    Note: The ERC-165 identifier for this interface is 0x0e89341c.
*/
interface ERC1155Metadata_URI {
    /**
        @notice A distinct Uniform Resource Identifier (URI) for a given token.
        @dev URIs are defined in RFC 3986.
        The URI MUST point to a JSON file that conforms to the "ERC-1155 Metadata URI JSON Schema".
        @return URI string
    */
    function uri(uint256 _id) external view returns (string memory);
}



uint256 baseTokenNFT = 12345 << 128;
uint128 indexNFT = 50;

uint256 baseTokenFT = 54321 << 128;

balanceOf(baseTokenNFT, msg.sender); // Get balance of the base token for non-fungible set 12345 (this MAY be used to get balance of the user for all of this token set if the implementation wishes as a convenience).
balanceOf(baseTokenNFT + indexNFT, msg.sender); // Get balance of the token at index 50 for non-fungible set 12345 (should be 1 if user owns the individual non-fungible token or 0 if they do not).
balanceOf(baseTokenFT, msg.sender); // Get balance of the fungible base token 54321.


```


## ERC-223

## ERC-998 (可组合的 NFT，composable NFTs， CNFT)

它的结构设计是一个标准化延伸可以让任何一个NFT可以拥有其他NFT或FT。转移CNFT时，就是转移CNFT所拥有的整个层级结构和所属关系。简单来说就是ERC-998可以包含多个ERC-721和ERC-20形式的代币。

ERC998ERC721 top-down:必须实现ERC721的接⼝

ERC998ERC20 bottom-up:必须实现ERC20的接⼝

ERC165标准必须适⽤用于所使⽤用的每个ERC998接⼝

http://192.168.10.146:6789


## ERC-4907 (租赁NFT协议)


## ERC-672 (逆向 ENS 伪自省)


```
pragma solidity ^0.4.18;

interface IENS {
    function owner(bytes32 _node) public constant returns(address);
    function resolver(bytes32 _node) public constant returns(address);
    function ttl(bytes32 _node) public constant returns(uint64);
    function setOwner(bytes32 _node, address _owner) public;
    function setSubnodeOwner(bytes32 _node, bytes32 _label, address _owner) public;
    function setResolver(bytes32 _node, address _resolver) public;
    function setTTL(bytes32 _node, uint64 _ttl) public;
}

interface IReverseRegistrar {
    function claimWithResolver(address _owner, address _resolver) public returns (bytes32 node);
}

interface IPublicResolver {
    function addr(bytes32 _node) public constant returns (address ret);
    function setAddr(bytes32 _node, address _addr) public;
}

// [functionSig or interfaceId].[address].addr.reverse

// Base contract for any contract that uses EnsPseudoIntrospection
contract EIP672 {
    address constant ENS_MAIN = 0x314159265dD8dbb310642f98f50C066173C1259b;
    address constant ENS_ROPSTEM = 0x112234455C3a32FD11230C42E7Bccd4A84e02010;
    address constant ENS_RINKEBY = 0xe7410170f87102DF0055eB195163A03B7F2Bff4A;
    address constant ENS_SIMULATOR = 0x8cDE56336E289c028C8f7CF5c20283fF02272182;
    bytes32 constant public REVERSE_ROOT_NODE = keccak256(keccak256(bytes32(0), keccak256('reverse')), keccak256('addr'));
    bytes32 constant public PUBLICRESOLVE_ROOT_NODE = keccak256(keccak256(bytes32(0), keccak256('eth')), keccak256('resolver'));
    IENS public ens;

    function EIP672() public {
      if (isContract(ENS_MAIN)) {
        ens = IENS(ENS_MAIN);
      } else if (isContract(ENS_ROPSTEM)) {
        ens = IENS(ENS_ROPSTEM);
      } else if (isContract(ENS_RINKEBY)) {
        ens = IENS(ENS_RINKEBY);
      } else if (isContract(ENS_SIMULATOR)) {
        ens = IENS(ENS_SIMULATOR);
      } else {
        assert(false);
      }

      IReverseRegistrar reverseRegistrar = IReverseRegistrar(ens.owner(REVERSE_ROOT_NODE));

      IPublicResolver resolver = IPublicResolver(ens.resolver(PUBLICRESOLVE_ROOT_NODE));
      IPublicResolver publicResolver = IPublicResolver(resolver.addr(PUBLICRESOLVE_ROOT_NODE));

      reverseRegistrar.claimWithResolver(
        address(this),
        address(publicResolver));
    }

    function setInterfaceImplementation(string ifaceLabel, address impl) internal {

        bytes32 node = rootNodeForAddress(address(this));
        bytes32 ifaceLabelHash = keccak256(ifaceLabel);
        bytes32 ifaceNode = keccak256(node, ifaceLabelHash);

        ens.setSubnodeOwner(node, ifaceLabelHash, address(this));

        IPublicResolver resolver = IPublicResolver(ens.resolver(PUBLICRESOLVE_ROOT_NODE));
        IPublicResolver publicResolver = IPublicResolver(resolver.addr(PUBLICRESOLVE_ROOT_NODE));

        ens.setResolver(ifaceNode, publicResolver);
        publicResolver.setAddr(ifaceNode, impl);
    }

    function interfaceAddr(address addr, string ifaceLabel) internal constant returns(address) {
        bytes32 node = rootNodeForAddress(address(addr));
        bytes32 ifaceNode = keccak256(node, keccak256(ifaceLabel));
        IPublicResolver resolver = IPublicResolver(ens.resolver(ifaceNode));
        if (address(resolver) == 0) return 0;
        return resolver.addr(ifaceNode);
    }

    function rootNodeForAddress(address addr) internal constant returns (bytes32) {
        return keccak256(REVERSE_ROOT_NODE, keccak256HexAddress(addr));
    }

    function keccak256HexAddress(address addr) private constant returns (bytes32 ret) {
        addr; ret; // Stop warning us about unused variables
        assembly {
            let lookup := 0x3031323334353637383961626364656600000000000000000000000000000000
            let i := 40
        loop:
            i := sub(i, 1)
            mstore8(i, byte(and(addr, 0xf), lookup))
            addr := div(addr, 0x10)
            i := sub(i, 1)
            mstore8(i, byte(and(addr, 0xf), lookup))
            addr := div(addr, 0x10)
            jumpi(loop, i)
            ret := keccak256(0, 40)
        }
    }

    /// @dev Internal function to determine if an address is a contract
    /// @param _addr The address being queried
    /// @return True if `_addr` is a contract
    function isContract(address _addr) constant internal returns(bool) {
        uint size;
        if (_addr == 0) return false;
        assembly {
            size := extcodesize(_addr)
        }
        return size>0;
    }
}
```



EIP 191 EIP-191（关于如何在以太坊合约中处理签名数据） 为了与普通的交易信息区分开，EIP-191定义了presigned message标准

```

0x19 <1 byte version> <version specific data> <data to sign>


注意： 选择0x19作为开始字段的原因是普通的交易信息采用RLP编码，而RLP编码中的0x19开头只能是代表一个值：0x19本身，无法去对后续信息编码。从而与普通的RLP编码的普通交易区分开。



对于目前的Presigned message来讲，一共有三种不同的实现版本：

| Version | Type    | EIP Desc | 
|   ---   |   ---   |   ---   | 
| 0x00    | EIP-191 | Data with intended validator | 
| 0x01    | EIP-712 | Structured data | 
| 0x45    | EIP-191 | personal_sign message | 



对于version type = 0x00, 中存放的是32位的验证者地址

对于version type = 0x45, 在中，将\x19Ethereum Signed Message:\n+len(message)添加到了的前面，然后在进行哈希，签名。

在 Gnosis safe 中，我们主要关注version type = 0x01, 即EIP-712的实现：


```



EIP 712 大对象签名 (结构化数据的签名/验证)




EIP712Domain
顾名思义，是一个与域相关的结构体，总共包含五个字段：

name，合约或者协议的名称
version，合约的版本
chainId，合约部署的链 Id，一般使用 block.chainid，即当前链 Id
verifyingContract，签名的合约地址，一般使用 address(this)，即当前合约
salt，随机数盐，一般不常用


**domainSeparator 域对象**

domainSeparator 是 EIP712 中非常重要的概念，它的作用主要是保证不同的合约和链上的签名是不同的、隔离的。 


EIP712DOMAIN_TYPEHASH 是 EIP712 类型的Hash， 

```
EIP712DOMAIN_TYPEHASH =  keccak256(
    "EIP712Domain(uint256 chainId,address verifyingContract)"
);


bytes32 constant EIP712DOMAIN_TYPEHASH = keccak256(
    "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
);
```

DOMAIN_SEPARATOR 是  EIP712Domain 数据的哈希值，即：

```
DOMAIN_SEPARATOR = keccak256(
    abi.encode(
        EIP712DOMAIN_TYPEHASH,
        keccak256(name，即合约名称),
        keccak256(version，即合约版本),
        chainId,
        verifyingContract
    )
);
```

**EIP712最终的可签名的hash生成公式**

encode(domainSeparator : bytes32, message : Struct) = "x19x01" ‖ domainSeparator ‖ hashStruct(message)


**hashStruct函数的定义为：**

hashStruct(s : Struct) = keccak256(typeHash ‖ encodeData(s))




EIP 1272 合约校验签名方法 (合约签名/验证)

```
外部拥有账户 (EOA) 可以使用其关联的私钥签署消息，但目前合约不能。我们为任何合约提出了一种标准方法来验证 【代表给定合约】 的签名是否有效。这可以通过在isValidSignature(hash, signature)签名合约上实现一个函数来实现，可以调用该函数来验证签名

```



授权签名（EIP-2612）


以太坊登录（EIP-4361）  使用同一个私钥实现 单点登录 ？？ EIP-4361 的目标是为中心化身份提供者提供一种自我托管的替代方案，提高基于以太坊身份验证的链下服务的互操作性，并为钱包供应商提供一致的机器可读消息格式，以实现更好的用户体验和同意管理。


EIP-86 里首次提出的 (抽象账户)，其目的是实现 “交易来源和签名的抽象”


EIP 3074：操作码 AUTH 和 AUTHCALL；EIP 4337 之前的提案


账户抽象化（EIP-2938）： 抽象账户交易需要增加两个新的操作码：NONCE 和 PAYGAS；账户类型仍然保持现有的两种（外部账户和合约账户）；EIP 4337 之前的提案



```

EOA的签名由三部分组成，R,S,v. 其中V值在EIP-155后的定义为：由{0,1}+27 -> {0,1} + chian_id*2 + 35
r: 0xbde0b9f486b1960454e326375d0b1680243e031fd4fb3f070d9a3ef9871ccfd5
s: 0x7d1a653cffb6321f889169f08e548684e005f2b0c3a6c06fba4c4a68f5e00624 
v: 0x1c
=> 编码后的签名为：
bde0b9f486b1960454e326375d0b1680243e031fd4fb3f070d9a3ef9871ccfd5
7d1a653cffb6321f889169f08e548684e005f2b0c3a6c06fba4c4a68f5e00624
1c


```


EIP 1127 


```


EIP 1127 提议对以太坊虚拟机 (EVM) 进行扩展，以允许创建可以执行复杂算术运算的新操作码。这些操作码旨在实现需要复杂数学计算的高级智能合约和去中心化应用程序 (dApp)。

引入 EIP 1127 是为了解决 EVM 计算能力有限的问题，这可能导致难以在以太坊平台上实施某些类型的智能合约和 dApp。EIP 1127 引入的新操作码将允许开发人员执行更复杂的算术运算，从而可以更轻松地在以太坊上构建更高级、更强大的 dApp。

EIP 1127 仍处于提案阶段，尚未在以太坊区块链上实施。如果它获得批准和实施，它可能会对以太坊生态系统和可以在该平台上构建的 dApp 类型产生重大影响。

```


EIP 1077  提供智能合约支付gas的抽象接口



EIP 2771 通过可信转发器接收元交易的合约接口， Recipient合约通过可信Forwarder合约接受元交易 (定义 Recipient 合约接口)



EIP-1474：远程过程调用规范 定义 jsonrpc 的返回状态码


EIP-1167  代理(部署)合约 供了一种低成本克隆合约的方法

```

contract Deployer {
    event InstanceDeployed(address instance);
    
    /**
     * @dev deploy
     *      deploy new contract instance 
     * @param _logic the logic contract address
     * @return address of the new instance
     */
    function deploy(
        address _logic
    ) 
      internal 
      returns (address instance) 
    {
        bytes20 targetBytes = bytes20(_logic);
        assembly {
          let clone := mload(0x40)
          mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
          mstore(add(clone, 0x14), targetBytes)
          mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
          instance := create(0, clone, 0x37)
        }
        emit InstanceDeployed(address(instance));
    }
}




/**

执行：

let clone := mload(0x40)
mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
mstore(add(clone, 0x14), targetBytes)
mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)

将会得到 '0x3d602d80600a3d3981f3363d3d373d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af43d82803e903d91602b57fd5bf3'

而 instance := create(0, clone, 0x37) 则是将 '0x3d602d80600a3d3981f3363d3d373d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af43d82803e903d91602b57fd5bf3' 丢到 go-ethereum 的 create() 函数中执行 [通过 opcreate() 进来]， 查看 go-ethereum 的源码可知 create() 如下：


func (evm *EVM) create(caller ContractRef, codeAndHash *codeAndHash, gas uint64, value *big.Int, address common.Address, typ OpCode) ([]byte, common.Address, uint64, error) {
    // Depth check execution. Fail if we're trying to execute above the
    // limit.
    if evm.depth > int(params.CallCreateDepth) {
        return nil, common.Address{}, gas, ErrDepth
    }
    if !evm.Context.CanTransfer(evm.StateDB, caller.Address(), value) {
        return nil, common.Address{}, gas, ErrInsufficientBalance
    }
    nonce := evm.StateDB.GetNonce(caller.Address())
    if nonce+1 < nonce {
        return nil, common.Address{}, gas, ErrNonceUintOverflow
    }
    evm.StateDB.SetNonce(caller.Address(), nonce+1)
    // We add this to the access list _before_ taking a snapshot. Even if the creation fails,
    // the access-list change should not be rolled back
    if evm.chainRules.IsBerlin {
        evm.StateDB.AddAddressToAccessList(address)
    }
    // Ensure there's no existing contract already at the designated address
    contractHash := evm.StateDB.GetCodeHash(address)
    if evm.StateDB.GetNonce(address) != 0 || (contractHash != (common.Hash{}) && contractHash != emptyCodeHash) {
        return nil, common.Address{}, 0, ErrContractAddressCollision
    }
    // Create a new account on the state
    snapshot := evm.StateDB.Snapshot()
    evm.StateDB.CreateAccount(address)
    if evm.chainRules.IsEIP158 {
        evm.StateDB.SetNonce(address, 1)
    }
    evm.Context.Transfer(evm.StateDB, caller.Address(), address, value)

    // Initialise a new contract and set the code that is to be used by the EVM.
    // The contract is a scoped environment for this execution context only.
    contract := NewContract(caller, AccountRef(address), value, gas)
    contract.SetCodeOptionalHash(&address, codeAndHash)

    if evm.Config.Debug {
        if evm.depth == 0 {
            evm.Config.Tracer.CaptureStart(evm, caller.Address(), address, true, codeAndHash.code, gas, value)
        } else {
            evm.Config.Tracer.CaptureEnter(typ, caller.Address(), address, codeAndHash.code, gas, value)
        }
    }

    start := time.Now()


    ///
    ///
    ///
    /// 这一行就是最终执行 '0x3d602d80600a3d3981f3363d3d373d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af43d82803e903d91602b57fd5bf3' 的入口
    ///
    /// ret 是最后得到的部署完成后的 contract code，但是使用 EIP 1167 得到的其实不是 contract code 而是 一串 runtime code <后续执行该合约都会拉起的 delegatecall 的逻辑>
    /// 
    /// ret 值为 '363d3d373d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af43d82803e903d91602b57fd5bf3'
    ///
    ret, err := evm.interpreter.Run(contract, nil, false)

    // Check whether the max code size has been exceeded, assign err if the case.
    if err == nil && evm.chainRules.IsEIP158 && len(ret) > params.MaxCodeSize {
        err = ErrMaxCodeSizeExceeded
    }

    // Reject code starting with 0xEF if EIP-3541 is enabled.
    if err == nil && len(ret) >= 1 && ret[0] == 0xEF && evm.chainRules.IsLondon {
        err = ErrInvalidCode
    }

    // if the contract creation ran successfully and no errors were returned
    // calculate the gas required to store the code. If the code could not
    // be stored due to not enough gas set an error and let it be handled
    // by the error checking condition below.
    if err == nil {
        createDataGas := uint64(len(ret)) * params.CreateDataGas
        if contract.UseGas(createDataGas) {
            evm.StateDB.SetCode(address, ret)
        } else {
            err = ErrCodeStoreOutOfGas
        }
    }

    // When an error was returned by the EVM or when setting the creation code
    // above we revert to the snapshot and consume any gas remaining. Additionally
    // when we're in homestead this also counts for code storage gas errors.
    if err != nil && (evm.chainRules.IsHomestead || err != ErrCodeStoreOutOfGas) {
        evm.StateDB.RevertToSnapshot(snapshot)
        if err != ErrExecutionReverted {
            contract.UseGas(contract.Gas)
        }
    }

    if evm.Config.Debug {
        if evm.depth == 0 {
            evm.Config.Tracer.CaptureEnd(ret, gas-contract.Gas, time.Since(start), err)
        } else {
            evm.Config.Tracer.CaptureExit(ret, gas-contract.Gas, err)
        }
    }
    return ret, address, contract.Gas, err
}



通过 create() 将执行 前小段代码 '3d602d80600a3d3981f3',其指令为：

[00]    RETURNDATASIZE  
[01]    PUSH1   2d
[03]    DUP1    
[04]    PUSH1   0a
[06]    RETURNDATASIZE  
[07]    CODECOPY    
[08]    DUP2    
[09]    RETURN   // 这行会结束部署

最终得到 ret 为 '363d3d373d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af43d82803e903d91602b57fd5bf3', 这里的含义就是，执行这段 code 时都是执行 delegatecall 逻辑，
即 当EVM运行完上述代码后，go-ethereum客户端会把返回的字节码存储到ret变量中，通过evm.StateDB.SetCode(address, ret)将合约地址和合约代码保存到StateDB数据库中，在我们后期进行调用时，仅运行EVM返回的以下字节码 '363d3d373d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af43d82803e903d91602b57fd5bf3'


【调用】

在后续的调用中每次使用 ret 中的 '363d3d37' 进行获取 用户入参的 calldata ,其中 '363d3d37' 为：

36  CALLDATASIZE    获得calldata的长度  
3d  RETURNDATASIZE  如前文所述，一种向堆栈中推入0的廉价方式   
3d  RETURNDATASIZE  同上  
37  CALLDATACOPY    如前所述复制calldata到内存   


最终吧用户的 calldata 复制到 memory 中，然后 '3d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af4' 为：

3d  RETURNDATASIZE   
3d  RETURNDATASIZE  
3d  RETURNDATASIZE   
36  CALLDATASIZE     
3d  RETURNDATASIZE   
73bebebebebebebebebebebebebebebebebebebebe  PUSH20 bebebebebebebebebebebebebebebebebebebebe 
5a  GAS  
f4  DELEGATECALL  

意思为 进行 delegatecall 调用，被调用的合约为 0xbebebebebebebebebebebebebebebebebebebebe，调用参数为 上述获取的用户 calldata。

最后使用 '3d82803e' 核心操作码为RETURNDATACOPY 读取 delegatecall 的结果 returndata 中的数据到 memory 中。

3d  RETURNDATASIZE  rds success 0   [0-cds]Calldata
82  DUP3    0 rds success 0 [0-cds]Calldata
80  DUP1    0 0 rds success 0   [0-cds]Calldata
3e  RETURNDATACOPY  success 0   [0-rds]Returndata


(辅助操作码为DUPn(其中n∈[1, 16])，其主要为将堆栈中的第n个元素复制并推入堆栈, 如目前堆栈中存在0 1两个元素，使用DUP2后运行完后堆栈为1 0 1，即将第二个元素1复制并推入堆栈)


最最后使用 '903d91602b57fd5bf3' 将结果返回给用户。

|           0x00000024      90             swap1                 0 suc
|           0x00000025      3d             returndatasize        rds 0 suc
|           0x00000026      91             swap2                 success 0 rds
|           0x00000027      602b           push1 0x2b            0x2b success 0 rds
|       ,=< 0x00000029      57             jumpi                 0 rds
|       |   0x0000002a      fd             revert
|       `-> 0x0000002b      5b             jumpdest              0 rds
\           0x0000002c      f3             return






*/





```



EIP-1967  代理存储槽


一个一致的位置，代理存储它们委托给的逻辑合约的地址，以及其他特定于代理的信息。


EIP-897 (PROXY)

EIP-1822 (UUPS)