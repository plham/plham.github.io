---
title: CI2002Main
author: Takuma Torii
math: false
---

# CI2002Main

本記事では Chiarella & Iori (2002) が行った単一銘柄シミュレーションを作成することを通して，本ソフトウェアの使い方を説明する（ただし，パラメータ設定の方法など若干の違いがある）．

本記事で扱う内容：

  * エージェントの作成
  * メインの作成
  * JSON との連携

関連するファイル：

  * `samples/CI2002/CI2002Main.x10`
  * `plham/agent/FCNAgent.x10`
  * `plham/Market.x10`


## Components of this simulation

本ソフトウェアは市場の要素を表す以下の基本クラスからなる．

  * `plham.Agent`        ... エージェント
  * `plham.Market`       ... マーケット・銘柄
  * `plham.IndexMarket`  ... 複数銘柄と紐づいた銘柄（e.g. 指数先物，ETF ）
  * `plham.Order`        ... 注文
  * `plham.OrderBook`    ... 板（オーダーブック）
  * `plham.Fundamentals` ... 理論価値

人工市場シミュレーションはこれらの基本クラスおよび派生クラスを組み合わせることで実現される．
本記事の目的は，その組み合わせ方，ひとつのシミュレーションを構築する手順を説明することにある．

ただし，以下の用語に留意する．

  * 「エージェント」は現実における個人投資家か投資企業に相当する
    * 必ずしもひとりの人間と対応しない
  * 「マーケット」は市場というよりむしろ現実における個別の銘柄に相当する
    * ある銘柄の注文を処理する単位をマーケットと呼ぶ．銘柄に対応するクラスは存在しない
    * 複数の銘柄を考える場合，複数のマーケットを用意する


## Compile and run

```
$ x10c++ samples/CI2002/CI2002Main.x10
$ ./a.out samples/CI2002/config.json
```


## Chiarella & Iori (2002)

Chiarella & Iori (2002) のモデルは

  * 連続ダブルオークション型のマーケット１つ
  * 理論価格，時系列分析，ノイズに基づき注文を決定するエージェント

をもつ．
したがって，通常のやり方でこの人工市場シミュレーションを行うには，ユーザは上記の機能を実現するマーケットおよびエージェントという部品を実装し，さらに金融市場の流れに沿ってそれらの部品を組み合わせる「メイン（`Main`）」プログラムを書かなければならない．
また，メイン（`Main`）には，設定ファイルを読み取り，マーケットやエージェントのパラメータを初期化する，各時点での途中経過を出力するといった処理も含まれる．
これらをフルスクラッチで実装するのは容易ではない．
そこで，本ソフトウェアを使えば，ユーザはモデルの拡張点に当たる部品を開発するだけで基本的な人工市場シミュレーションを実行できる．

本ソフトウェアを使う場合，上記の機能を実現するマーケットおよびエージェントはそれぞれ次のクラスに既に実装されており，ユーザはこれを利用できる．

  * [plham.Market](/Market)
  * [plham.agent.FCNAgent](/FCNAgent)

以下では `FCNAgent` の実装を概観し，エージェントクラス作成のポイントを押さえる．
意思決定の方法は原著 Chiarella & Iori (2002) や鳥居・中川・和泉 (2015) を参照してほしい．
また，本記事で用いる `Market` や `FCNAgent` をどのように X10 で実装したかは部分的には [`Market`](/Market) や [`FCNAgent`](/FCNAgent) で解説されているが，直接コードを確認してほしい．
自分で新しいエージェントクラスを作成するときには，原著論文と X10 プログラムを見比べると参考になるだろう．


## Market

`Market` クラスは以下のメソッドを含む．
Market クラスは連続ダブルオークション（ザラバ方式）で注文を処理する（詳しくは[こちら](/Market)）．

