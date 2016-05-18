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



## シミュレーションの流れ

Plham を用いた人工市場シミュレーションは基本的には２つのメソッド

  * `Agent#submitOrders(List[Market])` → `List[Order]`
  * `Market#handleOrders(List[Order])` → `void`

を特定のスケジュールで呼び出すことで進行する．

`Agent#submitOrders(List[Market])` では，エージェントはマーケットの価格情報などから注文を決定し，このメソッドの戻り値として注文のリストを返す．
このメソッドがエージェントが注文をだす唯一の方法となるべきである．
このメソッドの戻り値 `List[Order]` のなかにキャンセル要求 `Cancel` を含めることが可能である．

`Market#handleOrders(List[Order])` では，マーケットはエージェントから集められた注文を処理し，約定が成立した場合にはエージェントの資産情報を更新する．
このメソッドがマーケットが注文を受け付ける唯一の方法となるべきである．
これらのメソッドはスケジューラによって自動的に呼び出されるので，ユーザが明示的に呼び出すことはない．

上記の例からもわかる通り，各クラスに定義されているメソッドには「暗黙のアクセス制限」がある．
暗黙のアクセス制限はクラス間の関係で想定される．
たとえば，「エージェントはマーケットの何を参照許可されるか」などである．
本記事では，Plham で想定されている「エージェント」，「マーケット」，「システム」の間のアクセス制限について述べる（サンプルコードも活用してほしい）．
まずは各クラスを紹介する．


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

注文の処理は `handleOrders()` メソッドに実装する．
これがユーザがマーケットを作成するときの主要な作業内容である．

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

注文の決定は `submitOrders()` メソッドに実装する．
これがユーザがエージェントを作成するときの主要な作業内容である．

エージェントの保有する現金額，株式数はマーケットが注文を処理する過程で自動的に更新される．

エージェントの実装例は `plham.agent.FCNAgent` を参考にしてほしい．


## システム

システムは，`Agent#submitOrders()` および `Market#handleOrders()` の呼び出しを含む，シミュレーションの進行に伴うスケジューリングを行う．
また，システムは下記のメイン（`Main`）を含み，そこではユーザはシミュレーションに必要なインスタンスを生成したり，シミュレーションの初期状態を設定できる．


### メイン

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


## エージェントを実装するためのマーケットの API

エージェントは，マーケットから価格時系列や気配値の情報などを読み取る．
現実ではエージェントはマーケットの内部変数を書き換えることはできないので，シミュレーション上でもそうすべきではない．

マーケットがもつメソッドのうち，エージェントの `submitOrders()` で利用推奨なメソッドを示す．

```x10
// plham/Market.x10
public class Market {
	id:Long;                                // ID 番号
	name:String;                            // 名前（JSON で定義された）
	getTime():Long;                         // 時間ステップ
	getPrice(t:Long):Double;                // 市場価格
	getFundamentalPrice(t:Long):Double;     // ファンダメンタル価格
	getBestBuyPrice():Double;               // 買い気配値
	getBestSellPrice():Double;              // 売り気配値
	getMidPrice():Double;                   // 仲値
    getTradeVolume(t:Long):Long;            // 出来高（約定単位数）
	isRunning():Boolean;                    // マーケットが開いているか？
	getOutstandingShares():Long;            // 時価総額シェア（時価総額荷重指数にのみ使用）
	getTickSize():Double;                   // ティックサイズ
	roundBuyPrice(price:Double):Double;     // 価格をティックサイズに丸める
	roundSellPrice(price:Double):Double;    // 価格をティックサイズに丸める
}
```

次に，利用**非推奨**なメソッド（類似の名をもつメソッドも非推奨）の一部を示す．

```x10
// plham/Market.x10
public class Market {
	handleOrders(orders:List[Order]);    // 非推奨
	handleOrder(order:Order);            // 非推奨
	cancelOrder(order:Order);            // 非推奨
}
```


## マーケットを実装するためのエージェントの API

マーケットはシステムから与えられた注文を処理し，その途中でエージェントの資産を更新する．

エージェントがもつメソッドのうち，マーケットの `submitOrders()` で利用推奨なメソッドを示す．

```x10
// plham/Agent.x10
public class Agent {
	id:Long;                               // ID 番号
	name:String;                           // 名前（JSON で定義された）
	getCashAmount():Double;                // 現金量
	getAssetVolume(market:Market):Long;    // 株式数
	updateCashAmount(delta:Double);                    // 現金量を更新
	updateAssetVolume(market:Market, delta:Double);    // 株式数を更新
	orderExecuted(market:Market, orderId:Long, price:Double, cashAmountDelta:Double, assetVolumeDelta:Long);
}
```

次に，利用**非推奨**なメソッド（類似の名をもつメソッドも非推奨）の一部を示す．

```x10
// plham/Agent.x10
public class Agent {
	submitOrders(markets:List[Market]):List[Order];    // 非推奨
}
```


## システムを実装するためのマーケットとエージェントの API

一部のみ示すが，繰り返し述べているとおり，`Agent#submitOrders()` および `Market#handleOrders()` を確実に呼び出すことが重要である．

```x10
// plham/Market.x10
public class Market {
	handleOrders(orders:List[Order]);
	handleOrder(order:Order);
}
```

```x10
// plham/Agent.x10
public class Agent {
	submitOrders(markets:List[Market]):List[Order];
}
```

