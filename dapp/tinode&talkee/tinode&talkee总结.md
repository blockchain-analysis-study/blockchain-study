# tinode 和 talkee 总结


## <span style="color: red;"> tinode </span>

#### 消息类型

**客户端**

```
hi: hi 消息(进入web app 界面时触发，有点 ping 的效果),打开连接后，客户端必须向服务器发出 {hi} 消息。服务器以 {ctrl} 消息响应，指示成功或错误。
acc: 创建 或 更新user 
login: 登录消息
sub: 订阅请求消息
leave: 取消订阅
pub: 客户端向主题订阅者｛pub｝发布数据
get: 对主题状态｛get｝的查询
set: 主题状态｛set｝的更新
del: 删除消息或主题｛del｝
note: 客户端为主题订阅者生成的通知
```

**服务端**

```
ctrl: 指示错误或成功条件的一般响应。消息被发送到原始会话 （其实回应对应的客户端消息）
data: 服务端的 data 消息，将广播给所有具有 R 权限的主题订阅者
meta: 有关主题元数据或订阅者的信息，作为对 {get} 、 {set} 或 {sub} 消息的响应而发送到原始会话
pres: Tinode 使用 {pres} 消息通知客户端重要事件
info: 已转发客户端生成的通知 {note} 。服务器保证消息符合此规范并且 topic 和 from 字段的内容是正确的。其他内容是从 {note} 消息中逐字复制的，可能会不正确或具有误导性。
```

#### 用户

**身份验证级别**  

首次建立连接时，客户端应用程序可以发送 {acc} 或 {login} 消息，在一个级别对用户进行身份验证

1. auth: 需要经过身份校验的
2. anon: 匿名用户

**身份校验**

用于 {acc} 和 {login} 消息

1. token: 通过加密令牌提供身份验证
2. basic: 通过登录密码对提供身份验证
3. anonymous: 专为临时用户的情况而设计，例如：通过聊天处理客户支持请求

用于创建帐户，但不能用于登录：用户使用 anonymous 方案创建帐户并获得用于后续 token 登录的加密令牌。如果令牌丢失或过期，用户将无法再访问该帐户

4. rest: 是一种元方法，它允许通过 JSON RPC 使用外部身份验证系统 (即 第三方登录方式)



#### 注册流程 (重置密码也走这个流程)

![](./img/tinode_acc.svg)


在创建帐户期间只能使用 basic 和 anonymous 。 basic 要求用户生成唯一的登录名和密码并将其发送到服务器。 anonymous 不会。

用户可以选择设置 {acc login=true} 以使用新帐户进行即时身份验证。

- 当 login=false （或未设置）时，创建新帐户，但创建该帐户的会话的身份验证状态保持不变。
- 当 login=true 服务器将尝试使用新帐户对会话进行身份验证时，对 {acc} 请求的 {ctrl} 响应将在成功时包含身份验证令牌。


用户可以通过发出 {acc} 请求来更改身份验证参数，例如更改登录名和密码。目前只有 basic 认证支持更改参数：

```
acc: {
  id: "1a2b3", // string, client-provided message id, optional
  user: "usr2il9suCbuko", // user being affected by the change, optional
  token: "XMg...g1Gp8+BO0=", // authentication token if the session
                             // is not yet authenticated, optional.
  scheme: "basic", // authentication scheme being updated.
  secret: base64encode("new_username:new_password") // new parameters
}
```

为了仅更改密码， username 应留空，即 secret: base64encode(":new_password") 。

对于 token 来说：

- 如果会话未通过身份验证 (acc 或者 login)，则请求必须包含 token 。它可以是在登录期间获得的常规身份验证令牌，也可以是通过重置密码过程收到的受限令牌。
- 如果会话已通过身份验证，则不得包含 token 。



**交互消息,out是客户端发出,in是客户端接收**


1. 客户端发出 `hi` 消息,里面主要包括了版本,ua,lang消息
2. 服务端解析客户端发来的 `Hi` 消息，并组装 `Ctrl` 消息返回
3. 服务端返回 `ctrl` 消息;其实就是返回了服务端的一堆配置,返回的代码是 `201`
4. 客服端登录,发送 `acc` 消息
5. 服务端判断当前 `acc` 消息是注册还是更新密码。
6. 如果为更新密码，则调用 `auth_basic.Authenticate()` 对客户端发来的旧密码进行校验
7. 如果为注册，则服务端调用 `Create()` 创建新用户，并调用 `AddRecord()` 创建新用户的 auth 信息 (用于未来 `login` 用)
8. 服务端创建 `token` (jwt) ，组装 `Ctrl` 消息返回
9. 服务端返回ctrl消息,主要内容: `authlvl`, `token` 和 `expires`



#### 登录流程

![](./img/tinode_login.svg)


