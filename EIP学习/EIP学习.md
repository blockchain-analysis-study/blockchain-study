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




## ERC-4907 (租赁NFT协议)  作为 EIP-721 的拓展


他作为 ERC-721 的扩展， EIP-4907 增加了一个变量UserInfo，让应用可以查询此NFT当前被租出去的目标地址“user”和出租时间”expires"。如果发现已经超出出租时间，则租赁关系宣告失效。

代码极为简单仅有72行，使用这个标准，就是在原来的ERC721之上新增

1个事件（用于通知链下应用称为事件）

3个方法（用于实现链上数据管理功能）

分别是

UpdateUser 事件：当NFT转移，租赁校色设置时，发出租赁用户改变的通知

setUser 方法：NFT所有者授权者可用，设置此NFTID的出租用户和过期时间

userOf 方法：任何人可用，查询此NFTID的出租用户

userExpires 方法：任何人可用，查询此NFTID的过期时间


传统的 EIP-721 NFT只是通过2个映射 owners 、balances, 即一种字典形式的key-value对应关系的存储结构去记录数据。

```
mapping(uint256 => address)  _owners;// 记录每一个NFTID当前对应的所有者地址
mapping(address => uint256)  _balances; //记录了当前所有者总计持有的NFT数量
```


而 EIP-4907 则是新增了一个数据对象 UserInfo 在所有权的概念之外增加[用户]的维度。

```
struct UserInfo {
        address user;   // 用户地址
        uint64 expires; //用户到期时间
}
```


主要函数：


```

function setUser(uint256 tokenId, address user, uint64 expires) public virtual{

  require(_isApprovedOrOwner(msg.sender, tokenId),"ERC721: transfer caller is not owner nor approved");

  UserInfo storage info =  _users[tokenId];//新增存储登记信息

  info.user = user;   
  info.expires = expires;

  emit UpdateUser(tokenId,user,expires); //发出事件通知链下应用
}



function userOf(uint256 tokenId)public view virtual returns(address){


  if( uint256(_users[tokenId].expires) >=  block.timestamp){ 

    //执行此函数，在未到期的情况下，返回此ID的当前用户地址
    return  _users[tokenId].user; 
  } else {

    //到期情况下，则返回0地址，意未占用
    return address(0);
  }
}



function userExpires(uint256 tokenId) public view virtual returns(uint256){
        return _users[tokenId].expires; //执行此函数，返回此ID的用户过期时间
}



/// 此外， EIP-4907 对标准交易方法 Transfer 增加了一部分内容，通过 _beforeTokenTransfer 实现，就是强制在进行 Transfer 交易转移后就删除掉这部分对用户的信息 (删掉旧的 user 信息)，并且发出事件通知已经用户失效了。

function _beforeTokenTransfer(address from,address to,uint256 tokenId
) internal virtual override{
        super._beforeTokenTransfer(from, to, tokenId);
        //当交易不是自己转自己的情况下，如果有设置“用户”则删除他
        if (from != to && _users[tokenId].user != address(0)) {
            delete _users[tokenId];// 删除用户信息
            emit UpdateUser(tokenId, address(0), 0);// 发出事件通知已删除
        }
}

```


## EIP-5006 (租赁NFT)  针对 EIP-1155 的NFT租赁标准

EIP-5006 的核心价值则是将进一步强化围绕用户创作应用场景上所有权和使用权的分离，明确NFT扩大应用价值的方向。


该标准是EIP-1155的扩展。它提出了一个额外的角色 ( user)，可以授予代表user资产的地址而不是owner.


```

/ SPDX-License-Identifier: CC0-1.0

pragma solidity ^0.8.0;

interface IERC5006 {


    struct UserRecord {
        uint256 tokenId;
        address owner;
        uint64 amount;
        address user;
        uint64 expiry;
    }
    
    /**
     * @dev Emitted when permission for `user` to use `amount` of `tokenId` token owned by `owner`
     * until `expiry` are given.
     */
    event CreateUserRecord(
        uint256 recordId,
        uint256 tokenId,
        uint64  amount,
        address owner,
        address user,
        uint64  expiry
    );

    /**
     * @dev Emitted when record of `recordId` are deleted. 
     */
    event DeleteUserRecord(uint256 recordId);

    /**
     * @dev Returns the usable amount of `tokenId` tokens  by `account`.
     */
    function usableBalanceOf(address account, uint256 tokenId)
        external
        view
        returns (uint256);

    /**
     * @dev Returns the amount of frozen tokens of token type `id` by `account`.
     */
    function frozenBalanceOf(address account, uint256 tokenId)
        external
        view
        returns (uint256);

    /**
     * @dev Returns the `UserRecord` of `recordId`.
     */
    function userRecordOf(uint256 recordId)
        external
        view
        returns (UserRecord memory);

    /**
     * @dev Gives permission to `user` to use `amount` of `tokenId` token owned by `owner` until `expiry`.
     *
     * Emits a {CreateUserRecord} event.
     *
     * Requirements:
     *
     * - If the caller is not `owner`, it must be have been approved to spend ``owner``'s tokens
     * via {setApprovalForAll}.
     * - `owner` must have a balance of tokens of type `id` of at least `amount`.
     * - `user` cannot be the zero address.
     * - `amount` must be greater than 0.
     * - `expiry` must after the block timestamp.
     */
    function createUserRecord(
        address owner,
        address user,
        uint256 tokenId,
        uint64 amount,
        uint64 expiry
    ) external returns (uint256);

    /**
     * @dev Atomically delete `record` of `recordId` by the caller.
     *
     * Emits a {DeleteUserRecord} event.
     *
     * Requirements:
     *
     * - the caller must have allowance.
     */
    function deleteUserRecord(uint256 recordId) external;
}


为了使用，提供了4个接口来管理1155的租赁关系

setUser：设置某个NFT-id下的某个所有者，设置多少个token数量给某个用户

balanceOfUser ：查询哪个NFT-ID的哪个用户租赁到多少

balanceOfUserFromOwner：查询某个NFT-ID的某所有者下的某个用户租赁到多少个

frozenAmountOfOwner：查询某个NFT-ID的所有者其持有的token，已经被租出去多少个了（要冻结掉防止重复出租）


```



EIP-4907 的核心价值是为链上 [原生租赁] 提供了技术支撑，实现了 NFT 的所有权和使用权的分离，是解决NFT流动性短缺问题的重要基础设施。

EIP-5006 的核心价值则是将进一步强化围绕 [用户创作应用场景上] 所有权和使用权的分离，明确NFT扩大应用价值的方向，将会涌现更多丰富的玩法、应用场景和衍生品。




