---
title: MarketShareMain UseCases 02
author: Takuma Torii
math: true
---

# MarketShareMain UseCases 02

本記事では，[tutorial/MarketShareMain](MarketShareMain) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．
草田ら (2015) が行ったマーケットメイカーの影響を調べる．


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
`MarketShare/config-02.json` の主要なパラメータをまとめる．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `MarketMakerAgent` | `netInterestSpread` | 0.02
| `Market-A` | `tickSize`    | 0.00001
| `Market-B` | `tickSize`    | 0.00001
| `Market-A` | `tradeVolume` | 90
| `Market-B` | `tradeVolume` | 10

出来高シェアの初期値は草田ら (2015) より，市場 A（`Market-A`）に 90%，市場 B（`Market-B`）に 10% とした．

以下の手順でグラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/MarketShare/config-02.json >output.dat
$ Rscript samples/MarketShare/plot.R output.dat output.png
```

以下に output.png を示す．
赤線が市場 A の市場価格の時系列，緑線が市場 B の市場価格の時系列，黒線がファンダメンタル価格の時系列である．

市場 B にのみ，高頻度取引を行うマーケットメイカーが介入する．
マーケットメイカーは仲値の上下に等間隔で売買両方の注文を出すが，注文価格の間隔はスプレッドの値に依存し，スプレッドが小さいほど間隔は狭くなる．
また，一般にマーケットメイカーは約定しやすい位置に注文をだすため，実質的に価格変動の上限下限を制御する機能を有する．
このため，図では，市場 B のほうが価格の変動が小さい．

![small](/tutorial/MarketShareMain.figs/fig01.png)


## マーケットメイカーのスプレッドの影響分析

草田ら (2015) は３つの要因，すなわち，(1) マーケットメイカの利益率（スプレッド），(2) マーケットのティックサイズ（最小価格単位），(3) 出来高シェアの初期値の関係を分析している．
本記事では，スプレッドの影響のみを調べる．
ティックサイズは十分に小さい値とした．
出来高シェアの初期値は草田ら (2015) より，市場 A（`Market-A`）に 90%，市場 B（`Market-B`）に 10% とした．

草田ら (2015) によれば，マーケットメイカーのスプレッドが小さくなるほど，出来高シェアを奪いとるまでに必要な時間が短くなる．
より具体的には，過去ある期間における市場 A の出来高を $N_A$ としたとき，市場 B の出来高シェア $P_B = N_B / (N_A + N_B)$ が，初期値 $P_B = 0.1$ から始めて，$P_B = 1$ に収束するまでの時間が短くなる．
シミュレーションの要は，$P_B$ は `FCNAgent` が市場 B に注文をだす確率として使用される点にある．
以下ではマーケットメイカーのスプレッド `netInterestSpread` を変化させた場合の収束時間を比較する．


## マーケットメイカーのスプレッドが小さい場合

**問.2**

> マーケットメイカーのスプレッドが 0.01 の場合をシミュレーションし，過去 100 ステップにおける市場 B の出来高シェア $P_B$ の時系列をグラフ化せよ．

**解説**

下記の通り，`MarketShare/config-02.json` を設定する．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `MarketMakerAgent` | `netInterestSpread` | 0.01

シミュレーションでは各時点 t の出来高を出力するにとどめ，「過去 100 ステップにおける市場 B の出来高シェア」は R の解析プログラムにより求めた．
`print()` メソッドについては[前記事](MarketShareMain)を参照してほしい．
また，R で出来高シェアを計算する方法については `/samples/MarketShare/plot-tradeshare.R` を参照してほしい．

シミュレーションを実行し，グラフを描画する手順は以下である．

```
$ ./a.out samples/MarketShare/config-02.json >output.dat
$ Rscript samples/MarketShare/plot-tradeshare.R output.dat output.png
```

以下に output.png を示す．
赤線は市場 B の出来高シェア（0 〜 1 の範囲）である．
初期の出来高シェア 0.1 から次第に増加しており，出来高シェアを奪っている（図は１標本であるため，ランによりバラつきがある）．

![small](/tutorial/MarketShareMain.figs/fig02.png)


## マーケットメイカーのスプレッドがさらに小さい場合

**問.3**

> マーケットメイカーのスプレッドが 0.0001 の場合をシミュレーションし，過去 100 ステップにおける市場 B の出来高シェア $P_B$ の時系列をグラフ化せよ．

**解説**

下記の通り，`MarketShare/config-02.json` を設定する．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `MarketMakerAgent` | `netInterestSpread` | 0.0001

シミュレーションを実行し，グラフを描画する手順は以下である．

```
$ ./a.out samples/MarketShare/config-02.json >output.dat
$ Rscript samples/MarketShare/plot-tradeshare.R output.dat output.png
```

以下に output.png を示す．
赤線は市場 B の出来高シェア（0 〜 1 の範囲）である．
問.2 と同様に，初期の出来高シェア 0.1 から次第に増加しており，出来高シェアを奪っている．
しかし，問.2 と比較して，より短い時間で出来高シェアを奪えている（図は１標本であるため，ランによりバラつきがある）．

![small](/tutorial/MarketShareMain.figs/fig03.png)