```x10
// plham/Market.x10
public class Market {

    public def handleOrders(orders:List[Order]) {}       // 注文の処理（約定）
    public def handleOrder(order:Order) {}               // 注文の処理（約定）

    public def getTime():Long {}                         // ステップ時間の取得

    public def getPrice(t:Long):Double {}                // ステップ t の市場価格
    public def getFundamentalPrice(t:Long):Double {}     // ステップ t の理論価格

    public def getBestBuyPrice();     // 買い気配値（最新の値のみ）
    public def getBestSellPrice();    // 売り気配値（最新の値のみ）
    public def getMidPrice();         // 仲値（最新の値のみ）
}
```

エージェントの意思決定を実装するときにはマーケットの情報を取得する必要がある．
マーケットの情報を取得するときには上記のうち，`handleOrders()`，`handleOrder()` を**除く**，他のメソッドを使うことになる．
`handleOrders()` 類は本ソフトウェアから自動的に呼び出されるので，通常，ユーザがこのメソッドを呼び出すことはない．

留意点として，計算実行モデルによってはマーケットごとにステップ時刻が異なる可能性がある．
そのため，すべてのマーケットで `getTime()` が一致しているという前提で意思決定を実装することは望ましくない．


## FCNAgent

`FCNAgent` クラスは次のような構造をもつ．
`FCNAgent` クラスは Chiarella & Iori (2002) 型の意思決定を実装する（詳しくは[こちら](/FCNAgent)）．

```x10
// plham/agent/FCNAgent.x10
public class FCNAgent extends Agent {

    public var fundamentalWeight:Double;    // 理論価格分析
    public var chartWeight:Double;          // 時系列分析
    public var noiseWeight:Double;          // ノイズ
    public var isChartFollowing:Boolean;
    public var fundamentalMeanReversion:Double;
    public var timeWindowSize:Long;
    public var noiseScale:Double;
    public var orderMargin:Double;

    public def submitOrders(market:Market):List[Order] {}    // 注文の意思決定
}
```

各フィールドのうち

  * `fundamentalWeight`  ... 理論価格，ファンダメンタル成分
  * `chartWeight`        ... 時系列分析，チャート成分
  * `noiseWeight`        ... ランダム，ノイズ成分

はそれぞれ取引戦略に対する荷重を表す．
これらの頭文字をとって FCN エージェントと呼ぶ．
エージェントの注文の決定は以下のメソッドに実装する．

  * `submitOrders(List[Market]):List[Order]`

`submitOrders(List[Market])` は `Market` のリストを受けとり，`Order` のリストを返す．
FCN エージェントは単一銘柄しか取引しないので，`FCNAgent.x10` では次のように定義されている．

```x10
// plham/Agent.x10
public def submitOrders(markets:List[Market]):List[Order] {
    val orders = new ArrayList[Order]();
    for (market in markets) {
        orders.addAll(this.submitOrders(market));
    }
    return orders;
}

public def submitOrders(market:Market):List[Order] {
    /* Chiarella & Iori (2002) */
}
```


## CI2002Main

メインクラスの役割は JSON ファイルに基づき，要求されるモデルを構築し，要求されるシミュレーションを実行することにある．

ユーザは `plham.Main` を継承して，新しいメインクラスを作成できる．
`Main` は JSON ファイルにもとづき，シミュレーションに必要な部品を構築する際の基本的な処理や，標準的な人工市場シミュレーションを実行する機能をもつ．
そのため，ユーザは自ら拡張定義したエージェントやマーケットを登録するだけでよい．
以下ではまず，このやり方を解説する．
次いで，`Main` で定義されたシミュレーションの流れを JSON ファイルから制御する方法を解説する．

まず，`CI2002Main` は次のような構造をもつ．

```x10
// samples/CI2002/CI2002Main.x10
public class CI2002Main extends Main {

    public static def main(args:Rail[String]) {
        new SequentialRunner(new CI2002Main()).run(args);
    }

    public def createMarkets(json:JSON.Value):List[Market] {}

    public def createAgents(json:JSON.Value):List[Agent] {}
}
```

