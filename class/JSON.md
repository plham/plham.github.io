---
title: JSON
author: Takuma Torii
math: false
---

# JSON class

JSON（JavaScript Object Notation）は軽量のデータ交換フォーマットである．
詳しくは [www.json.org](http://www.json.org/json-ja.html) を参照してほしい．


## JSON.x10

X10JSON `plham.util.JSON` は JSON ファイルを解析し，そのデータにアクセスする手段を提供する．
具体的に，下記のデータが `input.json` に保存されているとする．

```json
// input.json
{
   "n": 10,
   "a": ["item0", "item1"],
   "d": {"X": 1.5, "Y": 2.5}
}
```

このファイル  `input.json` を X10 から読み込むには次のように書く．
なお，`plham.main.BaseMain` を継承した場合，コマンドライン引数 `args` に指定される JSON ファイルは `BaseMain` 内で読み込み処理され，フィールド変数 `CONFIG` に保存される．

```x10
val json = JSON.parse(new File("input.json"));
```

X10 上から，JSON ファイルの内容を取得する例を以下に示す．
なお，X10JSON では整数，実数などの区別はなくすべて文字列型で格納されるため，`toLong()`，`toDouble()` などで型変換する必要がある．
また，文字列を取得する場合にも `toString()` を呼び出す．

```x10
val n = json("n").toLong();               // "n" を Long 型として取得する
val a0 = json("a")(0).toString();         // 配列 "a" の第 0 要素 "item0" を取得する
val dkey1 = json("d")("X").toDouble();    // 辞書 "d" の鍵 "X" に対する値 1.5 を取得する

val a = json("a");
val a1 = a(1).toString();                 // 配列 "a" の第 1 要素 "item1" を取得する

val d = json("d");
val dkey2 = d("Y").toDouble();            // 辞書 "d" の鍵 "Y" に対する値 2.5 を取得する
```


## 継承（extends）

X10JSON `plham.util.JSON` はオブジェクト属性の継承をサポートしている．
継承はオブジェクトの任意の箇所で `"extends"` キーワードを使用することで認識され，その値として基本オブジェクト（継承元）を指定する： `"extends": base-object-name`．
基本オブジェクト（継承元）と派生オブジェクト（継承先）で同じ名前の属性が再定義されている場合，`"extends`" の前で宣言されたか後で宣言されたかにかかわらず，すべて派生オブジェクトの値でオーバーライドされる．
継承の効果は，`"extends"` の出現箇所に影響されないので，オブジェクト定義の冒頭に書くことを推奨する．
たとえば，以下の JSON ファイルの場合，

```json
{
    "base-object": {
        "a": 1,
        "b": 2,
        "c": 3
    },
    "derived-object": {
        "a": 11,
        "extends": "base-object",
        "b": 12,
        "d": 14
    }
}
```

`"derived-object"` は次の定義と等価である．

```json
    "derived-object": {
        "a": 11,
        "b": 12,
        "c": 3,
        "d": 14
    }
```

X10JSON で継承を作用させるには以下のように書く．

```x10
JSON.extend(json)
```

