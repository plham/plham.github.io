---
title: Terminology
author: Takuma Torii
math: false
---

# Terminology

本ソフトウェアで用いる独特の用語を整理する．


#### エージェント

すべてのトレーダ（高頻度取引を含む）を「エージェント」と呼ぶ．

`Agent` クラスを継承する．


#### マーケット

現実における１つの金融商品を１つの「マーケット」として抽象化している．
通常「市場」とは複数の金融商品を交換する場あるいはシステムをさすが，本ソフトウェアで「マーケット」というとき，それは独立した個別の金融商品を取引するシステムを意味する．

`Market` クラスを継承する．


#### イベント

イベントとは金融ショックや金融規制など，何らかの条件を引き金として発動する事象を「イベント」と呼ぶ．

`Event` クラスを継承する．


#### ステップとティック

シミュレーションの１ステップは複数のティックからなる．
１ティックとは１つの取引成立を単位とする時間である．
他方，１ステップは現実でいう「１秒」や「１分」に類する時間単位だと考えてよい．

逐次実行モデルにおいては，１ステップは低頻度取引エージェントの同期的な意思決定を単位とする時間といえる．
他方，１ティックは注文処理を単位とする時間である．
高頻度取引エージェントの意思決定はティック時間で可能である．

`Market` クラスの `getPrice(t)` は各ステップにおける最終的な価格を返す．


#### 人工市場モデル（simulation model）

エージェントシミュレーションの用語でいう「モデル」をさす．
金融・経済の分野の研究者が作成するのはこの意味でのモデルである． 
この意味でのモデルは，エージェントの取引戦略，マーケットの注文処理，金融ショックや金融規制などを含む．
詳しくは[こちら](/Platform)．


#### 計算実行モデル（execution model）

計算機科学の分野の研究者が開発するのは，「人工市場モデル」（あるいは「問題」）をいかに能率的に計算するか，その計算の実行方法に関する「モデル」である． 
この意味でのモデルは，データ配置の問題，データ通信の問題，タスク管理の問題を含む．
詳しくは[こちら](/Platform)．
