---
title: JSON for Main
author: Takuma Torii
math: false
---

# JSON for Main

JSON（JavaScript Object Notation）は軽量のデータ交換フォーマットである．
詳しくは [www.json.org](http://www.json.org/json-ja.html) を参照してほしい．

本記事ではシミュレーション設定を記述する JSON ファイルの仕様を解説する．
この仕様は `Main` で想定されるものである．
また，２つの拡張機能，継承（`extends`）とグループ化，および，キーワード・チェインという階層関係の定義方法について解説する．
２つの拡張機能はユーザによる JSON の記述を助ける．
他方，キーワード・チェインはマーケットの階層的依存関係を記述する仕組みを提供する．


## "simulation" JSON object

シミュレーション設定の読み出しは `"simulation"` オブジェクトから始まる．
以下に例を示す．

```json
// samples/CI2002/config.json
"simulation": {
    "markets": ["Market"],
    "agents": ["FCNAgents"],
    "sessions": [
        {    "sessionName": 0,
             "iterationSteps": 100,
             "withOrderPlacement": true,
             "withOrderExecution": false,
             "withPrint": true
        },
        {    "sessionName": 1,
             "iterationSteps": 500,
             "withOrderPlacement": true,
             "withOrderExecution": true,
             "withPrint": true
        }
    ]
},
```

キーワード `"simulation"` は `Main` で約束されており，JSON ファイルの処理は概ねここから始まる．
シミュレーションは以下の属性をもたねばらない．

  * `"markets"`  ... シミュレーションで使われるマーケットのリスト
  * `"agents"`   ... シミュレーションで使われるエージェントのリスト
  * `"sessions"` ... セッションのリスト（下記参照）

`"markets"` や `"agents"` の内容はこれまで本記事で登場している．

セッションは異なるマーケットの状態を扱うための仕組みであり，たとえば，
「注文を出せば約定される状態（マーケット開放時）」の他に，
「注文を出せるが約定されない状態（マーケット閉鎖時）」などを設定できる．
セッションは定義された順番に実行される．
セッションのもつパラメータは以下である．

  * `"sessionName"`        ... セッション名（出力データ用）（String）
  * `"iterationSteps"`     ... シミュレーションのステップ数（Long）
  * `"withOrderPlacement"` ... 注文を許可するか（Boolean）
  * `"withOrderExecution"` ... 約定を許可するか（Boolean）
  * `"withPrint"`          ... 出力データありなし（Boolean）

もし `"withOrderPlacement" == false` かつ `"withOrderExecution" == false`
の場合，市場価格を理論価格で埋め合わせて擬似価格時系列が生成される．
また，擬似時系列の生成は以下のパラメータで明示的に設定できる．

  * `"forDummyTimeseries"` ... 擬似時系列を生成するか（Boolean）

擬似時系列の生成は，シミュレーション開始時点でのチャーチストの振る舞いを定めるために必要となる．

各ステップ毎に，最大何体の**低頻度取引**（**非**高頻度取引）エージェントが注文を出せるか（尋ねるかではなく出せるか）は以下のパラメータで制御できる．

  * `"maxNormalOrders"` ... 各ステップで最大何体から注文を受けるか（低頻度取引）

`maxNormalOrders` のデフォルト値はマーケットの個数に設定される．この場合，長期的に見れば，各銘柄が各ステップで１つの注文を受ける．

また，各**ティック**毎に，最大何体の**高頻度取引**エージェントが注文を出せるかは以下のパラメータで制御できる．

  * `"maxHifreqOrders"` ... 各ステップで最大何体から注文を受けるか（高頻度取引）

`maxHifreqOrders` のデフォルト値は `0` である．

**注意**：
繰り返しになるが，`maxNormalOrders` は各ステップ毎，他方，`maxHifreqOrders` は各ティック毎であることに注意する．
なお，本ソフトウェアでは，「ティック毎」とは「低頻度取引エージェントの注文を１つ処理する度に」を意味する．


## "extends" keyword

See [JSON](/class/JSON).


## Group pseudo-class

JSON オブジェクトによる複数のエージェント定義やマーケット定義をまとめるグループ化の仕組みとして，`AgentGroup` および `MarketGroup` キーワードが用意されている．
注：X10 クラスとしては存在しない．
`AgentGroup` および `MarketGroup` は `"class"` の値として使用できる．
それぞれ `"agents"`，`"markets"` 属性にグループに属する JSON オブジェクトを指定する．
以下に例を示す．

```json
{
    "agents-1": {
        "class": "FCNAgent",
        ...
    },

    "agents-2": {
        "class": "FCNAgent",
        ...
    },

    "agents-1&2": {
        "class": "AgentGroup",
        "agents": ["agents-1", "agents-2"],
        ...
    }
}
```


## Keyword chaining

指数銘柄のように他の銘柄と主従関係にあるような銘柄を定義する場合，キーワード・チェインという約束に従うように注意する．
キーワード・チェインは JSON ファイルのみから，マーケット間，エージェント間の階層的主従関係を把握するための仕組みである．

キーワード・チェインでは，階層的に主従関係にある下位オブジェクトを指定するときに，同じキーワード（`"markets"` か  `"agents"`）に対する値として記述することで，その階層的主従関係を JSON ファイル上で表現できる．
例えば，以下の例ではキーワード `"sub"` を辿ることによって D :> C :> B :> A という階層関係が定義されている．
このようにあるキーワード（`"markets"`）の連鎖として階層関係を記述するので，キーワード・チェインと呼んでいる．
なお，循環的相互依存関係は許容されていない．

```json
{
    "A": {},
    "B": { "sub": ["A"] },
    "C": { "sub": ["B"] },
    "D": { "sub": ["C"] }
}
```

キーワード・チェインは以下の技術的問題を回避する．
例として，`createMarkets()` 内で `IndexMarket` をインスタンス化することを考えよう．
`IndexMarket` は現物銘柄に相当するマーケットへの参照をもつため，`IndexMarket` のインスタンス化に直面した時点で，要求されるすべての現物銘柄マーケットのインスタンス化が完了していなければならない．
換言すれば，マーケットのインスタンス化に優先順序が生じており，依存関係のないもの，依存関係を解消できたものから順に処理する必要がある．
この階層的順序関係はインスタンス化を始める前に把握する必要があり，それを実現する仕組みがキーワード・チェインである．

具体例を見てみよう．
`"market-1"`，`"market-2"`，`"market-3"` は他のマーケットに依存しない独立したマーケットである．
他方，`"markets-1-2"` は先述のマーケットグループで，`"market-1"`，`"market-2"` に依存する．
また，`"market-INDEX"` は `"markets-1-2"` と `"market-3"` に依存するが，マーケットグループを含んでいるので，実際には `"market-1"`，`"market-2"`，`"market-3"` さらに `"markets-1-2"` に依存している．
この具体例から，`"markets"` キーワードを辿っていくことで階層的主従関係が把握できることがわかるだろう．
この階層的主従関係は `"simulation"/"markets"` のリストだけからは把握できないため，キーワード・チェインによって階層関係を把握する．

```json
{
    "market-1": {},

    "market-2": {},

    "markets-1-2": {
        "class": "MarketGroup",
        "markets": ["market-1", "market-2"]
    },

    "market-3": {},

    "market-INDEX": {
        "class": "IndexMarket",
        "markets": ["markets-1-2", "market-3"]
    },

    "simulation": {
        "markets": ["market-1", "market-2", "market-3", "market-INDEX"]
    }
}
```

