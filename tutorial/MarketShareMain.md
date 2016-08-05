---
title: MarketShareMain
author: Takuma Torii
math: false
---

# MarketShareMain

本記事では，水田ら (2013) が行ったティックサイズ（価格の最小単位）が出来高シェア競争に与える影響分析，および，草田ら (2015) が行ったマーケットメイカーが出来高シェア競争に与える影響分析のシミュレーションを作成することを通して，本ソフトウェアの使い方を説明する．

本記事で扱う内容：

  * 複数のマーケット
  * ティックサイズ
  * 高頻度エージェント，マーケットメイカー

関連するファイル：

  * `samples/MarketShare/MarketShareMain.x10`
  * `samples/MarketShare/MarketShareFCNAgent.x10`
  * `samples/MarketShare/MarketMakerAgent.x10`


## Preface

複数の取引所が存在する現代において，各取引所（e.g. 日本証券取引所 JPX，ニューヨーク証券取引所 NYSE）にとって自らの営業する取引所の市場シェア（利用率）は重要な問題である．

水田ら (2013) では，ティックサイズの異なる２つの市場（シミュレーション上は個別銘柄だが，だた１つの同じ銘柄が２つの市場で取引されているとみなす）の間の出来高シェア競争を取り上げ，ティックサイスの小さい市場のほうが出来高シェアを高めることを示している．

ティックサイズは取引価格の最小単位であり，ティックサイズが小さいほどより細かい値段での売買が可能となる．

草田ら (2015) では，２つの市場の間の出来高シェア競争を取り上げ，出来高シェア競争に対してマーケットメイカーの介入が有効であることを示している．

マーケットメイカー（`MarketMakerAgent`）は，典型的には，仲値の上下に売り・買い両方の注文を高頻度で差し込むことで取引（売買の成立）の活性化を促す役割を担う．
マーケットメイカーは証券会社などの投資機関が担当する．

個人投資家に対応するトレーダとして，水田ら (2013) の拡張版 `FCNAgent`（`MarketShareFCNAgent`）は，２つの市場をその出来高で比較し，より出来高の多い市場により高い確率で注文をだす．
すなわち，取引がより活発な市場を好む．


## Compile & run

```
$ x10c++ samples/MarketShare/MarketShareMain.x10
$ ./a.out samples/MarketShare/config.json
```


## MarketMakerAgent

草田ら (2015) は水田ら (2013) の研究を拡張し，新たにマーケットメイカーというエージェントを加えている．

本記事では草田ら (2015) がシンプルマーケットメイカーと呼ぶ取引戦略を用いた．
シンプルマーケットメイカーの取引戦略は「仲値の上下に売り・買い両方の注文を高頻度で差し込む」というものだが，注文価格と仲値の幅を決めるパラメータ「スプレッド」（`netInterestSpread`）をもつ．

`MarketMakerAgent` の実装に関して特筆すべき点はない．
草田ら (2015) の論文と X10 プログラムを比較してほしい．


## MarketShareFCNAgent

水田ら (2013) の `MarketShareFCNAgent` は，２つの市場をその出来高で比較し，より出来高の多い市場により高い確率で注文をだす．
各市場の各時点での出来高は `Market#getTradeVolume(t)` により取得できる．
したがって，市場選択を実装するには，`timeWindowSize` の期間に渡る合計出来高を「比重」として確率的にルーレット選択を行えば良い．
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

各時点 t の出来高を出力するにとどめ，「過去 100 ステップにおける市場 B の出来高シェア」は R の解析プログラムにより求める．
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

次に，このプログラムを使って，シミュレーションを実行してみよう．
まず，水田ら (2013) が行ったティックサイズの影響を調べてみよう（[こちら](/tutorial/MarketShareMain_UseCases01)）．
次に，草田ら (2015) が行ったマーケットメイカーの影響を調べてみよう（[こちら](/tutorial/MarketShareMain_UseCases02)）．


## Related works

  * 水田・早川・和泉・吉村 (2013) 人工市場シミュレーションを用いた取引市場間におけるティックサイズと取引量の関係分析
  * 草田・水田・早川・和泉・吉村 (2014) 人工市場を用いたマーケットメーカーのスプレッドが市場出来高に与える影響の分析
  * 草田・水田・早川・和泉 (2015) 保有資産を考慮したマーケットメイク戦略が取引所間競争に与える影響:人工市場アプローチによる分析

