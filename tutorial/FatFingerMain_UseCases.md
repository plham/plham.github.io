---
title: FatFingerMain UseCases
author: Takuma Torii
math: false
---

# FatFingerMain UseCases

本記事では，[tutorial/FatFingerMain](FatFingerMain) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．


## シミュレーションの実行とグラフ化

**問.1**

> プログラムをコンパイル・実行し，価格の時系列をグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/FatFinger/FatFingerMain.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行し，グラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/FatFinger/config.json >output.dat
$ Rscript samples/FatFinger/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．

このシミュレーションでは，図中 200 ステップの時点で誤発注ショックが発生しており，そのため，市場価格が突如 285 付近へ変化している．今回の誤発注では 1 万単位の売り注文が出されている．
そのため，この誤発注売り注文が「壁」となり，市場価格の上限を作り出している．

![small](/tutorial/FatFingerMain.figs/fig01.png)


**問.2**

> 問.1 の実行結果から，売り板，買い板にある注文を可視化せよ．

**解説**

`FatFingerMain` の `print()` メソッドには OrderBook の状態を出力するコードが書かれている．
具体的には以下である．

```x10
// samples/FatFinger/FatFingerMain.x10
public def print(sessionName:String) {
	super.print(sessionName);
	val markets = getMarketsByName("markets");
	for (market in markets) {
		market.getBuyOrderBook().dump();  /* WARNING: This dumps all orders in the orderbook!! */
		market.getSellOrderBook().dump(); /* WARNING: This dumps all orders in the orderbook!! */
	}
}
```

留意点として，`OrderBook#dump()` メソッドはすべての注文を出力するため，通常，出力データのサイズが巨大になる．
長期的なシミュレーションでは気をつけること．

売り板，買い板の可視化は `samples/FatFinger/book.R` で行える．
具体的には以下である．

```
$ Rscript samples/FatFinger/book.R output.dat output-book.png 0    # 0 はマーケットID
```

以下に output-book.png を示す．
赤丸はその価格に売り注文があることを示し，青丸は買い注文があることを示す．

さきほどの時系列と同じく，時点 200 で誤発注が生じた結果，最安売り気配値が 285 まで急落している．
この際，いくつかの買い注文と約定が成立し，買い板から注文が消滅していることが見てとれる．
また，その後も，誤発注で出された売り注文が「壁」として売り板に残り続けていることも見てとれる．

![full](/tutorial/FatFingerMain.figs/fig02.png)


