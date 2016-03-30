---
title: TradingHaltMain
author: Takuma Torii
math: false
---

# TradingHaltMain

本記事では Chiarella & Iori (2002) が行った単一銘柄モデルを拡張し，理論価格（企業価値）の急落という金融ショックをシミュレーションする．
また，金融ショックを事後的に緩和するとされる金融制度「取引停止」を扱えるモデルを構築する．

本記事で扱う内容：

  * 金融制度の追加（取引停止）
  * 金融ショックの追加（理論価格ショック）
  * JSON ファイルで宣言した他のオブジェクトへのアクセス

関連するファイル：

  * `samples/TradingHalt/TradingHaltMain.x10`
  * `plham/event/TradingHalt.x10`
  * `plham/event/FundamentalPriceShock.x10`


## Preface

[`CI2002Main`](/tutorial/CI2002Main) を十分に理解できていることを前提とする．
`TradingHaltMain.x10` は `CI2002Main.x10` を継承して作成されており，エージェントやマーケットの生成に関する部分はすべて継承したものを使う．
本記事では実装済みの理論価格ショック `FundamentalPriceShock` と取引停止制度 `TradingHalt` の使い方を説明し，それらの実装は説明しない．


## Compile & run

```
 $ x10c++ samples/TradingHalt/TradingHaltMain.x10
 $ ./a.out samples/TradingHalt/config.json
```


## TradingHaltMain

`TradingHaltMain` は `CI2002Main` を継承し，次のような構造をもつ．

```x10
// samples/TradingHalt/TradingHaltMain.x10
public class TradingHaltMain extends CI2002Main {

    public static def main(args:Rail[String]) {
        new SequentialRunner(new TradingHaltMain()).run(args);
    }

    public def createEvents(json:JSON.Value):List[Event] {}
}
```

主要なメソッドは `createEvents()` であり，スーパークラス `Main` で定義されており，シミュレーションモデルを構築する過程で呼び出される．
ユーザに課された責任は `createEvents()` をオーバーライドすることで，JSON と連携し，金融ショックや金融規制をインスタンス化することである．


### createEvents()

`createEvents()` では `FundamentalPriceShock` および `TradingHaltRule` をインスタンス化する．
後続の節でそれぞれの内部を見ていく．

```x10
// samples/TradingHalt/TradingHaltMain.x10
public def createEvents(json:JSON.Value):List[Event] {
	val random = new JSONRandom(getRandom());
	val events = new ArrayList[Event]();
	if (!json("enabled").toBoolean()) {
		return events;
	}
	if (json("class").equals("FundamentalPriceShock")) {
		val shock = new FundamentalPriceShock();
		setupFundamentalPriceShock(shock, json, random);
		events.add(shock);
	}
	if (json("class").equals("TradingHaltRule")) {
		val rule = new TradingHaltRule();
		setupTradingHaltRule(rule, json, random);
		events.add(rule);
	}
	return events;
}
```


### createEvents(): TradingHaltRule

以下のコードは `TradingHaltRule` クラスを初期設定する．

```x10
// samples/TradingHalt/TradingHaltMain.x10
public def setupTradingHaltRule(rule:TradingHaltRule, json:JSON.Value, random:JSONRandom) {
	val market = getMarketByName(json("referenceMarket"));
	rule.referenceMarketId = market.id;
	rule.referencePrice = market.getPrice();
	rule.triggerChangeRate = json("triggerChangeRate").toDouble();
	rule.haltingTimeLength = json("haltingTimeLength").toLong();
	val targetMarkets = json("targetMarkets");
	val targetMarkets = getMarketsByNames(json("targetMarkets"));
	rule.addTargetMarkets(targetMarkets);
	market.addAfterOrderHandlingEvent(rule);
}
```

上記で使用されている JSON ファイルの一部は以下である．
取引停止の属性を定義している．

```json
// samples/TradingHalt/config.json
"TradingHaltRule": {
	"class": "TradingHaltRule",
	"referenceMarket": "Market",
	"targetMarkets": ["Market"],
	"triggerChangeRate": 0.05,
	"haltingTimeLength": 100,
	"enabled": true
},
```

エージェントやマーケットとまったく同じ仕方で X10 と JSON が連携していることがわかるだろう．

