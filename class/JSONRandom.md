---
title: JSONRandom
author: Takuma Torii
math: false
---

# JSON random number specification

本記事では，JSON ファイルで各種パラメータを指定するときに用いる乱数の書式を定義する．
この書式を用いた乱数生成は `plham.util.JSONRandom#nextRandom(...)` メソッドを呼び出すことで利用できる．

```x10
val random = new JSONRandom(new Random());
val json = JSON.parse(inputFile);
val param = random.nextRandom(json("param"));    // Random number or Constant
```

ここで，`nextRandom(...)` は `json("param")` が返す JSON オブジェクト（定数，配列，辞書など）を引数にとる．
後続の節で述べる定義書式をサポートしている．

註）JSON で記述するあるパラメータの値がこの定義書式をサポートしているかは，X10 で書かれたプログラムが `JSONRandom#nextRandom(...)` メソッドを呼び出しているかで決まる．
説明書を記述する際には `(JSONRandom compliant)` と書くとよいだろう．


## Constant

```json
{  "param": 100  }
{  "param": [100, 100]  }
{  "param": {"const": [100]}  }
```


## Uniform [min, max]

下限 `min`，上限 `max` の一様分布乱数を与える．

```json
{  "param": [100, 200]  }
{  "param": {"uniform": [100, 200]}  }
```


## Normal [mu, sigma]

平均 `mu`，分散 `sigma^2` の正規分布乱数を与える．

```json
{  "param": {"normal": [0, 1.5]}  }
```


## Exponential [lambda]

期待値 `lambda` の指数分布乱数を与える．

```json
{  "param": {"expon": [10.0]}  }
```


