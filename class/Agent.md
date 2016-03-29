---
title: Agent
author: Takuma Torii
---

# Agent class

## Description

Agent クラスはすべてのトレーダ・エージェントの基底クラスを担う．
このクラスは特別な取引戦略をもたない．

Agent クラスは，エージェントに注文の意思決定を実行させる役割を担うメソッド `submitOrders(...)` をもつ．
プログラマは `submitOrders(...)` をオーバーライドし，任意の取引戦略を実装できる．
Agent クラスは，トレーダに共通する情報として，現金の総額と資産の数量をフィールドとしてもつ．


## Model

なし．


## Parameters

| | JSON keyword | Field variable  |
|-|--------------|-----------------|----
| | `markets`      | --              | 取引銘柄
| | `cashAmount`   | `cashAmount`      | 現金の総額
| | `assetVolumes` | `assetVolumes(s)` | 資産の数量（銘柄 s）


## Complete JSON example

```json
"My-Agent": {
    "class": "Agent",
    "numAgents": 100,

    "MEMO": "Agent class",
    "markets": ["Market-A", "Market-B"],
    "assetVolumes": 0,
    "cashAmount": 0
}
```


## Parameter setup from JSON

### markets

  * Class: `plham.Agent`
  * JSON key: `markets`
  * JSON value: `List[String]`

エージェントが取引対象とする銘柄（マーケット）のリストを指定する．

例）

```json
{  "markets": ["Market-A", "Market-B"]  }
```


### cashAmount

  * Class: `plham.Agent`
  * JSON key: `cashAmount`
  * JSON value: `Double >= 0`

エージェントの初期現金の総額を指定する．

例）

```json
{  "cashAmount": 10000  }
```


### assetVolumes

  * Class: `plham.Agent`
  * JSON key: `assetVolumes`
  * JSON value: `Map[String,Long] >= 0  or  Long >= 0`

エージェントの初期の資産の数量を指定する．

例）辞書型を用いて個別銘柄ごとに数量を指定できる．

```json
{  "assetVolumes": { "Market-A": 100, "Market-B": 200 }  }
```

例）単一の整数を指定した場合，すべての銘柄（`markets`）にその値が設定される．

```json
{  "assetVolumes": 100  }
```


## Fields & Methods

### submitOrders(List[Market]) → List[Order]

エージェントによる注文の意思決定を実装するメソッドである．

註）Agent クラスの `submitOrders(...)` は抽象メソッドではない．
テストを容易にする意図で，現在の市場価格の周りでランダムに買い／売り注文をだすよう実装されている．


### isMarketAccessible(Market) → Boolean

エージェントの取引対象とする銘柄（マーケット）かどうかを返す．

註）メソッド `getAssetVolume(m)` など資産情報にアクセスする場合，`isMarketAccessible(m) == true` でなければアサーションエラーとなる．
 

### setMarketAccessible(Market), setMarketsAccessible(List[Market])

ある銘柄（マーケット）を取引対象に指定する．

註）メソッド `getAssetVolume(m)` など資産情報にアクセスする場合，`setMarketAccessible(m)` を実行した後なければアサーションエラーとなる．


## References

なし．

