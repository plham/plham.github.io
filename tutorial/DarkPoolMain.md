---
title: DarkPoolMain
author: Takuma Torii
math: true
---

# DarkPoolMain

本記事では，水田ら (2014) が行ったダークプール市場の影響分析のシミュレーションを作成することを通して，本ソフトウェアの使い方を説明する．
ダークプールは通常の連続二重オークションとは異なる取引メカニズムをもつ．
本記事の主要なテーマは，`Market` クラスを拡張し，別な取引メカニズムを実装する方法にある．

本記事で扱う内容：

  * ダークプール市場の取引メカニズム

関連するファイル：

  * `samples/DarkPool/DarkPoolMain.x10`
  * `samples/DarkPool/DarkPoolMarket.x10`
  * `samples/DarkPool/DarkPoolFCNAgent.x10`


## Preface

水田ら (2014) は通常の市場（リット市場と呼ばれる）に加えて，ダークプール市場が存在することが，市場の価格発見機能に及ぼす影響を分析している．
ダークプールでは株式を取引所内で突き合わせるため，株価や板情報が公開されない．
そのため，大規模な取引や価格の変化が市場の他のトレーダに与える心理的影響を減らす効果をもつ．
反面，取引情報が公開されないため，市場の公平性を損なうことが危惧される．

ダークプールはリット市場の特定の銘柄と紐づけられており，ダークプールでの取引価格はリット市場での仲値が採用される．
必ず仲値で取引されるという性質から，ダークプールでは相対する注文が存在すれば必ず約定が成立する．
また，この性質から，ダークプールでは売り板か買い板か，いずれか一方にしか注文が残らない．


## Compile & run

```
 $ x10c++ samples/DarkPool/DarkPoolMain.x10
 $ ./a.out samples/DarkPool/config.json
```


## DarkPoolMarket

先述の通り，ダークプールでの取引価格はリット市場での仲値が採用される．
換言すれば，エージェントは注文価格を決めることができない．
この点すなわち価格優先の原則が不在という点において，約定時に板にある注文の価格が採用される通常の連続二重オークションとは異なるが，時間優先の原則は共通している．
以上から，「仲値の使用」および「価格優先の原則の不在」という点で異なるが，大部分においては，連続二重オークションを実装した `Market` の実装を使い回せる．

ダークプールの取引メカニズム（価格優先の原則の不在）を実装するにあたって，注文の取引価格に関して仕様を定めてしまうと，のちの実装を進めやすい．
本実装では以下の仕様を定めた．

> ダークプールへの注文は注文価格が `Order.NO_PRICE` でなければならない．
> （さもなければ，プログラムは実行時エラーをだす．）

この定義に従えば，「価格優先の原則の不在」は「すべての注文が同じ値段（= `NO_PRICE`）である」ことにより実現できる．
また，時間優先の原則は相変わらず機能したままである．

この仕様の下，ダークプールの取引メカニズムを実装するには，少なくとも下記のメソッドだけをオーバーライドし，「ダークプールでの取引価格はリット市場での仲値が採用される」ようにすればよい．

```x10
// samples/DarkPool/DarkPoolMarket.x10
public class DarkPoolMarket extends Market {

	protected def executeBuyOrders(buyOrder:Order, sellOrder:Order) {
		assert buyOrder.getPrice() == Order.NO_PRICE : "The price must be Order.NO_PRICE"; // Check it now (easy impl)
		assert sellOrder.getPrice() == Order.NO_PRICE : "The price must be Order.NO_PRICE";
		executeOrders(roundSellPrice(getLitMidPrice()), buyOrder, sellOrder, true); // Always use the mid price.
	}
	
	protected def executeSellOrders(sellOrder:Order, buyOrder:Order) {
		assert buyOrder.getPrice() == Order.NO_PRICE : "The price must be Order.NO_PRICE"; // Check it now (easy impl)
		assert sellOrder.getPrice() == Order.NO_PRICE : "The price must be Order.NO_PRICE";
		executeOrders(roundBuyPrice(getLitMidPrice()), buyOrder, sellOrder, false); // Always use the mid price.
	}

	public def getLitMidPrice():Double {
		val lit = getLitMarket();
		var litPrice:Double = lit.getMidPrice();
		if (litPrice.isNaN()) { // If the lit's orderbooks are empty.
			litPrice = getLitMarket().getPrice();
		}
		return litPrice;
	}
}
```


## DarkPoolFCNAgent

`DarkPoolFCNAgent` は確率 $d$ でダークプールに注文を出し，確率 $1 - d$ でリット市場に注文をだす．
この確率 $d$ はフィールド `darkPoolChance` とした．
他に特筆すべき点はない．
X10 プログラムを参照してほしい．


## DarkPoolMain

### JSON configuration file

以下は JSON ファイルの `"simulation"` の部分である．

```json
// samples/DarkPool/config.json
"simulation": {
	"markets": ["LitMarket", "DarkPoolMarket"],
	"agents": ["DarkPoolFCNAgents"],
	"sessions": [
		{	"sessionName": 0,
			"iterationSteps": 100,
			"withOrderPlacement": true,
			"withOrderExecution": false,
			"withPrint": true
		},
		{	"sessionName": 1,
			"iterationSteps": 500,
			"withOrderPlacement": true,
			"withOrderExecution": true,
			"withPrint": true
		}
	]
},
```


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/DarkPoolMain_UseCases)）．


## Related works

  * 水田・小杉・楠本・松本・和泉 (2014) 人工市場シミュレーションを用いたダークプールによる市場効率化の分析
  * 西岡・鳥居・和泉 (2016) マーケットメーカーがダークプールの存在する市場の効率性に与える影響：人工市場アプローチによる分析


