---
title: FatFingerMain
author: Takuma Torii
math: false
---

# FatFingerMain

本記事では，理論価格（企業価値）の急落とは異なる，誤発注という金融ショックをシミュレーションする．
誤発注は別名 fat finger（太い指）エラーと呼ばれ，注文価格や注文量の入力ミスをさす．
誤発注が生じると，前日の取引値からかけ離れて安い（高い）価格で売買が成立し，その結果，価格発見という市場の機能を阻害すると考えられる．

本記事で扱う内容：

  * 金融ショックの追加（誤発注ショック）

関連するファイル：

  * `samples/FatFinger/FatFingerMain.x10`
  * `plham/event/OrderMistakeShock.x10`


## Compile & run

```
 $ x10c++ samples/FatFinger/FatFingerMain.x10
 $ ./a.out samples/FatFinger/config.json
```


## FatFingerMain

`FatFingerMain` は `CI2002Main` を継承し，次のような構造をもつ．

```x10
// samples/FatFinger/FatFingerMain.x10
public class FatFingerMain extends CI2002Main {

	public static def main(args:Rail[String]) {
		new SequentialRunner(new FatFingerMain()).run(args);
	}

    public def createEvents(json:JSON.Value):List[Event] {}
}
```


### createEvents(): OrderMistakeShock

`createEvents()` では `OrderMistakeShock` をインスタンス化する．

以下のコードは `OrderMistakeShock` クラスを初期設定する．

```x10
// samples/FatFinger/FatFingerMain.x10
public def setupOrderMistakeShock(shock:OrderMistakeShock, json:JSON.Value, random:JSONRandom) {
	val market = getMarketByName(json("target"));
	val agent = getAgentsByName(json("agent"))(0); // Use the first one.
	val t = market.getTime();
	shock.marketId = market.id;
	shock.agentId = agent.id;
	shock.triggerTime = t + json("triggerTime").toLong();
	shock.priceChangeRate = json("priceChangeRate").toDouble();
	shock.orderVolume = json("orderVolume").toLong();
	shock.orderTimeLength = json("orderTimeLength").toLong();
	market.addBeforeSimulationStepEvent(shock);
}
```

対応する JSON ファイルの一部を示す．

```json
// samples/FatFinger/config.json
"OrderMistakeShock": {
	"class": "OrderMistakeShock",
	"target": "Market",
	"agent": "FCNAgents", "MEMO": "One of them is selected",
	"triggerTime": 100,       "MEMO": "At the 100th step of the session 2",
	"priceChangeRate": -0.05, "MEMO": "Sign: negative for down; positive for up; zero for no change",
	"orderVolume": 10000,     "MEMO": "Very much",
	"orderTimeLength": 10000, "MEMO": "Very long",
	"enabled": true
},
```

これまでのチュートリアルから理解できるだろう．


### JSON configuration file

特筆すべき点はないが，`OrderMistakeShock` を使用している点に注意する．

```json
// samples/FatFinger/config.json
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
			"events": ["OrderMistakeShock"]
		}
	]
},
```


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/FatFingerMain_UseCases)）．



## Related works

  * 


