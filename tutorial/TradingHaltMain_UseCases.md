---
title: TradingHaltMain UseCases
author: Takuma Torii
math: false
---

# TradingHaltMain UseCases

本記事では，[tutorial/TradingHaltMain](TradingHaltMain) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．


## シミュレーションの実行とグラフ化

**問.1**

> プログラムをコンパイル・実行し，価格の時系列をグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/TradingHaltMain/TradingHaltMain.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行し，グラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/TradingHalt/config.json >output.dat
$ Rscript samples/TradingHalt/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．
価格時系列の背後が薄い青色の部分は取引停止の最中を表す．

このシミュレーションでは，図中 100 ステップの時点でファンダメンタル価格のショックが発生しており，そのため，ファンダメンタル価格が突如 270 へ変化している（10% 急落）．
その後，ファンダメンタル重視型の FCN エージェントが取引を活発化させ，市場価格は急落したファンダメンタル価格を追いかけるように下落してゆく．
そして，価格の下落（変化率）が発動閾値 5% × n （n は発動回数）を超えたところで取引停止が発動している．

![small](/tutorial/TradingHaltMain.figs/fig01.png)



## 取引停止のパラメータ

取引停止には２つのパラメータ，停止期間と発動閾値，がある．
停止期間の影響度は FCN エージェントの時間窓長との相対的な長さと関係することが予想される．
また，発動閾値の影響度はショックの大きさと関係することが予想される．

以下は，問.1 のシミュレーションで用いた `TradingHalt/config.json` の設定値である．

| Section       | Parameter           | Value
|---------------|---------------------|--------
| `TradingHalt` | `haltingTimeLength` | 100
| `TradingHalt` | `triggerChangeRate` | 0.05


## 取引停止の停止期間を変化させた場合

**問.2**

> 停止期間を長くした場合の取引をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

停止期間は，JSON ファイル `TradingHalt/config.json` では `haltingTimeLength` の箇所を書き換えることで変更できる．
JSON ファイルでは FCN エージェントの時間窓長が 100〜200 の一様乱数となっているので，今回のシミュレーションでは停止期間を 200 へ変更する．

```json
{    "haltingTimeLength": 200    }
```

`TradingHalt/config.json` の主要なパラメータをまとめる．

| Section       | Parameter           | Value
|---------------|---------------------|--------
| `TradingHalt` | `haltingTimeLength` | 200
| `TradingHalt` | `triggerChangeRate` | 0.05

以下の手順でグラフを描画した（出力ファイルは output.png）．

```
$ ./a.out samples/TradingHalt/config.json >output.dat
$ Rscript samples/TradingHalt/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．
価格時系列の背後が薄い青色の部分は取引停止の最中を表す．
問.1 の結果と比較すると，各回の停止期間が長くなっていることがわかる．

![small](/tutorial/TradingHaltMain.figs/fig02.png)


## 取引停止の発動閾値を変化させた場合

**問.3**

> 発動閾値を小さく場合の取引をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

発動閾値は，JSON ファイル `TradingHalt/config.json` では `triggerChangeRate` の箇所を書き換えることで変更できる．
JSON ファイルではショックの大きさ（ファンダメンタル価格の変化率）が −10% となっているので，今回のシミュレーションでは発動閾値を 0.02 へ変更する．

```json
{    "triggerChangeRate": 0.02    }
```

`TradingHalt/config.json` の主要なパラメータをまとめる．

| Section       | Parameter           | Value
|---------------|---------------------|--------
| `TradingHalt` | `haltingTimeLength` | 100
| `TradingHalt` | `triggerChangeRate` | 0.02

以下の手順でグラフを描画した（出力ファイルは output.png）．

```
$ ./a.out samples/TradingHalt/config.json >output.dat
$ Rscript samples/TradingHalt/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．
価格時系列の背後が薄い青色の部分は取引停止の最中を表す．
問.1 の結果と比較すると，より小さな価格変化でも取引停止が発動するため，発動回数が多くなっていることがわかる．

![small](/tutorial/TradingHaltMain.figs/fig03.png)

