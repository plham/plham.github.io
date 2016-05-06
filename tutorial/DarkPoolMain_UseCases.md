---
title: DarkPoolMain UseCases
author: Takuma Torii
math: false
---

# DarkPoolMain UseCases

本記事では，[tutorial/DarkPoolMain](DarkPoolMain) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．

水田ら (2014) は統計的な結果のみを議論しているが，本記事では価格時系列に現れる傾向を観察するに留める．
興味のある読者は複数のパラメータ設定を行い，水田らの結果を再現してみるとよい．


## シミュレーションの実行とグラフ化

**問.1**

> プログラムをコンパイル・実行し，価格の時系列をグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/DarkPool/DarkPoolMain.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行する．
`DarkPool/config.json` の主要なパラメータをまとめる．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `DarkPoolFCNAgent` | `darkPoolChance` | 0.1

以下の手順でグラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/DarkPool/config.json >output.dat
$ Rscript samples/DarkPool/plot.R output.dat output.png
```

以下に output.png を示す．
赤線が取引価格の時系列（ダークプールとリット市場のうち，取引があったほうの価格），黄線（明るい色）がリット市場の市場価格の時系列，青線（暗い色）がダークプールの市場価格の時系列（正確には最新取引価格の時系列），黒線がファンダメンタル価格の時系列である．

このシミュレーションではダークプールの利用率は 10% （`darkPoolChance` = 0.1）であるため，ダークプールで取引が生じる確率は概ね 10% 程度である．
取引が生じないためダークプールの価格時系列に関しては，価格が動いていない期間（青線横ばい）が目立つ（注意：ここでいう「ダークプールの価格」はトレーダが観察できるものではない）．

![small](/tutorial/DarkPool.figs/fig01.png)


# ダークプールの利用率が高い場合

**問.2**

> ダークプールの利用率が 50% の場合をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

下記の通り，`DarkPool/config.json` を設定する．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `DarkPoolFCNAgent` | `darkPoolChance` | 0.5

以下の手順でグラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/DarkPool/config.json >output.dat
$ Rscript samples/DarkPool/plot.R output.dat output.png
```

以下に output.png を示す．
ダークプールの利用率が 50% であるため，問.1（10% の場合）と比較すると，ダークプールでの価格更新が頻繁に生じている（青線横ばいが減少）（注意：トレーダが観察できる量ではない）．
他方，ダークプールでの取引頻度が増すと同時に，リット市場での取引頻度は少なくなるため，今度はリット市場の価格更新が遅くなっている（黄線横ばいが増加）．
取引価格（赤線）に関しても，横ばいの期間が増え，価格の変動は小さくなっているが，表面的には取引が活発でない印象をうける．

![small](/tutorial/DarkPool.figs/fig02.png)



# ダークプールの利用率がさらに高い場合

**問.3**

> ダークプールの利用率が 90% の場合をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

下記の通り，`DarkPool/config.json` を設定する．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `DarkPoolFCNAgent` | `darkPoolChance` | 0.9

以下の手順でグラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/DarkPool/config.json >output.dat
$ Rscript samples/DarkPool/plot.R output.dat output.png
```

以下に output.png を示す．
ダークプールの利用率が 90% であるため，リット市場での取引頻度は少なく，リット市場の価格更新は希にしか起こらない（黄線横ばい）．
このことは転じて，ダークプールでの取引が活発であるにもかかわらず，ダークプールでの取引価格（すなわちリット市場の仲値）がほとんど変化しない（青線横ばい）という結果を引き起こしている．
取引価格（赤線）に関しても，ほとんど横ばいの期間となり，実際にはダークプールで取引が活発なのだが，表面的には取引がさらに活発でない印象をうける．

![small](/tutorial/DarkPool.figs/fig03.png)

このように，ダークプールの利用は価格の変動を抑制する働きをもつことが予想される反面，市場が活発でないようにみえる，ファンダメンタル価格への復帰を遅らせるなど副作用をもたらす可能性も示唆される．
水田ら (2014) は価格発見機能という観点からダークプールを評価し，その効率性を論じている．


