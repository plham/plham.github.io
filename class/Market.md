---
title: Market
author: Takuma Torii
---

# Market class

## Description

Market クラスは，連続ダブルオークション（ザラバ方式）に従う注文の処理を行う．
このクラスはすべてのマーケットの基底クラスを担う．

本ソフトウェアにおいて，Market クラスは金融用語としての「市場」とは異なり，ある個別特定の「銘柄」に対する注文を処理するシステムを抽象化している．
したがって，モデル上で扱いたい「銘柄」の数だけ，Market クラスのインスタンスを作る必要がある．
本稿では，Market クラスのインスタンス（個別特定の銘柄に対応する）を表す言葉として「マーケット」を用いる．

Market クラスは，マーケットに注文の処理を実行させる役割を担うメソッド `handleOrders(...)` をもつ．
プログラマは `handleOrders(...)` をオーバーライドし，任意の注文処理を実装できる．
Market クラスは，マーケットの情報として，価格時系列，買い板・売り板（また気配値）をフィールドとしてもつ．


## Model

連続ダブルオークションでは，トレーダ・エージェントはいつでも注文をだすことができる．
注文には，対象銘柄，売り買いの区別，価格，単位数，保留期間といった情報を指定する．
注文は，板（オーダーブック）上で価格優先・時間優先の原則に従って順序づけされ，順序の先頭から約定の対象となる．
注文が届いたとき，もし相対する注文が板にあればただちに約定し，現金と株式の交換（取引）が実行される．
もし相対する注文がなければ，その注文は板に追加される．


### Arrow of time

シミュレーション上での時間の進み方を説明する．
マーケットは時点 $t$ を含む現在までの価格時系列を保管している．
時点 $t$ でエージェントが出した注文は時点 $t + 1$ の価格を決める．
時点 $t$ はティック時間とは区別され，ある時点 $t$ から $t + 1$ までの間に複数の注文が約定することもありうる（シミュレーションの設定に依存する）．


### Determination of next prices

時点を跨ぐときの価格更新の手続きを説明する．
ある時点 $t$ から $t + 1$ までの間に複数の注文が約定した場合，その最後の取引価格が次時点 $t + 1$ の市場価格となる．
もし時点 $t$ で約定が起きなければ，次時点 $t + 1$ の市場価格には仲値が設定される．
ただし，もし板に注文が存在せず，仲値を計算できなければ，次時点 $t + 1$ の市場価格は時点 $t$ の市場価格が設定される． 
以上の価格更新処理のあと，次の時点 $t + 1$ での注文を受け付ける．


## Parameters

| | JSON keyword | Field variable       | 
|-|--------------|----------------------|----------------------
| | `marketPrice`  | `marketPrices`         | 初期の市場価格
| | `fundamentalPrice` | `fundamentalPrices` | 初期のファンダメンタル価格
| | `tickSize`     | `tickSize`             | ティックサイズ


## Parameter setup from JSON

### marketPrice

  * Class: `plham.Market`
  * JSON key: `marketPrice`
  * JSON value: `Double >=0`

初期の市場価格を設定する．
例）

```json
{  "marketPrice": 10000.0  }
```


### fundamentalPrice

  * Class: `plham.Market`
  * JSON key: `fundamentalPrice`
  * JSON value: `Double >=0`

初期のファンダメンタル価格を設定する．
JSON ファイルのマーケットに関する記述に，もし `fundamentalPrice` が未定義ならば，`marketPrice` の値が代わりに設定される．
例）

```json
{  "fundamentalPrice": 10000.0  }
```


### tickSize

  * Class: `plham.Market`
  * JSON key: `tickSize`
  * JSON value: `Double > 0`

ティックサイズを指定する．
売り注文の場合，ティックサイズより小さな端数は切り上げる（`Math.ceil`）．
買い注文の場合，ティックサイズより小さな端数は切り下げる（`Math.floor`）．
ただし，`tickSize < 0` を指定した場合，ティックサイズなし（無限小）に設定できる．
例）

```json
{  "tickSize": 0.001  }
```


## Fields & Methods

### getTime()

マーケットの現在の時点 $t$ を返す．
マーケットは時点 $t$ を含む現在までの価格時系列を保管している．
時点 $t$ でエージェントが出した注文は時点 $t + 1$ の価格を決める．


### getPrice(t:Long), getMarketPrice(t:Long)

時点 $t$ の市場価格を返す．
マーケットは `getTime()` メソッドが返す時点を含む現在までの価格時系列を保管している．


### getFundamentalPrice(t:Long)

時点 $t$ のファンダメンタル価格を返す．
マーケットは `getTime()` メソッドが返す時点を含む現在までの価格時系列を保管している．


### getBuyOrderBook(), getSellOrderBook()

Market クラスは，買い板と売り板に対応した２つの OrderBook クラスのインスタンスをもつ．
いずれも Market クラスのコンストラクタでインスタンス化される．
各フィールドへのアクセスは以下のメソッドを呼ぶ．

  * `getBuyOrderBook()`
  * `getSellOrderBook()`

なお，買い気配値，売り気配値の取得には `getBestBuyPrice()`，`getBestSellPrice()` を使える．


### getBestBuyPrice(), getBestSellPrice()

買い気配値（best buy/bid），売り気配値（best sell/ask）を返す．
ただし，もし買い・売り注文が板に存在しなければ，`Double.NaN` を返す．


### getMidPrice()

仲値（買い気配値と売り気配値の中間の値）を返す．
ただし，もし買い・売り注文が板に存在しなければ，`Double.NaN` を返す．