## EIP-5058 (可锁定的 NFT代币) 该协议海贼审核中，不建议使用





本质上他是 ERC721 的拓展，让项目方可以对NFT资产，执行 [锁定] 而不是转移，他新增函数setLockApprovalForAll()以及lockApprove()，这样一来在锁定期结束之前被锁定的 NFT 不能转移。


>有些类似 ERC-721R 提案 (可退款的 NFT) 的做法一样，先将 资金锁定。


```

// SPDX-License-Identifier: CC0-1.0

pragma solidity ^0.8.8;

/**
 * @dev EIP-721 Non-Fungible Token Standard, optional lockable extension
 * ERC721 Token that can be locked for a certain period and cannot be transferred.
 * This is designed for a non-escrow staking contract that comes later to lock a user's NFT
 * while still letting them keep it in their wallet.
 * This extension can ensure the security of user tokens during the staking period.
 * If the nft lending protocol is compatible with this extension, the trouble caused by the NFT
 * airdrop can be avoided, because the airdrop is still in the user's wallet
 */
interface IERC5058 {
    /**
     * @dev Emitted when `tokenId` token is locked by `operator` from `from`.
     */
    event Locked(address indexed operator, address indexed from, uint256 indexed tokenId, uint256 expired);

    /**
     * @dev Emitted when `tokenId` token is unlocked by `operator` from `from`.
     */
    event Unlocked(address indexed operator, address indexed from, uint256 indexed tokenId);

    /**
     * @dev Emitted when `owner` enables `approved` to lock the `tokenId` token.
     */
    event LockApproval(address indexed owner, address indexed approved, uint256 indexed tokenId);

    /**
     * @dev Emitted when `owner` enables or disables (`approved`) `operator` to lock all of its tokens.
     */
    event LockApprovalForAll(address indexed owner, address indexed operator, bool approved);

    /**
     * @dev Returns the locker who is locking the `tokenId` token.
     *
     * Requirements:
     *
     * - `tokenId` must exist.
     */
    function lockerOf(uint256 tokenId) external view returns (address locker);

    /**
     * @dev Lock `tokenId` token until the block number is greater than `expired` to be unlocked.
     *
     * Requirements:
     *
     * - `tokenId` token must be owned by `owner`.
     * - `expired` must be greater than block.number
     * - If the caller is not `owner`, it must be approved to lock this token
     * by either {lockApprove} or {setLockApprovalForAll}.
     *
     * Emits a {Locked} event.
     */
    function lock(uint256 tokenId, uint256 expired) external;

    /**
     * @dev Unlock `tokenId` token.
     *
     * Requirements:
     *
     * - `tokenId` token must be owned by `owner`.
     * - the caller must be the operator who locks the token by {lock}
     *
     * Emits a {Unlocked} event.
     */
    function unlock(uint256 tokenId) external;

    /**
     * @dev Gives permission to `to` to lock `tokenId` token.
     *
     * Requirements:
     *
     * - The caller must own the token or be an approved lock operator.
     * - `tokenId` must exist.
     *
     * Emits an {LockApproval} event.
     */
    function lockApprove(address to, uint256 tokenId) external;

    /**
     * @dev Approve or remove `operator` as an lock operator for the caller.
     * Operators can call {lock} for any token owned by the caller.
     *
     * Requirements:
     *
     * - The `operator` cannot be the caller.
     *
     * Emits an {LockApprovalForAll} event.
     */
    function setLockApprovalForAll(address operator, bool approved) external;

    /**
     * @dev Returns the account lock approved for `tokenId` token.
     *
     * Requirements:
     *
     * - `tokenId` must exist.
     */
    function getLockApproved(uint256 tokenId) external view returns (address operator);

    /**
     * @dev Returns if the `operator` is allowed to lock all of the assets of `owner`.
     *
     * See {setLockApprovalForAll}
     */
    function isLockApprovedForAll(address owner, address operator) external view returns (bool);

    /**
     * @dev Returns if the `tokenId` token is locked.
     */
    function isLocked(uint256 tokenId) external view returns (bool);

    /**
     * @dev Returns the `tokenId` token lock expired time.
     */
    function lockExpiredTime(uint256 tokenId) external view returns (uint256);
}

```


用户授权项目方：lockApprove（许可锁定单个NFT），setLockApprovalForAll（许可锁定该地址下全部NFT）

项目方合约调用：lockFrom（锁定用户的NFT），unlockFrom（解锁用户的NFT）



设定锁定期：

    项目方（第三方）锁定 NFT 时，

    需要指定锁定过期的区块高度，该高度必须大于当前区块高度。

    锁到期后，NFT 自动释放，才可以进行转移。



## EIP-2981 (NFT 版税标准)


该标准允许合约（例如支持ERC-721和ERC-1155接口的 NFT）在每次出售或转售 NFT 时发出要支付给 [NFT 创建者] 或 [权利持有人] 的特许权使用费金额。这适用于希望支持艺术家和其他 NFT 创作者持续资助的 NFT 市场。

特许权使用费必须是自愿的，因为 transferFrom() 包括钱包之间的 NFT 转移在内的转移机制并不总是意味着发生了销售。

市场和个人通过检索特许权使用费支付信息来实施此标准 royaltyInfo()，它指定为给定的销售价格向哪个地址支付多少。支付和通知接收者的确切机制将在未来的 EIP 中定义。该 ERC 应被视为 NFT 版税支付进一步创新的最小、节省 gas 的构建块。


EIP-2981 合约必须支持 EIP-165 


```
pragma solidity ^0.6.0;
import "./IERC165.sol";

///
/// @dev Interface for the NFT Royalty Standard
///
interface IERC2981 is IERC165 {

    /// ERC165 bytes to add to interface array - set in parent contract
    /// implementing this standard
    ///
    /// bytes4(keccak256("royaltyInfo(uint256,uint256)")) == 0x2a55205a
    /// bytes4 private constant _INTERFACE_ID_ERC2981 = 0x2a55205a;
    /// _registerInterface(_INTERFACE_ID_ERC2981);

    /// @notice Called with the sale price to determine how much royalty
    //          is owed and to whom.
    /// @param _tokenId - the NFT asset queried for royalty information
    /// @param _salePrice - the sale price of the NFT asset specified by _tokenId
    /// @return receiver - address of who should be sent the royalty payment
    /// @return royaltyAmount - the royalty payment amount for _salePrice
    function royaltyInfo(
        uint256 _tokenId,
        uint256 _salePrice
    ) external view returns (
        address receiver,
        uint256 royaltyAmount
    );
}

interface IERC165 {
    /// @notice Query if a contract implements an interface
    /// @param interfaceID The interface identifier, as specified in ERC-165
    /// @dev Interface identification is specified in ERC-165. This function
    ///  uses less than 30,000 gas.
    /// @return `true` if the contract implements `interfaceID` and
    ///  `interfaceID` is not 0xffffffff, `false` otherwise
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}



/// 必须检查是否实现了  'royaltyInfo(uint256,uint256)'

/// bytes4(keccak256("royaltyInfo(uint256,uint256)")) == 0x2a55205a

bytes4 private constant _INTERFACE_ID_ERC2981 = 0x2a55205a;

function checkRoyalties(address _contract) internal returns (bool) {
    (bool success) = IERC165(_contract).supportsInterface(_INTERFACE_ID_ERC2981);
    return success;
 }


```


