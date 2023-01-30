### 1. 功能介绍

1. 主要实现链上获取链下数据的中间件。 提供可扩展、 分布式的基础设施。
2. 主要的功能。 
	1. DATA FEEDS： 将链下数据定期同步到链上合约。 使用者直接通过call调用获取数据。
	2. CONNECT TO ANY API： 链上合约异步调用链下api。
	3. USING RANDOMNESS： 提供随机数功能。
3. 面向的角色。
	1. 合约开发者： 使用chainlink功能的人，通过引入chainlink入口合约。
	2. 节点运行者： 部署DataOracle合约, 并维护chainlink节点的人。 节点上面可以设置job。
	3. 数据提供者： 提供特定领域api接口，供chainlink节点调用。
4. 支持的链。 官方运营的ETH链。 主要是支持evm合约。 因为PlatON节点接口和ETH兼容，所以也可以使用。

### 2. 官网对DataOracle的功能依赖

1. 官网主要暴露一些统计或者计算数据到合约。
2. 并且需要收取手续费。
3. 一般统计、隐私计算的接口为异步的

### 3. Chainlink在PlatON搭建及设置

1. 部署LinkToken代币合约

	
```
LinkToken.sol

pragma solidity ^0.4.11;
import "@chainlink/contracts/src/v0.4/LinkToken.sol";
	
```


2. 部署Oracle合约

```
Oracle.sol

// SPDX-License-Identifier: MIT
pragma solidity 0.6.6;
import "@chainlink/contracts/src/v0.6/Oracle.sol";
	
```

3. 部署chainlink节点

```
1. 创建工作目录
mkdir chainlink-platon

2. 配置管理台账户及密码
cat chainlink-platon/.api 
chenxiaodai@126.com
admin123!admin123!

3. 配置节点的钱包密码
cat chainlink-platon/.password 
admin123!dmin123!

4. docker-stack脚本及配置

version: "3"

services:

  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: admin123!admin123!
    volumes:
      - dev_postgres_data:/var/lib/postgresql/data
    ports:
      - "18000:5432"   
    networks:
      - frontend
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
        
  chainlink:
    image: smartcontract/chainlink:1.8.0-root
    command: ["local", "n", "-p", "/chainlink/.password", "-a", "/chainlink/.api"]
    depends_on:
      - postgres
    environment:
      ROOT: /chainlink
      LOG_LEVEL: debug
      ETH_CHAIN_ID: 2203181
      CHAINLINK_TLS_PORT: 0
      SECURE_COOKIES: "false"
      ALLOW_ORIGINS: "*"
      ETH_URL: ws://8.219.126.197:6790
      DATABASE_URL: postgresql://postgres:admin123!admin123!@192.168.120.144:18000/postgres?sslmode=disable
      LINK_CONTRACT_ADDRESS: "0xe7b6c68fFB952C3EaF7433DC9c36536b431d73ee"
      BRIDGE_RESPONSE_URL: http://192.168.120.144:6688
    ports:
      - "6688:6688"
    volumes:
      - /opt/docker-app/chainlink/chainlink-platon:/chainlink
    networks:
      - frontend
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
        
volumes:
  dev_postgres_data:
  
networks:
  frontend:

5. 调用oracle合约中设置节点地址。

6. 对节点地址充LAT

```

4. 登录界面 http://192.168.120.144:6688/

5. 设置外部适配器支持异步接口。

```
1. 节点管理台设置外部Bridges服务

name: bridge_1
URL: http://192.168.10.161:8080/chainLink/async


2. 节点管理设置job

type = "directrequest"
schemaVersion = 1
name = "task-1-V14"
maxTaskDuration = "180s"
contractAddress = "0x79D7857D6659A0Ebf95b1f60C54630c9adfFbeBF"
minContractPaymentLinkJuels = "1"
observationSource = """
    decode_log   [type="ethabidecodelog"
                  abi="OracleRequest(bytes32 indexed specId, address requester, bytes32 requestId, uint256 payment, address callbackAddr, bytes4 callbackFunctionId, uint256 cancelExpiration, uint256 dataVersion, bytes data)"
                  data="$(jobRun.logData)"
                  topics="$(jobRun.logTopics)"]

    decode_cbor  [type="cborparse" data="$(decode_log.data)"]
    fetch        [type="bridge" name="bridge_1" requestData="{\\"data\\":{\\"param1\\": $(decode_cbor.param1), \\"param2\\": $(decode_cbor.param2), \\"param3\\": $(decode_cbor.param3)}}"  async="true"]
    parse        [type="jsonparse" path="result1" data="$(fetch)"]
    encode_data  [type="ethabiencode" abi="(uint256 value)" data="{ \\"value\\": $(parse) }"]
    encode_tx    [type="ethabiencode"
                  abi="fulfillOracleRequest(bytes32 requestId, uint256 payment, address callbackAddress, bytes4 callbackFunctionId, uint256 expiration, bytes32 data)"
                  data="{\\"requestId\\": $(decode_log.requestId), \\"payment\\": $(decode_log.payment), \\"callbackAddress\\": $(decode_log.callbackAddr), \\"callbackFunctionId\\": $(decode_log.callbackFunctionId), \\"expiration\\": $(decode_log.cancelExpiration), \\"data\\": $(encode_data)}"
                 ]
    submit_tx    [type="ethtx" to="0x79D7857D6659A0Ebf95b1f60C54630c9adfFbeBF" data="$(encode_tx)"]

    decode_log -> decode_cbor -> fetch -> parse -> encode_data -> encode_tx -> submit_tx
"""

```


