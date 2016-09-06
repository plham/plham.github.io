---
title: OptionMain UseCases 02
author: Takuma Torii
math: false
---

# OptionMain UseCases 02

本記事では，[tutorial/OptionMain](OptionMain) で解説したプログラムを拡張し，さまざまなオプション取引戦略を加えたときのボラティリティ・サーフェイスへの影響を調べる．
具体的には，川久保 (2015) で取り上げられた下記の戦略を実装した．
詳細は，川久保 (2015) を参照されたい．

  * `StraddleOptionAgent` ... ストラドル戦略
  * `StrangleOptionAgent` ... ストラングル戦略
  * `SyntheticOptionAgent` ... シンセティック戦略
  * `DeltaHedgeOptionAgent` ... デルタヘッジ戦略
  * `ExCoverDashOptionAgent` ... 満期日近辺で原資産取引する戦略
  * `PutCallParityOptionAgent` ... プット・コール・パリティに基づく戦略
  * `LeverageFCNOptionAgent` ... 効用関数に基づきレバレッジをかける戦略
  * `ProspectFCNOptionAgent` ... プロスペクト理論に基づく戦略


## StraddleOptionAgent

同じ満期で同じ権利行使価格のコール・オプションとプット・オプションを組み合わせる取引戦略をいう．
実装では，同じ原資産，同じ満期，同じ権利行使価格の銘柄ペアをランダムに選択し，原資産のボラティリティが十分に高い場合に取引を行う．
この戦略は

  * `samples/Option/agent/StraddleOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "StraddleOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-straddle.png)


## StrangleOptionAgent

同じ満期で**異なる**権利行使価格のコール・オプションとプット・オプションを組み合わせる取引戦略をいう．
実装では，同じ原資産，同じ満期，**異なる**権利行使価格の銘柄ペアをランダムに選択し，原資産のボラティリティが十分に高い場合に取引を行う．
この戦略は

  * `samples/Option/agent/StrangleOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "StrangleOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-strangle.png)


## SyntheticOptionAgent

コール・オプションとプット・オプションを組み合わて合成ポジションを作り，この合成ポジションの価格と先物ポジション（このシミュレーションでは任意の原資産を含む）の価格を比較して裁定取引（安い方を買い，高い方を売る）を行う取引戦略をいう．
取引戦略には 2 つの可能性があり，

  * コンバージョン ... 合成ポジション → 売り，先物ポジション → 買い
  * リバーサル ... 合成ポジション → 買い，先物ポジション → 売り

と呼ばれる．
この戦略は

  * `samples/Option/agent/SyntheticOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "SyntheticOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-synthetic.png)

両スマイルではなく，片スマイルがみられる．


## DeltaHedgeOptionAgent

Black-Scholes モデルに含まれるデルタ（オプションのリスク指標）を利用した取引戦略をいう．
詳しくは他の文献を参照されたい．
この戦略は

  * `samples/Option/agent/DeltaHedgeOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "DeltaHedgeOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-deltahedge.png)


## ExCoverDashOptionAgent

通常はデルタヘッジを行うが，満期日の数日前からはリスク回避の目的で戦略的に原資産を売買する取引戦略をいう．
詳しくは川久保 (2015) を参照されたい．
この戦略は

  * `samples/Option/agent/ExCoverDashOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "ExCoverDashOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-excoverdash.png)


## PutCallParityOptionAgent

プット・コール・パリティと知られるオプション銘柄間に成立する価格の関係式に基づき，実際の取引価格の関係式からの乖離（裁定機会）を利用する取引戦略をいう．
詳しくは他の文献を参照されたい．
この戦略は

  * `samples/Option/agent/PutCallParityOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "PutCallParityOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-putcallparity02.png)

満期日近辺でのインプライド・ボラティリティの広がりがみられる．


## LeverageFCNOptionAgent

各資産の価値を，効用関数に基づき重みづけし，その重みづけされた価値に基づく取引戦略である．
効用関数などの詳細は川久保 (2015) を参照されたい．
シミュレーションでは効用最大の資産のみを取引させる．
この戦略は

  * `samples/Option/agent/LeverageFCNOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "LeverageFCNOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-leverage.png)


## ProspectFCNOptionAgent

プロスペクト理論に基づき，各資産の価値を重みづけし，その重みづけされた価値に基づく取引戦略である．
実装では，木村 (2006) で提案された価格づけを採用している（ただし 1 資産のみを考慮する）．
詳しくは木村 (2006) を参照されたい．
この戦略は

  * `samples/Option/agent/ProspectFCNOptionAgent.x10`

に実装されている．

シミュレーションを行うため，`config.json` の箇所を書き換える．

```json
"simulation": {
    "agents": ["UnderFCNAgents", "FCNOptionAgentGroup", "ProspectFCNOptionAgents"]
}
```

この設定でえられたボラティリティ・サーフェイスを以下に示す．

![small](/tutorial/OptionMain.figs/figXX-prospect.png)


## References

  * 川久保 (2015) 連成型人工市場モデルの構築によるデリバティブ市場の構造分析（東京大学 学位論文）
  * 木村 (2006) プロスペクト理論を用いたオプション価格評価（早稲田大学 修士論文）
  * Frijns, Lehnert, Zwinkels (2010) Behavioral heterogeneity in the option market
  * Tompkins (2001) Implied volatility surfaces: uncovering regularities for options on financial futures