**EIP-2981 是可选的**

不可能知道哪些 [ NFT 转移] 是销售的结果，哪些只是钱包移动或合并他们的 NFT。因此，我们不能强制每个转账功能（例如transferFrom()在 ERC-721 中）都涉及特许权使用费，因为并非每个转账都是需要此类付款的销售。我们相信 NFT 市场生态系统将自愿实施这一版税支付标准，为艺术家/创作者提供持续的资金。NFT 购买者在做出 NFT 购买决定时会将特许权使用费作为评估因素。


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


## EIP-173 (合约所有权标准)


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




EIP-4626 提供了一种将代币投资到投资池 ( 通常称为金库 ) 的标准方法。

主要针对 Vault 进行优化，旨在将所有的 DeFi Vault 标准化，统一变成 ERC-20 形式的交易合约，提供铸造、 存取、查询余额等功能;降低 Vault 的工作量，提高运作效率。

允许为代表单个基础EIP-20 代币份额的代币化保险库实施标准 API。该标准是 EIP-20 代币的扩展，提供了存取代币和读取余额的基本功能。


**动机**

当前的 Vault 缺乏标准化，借贷、聚合器等利息代币有着不一样的实施细节，这使得许多协议在聚合器或插件层的集成变得困难，协议开发者需要实现自己的适配器，这一过程容易出错并且浪费开发资源。


**做法**

所有的 ERC-4626 代币 Vault 都必须先实现 ERC-20 来作为股权代币。 如果一个 Vault 是不可转账的，它需要为 transfer 和 transferFrom 方法实现回滚 Revert 操作。 和 ERC-20 代币有关的操作如 balanceOf、 transfer、totalSupply 将在 Vault 的股份上进行操作，这代表了对 Vault 基础持有量对一小部分的所有权需求。

所有的 ERC-4626 代币 Vault 必须实现 ERC-20 的元数据拓展功能，name 和 symbol 需要反映出底层代币的相关数据。


**方法**


```
asset
totalAssets
convertToShares
convertToAssets
maxDeposit
previewDeposit：允许用户去模拟在当前区块下存入资产后 Vault 发生的变化
deposit：存入底层资产，铸造股权代币
maxMint
previewMint
mint
maxWithdraw
previewWithdraw
withdraw：烧掉股权代币，取回底层资产
maxRedeem
previewRedeem
redeem
```



**事件**


```
// Deposit

event Deposit(address indexed caller, address indexed owner, uint256 assets, uint256 shares);



// Withdraw

event Withdraw(
        address indexed caller,
        address indexed receiver,
        address indexed owner,
        uint256 assets,
        uint256 shares
);


```


**IMMUTABLES**


```
ERC20 public immutable asset;

constructor(
    ERC20 _asset,
    string memory _name,
    string memory _symbol
) ERC20(_name, _symbol, _asset.decimals()) {
    asset = _asset;
}

```


**主逻辑**


```

// Deposit函数：

function deposit(uint256 assets, address receiver) public virtual returns (uint256 shares) {
    
    // Check for rounding error since we round down in previewDeposit.
    require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");

    // Need to transfer before minting or ERC777s could reenter.
    asset.safeTransferFrom(msg.sender, address(this), assets);

    _mint(receiver, shares);

    emit Deposit(msg.sender, receiver, assets, shares);

    afterDeposit(assets, shares);
}


// Mint函数：

function mint(uint256 shares, address receiver) public virtual returns (uint256 assets) {
    assets = previewMint(shares); // No need to check for rounding error, previewMint rounds up.

    // Need to transfer before minting or ERC777s could reenter.
    asset.safeTransferFrom(msg.sender, address(this), assets);

    _mint(receiver, shares);

    emit Deposit(msg.sender, receiver, assets, shares);

    afterDeposit(assets, shares);
}


// Withdraw函数：


function withdraw(
    uint256 assets,
    address receiver,
    address owner
) public virtual returns (uint256 shares) {
    
    shares = previewWithdraw(assets); // No need to check for rounding error, previewWithdraw rounds up.

    if (msg.sender != owner) {
        uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

        if (allowed != type(uint256).max) allowance[owner][msg.sender] = allowed - shares;
    }

    beforeWithdraw(assets, shares);

    _burn(owner, shares);

    emit Withdraw(msg.sender, receiver, owner, assets, shares);

    asset.safeTransfer(receiver, assets);
}


// Redeem函数：

function redeem(
    uint256 shares,
    address receiver,
    address owner
) public virtual returns (uint256 assets) {
    
    if (msg.sender != owner) {
        uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

        if (allowed != type(uint256).max) allowance[owner][msg.sender] = allowed - shares;
    }

    // Check for rounding error since we round down in previewRedeem.
    require((assets = previewRedeem(shares)) != 0, "ZERO_ASSETS");

    beforeWithdraw(assets, shares);

    _burn(owner, shares);

    emit Withdraw(msg.sender, receiver, owner, assets, shares);

    asset.safeTransfer(receiver, assets);
}


```


**账户逻辑**


