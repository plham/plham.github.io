---
title: ShockTransferMain
author: Takuma Torii
math: false
---

# ShockTransferMain

本記事では，鳥居，中川，和泉 (2015) が行ったショック伝搬のシミュレーションを作成することを通して，本ソフトウェアの使い方を説明する．

本記事で扱う内容：

  * 複数マーケットの使用
  * 指数マーケットの使用
  * 裁定エージェントの使用

関連するファイル：

  * `samples/ShockTransfer/ShockTransferMain.x10`
  * `plham/IndexMarket.x10`
  * `plham/agent/ArbitrageAgent.x10`
  * `plham/event/FundamentalPriceShock.x10`


## Preface

鳥居，中川，和泉 (2015) のシミュレーションは，2 つの現物銘柄（単に `Market`）と 1 つの指数銘柄（`IndexMarket`）から構成され，個別銘柄を独立に取引する FCN エージェントの他に，現物と指数の価格差から利益をえる裁定エージェント（`ArbitrageAgent`）が取引を行う．
研究目的は，ある現物銘柄で生じた価格の急落が，裁定取引を媒介して，指数を構成する他の銘柄へどのようにショック伝搬するかを調べることにある．

`FCNAgent` および `Market` はすでに [`CI2002Main`](/tutorial/CI2002Main) の記事で説明済みであるので，本稿では解説しない．
また，ショックに関しては [`TradingHaltMain`](/tutorial/TradingHaltMain) の記事を参照してほしい．

本記事では，指数銘柄（`IndexMarket`）および裁定エージェント（`ArbitrageAgent`）をシミュレーションに組み込む方法を説明する．


## IndexMarket

以下に `IndexMarket` の骨格を示す．
指数銘柄に相当する `IndexMarket` は，現物銘柄に相当する複数の `Market` と紐付けられる．
株価指数の計算方式には一般に価格平均指数と時価総額平均指数の 2 種類があるが，下記で述べるように，それぞれのクラスが用意されている．

```x10
// plham/IndexMarket.x10
public class IndexMarket extends Market {

    public def addMarket(market:Market) {}            // 指数構成銘柄に追加
    public def addMarkets(markets:List[Market]) {}    // 指数構成銘柄に追加
    public def getMarkets():List[Market] {}           // 指数構成銘柄を取得

    public def setMarketIndexMethod(method:MarketIndex) {}         // 指数計算方式
    public def setFundamentalIndexMethod(method:MarketIndex) {}    // 指数計算方式

    public def getMarketIndex(t:Long) {}              // 市場価格指数の取得（時点 t）
    public def getFundamentalIndex(t:Long) {}         // 理論価格指数の取得（時点 t）

    public def computeMarketIndex():Double {}         // 市場価格指数の計算（再計算）
    public def computeFundamentalIndex():Double {}    // 理論価格指数の計算（再計算）
}
```

株価指数の計算方式には価格平均指数と時価総額平均指数の 2 種類はそれぞれ

  * `plham.index.PriceWeightedIndex`
  * `plham.index.CapitalWeightedIndex`

に実装されている．
具体的なインスタンス化の手順は `ShockTransferMain.x10` を参照してほしい．


## ArbitrageAgent

現物-指数間の裁定取引を行うエージェントは `ArbitrageAgent` クラスに実装されている．

```x10
// plham/agent/ArbitrageAgent.x10
public class ArbitrageAgent extends HighFrequencyAgent {

    public def submitOrders(market:Market):List[Order] {
        val index = market as IndexMarket;
        val spots = index.getMarkets();
        /* ... */
    }
}
```

`FCNAgent` との違いは (a) `HighFrequencyAgent` を継承している点，(b) `IndexMarket` のみを取引対象とする点にある．
`IndexMarket#getMarkets()` メソッドを介して，現物銘柄のリストへアクセスする．

(a) `HighFrequencyAgent` はマーカーの役割を果たし，本ソフトウェアがエージェントのタイプ「高速取引」を識別するために用いられる．
具体的には，`HighFrequencyAgent` を継承したエージェントクラスは，エージェントシミュレーションの流れのうち，高速取引を行える位置に組み込まれる．