取引停止は対象とするマーケットの価格変化がある閾値を越えたら，そのマーケットの取引を停止させる．
`TradingHaltRule` の現状の実装では `Market` のインスタンスを参照している．
上記の X10 コードには，どうやってインスタンス化されたマーケットを取得するか，が示されている．

具体的には，`GLOBAL` には，インスタンス化済みのマーケット，エージェント，イベントなどが JSON で宣言したマーケット名を `key` として `GLOBAL(key)` に保存されている．
インスタンス化は (1) マーケット，(2) エージェント，(3) イベント（金融規制／金融ショック）（順不定）の順番で行われ，先にインスタンス化されたオブジェクトにのみ，`GLOBAL` を介してアクセスできる．
マーケットのインスタンス化は相互依存関係を調べたうえで実行されるため，マーケット名が間違っていたり，循環関係が宣言されていない限り，マーケット `key` が `GLOBAL` に不在ということはない．
`GLOBAL` には `createMarkets()`，`createAgents()`，`createEvents()` の返り値がそのまま登録されているため，これらに関して言えば，key に対応する value すなわち `GLOBAL(key)` はすべて `List[T]` 型である．

実装上では，下記の補助メソッド群が提供されており，ユーザが `GLOBAL` に直接アクセスすることは少ないと思われる．

| Keys | Value | Market                | Agent                | Event
|------|-------|-----------------------|----------------------|----------------------
| Many | Many  | `getMarketsByNames()` | `getAgentsByNames()` | `getEventsByNames()`
| One  | Many  | `getMarketsByName()`  | `getAgentsByName()`  | `getEventsByName()`
| One  | One   | `getMarketByName()`   | `getAgentByName()`   | `getEventByName()`

上記の JSON ファイルでは，`"targetMarkets": ["Market"]` のうち，`"Market"` の部分がマーケットの定義名と一致していることが重要である．
上記の X10 コードでは，`getMarketsByNames(json("targetMarkets"))` という書き方で，既にインスタンス化されているマーケット `"Market"` を取得している．


### createEvents(): FundamentalPriceShock

以下のコードは `FundamentalPriceShock` クラスを初期設定する．

```x10
// samples/TradingHalt/TradingHaltMain.x10
public def setupFundamentalPriceShock(shock:FundamentalPriceShock, json:JSON.Value, random:JSONRandom) {
	val market = getMarketByName(json("target"));
	shock.marketId = market.id;
	shock.triggerTime = json("triggerTime").toLong();
	shock.shockTimeLength = FundamentalPriceShock.NO_TIME_LENGTH;
	shock.priceChangeRate = json("priceChangeRate").toDouble();
	market.addBeforeSimulationStepEvent(shock);
}
```

対応する JSON ファイルの一部を示す．

```json
// samples/TradingHalt/config.json
"FundamentalPriceShock": {
	"class": "FundamentalPriceShock",
	"target": "Market",
	"triggerTime": 0,    "MEMO": "At the beginning of the session 2",
	"priceChangeRate": -0.1,    "MEMO": "Sign: negative for down; positive for up; zero for no change",
	"enabled": true
},
```


### JSON configuration file

最後に，シミュレーションにこれまで定義した `TradingHaltRule` および `FundamentalPriceShock` を追加する方法を示す．
以下は JSON ファイルの `"simulation"` の部分である．
`CI2002/config.json` と比較すれば，`"MEMO"` で注釈づけられた部分が異なる．

```json
// samples/TradingHalt/config.json
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
			"events": ["FundamentalPriceShock", "TradingHaltRule"],    "MEMO": "ショック & "取引停止"
        }
    ]
}
```

JSON ファイル中のコメントにあるように，`"events"` にシミュレーションに取り込むショックや金融規制のリストを指定している．

取引停止は根拠の不十分な制度であるが，その目的のひとつはチャーチストによる市場価格の加速的な急落を抑制することにある．
エージェントの時系列分析の重視度 `"chartWeight"` と，取引停止の発動閾値 `"triggerChangeRate"`，発動期間 `"haltingTimeLength"` がどのような関係にあるかはシミュレーション分析により検討されている．


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/TradingHaltMain_UseCases)）．



## Related works

  * 清水，村永 (1999) 取引停止措置が市場機能に及ぼす影響：人為的シャットダウンを備えた市場の挙動に関するシミュレーション分析
  * 小林，橋本 (2006) サーキットブレーカー制度の有効性とその限界 ～人工市場シミュレーションによる検討～