主要なメソッドは `createMarkets(JSON.Value)` と `createAgents(JSON.Value)` であり，いずれもスーパークラス `Main` で定義されており，JSON ファイルをもとにシミュレーションモデルを構築する過程で呼び出される．
ユーザに課された責任は `createMarkets()` と `createAgents()` をオーバーライドすることで，JSON と連携し，エージェントやマーケットをインスタンス化することである．


### createMarkets()

まずは `createMarkets()` を見てみよう．
Chiarella & Iori (2002) のマーケットモデルは `Market` に実装済みなのでこれを利用している．

```x10
// samples/CI2002/CI2002Main.x10
public def createMarkets(json:JSON.Value):List[Market] {
    val random = new JSONRandom(getRandom());
    val markets = new ArrayList[Market]();
    if (json("class").equals("Market")) {
        val market = new Market();
        setupMarket(market, json, random);
        markets.add(market);
    }
    return markets;
}
```

具体的に，Market の初期設定は `setupMarket()` メソッドを呼び出している．

```x10
// samples/CI2002/CI2002Main.x10
public def setupMarket(market:Market, json:JSON.Value, random:JSONRandom) {
    market.setTickSize(random.nextRandom(json("tickSize", "-1.0"))); // " tick-size <= 0.0 means no tick size.
    market.setInitialMarketPrice(random.nextRandom(json("marketPrice")));
    market.setInitialFundamentalPrice(random.nextRandom(json("marketPrice")));
    market.setOutstandingShares(random.nextRandom(json("outstandingShares")) as Long);
}
```

各所でみられる `json(key)` は JSON ファイルで定義された key-value ペアを取りだす操作である．
上記のコードでは，[Market の属性](/Market) の初期値を JSON から読み込み，設定している．
JSON 上で乱数分布を指定する記法を可能にするため，`JSONRandom#nextRandom()` を経由して初期値を設定している（詳しくは [こちら](/JSONRandom)）．

`createMarkets()` の引数で与えられる `json:JSON.Value` は key `"Market"` に対応した value，すなわち `{ "class": "Market",... }` を格納した JSON.Value オブジェクトである．
以下に示すのはマーケットのプロパティに関する JSON ファイルの一部である．
X1０ コード（上記 `setupMarket()` メソッド）との対応関係が読みとれるだろう．

```javascript
// samples/CI2002/config.json
"Market": {
    "class": "Market",
    "tickSize": 0.00001,
    "marketPrice": 300.0,
    "outstandingShares": 25000
},
```

`Main` は JSON ファイルに関していくつかの制約を課しているが，それ以外ではユーザは自由に JSON の key 名を定めてよい．
JSON の詳細は[こちら](/tutorial/JSON_for_Main)を参照してほしい．


### createAgents()

`createAgents()` についても同じ仕方で書かれている．
Market 同じく，[FCNAgent の属性](/FCNAgent) の初期値を JSON から読み込み，設定している．
ただし，複数のエージェントを生成している点に留意する．

```x10
// samples/CI2002/CI2002Main.x10
public def createAgents(json:JSON.Value):List[Agent] {
    val random = new JSONRandom(getRandom());
    val agents = new ArrayList[Agent]();
    if (json("class").equals("FCNAgent")) {
        val numAgents = json("numAgents").toLong();
        for (i in 0..(numAgents - 1)) {
            val agent = new FCNAgent();
            setupFCNAgent(agent, json, random);
            agents.add(agent);
        }
    }
    return agents;
}
```

具体的な初期値の設定は次の `setupFCNAgent()` で行われる．

