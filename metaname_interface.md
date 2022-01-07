# metaname service 接口

## 接口地址

- main: not ready

## 接口介绍

## 1. 请求操作

在进行操作时先去请求必要的参数

### Request

- Method: **POST**

- URL: ```/reqargs```

- Body:
```
{
    nameHex: "746573746e616d65",
    address: "msREe5jsynP65899v1KJCydf6Sc9pJPb8S",
    op: 1,
    source: 'ccoincash'
}
```

> * nameHex: 需要注册或者更新信息的名字。该名字由utf-8编码然后转换为hex。
> * address: 用户的地址，用于接收nft。
> * op: operation: 1 注册, 2 续费, 3 更新信息。
> * source: 标记调用者的身份，方便查找错误。

### Response
```
{
    code: 0,
    msg: "",
    data: {
        requestIndex: 1,
        nftToAddress: '17r31WUcjV3fM7ZbM96CJmcHC5R9PegUNf',
        bsvToAddress: '1MApSB9rsEgZdR8EqbtgNPX4uvhLv8kcio',
        txFee: 117018,
        feePerYear: 5000000,
        op: 1,
        nftCodeHash: '0cf6d6e178584b7b34941604eab291958ba5db29',
        nftGenesisID: 'ea1b3cf61543c47b34f631651a6e687fc806fb4b',
        nftTokenIndex: '0'
    }
}
```

code为0时，表示正常返回data。code为1时，表示由错误。错误信息在msg中。**注意每次进行swap操作是都需要申请新的requestIndex，requestIndex不能重复使用。**

data格式如下：

> * requestIndex: 请求编号。
> * nftToAddress: 需要转入nft到此地址。register操作不需要。
> * bsvToAddress: 需要转入的矿工费以及bsv到如下地址。
> * txFee: 此操作需要到矿工费。
> * feePerYear: 维护此名字每年所需要的satoshis。
> * op: 操作类型。
> * nftCodeHash: 需要的nft的codeHash。register操作不需要。
> * nftGenesisID: 需要的nft的genesisID。register操作不需要。
> * nftTokenIndex: 需要的nft的tokenIndex。register操作不需要。

## 2. 注册

注册一个新的名字。此名字必须为没有注册的。

### Request
- Methos: **POST**
- URL: ```/register```
- Body: 
```
{
    data: compressedData
}
```

compressData是如下格式
```
data = {
    requestIndex: "1",
    bsvRawTx: "",
    bsvOutputIndex: 0,
    years: 10,
    infos: {
        "metaid": "",
        "bsv": "1MApSB9rsEgZdR8EqbtgNPX4uvhLv8kcio",
    }
}
compressData = gzip(JSON.stringify(data))
```

> * requestIndex: 之前通过reqargs获取的编号。
> * bsvRawTx: bsv转账raw tx。
> * bsvOutputIndex: bsv转账tx的outputIndex。
> * years: 名字注册的年数
> * infos: 需要名字对应的信息，目前支持两个metaid和bsv地址

**注意：这里转账的bsv数量为txFee + 注册费。注册费的计算方法为years * feePerYear.**

**注意2：rawTx不要广播到bsv网络上，直接发给后端。同时，在发送前必须对data进行gzip压缩, 然后设置header {'Content-Type': 'application/json'}。**

### Response

```
{
    "code": 0,
    "msg": "",
    "data": {
        txid: '1649c55319187fc7047f0bb372e89b5d2e2c716ce7e387470e3c0460d19065a6',
        expiredBlockHeight: 721187
    }
}
```
code为0时，表示正常返回data, txid表示此次操作的交易id，expiredBlockHeight是此名字的到期区块高度。code不为0时，表示执行错误。错误信息在msg中。

## 3. 续费

给名字续费，以年为单位

### Request
- Methos: **POST**
- URL: ```/renew```
- Body: 
```
{
    data: compressedData
}
```
compressData是如下格式
```
data = {
    requestIndex: "1",
    bsvRawTx: "",
    bsvOutputIndex: 0,
    nftRawTx: "",
    nftOutputIndex: 0,
    years: 10
}
compressData = gzip(JSON.stringify(data))
```

> * requestIndex: 之前通过reqargs获取的编号。
> * bsvRawTx: bsv转账raw tx。
> * bsvOutputIndex: bsv转账tx的outputIndex。
> * nftRawTx: nft转账raw tx。
> * nftOutputIndex: nft转账tx的outputIndex。
> * years: 名字注册的年数

**注意：这里转账的bsv数量为txFee + 注册费。注册费的计算方法为years * feePerYear.**

**注意2：rawTx不要广播到bsv网络上，直接发给后端。同时，在发送前必须对data进行gzip压缩, 然后设置header {'Content-Type': 'application/json'}。**

### Response

```
{
    "code": 0,
    "msg": "",
    "data": {
        txid: '1649c55319187fc7047f0bb372e89b5d2e2c716ce7e387470e3c0460d19065a6',
        expiredBlockHeight: 721187
    }
}
```
code为0时，表示正常返回data, txid表示此次操作的交易id，expiredBlockHeight是此名字的到期区块高度。code不为0时，表示执行错误。错误信息在msg中。

## 4. 更新信息

更新名字对应信息。

### Request
- Methos: **POST**
- URL: ```/updateinfo```
- Body: 
```
{
    data: compressedData
}
```

compressData是如下格式
```
data = {
    requestIndex: "1",
    bsvRawTx: "",
    bsvOutputIndex: 0,
    nftRawTx: "",
    nftOutputIndex: 0,
    infos: {
        "metaid": "",
        "bsv": "1MApSB9rsEgZdR8EqbtgNPX4uvhLv8kcio",
    }
}
compressData = gzip(JSON.stringify(data))
```

> * requestIndex: 之前通过reqargs获取的编号。
> * bsvRawTx: bsv转账raw tx。
> * bsvOutputIndex: bsv转账tx的outputIndex。
> * nftRawTx: nft转账raw tx。
> * nftOutputIndex: nft转账tx的outputIndex。
> * infos: 需要名字对应的信息，目前支持两个metaid和bsv地址

**注意：这里转账的bsv数量为txFee**

**注意2：rawTx不要广播到bsv网络上，直接发给后端。同时，在发送前必须对data进行gzip压缩, 然后设置header {'Content-Type': 'application/json'}。**

### Response

```
{
    "code": 0,
    "msg": "",
    "data": {
        txid: '1649c55319187fc7047f0bb372e89b5d2e2c716ce7e387470e3c0460d19065a6',
        expiredBlockHeight: 721187
    }
}
```
code为0时，表示正常返回data, txid表示此次操作的交易id，expiredBlockHeight是此名字的到期区块高度。code不为0时，表示执行错误。错误信息在msg中。