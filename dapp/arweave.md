### 1. 功能介绍

1. Arweave协议是将文件使用者和存储提供者打通。 类似打车平台。 主要提供去中心存储能力。
2. permaweb 是实现Arweave协议的基础网络。 类似platONGo 实现 PlatON协议。
3. gateways 提供内容检索服务GraphQL, 依赖于postgres数据库。 例如主网地址网关服务https://arweave.net/graphql
4. permaweb 提供一组api服务，主要是网络、节点、区块、交易查询接口，和文件上传接口（一次上传和断定上传下载) 。 https://github.com/ArweaveTeam/arweave/blob/master/http_iface_docs.md
5. AR作为Arweave网络的主币。 作为文件上传的唯一凭证。 文件的大小所使用的AR数量可参考 https://ar-fees.arweave.dev/。 permaweb 的api节点提供http接口估算AR消耗。
6. 一个重要的提案。 [https://github.com/ArweaveTeam/arweave-standards/blob/master/ans/ANS-104.md](https://github.com/ArweaveTeam/arweave-standards/blob/master/ans/ANS-104.md "Bundled Data v2.0 - Binary Serialization")。 主要可以一笔交易打包多个DataItem。 允许将 DataItem 的付款委托给第 3 方，同时保持创建 DataItem 的人的身份和签名，而不需要他们有一个有资金的钱包
7. Arweave的上层服务网， 依赖6的提案实现的。 目前是主流  https://docs.bundlr.network/docs/client/cli


### 2. 钱包的类型

> 采用RAS非对称秘钥。 格式为JWK格式。


### 3. 交易类型
> 主要有二类型交易。一种是AR转移， 一种是文件存储。

#### 3.1 AR转移交易

```
arweave.createTransaction({
  target: 'TRN0U3rzBuFaFlCH23hVm6LPFc9J5S7jYZuyGQmkkVU',
  quantity: arweave.ar.arToWinston('1')
},jwk)

``` 

#### 3.2 文件存储交易

```
arweave.createTransaction({
  data: '<html><head><meta charset="UTF-8"><title>Hello world!</title></head><body></body></html>'
}, jwk)

```

### 4. 文件上传
1. 单个大文件。 使用arweave-js 中断点上传。 http接口本来是支持的。
2. 大批量小文件。 使用 ArBundles。 类似第三服务，只要使用的功能介绍中6的协议。 目前主流。
3. 单个小文件。 使用arweave-js post。 


### 5. sdk
- js https://github.com/ArweaveTeam/arweave-js



### 6. 测试用的组件
1. 钱包。 安装插件钱包。 ArConnect.
2. 可测试的测试网关。 （在ArConnect中发现的 ）

```

const arweave = Arweave.init({
    host: 'testnet.redstone.tools',
    port: 443,
    protocol: 'https'
});


、、、

3. 有钱的钱包(水龙头未申请到， 下面钱包是测试组件内置的钱包， 意外发现在测试网有钱)

```

cat arweave-keyfile-MlV6DeOtRmakDOf6vgOBlif795tcWimgyPsYYNQ8q1Y.json

{
	"kty": "RSA",
	"e": "AQAB",
	"n": "zhTx5Kr9VNQrXGarf0EXySfbSePBbIQuSOpb07s3pM3q8HKCx-bbd_py8t-JxgwnKAmpGKt6UhOP0FeobGITCwr_O7ATFPrFgTbM-xLYG0JOzxUlPScyqdJ8rFRcSSpevfUyJ6UVTpA3LDQHEzf7kebjfMPeYwpsWuT3c9LP3j0kyPDOBini-LRUpKX3n4ljhJIHzl-Jdv6Z31U65kZRBR1LPwnjcBUg4hoc50i8JZsSLsrUYFfpYVuxM0L4ch0l2-FvPtmZs831mOQgT8e1s7GPB7kJBhrQBagGF3eVnAiImJjslXNQhy4eQr6Nffb5Wa61Tec52LX5-gmoNSuA0PW5yuYGuDO2faULW74u8ZfmMUxd2x3E3M6E0deP_rj27FUQCECdbO6ATVanA16wnW7MrySu2m-Kt83XyATdVoNDls-coxA4UxuX7Rmlr2eGM7ZRKtypt12GziKnZgNglK5c_4mmMP2xeeLU1fneBLkvuHSEnoFjqZnAaI0ei6pW8Jy3k8txI5MucaRkXdPOhCm3Nwj8B9rBAh0hU64NVVb7C28Gz8LCwZkRhtGRY_v2vzcS0DaomK2G63vyQMKx3VUc9_RnkxcI6bwy6xG2GBEjpV8tHxXgw8zGc53_8EMo-9EM1PpjOHHYyaYoubDbxHaSJPwCPqi_OlGbl2h8gIM",
	"d": "VTCEVB45Dd2NNSu9_iNW5VEkDdXoKec0SPEUV6DfXjG_SnlTxbYRiHXQGcU9a1CvyRXBQJD2RkKO4zWxSmh6bci0fKSLJtOJXKJeNvXxvsb41BLuK2ruPxRjdEuFQLuSoZzgCFJuTeVA4XV6bT_pr0UOSg-f-TogU6yt_EOrqTeGYshkqlibWmsVSGDRTbJaIL3LG00UAsw5qIBPkkyEBoS3C86XJcieKMlZpGRFXphNemlfRJpiv9vLEyE-mdGhylTVC1qhdpoPyg2Xq9MnMiqWsT8U02C3GHd-WSoWfwNqEAa7WgZqxg7S9I1X6Tf0mNWnXhZVK9gCB5IBZkVfAIR727N9eFu8QQfBUld3QNtT-JwIjXWvEsk6mc7vx3puiZiT75Adc40gV0-ynAjy2Pb1O51g9mqD19mqvwxUWeW0YBKO03dYUfw9P7lxNUaYZGoc7rUec94xznOZKxgyKXwGGPe1tROMVBZf3Ewv0yrsB6YmDXQVv1p9MHiJ6pd4iXfeyvZjzCAqwUUukMKC6C38KlVLhS_PvX1V4ebDmmfbhlQTwLyJIcL1fxrHDyOVA8ued-TI4ioXO1OXaPwjM6rwhvRYcah6UKRxnF23iJjDZ7sBedrowSYPAjDyeoU4go3MaJIcuuKl3wIrvBe-K8TczkrG4AFx7BktkRA80Lk",
	"p": "6lC25S1c8MQWSbw1a932A2MqjPMkB79H8VQ1eYxTCI7ihMGkubOwr-ZZTcGE_-S0O5GmHPklsAmdWqnNSEotuiCCLX-cvZwbiTi0EfD4P6A9BU8n2ElIuBwWxLZKoqM977Q-2qIUFnvpfPit8Shqu3x-OykEYHDnbSLbOphcbAI1-kGsBui-wmezbvXv-ULwLi3FKey_FJfP7nYf2RazNXk1b7lqe8l3UbotsC86FudHmJe6c7O0epcuUbADeu5cpKPTzf0CMJVsW152KWOc--tXTfA03gANl30qUGYnNm40qYntGxhpUtfUZYGTg9nHKKS-gteCobMVIxHNZk8q6Q",
	"q": "4SdWJW-MMFlrUhVLYmOTgzFNpZ9-xQ3GQ0-uJpjfqfFcwVaEyeLMQE2lDknCd58RwRpYHJBjKWIO8yhNkeIVcmSl1Fyg91g61gOBtXQoz6JQm5K9y2snGlj4BC4nQdP39dCdg7xjiRMa9nwEFnxL8hNN27xDq2n81F3VsFqG3VmXY5YMnALUjkNCVOZM_0h8L2RrOCLroKu_NI6WV1LIaAKe7y99ZiOsQl5MMjwp12n4ea_U7l_Oyp2NlEsp5idbYa1TrC8LOWOn80jgM_iE3DrxAB5JKuI8vx_A7Jrral-aLT4oTtJDKV9B5Y7PPuhDYZrqHOkI0fwUrDIRRzYUiw",
	"dp": "CnUxxIbCyCgoSoAw7jCI41vQsVvEtufNoTK99D_UEOS3rW8rF_KyJxej0rmZYwZlGOeGP3LLQNEdCcfcVqag5da_mKJCb6ABBp3WQ5q6qbRQJOWEhL24licCySLNr_aTNBiaWY20UdCT-jTrJoFESjvjMmbBQECpw5AzsqjMLzHmENZPhDttECYqtwAZBsn7CESYsSdU2-luqVjyUPEXbIKNZQAkhYPXZHlnwp5I_G60HlZfRvy1SGdo9NJjRWBQGDULpfzt1RdGL8nGglBk2EWHrv3Sjjn4YVN_yPjWNTKz_QEf6P6s7LqfSyx-VfspTWIU8qgFt4vTnK4VucQ8yQ",
	"dq": "BhZ2MdTuSXBhgnqo6yQeHPH8U3oYh2Nz9OX2o3yGr6WjCGc6d-r18tcmm1hLNcjLRhlcQIl25OuN0-1HC6a9RbaK9U772zQ7gwXdP_bAE70jyNES6KkhCYlWS2akEReWIMNfPuydFFu74uY_hgweUZFMDaDtg3j-KQ_Qc1A_TUTa3wpzlNROwvn2lS0U7-IZ2X4xl_b5wAJkzRr93aaTXJyVh4oVLenRAopiLQmLaBOpcEDc1QUqJjhUV6ogm-R8iAuTs5giCY80P1O9HCqgDQRa99HZ0JsFYXWOVddqfhnPpWGE3Xy57ChzM63E1MKa78ysf9OdNXBHbtB7vx0rOQ",
	"qi": "QTbN_oNqyg9Zi9k2zlR4qtBZwDlDwCU6E_QrCKDY1z3pIvLFmc9eVO8aLek_GEKvvjri8voEctK-lrPRipwpdXj7cxw93cfS86vK6FSkb9plkInBfoKnC7DEOTP2Gx7WpHNQPbR8C8yFAidiRKc7lqgvRSh0LvWBzZ-spUANKAfNBaR6RAy2EAavAojeRMGxANqjqqnYP3Vwl35ZwNtmzkK_qIvsjKe3xFWMiCfzeFatunV2xUhJNrenoBoNp4z_66Xu-jUPLwRcWDdI7fz3MD3kZBH3gH4t52Amod79WxRriRW0SYcUeuvKbAi9FJv0RiCnvt0vzkjpF0XtukyHIg"
}


```



