---
title: CI2002Main UseCases
author: Takuma Torii
math: false
---

# CI2002Main UseCases

本記事では，[tutorial/CI2002Main](CI2002Main) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．


## シミュレーションの実行とグラフ化

**問.1**

> プログラムをコンパイル・実行し，価格の時系列をグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/CI2002Main/CI2002Main.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行する．
`CI2002/config.json` の主要なパラメータをまとめる．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `FCNAgent` | `fundamentalWeight` | 1.0
| `FCNAgent` | `chartWeight`       | 0.0
| `FCNAgent` | `noiseWeight`       | 1.0

以下の手順でグラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/CI2002/config.json >output.dat
$ Rscript samples/CI2002/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．

![small](/tutorial/CI2002Main.figs/fig01.png)



## ファンダメンタル重視型の多い場合

**問.2**

> ファンダメンタル重視型の多い場合の取引をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

ファンダメンタル重視の度合いは `fundamentalWeight` が，`chartWeight` および `noiseWeight` に対して，相対的にどの程度大きいかによって決まる．
これらの大小関係は JSON ファイルの該当箇所を書き換えることで変更できる．

問.2 の場合，ファンダメンタル重視を高めたいので，`"FCNAgent"` に関する記述のうち，

```json
{  "fundamentalWeight": {"expon": [1.0]}  }
```

の数値を変更すればよい．

実行結果をグラフ化する．
実行にあたりパラメータは以下とし，`CI2002/config.json` を修正した．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `FCNAgent` | `fundamentalWeight` | 2.0
| `FCNAgent` | `chartWeight`       | 0.0
| `FCNAgent` | `noiseWeight`       | 1.0

その上で，以下の手順でグラフを描画した（出力ファイルは output.png）．

```
$ ./a.out samples/CI2002/config.json >output.dat
$ Rscript samples/CI2002/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．
問.1 の結果と比較すると，市場価格がファンタメンタル価格付近で小さく変動しており，ファンダメンタル重視の影響が強くなっていることがわかる．

![small](/tutorial/CI2002Main.figs/fig02.png)



## チャート重視型の多い場合

**問.3**

> チャート重視型の多い場合の取引をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

問.2 と本質的に同じだが，JSON ファイル中の `chartWeight` の数値を変更する点が異なる．

実行結果をグラフ化するにあたり，パラメータを以下のように `CI2002/config.json` を修正した．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `FCNAgent` | `fundamentalWeight` | 1.0
| `FCNAgent` | `chartWeight`       | 0.5
| `FCNAgent` | `noiseWeight`       | 1.0

その上で，以下の手順でグラフを描画した（出力ファイルは output.png）．

```
$ ./a.out samples/CI2002/config.json >output.dat
$ Rscript samples/CI2002/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．
問.1，問.2 の結果と比較すると，市場価格の時系列がより長期的なトレンドをもつためファンダメンタル価格への回帰頻度が少なくなり，チャート重視型の影響が強くなっていることがわかる．

![small](/tutorial/CI2002Main.figs/fig03.png)


