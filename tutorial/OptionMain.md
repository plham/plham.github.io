---
title: OptionMain
author: Takuma Torii
math: true
---

# OptionMain

本記事では，川久保（2015）が行ったオプション市場におけるボラティリティ・スマイルを再現するシミュレーションを作成することを通して，本ソフトウェアの使い方を説明する．

本記事で扱う内容：

  * オプション・マーケット
  * オプション・エージェント

関連するファイル：

  * `samples/Option/OptionMain.x10`
  * `samples/Option/OptionAgent.x10`
  * `samples/Option/agent/*`


## Preface

### オプション取引の概説

詳しくは[日本証券取引所の解説](http://www.jpx.co.jp/derivatives/options/)をご覧ください．

オプション市場ではある原資産（現物）を将来売買する「権利」を取引する（他方，将来売買する「約束」を取引する場合は先物取引と言われる[［参考］](http://www.jpx.co.jp/derivatives/options/outline/#heading_3)）．
そのため，オプション市場には，**将来買う権利**に関する取引「コール・オプション」と，**将来売る権利**に関する取引「プット・オプション」がある．
オプション市場の取引銘柄は，買う権利か売る権利か（**コール or プット**），将来のいつ取引する権利か（**満期**），いくらで取引する権利か（**権利行使価格**）という 3 つの属性によって，1 つ定まる．
具体的には，ある特定の原資産（現物）に対して，2 × 満期 種類数 × 行使価格 種類数 の数だけの派生銘柄が存在している．

<!-- ![small](/tutorial/OptionMain.figs/fig01.png) -->

オプション市場では将来売買する「権利」を取引するが，これは「約束」ではないため，満期が到来した時点で権利を**行使**するか**破棄**するかをトレーダは選択できる．
権利を行使した場合，権利の購入時に支払った金額と権利行使価格の差分がトレーダの利益となる．
一般に買い手（買う権利をもつトレーダ）は利益をえられる場合のみ権利を行使する（あるいは自動的に行使される）．
他方，売り手（売る権利をもつトレーダ）は買い手の権利行使に応じる義務をもち，買い手が権利を破棄した場合に差分だけの利益をえる．


### ボラティリティ・スマイル

オプション価格づけの数理モデルとして Black-Scholes モデルがある．
Black-Scholes モデルは，原資産価格，権利行使価格，満期までの残日数，ボラティリティなどをパラメータとして，オプション価格を算出する．
Black-Scholes モデルを，現在のオプション価格をパラメータに加え，ボラティリティを算出するよう変形することで，インプライド・ボラティリティと呼ばれる数量をえる．
インプライド・ボラティリティはトレーダのボラティリティに関する予想を反映した数量だといわれる．

オプション市場の構造を捉えるグラフのひとつに，ボラティリティ・サーフェイスがある（[Google 画像検索](https://www.google.co.jp/search?q=implied+volatility+surface&tbm=isch)）．
ボラティリティ・サーフェイスは 3 次元プロットで，X 軸「権利行使価格」，Y 軸「満期までの残日数」からなる平面上に，Z 軸「インプライド・ボラティリティ」をプロットして描かれる．
ただし，X 軸は 原資産価格 に最も近い 権利行使価格 が中心（ゼロの位置）となるよう標準化される（標準化された数値を moneyness という）．
これは 原資産価格 と 権利行使価格 の大小関係でトレーダの利益の有無が決まるためである．

Tompkins (2001) の論文では，現実の市場で観測されたボラティリティ・サーフェイスを可視化している．
論文の図のように，各 満期までの残日数 に関してみれば，正規化された権利行使価格を中心として，インプライド・ボラティリティはスマイル形状に変化することが知られている．
これを**ボラティリティ・スマイル**という．

<!-- ![small](/tutorial/OptionMain.figs/fig02.png) -->

この記事では，人工市場シミュレーションにおいてボラティリティ・スマイルを再現することを目指す．


## Compile & run

```
$ x10c++ samples/Option/OptionMain.x10
$ ./a.out samples/Option/config.json
```


## OptionMarket

`OptionMarket` はヨーロピアン型オプション市場の注文処理を実装している（[東証の解説](http://www.jpx.co.jp/derivatives/options/simulation/)）．
ヨーロピアン型では，満期を過ぎるまでは権利行使できない（他方，アメリカン型ではいつでも権利行使できる）．
また，言い換えると，満期到来時にすべての権利に関して利益が確定する．
この性質のため，現実のヨーロピアン型では，正の利益が確定している権利に関しては満期到来時に自動的に権利行使される場合もある．
ここでは，`OptionMarket` はこのタイプ（満期到来時に自動的に権利行使）の約定処理を実装している．
詳しくは，Plham のソースコードを参照してほしい．

  * `samples/Option/OptionMarket.x10`


## FCNOptionAgent

Frijns ら (2010) はオプション市場の統計的性質を人工市場シミュレーションで検証している．
Frijns ら (2010) はオプション銘柄を取引するエージェントとして，ファンダメンタリストとチャーチスト（いずれも将来のボラティリティを予想して意思決定する）を定義している．
ファンダメンタリストは将来のボラティリティがある速度（パラメータ）で無条件ボラティリティ（パラメータ）に近づくという予想のもとボラティリティを見積もる．
他方，チャーチストは原資産の過去ボラティリティ時系列のトレンドに基づき，そのトレンドが持続するという予想のもとボラティリティを見積もる．
さらに，ランダムにボラティリティの変動を予想するエージェントも加える．
詳しくは，Frijns ら (2010) の論文か，Plham のソースコードを参照してほしい．

  * `samples/Option/agent/FCNOptionAgent.x10`

この `FCNOptionAgent` は，Chiarella & Iori (2002) 方式（`FCNAgent`）をとり，ファンダメンタリスト，チャーチスト，ノイズの加重平均としてボラティリティを予想するよう実装されている．
本記事では Frijns ら (2010) に合わせ，ファンダメンタリスト，チャーチスト，ノイズトレーダ のうち，いずれかを戦略とするエージェントのみを用いるが，混合戦略をとるエージェントは用いない．


### ランダムエージェント

以下の実験では，売買行動をランダムに行う「ランダムエージェント」を用いる．
ランダムエージェントと呼ぶものは Chiarella & Iori (2002) のノイズトレーダと等しい．
オプションを取引するノイズエージェントはボラティリティをランダムに予想するが，これに対して，オプションを取引するランダムエージェントは価格予想と売買行動をランダムに行う．


## JSON configuration file

今回は Frijns ら (2010) が実データに照合して推定したパラメータを用いるため，オプションを取引するエージェントのパラメータは変更しない．
その代わりに，エージェントの種類を変更し，それがボラティリティ・サーフェイスに及ぼす影響を観察する．
以下に，`samples/Option/config.json` の一部を示す．

```json
// samples/Option/config.json
"simulation": {
	"markets": ["UnderlyingMarket", "OptionMarketCluster"],
	"agents": ["UnderFCNAgents", "FundamentalistOptionAgents", "ChartistOptionAgents", "NoiseOptionAgents"],
	"--agents": ["UnderRandomAgents", "FundamentalistOptionAgents", "ChartistOptionAgents", "NoiseOptionAgents"],
	"--agents": ["UnderFCNAgents", "OptionRandomAgents"],
	"--agents": ["UnderRandomAgents", "OptionRandomAgents"],
}
```

ここで，

| `"UnderFCNAgents"` | 原資産のみを取引する FCNAgent (Chiarella & Iori 2002)
| `"UnderRandomAgents"` | 原資産のみを取引するランダムエージェント (Chiarella & Iori 2002)
| `"FundamentalistOptionAgents"` | オプションを取引するファンダメンタリスト (Frijns 2010)
| `"ChartistOptionAgents"` | オプションを取引するチャーチスト (Frijns 2010)
| `"NoiseOptionAgents"` | オプションを取引するノイズエージェント (Frijns 2010)
| `"OptionRandomAgents"` | オプションを取引するランダムエージェント (Chiarella & Iori 2002)

比較実験設定の切り換えは `"--agents"` を書き換えて行う．
シミュレーションで使われるのは `"agents"` のみで，他の `"--agents"` はすべて無視される．
比較実験を行う場合はいずれかひとつのみ `"agents"` に変更すればよい．


## Simulations

次に，このプログラムを使って，シミュレーションを実行してみよう（[こちら](/tutorial/OptionMain_UseCases01)）．
さらに，エージェントを拡張して，オプション取引戦略が与える影響を分析してみよう（[こちら](/tutorial/OptionMain_UseCases02)）．


## Related works

  * 川久保 (2015) 連成型人工市場モデルの構築によるデリバティブ市場の構造分析（東京大学 学位論文）
  * Frijns, Lehnert, Zwinkels (2010) Behavioral heterogeneity in the option market
  * Tompkins (2001) Implied volatility surfaces: uncovering regularities for options on financial futures


