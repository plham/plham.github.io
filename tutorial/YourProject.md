---
title: Your Project
author: Takuma Torii
math: false
---

# Getting Started Your Project

本記事およぼチュートリアルでは，どのように本ソフトウェアを使用し，各自のプロジェクト，人工市場モデル，を開発していくかを説明する．
本記事ではまずファイル構成について述べる．


## Download from GitHub

[Install](/Install)の記事を参照せよ．


## Create your project

GitHub からダウンロードしたあと，ユーザのプロジェクトを作成する．
ユーザのプロジェクト関連のファイルは，本ソフトウェア `plham/plham` フォルダや `plham/samples` フォルダがある場所に配置することを推奨する．
以下では，ユーザがソースコードを配置するフォルダを `YourProject` とした．

```
$ cd plham
$ ls
README.md  doc/  hpc/  json/  plham/
$ mkdir YourProject
```


## Create your Main & JSON

ユーザは自作の Main クラスを作成し，`YourProject` に配置する．
Main クラスは `plham.Main` を継承し，`createAgents(JSON)` および `createMarkets(JSON)` をオーバーライドしてシミュレーションの部品をインスタンス化する．

また，ユーザは自作の Main クラスに合わせた，JSON 設定ファイルを用意する．

フォルダ構造は以下のようになるだろう．

```
$ ls
README.md  doc/  hpc/  json/  plham/  YourProject/
$ ls YourProject/
YourMain.x10  config.json
```


## Compile & run your Main

`plham` や `YourProject` のあるフォルダから，以下の手順でコンパイル・実行する．

```
$ ls
README.md  doc/  hpc/  json/  plham/  YourProject/
$ x10c++ YourProject/YourMain.x10
$ ./a.out YourProject/config.json
```