```

function totalAssets() public view virtual returns (uint256);

function convertToShares(uint256 assets) public view returns (uint256) {
    uint256 supply = totalSupply; // Saves an extra SLOAD if totalSupply is non-zero.

    return supply == 0 ? assets : assets.mulDivDown(supply, totalAssets());
}

function convertToAssets(uint256 shares) public view returns (uint256) {
    uint256 supply = totalSupply; // Saves an extra SLOAD if totalSupply is non-zero.

    return supply == 0 ? shares : shares.mulDivDown(totalAssets(), supply);
}

function previewDeposit(uint256 assets) public view virtual returns (uint256) {
    return convertToShares(assets);
}

function previewMint(uint256 shares) public view virtual returns (uint256) {
    uint256 supply = totalSupply; // Saves an extra SLOAD if totalSupply is non-zero.

    return supply == 0 ? shares : shares.mulDivUp(totalAssets(), supply);
}

function previewWithdraw(uint256 assets) public view virtual returns (uint256) {
    uint256 supply = totalSupply; // Saves an extra SLOAD if totalSupply is non-zero.

    return supply == 0 ? assets : assets.mulDivUp(supply, totalAssets());
}

function previewRedeem(uint256 shares) public view virtual returns (uint256) {
    return convertToAssets(shares);
}

/*///////////////////////////////////////////////////////////////
                 DEPOSIT/WITHDRAWAL LIMIT LOGIC
//////////////////////////////////////////////////////////////*/

function maxDeposit(address) public view virtual returns (uint256) {
    return type(uint256).max;
}

function maxMint(address) public view virtual returns (uint256) {
    return type(uint256).max;
}

function maxWithdraw(address owner) public view virtual returns (uint256) {
    return convertToAssets(balanceOf[owner]);
}

function maxRedeem(address owner) public view virtual returns (uint256) {
    return balanceOf[owner];
}

```


**钩子逻辑**

```

function beforeWithdraw(uint256 assets, uint256 shares) internal virtual {}

function afterDeposit(uint256 assets, uint256 shares) internal virtual {}

```


Vault 的接口是为聚合器设计的。


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



## EIP-3525 (半同质化token ERC3525 一种介于 ERC20 和 ERC721 的产物)



## ERC-223

## ERC-998 (可组合的 NFT，composable NFTs， CNFT)

它的结构设计是一个标准化延伸可以让任何一个NFT可以拥有其他NFT或FT。转移CNFT时，就是转移CNFT所拥有的整个层级结构和所属关系。简单来说就是ERC-998可以包含多个ERC-721和ERC-20形式的代币。

ERC998ERC721 top-down:必须实现ERC721的接⼝

ERC998ERC20 bottom-up:必须实现ERC20的接⼝

ERC165标准必须适⽤用于所使⽤用的每个ERC998接⼝

http://192.168.10.146:6789




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



## EIP-191（关于如何在以太坊合约中处理签名数据） 为了与普通的交易信息区分开，EIP-191定义了presigned message标准

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



## EIP-712 大对象签名 (结构化数据的签名/验证)




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




## EIP-1272 合约校验签名方法 (合约签名/验证)

```
外部拥有账户 (EOA) 可以使用其关联的私钥签署消息，但目前合约不能。我们为任何合约提出了一种标准方法来验证 【代表给定合约】 的签名是否有效。这可以通过在isValidSignature(hash, signature)签名合约上实现一个函数来实现，可以调用该函数来验证签名

```

**EIP1271的流程:**



1. 合约钱包拥有者使用自己的私钥进行签名;
2. 向目标合约提交合约钱包拥有者的签名
3. 目标合约使用此签名向合约钱包进行isValidSignature方法。
4. 合约钱包中的isValidSignature方法会检查签名是否属于合约拥有者。如果属于，合约钱包返回 0x1626ba7e  [其实就是 `bytes4(keccak256("isValidSignature(bytes32,bytes)")` 的值]，否则返回0xffffffff
5. 目标合约接受返回值，通过签名正误决定下一步操作


和 EIP-712 的区别是，限定了 (如：钱包合约)合约的签名交由 特定的用户EOA 代做，校验签名则需要  (如：钱包合约)合约来做。



**代码示例**


```


用户合约：


function callERC1271isValidSignature(
    address _addr,
    bytes32 _hash,
    bytes calldata _signature
) external view {

    // 将 交易签名的校验交给 钱包合约 (证明该交易确实是 钱包合约 发出的)
    bytes4 result = IERC1271Wallet(_addr).isValidSignature(_hash, _signature);
    require(result == 0x1626ba7e, "INVALID_SIGNATURE");
}




钱包合约：


function isValidSignature(
    bytes32 _hash,
    bytes calldata _signature
) external override view returns (bytes4) {
    // Validate signatures
    if (recoverSigner(_hash, _signature) == owner) {
        return 0x1626ba7e;
    } else {
        return 0xffffffff;
    }
}

其中对于 recoverSigner 需要根据具体的签名类型由用户自行编写，较为简单的方法是使用 ecrecover，(用户自行实现自己的钱包合约逻辑)




EIP-1271 合约



contract ERC1271 {

  // bytes4(keccak256("isValidSignature(bytes32,bytes)")
  bytes4 constant internal MAGICVALUE = 0x1626ba7e;

  /**
   * @dev Should return whether the signature provided is valid for the provided hash
   * @param _hash      Hash of the data to be signed
   * @param _signature Signature byte array associated with _hash
   *
   * MUST return the bytes4 magic value 0x1626ba7e when function passes.
   * MUST NOT modify state (using STATICCALL for solc < 0.5, view modifier for solc > 0.5)
   * MUST allow external calls
   */ 
  function isValidSignature(
    bytes32 _hash, 
    bytes memory _signature)
    public
    view 
    returns (bytes4 magicValue);
}



```



## EIP-2612（[链下]授权签名） EIP-20 签名批准的许可延期


在 ERC20 中 具备 approve 和之间的相互作用 transferFrom，这使得代币不仅可以在外部拥有账户 (EOA) 之间转移，还可以在特定应用条件下用于其他合约抽象msg.sender为令牌访问控制的定义机制。

EIP-2612在ERC-20标准上增加了一个新的函数：permit，该功能与approve函数相同，[但是将签名作为参数]。也就是说，**ERC2612的用户签名操作里面已经包含了approve信息。在合约验证成功后，该合约就可以执行transferFrom操作。**这样就能使用户可以通过在他们的交易中附加一个授权签名（Permit）信息来与应用合约交互，而不需要事先授权。

这样的机制是的类似 uniswap 等项目可以不需要用户事先授权，而直接转走代币。

**EIP-2612 使用原理：**

![](./img/EIP-2612_permit_using_flow.jpg)

[首先，EIP-2612 代币合约 (EIP-20的拓展)具备 permit()函数]

1. Alice签署一个链下的 "permit(签名授权)" 信息，表示她希望授予一个合约一个（EIP-2612）代币的使用权。
2. Alice提交签署的消息，作为她与所述协议合约(如： uniswap 合约)交互的一部分。
3. 协议合约(如： uniswap 合约)调用代币合约(EIP-2612 token 合约)上的 "permit()" 方法，它会使用签名授权信息和签名，同时授予协议合约一个授权。
4. 协议合约现在有了授权，所以它可以在代币合约上调用transferFrom()，转账由Alice持有的代币。

