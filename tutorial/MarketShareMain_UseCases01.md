---
title: MarketShareMain UseCases 01
author: Takuma Torii
math: true
---

# MarketShareMain UseCases 01

本記事では，[tutorial/MarketShareMain](MarketShareMain) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．
水田ら (2013) が行ったティックサイズの影響を調べる．


## シミュレーションの実行とグラフ化

**問.1**

> プログラムをコンパイル・実行し，価格の時系列をグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/MarketShare/MarketShareMain.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行する．
`MarketShare/config-01.json` の主要なパラメータをまとめる．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `Market-A` | `tickSize`    | 5.0
| `Market-B` | `tickSize`    | 1.0
| `Market-A` | `tradeVolume` | 90
| `Market-B` | `tradeVolume` | 10

出来高シェアの初期値は水田ら (2013) より，市場 A（`Market-A`）に 90%，市場 B（`Market-B`）に 10% とした．

以下の手順でグラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/MarketShare/config-01.json >output.dat
$ Rscript samples/MarketShare/plot.R output.dat output.png
```

以下に output.png を示す．
赤線が市場 A の市場価格の時系列，緑線が市場 B の市場価格の時系列，黒線がファンダメンタル価格の時系列である．

取引値の初期値 300 円に対して，市場 A のティックサイズは 5 円刻み，市場 B のティックサイズは 1 円刻みである． 
FCNAgent は出来高シェアに比例する確率で市場を選択するため，序盤は初期シェア 90 % の市場 A のほうが活発に取引されている．
しかし，次第に市場 A の取引頻度は低下し，ティックサイズの小さい市場 B の取引が活発になっている．

![small](/tutorial/MarketShareMain.figs/fig04.png)


## ティックサイズが出来高シェアに与える影響

次の図は，上記の価格時系列と同じ試行からえられた，市場 B の出来高シェアの推移を示している．
図から，出来高シェアは次第に市場 B に移り，時点 1000 付近ではすでに 100% のシェアを獲得している．
これは市場 A ではほとんど取引されていないことを意味するが，実際，市場 A の価格時系列は変化しておらず，取引がほとんど生じていない．

![small](/tutorial/MarketShareMain.figs/fig05.png)


水田ら (2013) によれば，ティックサイズが小さくなるほど約定が頻繁に起こり，したがって出来高シェアを奪いとるまでに必要な時間が短くなる．
シミュレーションの要は，$P_B$ は `FCNAgent` が市場 B に注文をだす確率として使用される点にある．
約定頻度はエージェントの予想価格（= 注文価格）と最良気配値との関係で決まるが，ティックサイズが大きすぎると，エージェントの予想価格が相対する最良気配値を越える機会が少なくなる．
そのため，エージェントはティックサイズの大きい市場 A において注文をだす誘因を次第に失い，ティックサイズの小さい，取引頻度の活発な市場 B を次第に好むようになる．
詳しくは水田ら (2013) を参照してほしい．


## ティックサイズの差が小さい場合

**問.2**

> 市場 A のティックサイズ = 2 の場合をシミュレーションし，過去 100 ステップにおける市場 B の出来高シェア $P_B$ の時系列をグラフ化せよ．

**解説**

下記の通り，`MarketShare/config-01.json` を設定する．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `Market-A` | `tickSize`    | 2.0

シミュレーションでは各時点 t の出来高を出力するにとどめ，「過去 100 ステップにおける市場 B の出来高シェア」は R の解析プログラムにより求めた．
`print()` メソッドについては[前記事](MarketShareMain)を参照してほしい．
また，R で出来高シェアを計算する方法については `/samples/MarketShare/plot-tradeshare.R` を参照してほしい．

シミュレーションを実行し，グラフを描画する手順は以下である．

```
$ ./a.out samples/MarketShare/config-01.json >output.dat
$ Rscript samples/MarketShare/plot-tradeshare.R output.dat output.png
```

以下に output.png を示す．
赤線は市場 B の出来高シェア（0 〜 1 の範囲）である．
初期の出来高シェア 0.1 から次第に増加しており，出来高シェアを奪っている（図は１標本であるため，ランによりバラつきがある）．
しかし，出来高シェアを奪う速度は市場 A のティックサイズ = 5 の場合に比べて遅い．

![small](/tutorial/MarketShareMain.figs/fig07.png)

