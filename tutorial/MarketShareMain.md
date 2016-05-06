---
title: MarketShareMain
author: Takuma Torii
math: false
---

# MarketShareMain

本記事では，草田ら (2015) が行ったマーケットメイカーが出来高シェア競争に与える影響分析のシミュレーションを作成することを通して，本ソフトウェアの使い方を説明する．

本記事で扱う内容：

  * 複数のマーケット
  * 高頻度エージェント，マーケットメイカー

関連するファイル：

  * `samples/MarketShare/MarketShareMain.x10`
  * `samples/MarketShare/MarketShareFCNAgent.x10`
  * `samples/MarketShare/MarketMakerAgent.x10`


## Preface

取引所（e.g. 日本証券取引所 JPX，ニューヨーク証券取引所 NYSE）は，「取引」というサービスを提供する見返りとして，手数料などから利益をえる．
このため，取引量あるいは出来高シェアは，取引所にとって重要な問題である．

草田ら (2015) では，２つの市場（シミュレーション上は個別銘柄）の間の出来高シェア競争を取り上げ，出来高シェア競争に対してマーケットメイカーの介入が有効であることを示している．

マーケットメイカー（`MarketMakerAgent`）は，典型的には，仲値の上下に売り・買い両方の注文を高頻度で差し込むことで取引（売買の成立）の活性化を促す役割を担う．
マーケットメイカーは証券会社などの投資機関が担当する．

個人投資家に対応するトレーダとして，草田ら (2015) の拡張版 `FCNAgent`（`MarketShareFCNAgent`）は，２つの銘柄をその出来高で比較し，より出来高の多い銘柄により高い確率で注文をだす．
すなわち，より取引が活発な市場を好む．


## Compile & run

```
$ x10c++ samples/MarketShare/MarketShareMain.x10
$ ./a.out samples/MarketShare/config.json
```


## MarketMakerAgent

`MarketMakerAgent` の実装に関して特筆すべき点はない．
草田ら (2015) の論文と X10 プログラムを比較してほしい．

本記事では草田ら (2015) がシンプルマーケットメイカーと呼ぶ取引戦略を用いた．
シンプルマーケットメイカーの取引戦略は「仲値の上下に売り・買い両方の注文を高頻度で差し込む」というものだが，注文価格と仲値の幅を決めるパラメータ「スプレッド」（`netInterestSpread`）をもつ．


## MarketShareFCNAgent

`MarketShareFCNAgent` は，２つの銘柄をその出来高で比較し，より出来高の多い銘柄により高い確率で注文をだす．
各銘柄の各時点での出来高は `Market#getTradeVolume(t)` により取得できる．
したがって，銘柄選択を実装するには，`timeWindowSize` の期間に渡る合計出来高を「比重」として確率的にルーレット選択を行えば良い．
これは下記のように実装できる．

```x10
// samples/MarketShare/MarketShareFCNAgent.x10
public def submitOrders(markets:List[Market]):List[Order] {
	val weights = new ArrayList[Double]();
	for (m in markets) {
		weights.add(getSumTradeVolume(m));
	}
	val k = Statistics.roulette(getRandom(), weights);
	val market = markets(k);
	return super.submitOrders(market);
}

public def getSumTradeVolume(market:Market):Long {
	val t = market.getTime();
	val timeWindowSize = Math.min(t, this.timeWindowSize);
	var volume:Long = 0;
	for (d in 1..timeWindowSize) {
		volume += market.getTradeVolume(t - d);
	}
	return volume;
}
```


## MarketShareMain

### print() メソッド

各時点 t の出来高を出力するにとどめ，「過去 100 ステップにおける銘柄 B の出来高シェア」は R の解析プログラムにより求める．
具体的には `print()` メソッドを以下のように書き，`getTradeVolume(t)` により各時点の出来高を出力しておく．

```x10
// samples/MarketShare/MarketShareMain.x10
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
			market.getTradeVolume(t),
			"", ""], " ", "", Int.MAX_VALUE));
	}
}
```


### JSON configuration file

以下は JSON ファイルの `"simulation"` の部分である．

```json
// samples/MarketShare/config.json
"simulation": {
	"markets": ["Market-A", "Market-B"],
	"agents": ["MarketShareFCNAgents", "MarketMakerAgent"],
	"sessions": [
		{	"sessionName": 0,
			"iterationSteps": 100,
			"withOrderPlacement": true,
			"withOrderExecution": false,
			"withPrint": true
		},
		{	"sessionName": 1,
			"iterationSteps": 2000,
			"withOrderPlacement": true,
			"withOrderExecution": true,
			"withPrint": true,
			"maxHifreqOrders": 1,    "MEMO": "マーケットメイカーの介入頻度"
		}
	]
},
```


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/MarketShareMain_UseCases)）．


## Related works

  * 草田・水田・早川・和泉・吉村 (2014) 人工市場を用いたマーケットメーカーのスプレッドが市場出来高に与える影響の分析
  * 草田・水田・早川・和泉 (2015) 保有資産を考慮したマーケットメイク戦略が取引所間競争に与える影響:人工市場アプローチによる分析