这解决了典型ERC20 授权方法的两个问题：

1. 用户永远不需要提交一个单独的approve()交易。
2. 不再有悬空授权的必要之恶，因为许可消息授予的是即时授权，通常会立即花费。因此也可以选择一个更合理的授权额度，更重要的是，在签名授权消息可以被使用代币的时间上有一个到期时间。


但是在 EIP-2612 之前推出的代币 (ERC-20) 并不支持签名授权功能，而且并非所有较新的代币都采用该功能，这就是悲催的现实。因此大多数时候，这种方法不可行。所以我们看 permit2() 的实现，其实就是在旧的(不支持 permit() 的 ERC-20 合约和协议合约间加一层 permit2 合约)。

![](./img/EIP-2612_permit2_using_flow.jpg)





1. Alice在一个ERC20合约 (旧的、不支持permit()的合约)上调用approve()，典型的方式为的Permit2合约授予一个无限的授权。
2. Alice签署一个链下 permit2 消息，该消息表明协议合约(如： uniswap 合约)被允许代表她转账代币。
3. Alice在协议合约(如： uniswap 合约)上调用一个交互函数，将签署的 permit2 消息作为参数传入。
4. 协议合约(如： uniswap 合约)在Permit2合约上调用 permitTransferFrom()，而Permit2合约又使用其授权（在1中授予）在ERC20合约上调用 "transferFrom()"，转账Alice持有的代币。


这样 permit2 合约相当于 为旧有不支持 permit() 的ERC20合约 充当了 permit() 的代理，而 permit2 合约和 ERC20 合约间的交互还是使用旧有的，先 approve() 再 transferFrom() 。
[这种间接性使得Permit2可以将类似于EIP-2612的好处扩展到每一个现有的ERC20代币上。]




## EIP-4361（以太坊登录）  使用同一个私钥实现 单点登录 ？？ EIP-4361 的目标是为中心化身份提供者提供一种自我托管的替代方案，提高基于以太坊身份验证的链下服务的互操作性，并为钱包供应商提供一致的机器可读消息格式，以实现更好的用户体验和同意管理。


## EIP-86 里首次提出的 (抽象账户)，其目的是实现 “交易来源和签名的抽象”


## EIP-3074：操作码 AUTH 和 AUTHCALL；EIP 4337 之前的提案


## EIP-2938（账户抽象化）： 抽象账户交易需要增加两个新的操作码：NONCE 和 PAYGAS；账户类型仍然保持现有的两种（外部账户和合约账户）；EIP 4337 之前的提案



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


## EIP-1127


```


EIP 1127 提议对以太坊虚拟机 (EVM) 进行扩展，以允许创建可以执行复杂算术运算的新操作码。这些操作码旨在实现需要复杂数学计算的高级智能合约和去中心化应用程序 (dApp)。

引入 EIP 1127 是为了解决 EVM 计算能力有限的问题，这可能导致难以在以太坊平台上实施某些类型的智能合约和 dApp。EIP 1127 引入的新操作码将允许开发人员执行更复杂的算术运算，从而可以更轻松地在以太坊上构建更高级、更强大的 dApp。

EIP 1127 仍处于提案阶段，尚未在以太坊区块链上实施。如果它获得批准和实施，它可能会对以太坊生态系统和可以在该平台上构建的 dApp 类型产生重大影响。

```


## EIP-1077  (提供智能合约支付gas的抽象接口) 是一个比 EIP-4337 更早提出的提案



采用 DApp 的主要障碍是需要多个 token 来执行链式操作。允许用户签名消息以显示执行意图，但允许第三方中继器执行消息可以避免此问题，尽管以太坊交易始终需要 ETH，但智能合约可以采用 EIP-191 签名并转发付款激励具有 ETH 的不受信任方执行交易。可以标准化它们的通用格式，以及用户允许以代币支付交易的方式，为应用程序开发人员提供了很大的灵活性，并且可以成为应用程序用户与区块链交互的主要方式。


实现函数：

```

function executeGasRelay(bytes calldata _execData, uint256 _gasPrice, uint256 _gasLimit, address _gasToken, address _gasRelayer, bytes calldata _signature) external;   


function executeGasRelayMsg(uint256 _nonce, bytes memory _execData, uint256 _gasPrice, uint256 _gasLimit, address _gasToken, address _gasRelayer) public pure returns (bytes memory);


function executeGasRelayERC191Msg(uint256 _nonce, bytes memory _execData, uint256 _gasPrice, uint256 _gasLimit, address _gasToken, address _gasRelayer) public view returns (bytes memory);


function lastNonce() public returns (uint nonce);



签名消息需要以下字段：

Nonce：随机数或时间戳；
Execute Data：账户合约要执行的字节码；
Gas Price：gas价格（以所选代币支付）；
Gas Limit：为中继执行预留的gas；
Gas Token：支付gas的代币（以太币为0）；
Gas Relayer：本次调用的gas返还受益人（留0 block.coinbase）


```


**消息签名**


消息必须按照EIP-191标准进行签名，被调用的合约也必须实现EIP-1271，EIP-1271 必须验证签名的消息。

消息必须由正在执行的帐户合约的所有者签名。如果所有者是合约，它必须实现EIP-1271接口并将验证转发给它。



如： 

为了合规，交易必须请求签署一个“messageHash”，它是多个字段的串联。

字段必须构造为此方法：

第一个和第二个字段是为了使其符合EIP-191。

开始交易以byte(0x19)确保签名数据不是有效的以太坊交易。

第二个参数是版本控制字节。

第三个是根据EIP-191版本 0 的验证者地址（账户合约地址） 。

其余参数是 gas 中继的应用程序特定数据：根据EIP-1344 的chainID 、执行随机数、执行数据、约定的 gas 价格、gas 中继调用的 gas 限制、要偿还的 gas 代币和授权接收奖励的 gas 中继器。


```

EIP-191消息必须构造如下：

keccak256(
    abi.encodePacked(
        byte(0x19), //ERC-191 - the initial 0x19 byte
        byte(0x0), //ERC-191 - the version byte
        address(this), //ERC-191 - version data (validator address)
        chainID,
        bytes4(
            keccak256("executeGasRelay(uint256,bytes,uint256,uint256,address,address)")
        ),
        _nonce, 
        _execData,
        _gasPrice,
        _gasLimit,
        _gasToken,
        _gasRelayer
    )
)

```