```x10
// samples/CI2002/CI2002Main.x10
public def setupFCNAgent(agent:FCNAgent, json:JSON.Value, random:JSONRandom) {
    val MARGIN_TYPES = JSON.parse("{'fixed': " + FCNAgent.MARGIN_FIXED + ", 'normal': " + FCNAgent.MARGIN_NORMAL + "}");

    agent.fundamentalWeight = random.nextRandom(json("fundamentalWeight"));
    agent.chartWeight = random.nextRandom(json("chartWeight"));
    agent.noiseWeight = random.nextRandom(json("noiseWeight"));
    agent.isChartFollowing = (random.nextDouble() < 1.0); // 100%

    agent.noiseScale = random.nextRandom(json("noiseScale"));
    agent.timeWindowSize = random.nextRandom(json("timeWindowSize")) as Long;
    agent.orderMargin = random.nextRandom(json("orderMargin"));
    agent.marginType = MARGIN_TYPES(json("marginType", "fixed")).toLong();

	assert json("markets").size() == 1 : "FCNAgents suppose only one Market";
	val market = getMarketByName(json("markets")(0));
    agent.setMarketAccessible(market);
    agent.setAssetVolume(market, random.nextRandom(json("assetVolume")) as Long);
    agent.setCashAmount(random.nextRandom(json("cashAmount")));
}
```

以下に示すのは FCN エージェントのプロパティに関する JSON ファイルの一部である．

```javascript
// samples/CI2002/config.json
"FCNAgents": {
    "class": "FCNAgent",
    "numAgents": 100,

    "MEMO": "Agent class",
    "markets": ["Market"],
    "assetVolume": 50,
    "cashAmount": 10000,

    "MEMO": "FCNAgent class",
    "fundamentalWeight": {"expon": [1.0]},
    "chartWeight": {"expon": [0.0]},
    "noiseWeight": {"expon": [1.0]},
    "noiseScale": 0.001,
    "timeWindowSize": [100, 200],
    "orderMargin": [0.0, 0.1]
}
```

上記 JSON ファイルの説明は[こちら](/FCNAgent)を参照してほしいが，たとえば，`"chartWeight"` をより大きな数値に設定すれば，チャート重視型戦略の多い場合の取引をシミュレーションできる．



### JSON configuration file

ここまで，ユーザが独自のエージェントやマーケットを作成したとき，X10 プログラムをどのように書き，JSON ファイルと連携すればよいかを示した．

最後に，`Main` で実装済みの人工市場シミュレーションの流れを JSON ファイルから制御する方法を解説する．
以下にその部分の JSON ファイルを示す．

```javascript
// samples/CI2002/config.json
"simulation": {
    "markets": ["Market"],
    "agents": ["FCNAgents"],
    "sessions": [
        {    "sessionName": 0,
             "iterationSteps": 100,
             "withOrderPlacement": true,
             "withOrderExecution": false,
             "withPrint": true
        },
        {    "sessionName": 1,
             "iterationSteps": 500,
             "withOrderPlacement": true,
             "withOrderExecution": true,
             "withPrint": true
        }
    ]
},
```

詳しくは[こちら](/tutorial/JSON_for_Main) を参照してほしい．


### Changing Console output

シミュレーションの出力データは `Main` の `print(...)` 関数に記述されている．
ユーザはこの `print(...)` 関数を書き換えることで出力内容を変更できる．
以下に標準の `print(...)` 関数を示す．

```x10
// samples/CI2002/CI2002Main.x10
public def print(sessionName:String) {
	val markets = getMarketsByName("markets");
	val agents = getAgentsByName("agents");
    for (market in markets) {
        val t = market.getTime();
        Console.OUT.println(StringUtil.formatArray([
            sessionName,
            t, 
            market.id,
            market.name,
            market.getPrice(t),
            market.getFundamentalPrice(t),
            "", ""], " ", "", Int.MAX_VALUE));
    }
}
```

出力データのうち，第 5 列目が市場価格，第 6 列目がファンダメンタル価格である．


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/CI2002Main_UseCases)）．


## Related works

  * Chiarella & Iori (2002) A simulation analysis of the microstructure of double auction markets
  * 鳥居，中川，和泉 (2015) 複数資産人工市場を用いた裁定取引によるショック伝搬の分析

