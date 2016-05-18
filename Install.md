---
title: Install
author: Takuma Torii
math: false
---

# Getting started

## About X10

本ソフトウェアは並列分散プログラミング言語 X10 により書かれている．
X10 は Java 的なオブジェクト指向言語であり，並列分散処理を記述できる言語設計をもつ．
X10 は C++ か Java をバックエンドとし，X10 で書かれたプログラムは１度 C++ か Java に翻訳されてコンパイルされる．

X10 については，公式サイト

  * http://x10-lang.org/
  * http://x10-lang.org/documentation/getting-started.html
  * http://x10.sourceforge.net/x10doc/latest/

などから情報を収集できる．


## Install X10

X10 コンパイラは Java を使用するため，Java をインストールし，環境変数 `JAVA_HOME` を設定しておく必要がある．

ここでは，X10 2.5.4 を Linux にインストールする場合を説明する．

まず，以下のウェブサイトから「`x10-2.5.4_linux_x86_64.tgz`」をダウンロードする．

  * http://sourceforge.net/projects/x10/files/x10/

コマンドラインターミナルを起動し，以下の手順を行う．

```
$ su
# ls
x10-2.5.4_linux_x86_64.tgz

# mkdir /usr/local/x10
# tar zxf x10-2.5.4_linux_x86_64.tgz -C /usr/local/x10
# exit
```

以上で `/usr/local/x10` に X10 をインストールできた．
最後に X10 へのパスを設定する．

```
$ echo 'export PATH=/usr/local/x10/bin:$PATH' >>~/.profile
```

一度ログアウトし再度ログインすれば，`x10`，`x10c`，`x10c++` などのコマンドが使える．

  * `x10c++` ... C++ バックエンドでコンパイルし，`a.out` を生成する
  * `x10c` ... Java バックエンドでコンパイル（`javac` 相当）
  * `x10`  ... Java バックエンドで実行（`java` 相当）


## Download from GitHub

ターミナルを起動し，以下の手順でソースコードを入手する．

```
$ git clone git@github.com:plham/plham.git
$ git clone https://github.com/plham/plham.git    # HTTPS を使う場合
```

正しく完了すれば，`plham` というフォルダができている．


## Run sample programs

本ソフトウェアは人工市場シミュレーションの研究例をサンプルコードをして含む．
ここでは一例として，Chiarella & Iori (2002) のシミュレーション（変更あり）を実行する．

先ほど `git clone` を実行したフォルダから，

```
$ cd plham
$ x10c++ samples/CI2002/CI2002Main.x10                   # C++ 経由でコンパイル
$ ./a.out samples/CI2002/config.json >output.dat         # 実行出力を output.dat に保存
$ Rscript samples/CI2002/plot.R output.dat output.png    # 価格時系列をプロット
```