### 4. Chainlink在异步调用过程

1. 用户发起合约调用

```

AsyncAPIConsumer.sol

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

import '@chainlink/contracts/src/v0.8/ChainlinkClient.sol';
import '@chainlink/contracts/src/v0.8/ConfirmedOwner.sol';

/**
 * Request testnet LINK and ETH here: https://faucets.chain.link/
 * Find information on LINK Token Contracts and get the latest ETH and LINK faucets here: https://docs.chain.link/docs/link-token-contracts/
 */

/**
 * THIS IS AN EXAMPLE CONTRACT WHICH USES HARDCODED VALUES FOR CLARITY.
 * THIS EXAMPLE USES UN-AUDITED CODE.
 * DO NOT USE THIS CODE IN PRODUCTION.
 */

contract AsyncAPIConsumer is ChainlinkClient, ConfirmedOwner {
    using Chainlink for Chainlink.Request;

    uint256 public volume;
    uint256 private fee;

    event RequestVolume(bytes32 indexed requestId, uint256 volume);

    constructor() ConfirmedOwner(msg.sender) {
        setChainlinkToken(0xe7b6c68fFB952C3EaF7433DC9c36536b431d73ee);
        setChainlinkOracle(	0x79D7857D6659A0Ebf95b1f60C54630c9adfFbeBF);
        fee = (1 * LINK_DIVISIBILITY) / 10; // 0,1 * 10**18 (Varies by network and job)
    }

    function requestVolumeData(bytes32 _specId) public returns (bytes32 requestId) {

        Chainlink.Request memory req = buildChainlinkRequest(_specId, address(this), this.fulfill.selector);

        req.add('param1', 'param1');
        req.add('param2', 'param2');
        req.add('param3', 'param3');
        return sendChainlinkRequest(req, fee);
    }

    /**
     * Receive the response in the form of uint256
     */
    function fulfill(bytes32 _requestId, uint256 _volume) public recordChainlinkFulfillment(_requestId) {
        emit RequestVolume(_requestId, _volume);
        volume = _volume;
    }

    /**
     * Allow withdraw of Link tokens from the contract
     */
    function withdrawLink() public onlyOwner {
        LinkTokenInterface link = LinkTokenInterface(chainlinkTokenAddress());
        require(link.transfer(msg.sender, link.balanceOf(address(this))), 'Unable to transfer');
    }
}


```

2. 外部适配服务会收到调用的参数同步响应pedding中。

```
request（POST）:
{
	"data": {
		"param1": "param1",
		"param2": "param2",
		"param3": "param3"
	},
	"meta": {
		"oracleRequest": {
			"callbackAddr": "0x3c7624Eb4E842f23C952BF3D4AE28a1792a46096",
			"callbackFunctionId": "0x4357855e",
			"cancelExpiration": "1663311982937",
			"data": "0x66706172616d3166706172616d3166706172616d3266706172616d3266706172616d3366706172616d33",
			"dataVersion": "1",
			"payment": "100000000000000000",
			"requestId": "0x0ca0fc14e7a92b4dc6154b36394239f74512aae7dfbf0c6af26debf4d130ef3f",
			"requester": "0x3c7624Eb4E842f23C952BF3D4AE28a1792a46096",
			"specId": "0x7f04c1b77529477ea9f6d8ba91dca28600000000000000000000000000000000"
		}
	},
	"responseURL": "http://192.168.120.144:6688/v2/resume/420927a0-9a2b-47e4-a439-fa40646c46a1"
}



response:

{
	"pending":true
}


```


3. 外部适配处理完成后响应结果。

```

requst(PATCH):

{
    "value": {
        "result1": 12222
    }
}


response:
1

```

4. 合约中会收到 12222 的值。

### 5. 节点获取 link收益。

调用 orcale合约提取link
