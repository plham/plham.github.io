---
title: Fundamentals
author: Takuma Torii
math: true
---

# Fundamentals

本記事ではシミュレーションで，ファンダメンタル価格（理論価格）を制御する仕方を述べる．
ファンダメンタル価格は[多変量幾何ブラウン運動](https://en.wikipedia.org/wiki/Geometric_Brownian_motion#Multivariate_version)として生成される．

関連するファイル：

  * `plham.Fundamentals`
  * `plham.util.MultiGeomBrownian`
  * `plham.main.BaseMain`


## 多変量幾何ブラウン運動

多変量幾何ブラウン運動の各変数は以下のパラメータをもつ．
カッコ内に JSON ファイルで用いるキーワードとともに示す．

  * 初期価格（`fundamentalPrice`）
  * ドリフト項（`fundamentalDrift`）
  * ボラティリティ項（`fundamentalVolatility`）
  * 価格相関行列（`fundamentalCorrelations`）

ドリフト項は 0.0 のとき，平均回帰する時系列（トレンドなし）を生み出すが，非ゼロの値をとるとき，上昇トレンド（正），下落トレンド（負）を生みだす．

ボラティリティ項（非負）は 0.0 のとき，時系列は定数となるが，非ゼロの値をとるとき，価格時系列にバラつきを生み出す．

価格相関行列は文字通り，各変数に対応する銘柄ペアの価格時系列の相関を定める．


## JSON ファイルでの指定

### 初期価格，ドリフト項，ボラティリティ項

最初の３項目

  * 初期価格（`fundamentalPrice`）
  * ドリフト項（`fundamentalDrift`）
  * ボラティリティ項（`fundamentalVolatility`）

はマーケットの記述に組み込む（約束事）．
たとえば，以下のように書く．

```json
    "market-1": {
        "class": "Market",
        "marketPrice": 300.0,
        "outstandingShares": 25000,
        "fundamentalPrice": 300.0,
        "fundamentalDrift": 0.0,
        "fundamentalVolatility": 0.001
    }
```

**補足**
`Fundamentals` は `Main` でインスタンス化されるが，マーケットの定義の一部として上記３項目が宣言されることを前提としている．
`Main` では上記３項目についてデフォルト値を設けており，上記３項目の宣言は欠落してもよい．
`"fundamentalPrice"` のデフォルト値は `"marketPrice"` であり，
`"fundamentalDrift"` と `"fundamentalVolatility"` のデフォルト値はともに `0.0` である．

**注意**
マーケット初期化時にファンダメンタル価格の初期値を渡す必要があるが，これは `Fundamentals` インスタンスの初期値とは独立している．
`samples/CI2002/CI2002Main.x10` ではファンダメンタル価格の初期値を `marketPrice` で決め打ちにしているが，`samples/ShockTransfer/ShockTransferMain.x10` では `"fundamentalPrice"` のデフォルト値を `"marketPrice"` とするよう X10 コードが書かれているので，参考にしてほしい．


### 価格相関行列

価格相関行列（`fundamentalCorrelations`）はマーケットペアごとに指定する必要がある．
仮に，３つのマーケットがあるとし，

  * `market-1` と `market-2` の相関を  0.9
  * `market-1` と `market-3` の相関を -0.1

と設定したいとしよう．
以下のように行う．

```json
	"simulation": {
		"markets": ["market-1"],
		"agents": ["agents-1"],
        "fundamentalCorrelations": {
            "pairwise": [
                ["market-1", "market-2",  0.9],
                ["market-1", "market-3", -0.1]
            ]
        },
		"sessions": []
	}
```

**補足**
`"fundamentalCorrelations"` の宣言そのものが欠落してもよい．
`Main` では，相関係数のデフォルト値を設けており，未指定のマーケットペアの相関は 0.0 となる（ただし，相関の定義上，自己相関は必ず 1.0 となる）．
これは `"pairwise": []` と等価である．


