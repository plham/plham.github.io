---
title: PriceLimitMain
author: Takuma Torii
math: false
---

# PriceLimitMain

本記事では Chiarella & Iori (2002) が行った単一銘柄モデルを拡張し，値幅制限という１日の価格変動幅の上限下限を設ける金融規制をシミュレーションする．
値幅制限は，シミュレーションモデル上では，(a) エージェントが注文を出すとき価格を値幅以内に確実に収める，(b) マーケットが出された注文の価格を値幅以内に変更する，という方法がありえる．
このシミュレーションでは，(a) および (b) の両方を実装する．

[取引停止](/tutorial/TradingHaltMain)と対比させると，取引停止が（注文とは独立に）価格の変化量を条件として発動し，市場の開閉を変化させたのに対し，値幅制限は注文の価格を引き金として発動し，個別の注文価格を書き換えるのみである．

本記事で扱う内容：

  * 金融制度の追加（値幅制限）

関連するファイル：

  * `samples/PriceLimit/PriceLimitMain.x10`
  * `samples/PriceLimit/PriceLimitFCNAgent.x10`
  * `plham/event/PriceLimitRule.x10`


## Preface

[`CI2002Main`](/tutorial/CI2002Main) を十分に理解できていることを前提とする．
`PriceLimitMain.x10` は `CI2002Main.x10` を継承して作成されており，エージェントやマーケットの生成に関する部分はすべて継承したものを使う．
本記事では値幅制限制度 `PriceLimit` の使い方を説明し，その実装は説明しない．

余談だが，値幅制限が市場に及ぼす影響は，水田ら (2014) によって研究されている．


## Compile & run

```
 $ x10c++ samples/PriceLimit/PriceLimitMain.x10
 $ ./a.out samples/PriceLimit/config.json
```


## PriceLimitFCNAgent

以下に `PriceLimitFCNAgent` のコード全体を示す．
実装は非常に単純で，まず `FCNAgent` の `submitOrders()` により注文を決定し，次にフィールド `priceLimit` で定義された仕方でその注文の価格を書き換えている．

```x10
// samples/PriceLimit/PriceLimitFCNAgent.x10
public class PriceLimitFCNAgent extends FCNAgent {

	public var priceLimit:PriceLimitRule;

	public def submitOrders(market:Market):List[Order] {
		val orders = super.submitOrders(market);
		if (orders.size() == 0) {
			return orders;
		}

		for (order in orders) {
			val oldPrice = order.getPrice();
			val newPrice = priceLimit.getLimitedPrice(order, market);
			if (newPrice != oldPrice) {
				order.setPrice(newPrice); // Adjust the price.
			}
		}
		return orders;
	}
}
```


## PriceLimitMain

`PriceLimitMain` は `CI2002Main` を継承し，次のような構造をもつ．

```x10
// samples/PriceLimit/PriceLimitMain.x10
public class PriceLimitMain extends CI2002Main {

	public static def main(args:Rail[String]) {
		new SequentialRunner(new PriceLimitMain()).run(args);
	}

	public def createAgents(json:JSON.Value):List[Agent] {}

	public def createEvents(json:JSON.Value):List[Event] {}
}
```

主要なメソッドは `createAgents()` および `createEvents()` であり，スーパークラス `Main` で定義されており，シミュレーションモデルを構築する過程で呼び出される．
これらをオーバーライドし，シミュレーションで使用するエージェントや値幅制限規制を生成する．


### createAgents()

`createAgents()` では `PriceLimitFCNAgent` をインスタンス化する．

```x10
// samples/PriceLimit/PriceLimitMain.x10
public def createAgents(json:JSON.Value):List[Agent] {
	val random = new JSONRandom(getRandom());
	val agents = super.createAgents(json); // Use FCNAgent defined in CI2002Main.
	if (json("class").equals("PriceLimitFCNAgent")) {
		val numAgents = json("numAgents").toLong();
		for (i in 0..(numAgents - 1)) {
			val agent = new PriceLimitFCNAgent();
			setupPriceLimitFCNAgent(agent, json, random);
			agents.add(agent);
		}
	}
	return agents;
}
```

以下のコードは `PriceLimitFCNAgent` の初期設定を行う．
`FCNAgent` から継承した属性に関しては，`CI2002Main` で定義された `setupFCNAgent()` メソッドを再利用している．
また，残りのフィールド `priceLimit` についてはここで設定している．

```x10
// samples/PriceLimit/PriceLimitMain.x10
public def setupPriceLimitFCNAgent(agent:PriceLimitFCNAgent, json:JSON.Value, random:JSONRandom) {
	setupFCNAgent(agent, json, random);
	agent.priceLimit = createEvents(CONFIG(json("priceLimit")))(0) as PriceLimitRule;
}
```

