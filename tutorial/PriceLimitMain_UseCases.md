---
title: PriceLimitMain UseCases
author: Takuma Torii
math: false
---

# PriceLimitMain UseCases

本記事では，[tutorial/PriceLimitMain](PriceLimitMain) で解説したプログラムを使い，いくつかのパラメータを変えて，実際にシミュレーションを行う．


## シミュレーションの実行とグラフ化

**問.1**

> プログラムをコンパイル・実行し，価格の時系列をグラフ化せよ．

**解説**

コンパイルは以下の手順で行う．

```
$ x10c++ samples/PriceLimitMain/PriceLimitMain.x10
```

コンパイルに成功すると実行ファイル `a.out` が生成される．

次に，シミュレーションを実行し，グラフを描画する（出力ファイルは output.png）．

```
$ ./a.out samples/PriceLimit/config.json >output.dat
$ Rscript samples/PriceLimit/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．

このシミュレーションでは，値幅をファンダメンタル価格 300 から上下 5% に設定している．
価格は最初ランダムに変動するが 200 ステップ付近で値幅の上限に到達し，その後しばらくの間，価格が上限（天井）に張り付き変動しなくなっている．
その後，価格は天井から離れ始めるが，今後は 400 ステップ付近からしばらくの間，価格が下限（床）に張り付き変動しなくなっている．

![small](/tutorial/PriceLimitMain.figs/fig01.png)



## 値幅制限のパラメータ

値幅制限には１つのパラメータ，値幅（発動閾値），がある．
値幅制限の影響度は FCN エージェントのトレンド追従型（チャート分析の成分）と関係することが予想される．
そこで，以下の練習問題では，値幅制限のパラメータはそのままに，トレンド追従型の影響力を変化させてみる．

以下は，問.1 のシミュレーションで用いた `PriceLimit/config.json` の設定値である．

| Section       | Parameter           | Value
|---------------|---------------------|--------
| `PriceLimit`  | `triggerChangeRate` | 0.05
| `FCNAgent`    | `chartWeight`       | `{"expon": [0.0]}`


## トレンド追従型が多い場合の値幅制限

**問.2**

> トレンド追従型の影響を大きくした場合の取引をシミュレーションし，価格の時系列をグラフ化せよ．

**解説**

トレンド追従型の影響力は，JSON ファイル `PriceLimit/config.json` では `chartWeight` の箇所を書き換えることで変更できる．
今回のシミュレーションでは `chartWeight` の平均値を 1.0 へ変更する．

```json
{    "chartWeight": {"expon": [1.0]}    }
```

`PriceLimit/config.json` の主要なパラメータをまとめる．

| Section       | Parameter           | Value
|---------------|---------------------|--------
| `PriceLimit`  | `triggerChangeRate` | 0.05
| `FCNAgent`    | `chartWeight`       | `{"expon": [1.0]}`

以下の手順でグラフを描画した（出力ファイルは output.png）．

```
$ ./a.out samples/PriceLimit/config.json >output.dat
$ Rscript samples/PriceLimit/plot.R output.dat output.png
```

以下に output.png を示す．
赤線は市場価格の時系列，黒線がファンダメンタル価格の時系列である．
問.1 の結果と比較すると，値幅の上限（下限）に張り付く期間が長くなっていることがわかる．
その理由としては，チャート成分を大きくしたことでノイズ成分の影響力が弱まり，偶然に剥がれる可能性が低下したことがあげられる．
また，今回の設定では，ファンダメンタル成分が小さいため，ファンダメンタル価格への回帰が生じにくい．

![small](/tutorial/PriceLimitMain.figs/fig02.png)

今回の設定では，上記の理由で，500 ステップ以内に価格が値幅の上限下限に張り付かないこともある．
その場合は，シミュレーションのステップ数を増やして（5000 ステップ程度）結果を観察することで，問.1 の場合よりも，値幅の上限（下限）に張り付く期間が長くなることを確認できるだろう．