通过发出 {login} 请求执行登录。只能使用 basic 和 token 登录。对任何登录的响应是一条 {ctrl} 消息，其中包含一个代码 200 和一个 token，可以在随后的 token 身份验证登录中使用，或者代码 300 请求附加信息，例如验证凭据或响应方法 -多步身份验证中的相关质询，或代码 4xx 错误。

**交互消息,out是客户端发出,in是客户端接收**


1. 客户端发出 `hi` 消息,里面主要包括了版本,ua,lang消息

```
out: {"hi":{"id":"83985","ver":"0.16.3","ua":"TinodeWeb/0.16.3 (Chrome/80.0; Win32); tinodejs/0.16.3","lang":"zh-CN"}} 
```

2. 服务端解析客户端发来的 `Hi` 消息，并组装 `Ctrl` 消息返回

3. 服务端返回 `ctrl` 消息;其实就是返回了服务端的一堆配置,返回的代码是 `201`

```
in: {"ctrl":{"id":"83985","params":{"build":"mongodb:undef","maxFileUploadSize":8388608,"maxMessageSize":262144,"maxSubscriberCount":128,"maxTagCount":16,"ver":"0.16"},"code":201,"text":"created","ts":"2020-03-23T07:55:58.439Z"}} 
```

4. 客服端登录,发送 `login` 消息

```
out: {"login":{"id":"83986","scheme":"basic","secret":"bWExMjM0Ok1hQDEyMzQ="}}
```

> - 这个 `secret`没有任何加密之类的保护措施,就是 `用户名:密码`
> - 登录的时候, 使用的是 `schema` 是 basic

5. 服务端调用 `auth_basic.Authenticate()` 对客户端发来的 `login` 消息进行校验
6. 服务端调用 `GetAuthUniqueRecord()` 查询用户的 `username` 和 `passhash` 并对 password 求 Hash 做对比
7. 服务端创建 `token` (jwt) ，组装 `Ctrl` 消息返回

8. 服务端返回ctrl消息,主要内容: `authlvl`, `token` 和 `expires`

```
in:{"ctrl":{"id":"83986","params":{"authlvl":"auth","expires":"2020-04-06T07:55:58.504Z","token":"Q31lre9hlo6O4IpeFAABAAEAUo6b0ItShDLjnW15vXYZzvVjg2zE++bUZ6swdvVbcro=","user":"usrQ31lre9hlo4"},"code":200,"text":"ok","ts":"2020-03-23T07:55:58.441Z"}} 
```
> - `expires` 是在配置文件的 `token.expire_in` 里指定的
> - `token`是没有存储在数据库或者redis之类的,比较像jwt
> - 配置文件的 `token.key` 设置的 salt,所有的用户用都是同一个


#### 支持的DB

1. mangodb
2. mysql
3. rethinkdb


#### <span style="color: red;"> 改造点 (最小化) <span/>

<span style="color: cornflowerblue;"> **合并登录&注册: acc&login => login** <span/>

1. 更改 username:password 形式的注册，为 username:address 形式，数据库中的 user 信息中的 username、passwordHash 变更为保存 username、address，保留其他流程 (如： token 等)
2. 移除 acc 消息中修改 password 部分逻辑
3. 合并 acc 注册和 login 登录流程为只有 login 流程
4. 变更以 EIP-4631 形式做签名校验，通过后 使用 address 拉起数据库中存储的 user 信息，其他流程保留


<span style="color: darkgreen;"> **客户端** <span/>

客户端需要配合 chat 的go代码逻辑做相应的变动，包括： webapp  (android、ios、命令行客户端、聊天机器人可以先不支持)

<span style="color: purple;"> **README 文档** <span/>

调整文档中 acc 和 login 相关描述


## <span style="color: red;"> talkee </span>

主页： https://pando.im/talkee

github： https://github.com/pandodao/talkee


建立在 Mixin Network 上的 im，属于 pandodao 。支持 EIP-4361 (基于 MVM， Mixin Virtual Machine)。


#### 登录流程

1. 基于 mixin_token 登录:

![](./img/talkee_login(mixin_token).svg)

Mixin Messenger 的 OAuth 形式登录，此方法仅适用于 Mixin Messenger app 。它与 EIP-4361 的方法有点不同。您将被重定向到 Mixin Messenger 应用程序以完成身份验证过程，而不是签署消息。



2. 基于 mvm 登录:

![](./img/talkee_login(mvm).svg)
EIP-4361 形式登录。这种方法适用于大多数以太坊兼容钱包（例如 Metamask、TrustWallet、Coinbase 钱包等）。钱包将使用用户帐户的私钥对消息进行签名。该消息包含依赖方（例如 Pando Proto）验证身份验证所需的信息。





#### <span style="color: red;"> 参考点 <span/>


基于 EIP-4361 形式的登录校验



## <span style="color: red;"> 总结 <span/>

原有 tinode 的注册/登录 考虑合并成 login 中，并将 username:password 形式的校验变更为 EIP-4631 形式校验。