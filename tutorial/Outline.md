---
title: Outline
author: Takuma Torii
math: false
---

# Outline

本記事では本ソフトウェアの概要を述べる．
とくに，本ソフトウェアを使い始めるにあたり，最初に参照するであろう，マーケット，エージェント，そしてメインの骨格を紹介する．


## 留意事項，実装ガイドライン

以下は，本ソフトウェアを使用し，実装を進める上でのガイドラインである．

**多くのフィールドは変更可能（var）だが．**
多くのフィールドは変更可能（var）であるが，基本的には独自作成したクラスのフィールドでない限り，派生クラス側で書き換えること（再代入）は望ましくない．
とくにプリミティブ型でない `List[T]` や `Map[T,U]` などは基本クラスのコンストラクタで初期化し，以後，派生クラス側では書き換えないことが望ましい．
たとえば，`market.id` や `agent.id` なども書き換え可能であるが，書き換えた場合のシミュレーションの挙動は保証されない．

**多くのメソッドは public だが．**
多くのメソッドは `public` で定義されているが，だからといって，常識的な範囲を超えてアクセスすることは望ましくない．
たとえば，マーケットはオーダーブックを保持し，`Market#getOrderBook()` を介してオーダーブック本体にアクセスできる．
オーダーブックのすべてのメソッドを呼び出せるので，あるエージェントがオーダーブックを全消しするなどできるが，人工市場を用いた取引システムのバグの影響でもテーマにしない限り，そんなことをする理由はないだろう．

**多くのフィールドは public だが．**
多くのフィールドおよびメソッドは `public` で定義されているが，フィールドへの直接アクセスは可能であれば避けるのが望ましい．
とくに `plham` 直下に配置される本ソフトウェアの核をなすクラス群（`plham/*.x10`）は仕様変更される可能性が高いので，メソッドを使用することを推奨する．
たとえば，`submitOrders()` が呼び出される度にあるエージェントがマーケットの市場価格を書き換えたりできるが，人工市場を用いた金融サイバーテロのシミュレーションでもテーマにしない限り，そんなことをする理由はないだろう．


## マーケット

マーケットは `plham/Market.x10` に実装されている．

マーケットの基本的な機能は以下である．

  * 注文の処理
    * 注文を受ける
    * 約定を判定する
    * オーダーブックを更新する
  * 価格情報の管理

各機能に対応したメソッドを以下に示す．

```x10
// plham/Market.x10
public class Market {

    public def handleOrders(orders:List[Order]) {}       // 注文の処理（約定）

    public def getTime():Long {}                         // ステップ時間の取得

    public def getPrice(t:Long):Double {}                // ステップ t の市場価格
    public def getFundamentalPrice(t:Long):Double {}     // ステップ t の理論価格
}
```

留意点として，モデルによってはマーケットごとにステップ時間が異なる可能性がある．
そのため，すべてのマーケットで `getTime()` が一致しているという前提で意思決定
を実装することは望ましくない．


## エージェント

エージェントは `plham/Agent.x10` に実装されている．

エージェントの基本的な役割は以下である．

  * 注文の意思決定
  * 資産（現金，株式）の管理

以下にエージェントクラスの概要を示す．

```x10
// plham/Agent.x10
public class Agent {

    public def submitOrders(markets:List[Market]):List[Order] {}    // 注文の決定（複数銘柄）

    public def isMarketAccessible(market:Market) {}                // そのマーケットで取引するか
    public def setMarketAccessible(market:Market) {}               // そのマーケットで取引するか

    public def getCashAmount():Double {}                           // 現金額の取得
    public def getAssetVolume(market:Market):Long {}               // 株式数の取得
}
```

注文の意思決定は `submitOrders()` メソッドに実装する．
これがユーザがエージェントを作成するときの主要な作業内容である．

エージェントの保有する現金額，株式数はマーケットが注文を処理する過程で自動的に更新される．

エージェントの実装例は `plham.agent.FCNAgent` を参考にしてほしい．


## メイン

通常，ユーザは `plham.Main` を継承した独自のメインクラスを作成する．
その際，以下のメソッドをオーバーライドし，JSON ファイルと連携しつつ，シミュレーションで使用したいインスタンスを生成すればよい（JSON との連携方法はここでは説明しないので，チュートリアルを参考にしてほしい）．

```x10
// plham/Main.x10
public class Main extends Simulator {

    public def createMarkets(json:JSON.Value):List[Market] {}
    public def createAgents(json:JSON.Value):List[Agent] {}
    public def createEvents(json:JSON.Value):List[Event] {}
}
```



