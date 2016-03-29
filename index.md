---
title: Readme
Author: Takuma Torii
math: false
---

# Preface

人工市場シミュレーションのためのソフトウェア **Plham** （ぷらむ）のドキュメントです．

本ドキュメントは実際に出版されている複数の研究事例をチュートリアルとして含んでいます．
チュートリアルでは，シミュレーションの開発，実行，可視化までを紹介しています．
シミュレーションの開発手順はこれらのチュートリアルをご覧ください．
チュートリアルで使われたソースコードはソフトウェアの一部として一緒にダウンロードされます．
これらのソースコードは自由に再利用していただいて構いません．
チュートリアルは今後も追加していく予定です．

本ソフトウェアを使用して行われた研究では下記の文献を引用していただけると幸いです．

> Takuma Torii, Tomio Kamada, Kiyoshi Izumi, Kenta Yamada (2016) Platform Design for Large-Scale Artificial Market Simulation and Preliminary Evaluation on the K computer. Proceedings of The 21st International Symposium on Artificial Life and Robotics (AROB 21st 2016), OS10-2, pp.1--6.

本ソフトウェアに関する問い合わせは下記までお願いいたします．
拡張機能や詳しい解説の要望などお待ちしています．

> `plham-discuss(at)socsim.org`


# Contents

  * [プロジェクトの概要 (Project)](/Project)
  * [インストールの仕方 (Install)](/Install)
  * [ソフトウェアの特徴 (Platform)](/Platform)
  * [用語について (Terminology)](/Terminology)
  * [シミュレーションの流れ (Runner)](/class/Runner)
  * Tutorial
    * [自分のプロジェクトを作る (YourProject)](/tutorial/YourProject)
    * [アウトラインとガイドライン (Outline)](/tutorial/Outline)
    * 事例 1: 単一銘柄の人工市場シミュレーション
      * [CI2002Main](/tutorial/CI2002Main)
      * [CI2002 UseCases](/tutorial/CI2002Main_UseCases)
    * 事例 2: 値幅制限規制
      * [PriceLimitMain](/tutorial/PriceLimitMain)
      * [PriceLimit UseCases](/tutorial/PriceLimitMain_UseCases)
    * 事例 3: 取引停止規制と理論価格ショック
      * [TradingHaltMain](/tutorial/TradingHaltMain)
      * [TradingHalt UseCases](/tutorial/TradingHaltMain_UseCases)
    * 事例 4: 誤発注型ショック（Fat finger）
      * [FatFingerMain](/tutorial/FatFingerMain)
      * [FatFinger UseCases](/tutorial/FatFingerMain_UseCases)
    * 事例 5: 高頻度取引によるショック伝搬（FlashCrash）
      * [ShockTransferMain](/tutorial/ShockTransferMain)
      * [ShockTransfer UseCases](/tutorial/ShockTransferMain_UseCases)
    * 事例 6: 数百銘柄シミュレーションの並列実行
      * [ParallelMain](/tutorial/ParallelMain)
    * 補足説明など
      * [JSON for Main](/tutorial/JSON_for_Main)
  * Utilities
	* [JSON](/class/JSON)
	* [JSONRandom](/class/JSONRandom)
    * [Fundamentals](/class/Fundamentals)
  * Models
    * [Market](/class/Market)
    * [IndexMarket](/class/IndexMarket)
	* [Agent](/class/Agent)
	* [FCNAgent](/class/FCNAgent)
    * [ArbitrageAgent](/class/ArbitrageAgent)
  * API (javadoc)
    * [(link)](/api)