(b) 現物銘柄へのアクセスは `index.getMarkets()` から行う．
意思決定の詳細は後続の節か，`plham/agent/ArbitrageAgent.x10` を直接参考にしてほしい．


## ShockTransferMain

`ShockTransferMain` は `CI2002Main` を継承し，次のような構造をもつ．

```x10
public class ShockTransferMain extends CI2002Main {

	public static def main(args:Rail[String]) {
		new SequentialRunner(new ShockTransferMain()).run(args);
	}

	public def createMarkets(json:JSON.Value):List[Market] {}

	public def createAgents(json:JSON.Value):List[Agent] {}

	public def createEvents(json:JSON.Value):List[Event] {}
}
```

コードに示されるように，`createMarkets()`，`createAgents()`，`createEvents()` のすべてにおいて，シミュレーションに必要な新しい部品を追加していく．


### createMarkets()

`createMarkets()` では `IndexMarket` をインスタンス化する．
通常の `Market` に関しては `CI2002Main` での定義を再利用する．

```x10
// samples/ShockTransfer/ShockTransferMain.x10
public def createMarkets(json:JSON.Value):List[Market] {
	val random = new JSONRandom(getRandom());
	val markets = super.createMarkets(json); // Use Market defined in CI2002Main.
	if (json("class").equals("IndexMarket")) {
		val market = new IndexMarket();
		setupIndexMarket(market, json, random);
		markets.add(market);
	}
	return markets;
}
```

`IndexMarket` をインスタンス化するさい，`addMarket()` を呼び出し，株価指数構成銘柄の追加を JSON ファイルと連携しながら行う必要がある．
これを行うには，ユーザはインスタンス化済みの他のマーケットを取得する必要があるが，これは `GLOBAL(String)` を介して行える．

具体的には，`GLOBAL` には，インスタンス化済みのマーケット，エージェント，イベントなどが JSON で宣言したマーケット名を `key` として `GLOBAL(key)` に保存されている．
インスタンス化は (1) マーケット，(2) エージェント，(3) イベント（金融規制／金融ショック）（順不定）の順番で行われ，先にインスタンス化されたオブジェクトにのみ，`GLOBAL` を介してアクセスできる．
マーケットのインスタンス化は相互依存関係を調べたうえで実行されるため，マーケット名が間違っていたり，循環関係が宣言されていない限り，マーケット `key` が `GLOBAL` に不在ということはない．
`GLOBAL` には `createMarkets()`，`createAgents()`，`createEvents()` の返り値がそのまま登録されているため，これらに関して言えば，key に対応する value すなわち `GLOBAL(key)` はすべて `List[T]` 型である．

実装上では，下記の補助メソッド群が提供されており，ユーザが `GLOBAL` に直接アクセスすることは少ないと思われる．

| Market                | Agent                | Event
|-----------------------|----------------------|----------------------
| `getMarketsByName()`  | `getAgentsByName()`  | `getEventsByName()`
| `getMarketByName()`   | `getAgentByName()`   | `getEventByName()`

具体的には以下のようにする．

```x10
// samples/ShockTransfer/ShockTransferMain.x10
public def setupIndexMarket(market:IndexMarket, json:JSON.Value, random:JSONRandom) {
	market.setTickSize(random.nextRandom(json("tickSize", "-1.0"))); // " tick-size <= 0.0 means no tick size.

	val spots = getMarketsByNames(json("markets"));
	market.addMarkets(spots);

	// WARN: Market's methods access to market.env is not available here :WARN

    // 市場価格指数
	val marketIndex = new CapitalWeightedIndexScheme(CapitalWeightedIndexScheme.MARKET_PRICE);
	marketIndex.setIndexDivisor(random.nextRandom(json("marketPrice")), marketIndex.getIndex(spots));
	market.setMarketIndexScheme(marketIndex);

    // 理論価格指数
	val fundamIndex = new CapitalWeightedIndexScheme(CapitalWeightedIndexScheme.FUNDAMENTAL_PRICE);
	fundamIndex.setIndexDivisor(random.nextRandom(json(["fundamentalPrice", "marketPrice"])), fundamIndex.getIndex(spots));
	market.setFundamentalIndexScheme(fundamIndex);

    // 指数の値を使い IndexMarket を初期設定
	market.setInitialMarketPrice(marketIndex.getIndex(spots));
	market.setInitialMarketIndex(marketIndex.getIndex(spots));
	market.setInitialFundamentalPrice(fundamIndex.getIndex(spots));
	market.setInitialFundamentalIndex(fundamIndex.getIndex(spots));
	market.setOutstandingShares(random.nextRandom(json("outstandingShares")) as Long);
}
```