JSON ファイルはほとんど `FCNAgent` の定義と同じなので省略する．


### createEvents()

`createEvents()` では `PriceLimitRule` をインスタンス化する．

```x10
// samples/PriceLimit/PriceLimitMain.x10
public def createEvents(json:JSON.Value):List[Event] {
	val random = new JSONRandom(getRandom());
	val events = new ArrayList[Event]();
	if (!json("enabled").toBoolean()) {
		return events;
	}
	if (json("class").equals("PriceLimitRule")) {
		val rule = new PriceLimitRule();
		setupPriceLimitRule(rule, json, random);
		events.add(rule);
	}
	return events;
}
```

以下のコードは `PriceLimitRule` の初期設定を行う．

```x10
// samples/PriceLimit/PriceLimitMain.x10
public def setupPriceLimitRule(rule:PriceLimitRule, json:JSON.Value, random:JSONRandom) {
	val referenceMarket = getMarketByName(json("referenceMarket"));
	rule.referenceMarketId = referenceMarket.id;
	rule.referencePrice = referenceMarket.getPrice();
	rule.triggerChangeRate = json("triggerChangeRate").toDouble();
	referenceMarket.addBeforeOrderHandlingEvent(rule);
}
```

上記で使用されている JSON ファイルの一部は以下である．
値幅制限の属性を定義している．

```json
// samples/PriceLimit/config.json
"PriceLimitRule": {
	"class": "PriceLimitRule",
	"referenceMarket": "Market",
	"targetMarkets": ["Market"],
	"triggerChangeRate": 0.05,
	"enabled": true
},
```

エージェントやマーケットとまったく同じ仕方で X10 と JSON が連携していることがわかるだろう．

値幅制限は対象とするマーケットの価格がある範囲を超えて変化しないよう制限する．
`PriceLimitRule` の現状の実装では `Market` のインスタンスを参照している．
上記の X10 コードには，どうやってインスタンス化されたマーケットを取得するか，が示されている．

具体的には，`GLOBAL` には，インスタンス化済みのマーケット，エージェント，イベントなどが JSON で宣言したマーケット名を `key` として `GLOBAL(key)` に保存されている．
インスタンス化は (1) マーケット，(2) エージェント，(3) イベント（金融規制／金融ショック）（順不定）の順番で行われ，先にインスタンス化されたオブジェクトにのみ，`GLOBAL` を介してアクセスできる．
マーケットのインスタンス化は相互依存関係を調べたうえで実行されるため，マーケット名が間違っていたり，循環関係が宣言されていない限り，マーケット `key` が `GLOBAL` に不在ということはない．
`GLOBAL` には `createMarkets()`，`createAgents()`，`createEvents()` の返り値がそのまま登録されているため，これらに関して言えば，key に対応する value すなわち `GLOBAL(key)` はすべて `List[T]` 型である．

実装上では，下記の補助メソッド群が提供されており，ユーザが `GLOBAL` に直接アクセスすることは少ないと思われる．

| Market                | Agent                | Event
|-----------------------|----------------------|----------------------
| `getMarketsByName()`  | `getAgentsByName()`  | `getEventsByName()`
| `getMarketByName()`   | `getAgentByName()`   | `getEventByName()`

上記の JSON ファイルでは，`"targetMarkets": ["Market"]` のうち，`"Market"` の部分がマーケットの定義名と一致していることが重要である．
上記の X10 コードでは，`getMarketsByNames(json("targetMarkets"))` という書き方で，既にインスタンス化されているマーケット `"Market"` を取得している．


### JSON configuration file

最後に，シミュレーションにこれまで定義した `PriceLimitRule` および `FundamentalPriceShock` を追加する方法を示す．
以下は JSON ファイルの `"simulation"` の部分である．
`CI2002/config.json` と比較すれば，`"MEMO"` で注釈づけられた部分が異なる．

```json
// samples/PriceLimit/config.json
"simulation": {
	"markets": ["Market"],
	"agents": ["FCNAgents"],
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
			"withPrint": true,
			"events": ["PriceLimitRule"], "MEMO": "値幅制限"
		}
	]
},
```

JSON ファイル中のコメントにあるように，`"events"` にシミュレーションに取り込む金融規制のリストを指定している．


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/PriceLimitMain_UseCases)）．


## Related works

  * 水田・和泉・八木・吉村 (2009) 人工市場を用いた値幅制限・空売り規制・アップティックルールの検証．


