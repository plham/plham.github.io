---
title: CI2002Main UseCases02
author: Takuma Torii
math: false
---

# CI2002Main UseCases

本記事では，[tutorial/CI2002Main](CI2002Main) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．
とくに，ファットテールやボラティリティ・クラスタリングと呼ばれる価格時系列がもつ統計的性質を調べる．

金融に関する理論には，利益率（rate of return）あるいは価格の変動率が正規分布に従うことを前提としたものが多い．
しかし，現実のデータでは，利益率の分布は正規分布に従わないことが指摘されている．
正規分布からの逸脱を示す証拠として，ファットテールやボラティリティ・クラスタリングと呼ばれる手法が用いられる．
ファットテールの手法を用いると，利益率の分布の裾が正規分布の裾よりも厚いことを確認できるが，これは大きな利益率（価格変動）が正規分布よりも高い確率で発生することを示している．
他方，ボラティリティ・クラスタリングの手法では，利益率の絶対値の自己相関（時間ラグに対する自己相関の強さ）を調べるが，利益率の分布の裾が厚い場合ほどより長期的ラグに対しても自己相関が高くなることが確認される．
これは次時点の市場価格がより長期的な過去の価格に依存している（長期記憶性）ことを示している．

本記事では，[tutorial/CI2002Main](CI2002Main) を用い，エージェントの取引戦略を変化させたときに，ファットテールやボラティリティ・クラスタリングの分析結果がどう変化するかを調べる．

統計的性質を分析するため，シミュレーションの実行期間を 60000 ステップと大きめに設定する（うち最初の 10000 ステップは使わない）．
また，エージェントの時間窓長を \[100, 500] と大きめにし，エージェント数を 1000 体と多様性をもたせる．
具体的には，JSON ファイルに以下の変更を行う．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `simulation/sessions[1]` | `iterationSteps`  | 60000
| `FCNAgent` | `numAgents`         | 1000
| `FCNAgent` | `timeWindowSize`    | \[100, 500]
| `FCNAgent` | `orderMargin`       | `normal`
 


## シミュレーションの実行とグラフ化

**問.4**

> ファンダメンタル重視型，チャート重視型のどちらも少ない場合をシミュレーションし，ファットテール および ボラティリティ・クラスタリングをグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/CI2002/CI2002Main.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行する．
前述の変更点に加えて，`CI2002/config.json` の主要なパラメータをまとめる．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `FCNAgent` | `fundamentalWeight` | 0.0
| `FCNAgent` | `chartWeight`       | 0.0
| `FCNAgent` | `noiseWeight`       | 1.0

まず，シミュレーションを実行する．

```
$ ./a.out samples/CI2002/config.json >output.dat
```

以下の手順でファットテールのグラフを描画する（出力ファイルは fattail.png）．

```
$ Rscript samples/CI2002/fattail.R output.dat fattail.png
```

以下の手順でボラティリティ・クラスタリングのグラフを描画する（出力ファイルは volcluster.png）．

```
$ Rscript samples/CI2002/volcluster.R output.dat volcluster.png
```

以下に fattail.png を示す．
赤点は利益率の密度分布（シミュレーション結果から推定），黒線が同じ標準偏差をもつ正規分布である．
図では分布の裾を強調するため，縦軸は対数軸になっている．
もし利益率が正規分布に従うならば，赤点は黒線と重なるはずである．
しかし，分布の裾でわずかに乖離しており，正規分布に従わないことが示唆される．

![small](/tutorial/CI2002Main.figs/fig04a.png)

次に，volcluster.png を示す．
図は利益率の絶対値の自己相関プロットで，横軸は時間ラグである．
図から，自己相関はラグが大きくなるにつれ比較的早く減衰することがわかる（問.5 と比較してほしい）．
これは価格時系列が長期的依存性をそれほどもって**いない**ことを示している．

![small](/tutorial/CI2002Main.figs/fig04b.png)

ファットテールにおける分布の裾（大きな価格変動）や，ボラティリティ・クラスタリングにおける長期的自己相関は，市場に参加するチャート重視型の割合が多いほど観察されやすいことが知られている．


## チャート重視型の多い場合

**問.5**

> チャート重視型の多い場合の取引をシミュレーションし，ファットテール および ボラティリティ・クラスタリングをグラフ化せよ．

**解説**

前述の変更点に加えて，`CI2002/config.json` の主要なパラメータをまとめる．

| Section    | Parameter           | Value
|------------|---------------------|--------
| `FCNAgent` | `fundamentalWeight` | 1.0
| `FCNAgent` | `chartWeight`       | 10.0
| `FCNAgent` | `noiseWeight`       | 1.0

まず，シミュレーションを実行する．

```
$ ./a.out samples/CI2002/config.json >output.dat
```

以下の手順でファットテールのグラフを描画する（出力ファイルは fattail.png）．

```
$ Rscript samples/CI2002/fattail.R output.dat fattail.png
```

以下の手順でボラティリティ・クラスタリングのグラフを描画する（出力ファイルは volcluster.png）．

```
$ Rscript samples/CI2002/volcluster.R output.dat volcluster.png
```

以下に fattail.png を示す．
赤点は利益率の密度分布（シミュレーション結果から推定），黒線が同じ標準偏差をもつ正規分布である．
図では分布の裾を強調するため，縦軸は対数軸になっている．
問.4 のグラフと比較してわかるとおり，利益率の分布の裾がより分厚くなり，正規分布からの逸脱は一段と顕著になっている．

![small](/tutorial/CI2002Main.figs/fig05a.png)

次に，volcluster.png を示す．
図は利益率の絶対値の自己相関プロットで，横軸は時間ラグである．
問.4 のグラフと比較すると，時間ラグが大きくなっても自己相関は大きく維持されている．
これは価格時系列が強い長期的依存性をもって**いる**ことを示している．

![small](/tutorial/CI2002Main.figs/fig05b.png)