## EIP-2771 通过可信转发器接收元交易的合约接口， Recipient合约通过可信Forwarder合约接受元交易 (定义 Recipient 合约接口)



该 EIP 定义了合约级协议，用于Recipient合约通过可信Forwarder合约接受元交易。没有进行任何协议更改。Recipient合同通过附加额外的调用数据发送有效msg.sender (称为 msgSender()) 和msg.data (称为 msgData())。


**几个概念**


1. 交易签名者

>签署交易并将交易发送到 Gas Relay

2. Gas Relay

>接收来自交易签名者的链下签名请求并支付 gas 以将其转化为通过 Trusted Forwarder 的有效交易

3. Trusted ForwarderRecipient

>在转发来自交易签名者的请求之前，受信任的合约可以正确验证签名和随机数

4. Recipient

>通过 Trusted Forwarder 接受元交易的合约


![](./img/EIP-2771_using_flow.jpg)


1. 用户先对交易做链下签名
2. 用户将签名好的链下交易发送给  Gas Relay
3. Gas Relay 接收来自交易签名者的链下签名请求并支付 gas 以将其转化为通过 Trusted Forwarder 的有效交易
4. 在转发来自交易签名者的请求之前，受信任的合约 (Trusted Forwarder) 可以正确验证签名和随机数。Trusted Forwarder负责调用Recipient合约，并且必须将[Transaction Signer的地址]（20 字节数据）附加到调用数据的末尾。
5. Trusted Forwarder 将交易转发给 Recipient 合约
6. Recipient 合约然后可以通过执行 3 个操作来提取交易签名者地址：

    - 检查 Gas Relay 是否可信。如何实施不在本提案的范围内。
    - 从调用数据的最后 20 个字节中提取[Transaction Signer的地址]，并将其用作sender交易的原始地址（而不是msg.sender, 因为 msg.sender 是 Gas Relay ）
    - 如果 msg.sender 不是 受信任的Gas Relay（或者如果msg.data短于 20 个字节），则按原样返回原件msg.sender。
(Recipient 必须检查它是否 信任转发方 (Trusted Forwarder) 以防止它提取从不受信任的合约附加的地址数据。这可能导致伪造地址)



```


Recipient 合约必须实现这个功能：

function isTrustedForwarder(address forwarder) external view returns(bool);


如：


contract RecipientExample {

    function purchaseItem(uint256 itemId) external {
        address sender = _msgSender();
        // ... perform the purchase for sender
    }

    address immutable _trustedForwarder;
    constructor(address trustedForwarder) internal {
        _trustedForwarder = trustedForwarder;
    }

    function isTrustedForwarder(address forwarder) public returns(bool) {
        return forwarder == _trustedForwarder;
    }

    function _msgSender() internal view returns (address payable signer) {
        signer = msg.sender;
        if (msg.data.length>=20 && isTrustedForwarder(signer)) {
            assembly {
                signer := shr(96,calldataload(sub(calldatasize(),20)))
            }
        }    
    }

}

```


## EIP-1474  远程过程调用规范 定义 jsonrpc 的返回状态码


## EIP-1167  代理(部署)合约 供了一种低成本克隆合约的方法

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


最终把用户的 calldata 复制到 memory 中，然后 '3d3d3d363d73bebebebebebebebebebebebebebebebebebebebe5af4' 为：

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



## EIP-897 (PROXY) 委托代理

具体的做法是在 proxy 合约中内置 实现(逻辑)合约 地址，然后通过 代理合约的 fallback() 中实现 assambly delegatecall 统一调用 用户入参的  calldata。

缺点：是可能出现 存储变量被覆盖问题。

解决：使用代理合约与逻辑合约继承同一合约以避免逻辑合约地址存储冲突的解决方案。

https://hugo.wongssh.cf/posts/foundry-contract-upgrade-part1/


```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract ProxyStorage {
    address public otherContractAddress;

    function setOtherAddressStorage(address _otherContract) internal {
        otherContractAddress = _otherContract;
    }
}

contract DataLayout is ProxyStorage {
    uint256 public number;
}

contract NumberStorage is DataLayout { 
    function setNumber(uint256 _uint) public {
        number = _uint;
    }

    function getNumber() public view returns (uint256) {
        return number;
    }
}

contract NumberStorageUp is DataLayout {
    function setNumber(uint256 _uint) public {
        number = _uint;
    }

    function getNumber() public view returns (uint256) {
        return number;
    }

    function addNumber() public {
        number += 1;
    }
}

contract ProxyEasy is ProxyStorage {
    constructor(address _otherContract) {
        setOtherAddress(_otherContract);
    }

    function setOtherAddress(address _otherContract) public {
        super.setOtherAddressStorage(_otherContract);
    }

    fallback() external {
        address _impl = otherContractAddress;

        assembly {
            let ptr := mload(0x40)
            calldatacopy(ptr, 0, calldatasize())
            let result := delegatecall(gas(), _impl, ptr, calldatasize(), 0, 0)
            let size := returndatasize()
            returndatacopy(ptr, 0, size)

            switch result
            case 0 {
                revert(ptr, size)
            }
            default {
                return(ptr, size)
            }
        }
    }
}


```



## EIP-1822 Universal Upgradeable Proxy Standard (UUPS) 通用可升级代理标准

类比EIP-897是使用了一个距0地址非常远的随机地址中存储逻辑合约地址

```
contract Proxy {
    constructor(bytes memory constructData, address contractLogic) public {
        assembly {
            sstore(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7, contractLogic)
        }
        (bool success, bytes memory _ ) = contractLogic.delegatecall(constructData);
        require(success, "Construction failed");
    }

    fallback() external payable {
        assembly {
            let contractLogic := sload(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7)
            calldatacopy(0x0, 0x0, calldatasize)
            let success := delegatecall(sub(gas, 10000), contractLogic, 0x0, calldatasize, 0, 0)
            let retSz := returndatasize
            returndatacopy(0, 0, retSz)
            switch success
            case 0 {
                revert(0, retSz)
            }
            default {
                return(0, retSz)
            }
        }
    }
}



/// 另一个标准   Proxiable Contract


contract Proxiable {
    // Code position in storage is keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"

    function updateCodeAddress(address newAddress) internal {
        require(
            bytes32(
                0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7
            ) == Proxiable(newAddress).proxiableUUID(),
            "Not compatible"
        );
        assembly {
            sstore(
                0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7,
                newAddress
            )
        }
    }

    function proxiableUUID() public pure returns (bytes32) {
        return
            0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7;
    }
}


```




