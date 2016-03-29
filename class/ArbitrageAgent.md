---
title: ArbitrageAgent
author: Takuma Torii
math: true
---

# ArbitrageAgent class

## Desciption

ArbitrageAgent クラスのエージェントは，１つの指数銘柄の市場価格と株価指数（複数の現物銘柄の平均）の間の価格差から利益をえる裁定取引を高頻度で行う．
より具体的には，上場投資信託（Exchange Traded Fund; ETF）を取引する．
ETF 裁定取引では，「安く買い高く売る」の原理のもと，１つの指数銘柄と複数の現物銘柄の集合を見比べ，安い方を買い高い方を売る（空売りを含む）．
裁定取引の原理によれば，指数銘柄の市場価格と株価指数は将来的に一致するはず（等価になるはず）なので，価格差が生じたときに「安く買い高く売る」取引を瞬時に同時に行う．

ArbitrageAgent の性質をまとめる．

  * 裁定機会があるときのみ注文をだす．
  * １つの指数銘柄とそれに紐づけられた全現物銘柄を取引する．
  * 取引の意思決定はエージェントの保有する資産（現金，株式）の数量に依存しない．


## Model

エージェント $i$ はある指数銘柄 $I$ と，それに紐づけられた現物銘柄 $s = 1,\ldots, M$ を取引する．
ある時点 $t$ における銘柄 $s$ の価格を $p^s_t$，ファンダメンタル価格を $v^s_t$ と記す．
また，ある指数計算方式（INDEX）に従い計算される株価指数を $I_t$ とし，

$$
  I_t := \text{INDEX}(p^1_t,\ldots, p^M_t)
$$

と定義する．
INDEX は Nikkei225 の場合は価格の平均値であり，TOPIX の場合は時価総額荷重平均である．

エージェント $i$ は取引の有無を決める価格差に関する閾値 $\theta^i$ をもつ．
エージェント $i$ は指数銘柄の市場価格 $p^I_t$ と株価指数 $I_t$ を比較し，

  * もし $p^I_t - I_t > +\theta^i$ ならば，各現物銘柄を $1$ 単位ずつ売り，指数銘柄を $M$ 単位だけ買う．
  * もし $p^I_t - I_t < -\theta^i$ ならば，各現物銘柄を $1$ 単位ずつ買い，指数銘柄を $M$ 単位だけ売る．

注文価格には現在の市場価格 $p^I_t$，$p^s_t$ を用いる．


## Parameters

|            | JSON keyword          | Field variable        | 
|------------|-----------------------|-----------------------|---------
| $\theta^i$ | `orderThresholdPrice` | `orderThresholdPrice` | 注文を決める閾値
| $\tau^i$   | `orderTimeLength`     | `orderTimeLength`     | 注文の保存期間


### Inherited parameters

  * [Agent](Agent) class : `markets`,  `cashAmount`,  `assetVolumes`


## Complete JSON example

```json
"My-ArbitrageAgent": {
    "class": "ArbitrageAgent",
    "numAgents": 1,

    "MEMO": "Agent class",
    "markets": ["IndexMarket-I"],
    "assetVolumes": 0,
    "cashAmount": 0,

    "MEMO": "ArbitrageAgent class",
	"orderThresholdPrice": [1, 5],
    "orderTimeLength": [10, 100]
}
```


## Parameter setup from JSON

### orderThresholdPrice

  * Class plham.agent.ArbitrageAgent
  * JSON key: `orderThresholdPrice`
  * JSON value: Double >= 0

エージェント $i$ のフィールド `orderThresholdPrice` の値 $\theta^i$ を決める．


### orderTimeLength

  * Class: `plham.agent.ArbitrageAgent`
  * JSON key: `orderTimeLength`
  * JSON value: `Double >= 0`

エージェント $i$ のフィールド `orderTimeLength` の値 $\tau^i$ を決める．
ArbitrageAgent は高頻度取引を行うので，通常，2〜5 などの小さな値を設定する．


## Fields & Methods

特筆すべきフィールドやメソッドをもたない．


## References

  * 鳥居，中川，和泉 (2015) 複数資産人工市場を用いた裁定取引によるショック伝搬の分析