### createAgents()

`createAgents()` の中では `ArbitrageAgent` をインスタンス化する．
`FCNAgent` に関しては `CI2002Main` での定義を再利用する．

```x10
// samples/ShockTransfer/ShockTransferMain.x10
public def createAgents(json:JSON.Value):List[Agent] {
	val random = new JSONRandom(getRandom());
	val agents = super.createAgents(json); // Use FCNAgent defined in CI2002Main.
	if (json("class").equals("ArbitrageAgent")) {
		val numAgents = json("numAgents").toLong();
		for (i in 0..(numAgents - 1)) {
			val agent = new ArbitrageAgent();
			setupArbitrageAgent(agent, json, random);
			agents.add(agent);
		}
	}
	return agents;
}
```

`ArbitrageAgent` の初期設定に関して特別な点はないので説明を省く．

```x10
// samples/ShockTransfer/ShockTransferMain.x10
public def setupArbitrageAgent(agent:ArbitrageAgent, json:JSON.Value, random:JSONRandom) {
	agent.orderVolume = json("orderVolume").toLong();
	agent.orderThresholdPrice = json("orderThresholdPrice").toDouble();

	assert json("markets").size() == 1 : "ArbitrageAgents suppose only one IndexMarket";
	assert getMarketByName(json("markets")(0)) instanceof IndexMarket : "ArbitrageAgents suppose only one IndexMarket";
	val market = getMarketByName(json("markets")(0)) as IndexMarket;
	agent.setMarketAccessible(market);
	for (id in market.getComponents()) {
		agent.setMarketAccessible(id);
	}

	agent.setAssetVolume(market, random.nextRandom(json("assetVolume")) as Long);
	for (id in market.getComponents()) {
		agent.setAssetVolume(id, random.nextRandom(json("assetVolume")) as Long);
	}
	agent.setCashAmount(random.nextRandom(json("cashAmount")));
}
```


### createEvents()

`createEvents()` では `FundamentalPriceShock` をインスタンス化する．
取引停止の事例と同じなので[こちら](/tutorial/TradingHaltMain)を参照してほしい．


### JSON ファイルによるシミュレーションの制御

`CI2002Main` からの最大の違いは，`"markets"` に複数のマーケットを指定していること，また `"agents"` に複数のエージェントを指定していることである．
これらの指定する順番は任意であり，シミュレーションの初期化の順番に影響しない．
すなわち，マーケット間の相互依存関係はソフトウェアにより自動的に認識され，依存関係を解消する順番でインスタンス化される．

```json
// samples/ShockTransfer/config.json
"simulation": {
	"markets": ["SpotMarket-1", "SpotMarket-2", "IndexMarket-I"],
	"agents": ["FCNAgents-1", "FCNAgents-2", "FCNAgents-I", "ArbitrageAgents"],
	"sessions": [
		{	"sessionName": 0,
			"iterationSteps": 100,
			"withOrderPlacement": true,
			"withOrderExecution": false,
			"withPrint": true,
			"maxNormalOrders": 3, "MEMO": "The same number as #markets",
			"maxHifreqOrders": 0
		},
		{	"sessionName": 1,
			"iterationSteps": 500,
			"withOrderPlacement": true,
			"withOrderExecution": true,
			"withPrint": true,
			"maxNormalOrders": 3, "MEMO": "The same number as #markets",
			"maxHifreqOrders": 5,
			"events": ["FundamentalPriceShock"]
		}
	]
}
```


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/ShockTransferMain_UseCases)）．



## Related works

  * Chiarella & Iori (2002) A simulation analysis of the microstructure of double auction markets
  * 鳥居，中川，和泉 (2015) 複数資産人工市場を用いた裁定取引によるショック伝搬の分析