## EIP-1967  代理存储槽  [EIP-1822 UUPS的进一步标准化版本]


一个一致的位置，代理存储它们委托给的逻辑合约的地址，以及其他特定于代理的信息。


由于EIP-1822较为古老，在其文档中仍存在大量的解释性内容由于解释合约运行的原理。但在EIP-1967中，由于其创建时间较晚，合约代理的基本原理已被智能合约开发者所熟知，所以在EIP-1967的文档中没有介绍代理合约的基本原理，主要是对存储槽、事件进行了标准化和解释。


**与 EIP-1822 的不同点：**


1. 使用了 `keccak256('eip1967.proxy.implementation') - 1` 而不是使用 `keccak256("PROXIABLE")`

在EIP-1822中我们一般采用keccak256("PROXIABLE")值，即0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7，该值其实可以有开发者自行决定。

但在EIP-1967中，为了方便区块链浏览器的访问，该地址被标准化为 keccak256('eip1967.proxy.implementation') - 1，即0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc。

使用 keccak256('eip1967.proxy.implementation') - 1 而不是 keccak256('eip1967.proxy.implementation') 的做法是【避免潜在的攻击】。

首先我们为什么使用 `bytes32 internal constant _IMPLEMENTATION_SLOT = keccak256('eip1967.proxy.implementation');` ? 假设 代理合约<存储所在> 和 逻辑合约中可能出现在同一个 slot 处出现不一样的key，最终可能导致原本在代理合约内部用来存储 逻辑合约实例地址的 slot 被逻辑合约的某个 状态值覆盖了 【原因： slot 一样，而通过 delegatecall 最终导致 代理合约中的 slot 上的值被修改】。EIP-1967 使用了【非结构化存储】选择一个远离 0 slot 的slot存储 逻辑合约地址。然而直接使用 `keccak256('eip1967.proxy.implementation')` 会导致上述问题没有从根源解决，因为假设 代理合约 和 逻辑合约的 代码都是开源的，且 逻辑合约中存在 mapping 类型的属性，我们知道 mapping 的存储 slot 为 `keccak256(key, slot)` 即为 mapping中某个key的存储 slot，而我们知道 mapping 属性的位置slot那么我们可以构造出一个key满足 `keccak256(key, slot) == keccak256('eip1967.proxy.implementation')` 即 构造出 `key, slot == 'eip1967.proxy.implementation'`, 从而最终将 代理合约中存储逻辑合约实例地址的 slot 覆盖。而使用 `keccak256('eip1967.proxy.implementation') - 1`, 则 攻击者需要有 `keccak256('eip1967.proxy.implementation') - 1 == keccak256(X) == keccak256(key, slot)`, 而该等式中的 X 此处无法推算出来 (是个未知数了)，则无法伪造一个满足 `keccak256(X) == keccak256(key, slot)` 的key。


2. 当[信标代理合约]地址不为空时，逻辑合约地址可以为空


ERC标准规定信标代理的地址存储在 `bytes32(uint256(keccak256('eip1967.proxy.beacon')) - 1)` 中，其值为 `0xa3f0ad74e5423aebfd80d3ef4346578335a9a72aeaee59ff6cb3582b35133d50`。


信标代理的作用是[同一逻辑合约]可以实现[多个代理合约]共同代理。**这里类似最小代理合约使用的代理部署的代理架构**



EIP-1967也规定了合约拥有者的地址存储位置(Admin address)，该存储操位于bytes32(uint256(keccak256('eip1967.proxy.admin')) - 1)，即 `0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103`。


**EIP-1967 的插槽位置：**


|插槽名称|计算公式|值|
|---|---|---|
|逻辑合约地址|bytes32(uint256(keccak256(’eip1967.proxy.implementation’)) - 1)|0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc|
|信标代理合约地址|bytes32(uint256(keccak256(’eip1967.proxy.beacon’)) - 1)|0xa3f0ad74e5423aebfd80d3ef4346578335a9a72aeaee59ff6cb3582b35133d50|
|代理合约拥有者|bytes32(uint256(keccak256(’eip1967.proxy.admin’)) - 1)|0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103|



**EIP-1967 的主要事件：**

|事件名称|代码|触发条件|
|---|---|---|
|Upgraded|`event Upgraded(address indexed implementation);`|逻辑合约地址升级|
|BeaconUpgraded|`event BeaconUpgraded(address indexed beacon);`|信标代理合约升级|
|AdminChanged|`event AdminChanged(address previousAdmin, address newAdmin);`|合约拥有者改变|


**EIP-1967 合约图解：**


代理合约的结构


![](./img/mermaid-diagram-2022-07-25-195542.svg)

UML

![](./img/mermaid-diagram-2022-07-25-210049.svg)


在UML图中，以#开头的函数代表此函数仅能在合约内调用internal; +开头的函数或变量代表public; -开头的函数或变量代表private，即不能在合约外调用; 斜体函数名为抽象函数，即在当前合约内仅注明了函数名，我们需要在继承合约内实现。



## EIP-2535  (钻石模型, 多面代理)   创建可在部署后扩展的模块化智能合约系统


**该标准是对 EIP-1538 的改进。该标准的相同动机适用于该标准。**原来的 EIP-1538 (https://eips.ethereum.org/EIPS/eip-1538) 已经被撤回，不做讨论。



>该提案将钻石标准化，钻石是模块化的智能合约系统，部署后可以升级/扩展，几乎没有大小限制。从技术上讲，菱形是一种具有外部功能的合约，这些功能由称为facets的合约提供。Facets 是单独的、独立的合约，可以共享内部函数、库和状态变量。


动机：

1.无限合同功能的单一地址。

>为合约功能使用单一地址使得部署、测试以及与其他智能合约、软件和用户界面的集成变得更加容易。

2. 合约超过了 24KB 的最大合约大小。

>可能拥有将其保留在单个合约或单个合约地址中的相关功能。钻石没有最大合约大小。

3. 菱形提供了一种组织合约代码和数据的方法。

>可能想要构建一个具有很多功能的合约系统。菱形提供了一种系统的方法来隔离不同的功能并将它们连接在一起，并根据需要以节省气体的方式在它们之间共享数据。

4. 钻石提供了一种升级功能的方法。

>可升级钻石可以升级以添加/替换/删除功能。因为钻石没有最大合同大小，所以随着时间的推移可以添加到钻石的功能数量没有限制。无需重新部署现有功能即可升级钻石。可以添加/替换/移除钻石的某些部分，同时保留其他部分。


5. 钻石可以是不变的。

>可以在以后部署不可变钻石或使可升级钻石不可变。

6. 钻石可以重用已部署的合约。

>链上合约可用于创建钻石，而不是将合约部署到已经部署的现有区块链。可以从现有部署的合约创建自定义钻石。这使得创建链上智能合约平台和库成为可能。



基本功能：


1. 在合约中原子性的增加、减少或替换函数
2. 将合约内函数的增加、减少、替换通过约定的event给出
3. 通过提供合约查询公开函数的信息
4. 解决以太坊合约最大24KB的限制
5. 允许可升级函数在未来更改为不可升级函数



**和 EIP-1967 区别**

此模型与我们之前介绍的EIP-1967等传统代理模型不同，此模型没有采用无序存储合约地址的方法，而是在通过映射约定不同的函数和对应的合约地址，此方法属于有序存储。

我们之前一直强调代理合约的关键在于逻辑合约地址存储和合约数据存储，在此之前我们介绍了通过继承解决合约存储问题和通过随机地址槽存储数据解决数据冲突问题。


相对于 EIP-897 的继承存储，和 EIP-1967 的非结构化存储 来说，EIP-2535 使用了 `Diamond Storage` 和 `AppStorage`，即：依旧是选择随机存储槽存储逻辑合约所需要的数据，此方案通常被称为Diamond Storage。与之前仅提供随机数据存储槽存储代理合约地址不同，在EIP2535中，我们需要为不同类型的逻辑合约设计存储地址以保证其数据存储不会与其他逻辑合约冲突。

![](./img/EIP-2535_DiamondStorage.png)



如: 两个代理合约调用两个逻辑合约。 类比下图显示了使用相同两个刻面的两颗钻石：

![](.img/EIP-2535_facetreuse.png)


1. Diamond Storage 代码示例：

```
library MyStructStorage {
  bytes32 constant MYSTRUCT_POSITION = 
    keccak256("com.mycompany.projectx.mystruct");

  struct MyStruct {
    uint var1;
    bytes var2;
    mapping (address => uint) var3;
  }

  function myStructStorage()
    internal 
    pure 
    returns (MyStruct storage mystruct) 
  {
    bytes32 position = MYSTRUCT_POSITION;
    assembly {
      mystruct.slot := position
    }
  }
}



contract TestStruct {
    function myFunction(uint256 inputUint) external {

        // library的调动默认用了 (只能) delegatecall (库合约的调用是不需要使用delegatecall关键词的), 我们可以使用 `LibraryName.functionName()` 的形式直接调用库中的函数。
        MyStructStorage.MyStruct storage mystruct = MyStructStorage.myStructStorage();

        mystruct.var1 = inputUint;
    }
}



缺点： 

此方案的问题在于我们每次进行数据调用都需要调用一次MyStructStorage.MyStruct storage mystruct = MyStructStorage.myStructStorage();代码获得mystruct对象再进行数据修改，这是麻烦和乏味的。

```



2. AppStorage 代码示例：


或者 我们通过合约的internal标识避免变量名冲突问题 (AppSotrage)。

如：

```

第一步，在AppStorage.sol文件中把所有需要的存储变量到AppStorage结构体中，如下代码:

struct AppStorage {
  uint256 secondVar;
  uint256 firstVar;
  uint256 lastVar;
}


第二步，在需要使用变量的切面函数使用AppStorage internal s声明结构体。


import "./AppStorage.sol"

contract StakingFacet {
  AppStorage internal s;

  function myFacetFunction() external {
    s.lastVar = s.firstVar + s.secondVar;
  }
}


如果读者后期需要升级合约，需要在struct AppStorage结构体后增加变量，不可以打乱变量排列顺序，这与继承存储方案是一致的。


```


3. 混用 Diamond Storage 和 AppStorage


```

需要在AppStorage.sol加入以下代码


library LibAppStorage {

    function diamondStorage() internal pure returns (AppStorage storage ds) {
        assembly {
            ds.slot := 0
        }
    }
}



```


**名词**


1. 钻石合约(Diamond)

>即直接与用户交互的代理合约，是delegatecall的发起者和状态变量的存储者

2. 切面合约(Facet)，即逻辑合约

>用于编写操作状态变量的函数

3. 放大镜合约(loupe)。

>一个特殊的切面合约，用于返回钻石中各个切面合约的具体内容，包括切面合约地址、切面合约中的函数选择器等内容。


在EIP-2535中，最重要的函数就是 `diamondCut()`，该函数的功能是[增加、修改或替换]钻石合约中[函数和切面合约]。如果钻石合约为不可变合约，则可以不实现此函数。该函数运行后会抛出 `DiamondCut` 事件。此函数的接口如下:

```
interface IDiamondCut {

    enum FacetCutAction {Add, Replace, Remove}
    // Add=0, Replace=1, Remove=2

    struct FacetCut {
        address facetAddress;
        FacetCutAction action;
        bytes4[] functionSelectors;
    }

    /// @notice Add/replace/remove any number of functions and optionally execute
    ///         a function with delegatecall
    /// @param _diamondCut Contains the facet addresses and function selectors
    /// @param _init The address of the contract or facet to execute _calldata
    /// @param _calldata A function call, including function selector and arguments
    ///                  _calldata is executed with delegatecall on _init
    function diamondCut(
        FacetCut[] calldata _diamondCut,
        address _init,
        bytes calldata _calldata
    ) external;

    event DiamondCut(FacetCut[] _diamondCut, address _init, bytes _calldata);
}


```

Diamond 是一个合约，它将调用委托给称为面的实现合约。Diamond 合约实现了一个 diamondCut 函数，它提供了添加/替换/删除的能力。除了 diamondCut 函数之外，还实现了一组 loupe 函数。这些函数显示了关于所实现的Diamond 的信息。Diamond 合约维护了一个selectorToFacet的映射，这基本上是一个函数签名到实现合约地址的映射。当用户调用一个函数时，Diamond 代理的回退函数使用函数签名来寻找这个映射中的实现地址。在找到函数签名的地址后，回退函数只是将调用委托给实现合约，并将响应返回给用户。

Diamond 代理依靠的是一种存储模式。该代理被一些被称为面的实现合约所共享。理想情况下，每个切面都有自己的存储。然而，根据实施要求，面可以与其他面共享存储。 DiamondStorage 和 AppStorage 是一些流行的存储模式，分别用于[隔离]和[共享]存储。




说白了就是将一些逻辑合约的地址和逻辑合约的 finction sig 进行映射，存到代理合约中，用来实现 modules 功能。






