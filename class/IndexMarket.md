---
title: IndexMarket
author: Takuma Torii
---

# IndexMarket class

## Description

IndexMarket クラスは，株価指数およびその構成銘柄（現物銘柄）と紐づけられた指数銘柄の機能を提供する．
現実の金融市場では指数先物や上場投資信託（ETF）に相当する．
このクラスの機能は

  * 固有の指数計算方式をもつ
  * 株価指数の情報を提供する

といった点で [Market](/class/Market) クラスと異なる．
他方，IndexMarket クラス自体が連続ダブルオークション形式の取引を提供する点は共通している．



## Model

株価指数の説明...



## Parameters

See [Market](/class/Market)


## Parameter setup from JSON

See [Market](/class/Market)


## Fields & Methods

### addMarket(Market), addMarkets(List[Market])

あるマーケットを株価指数の構成銘柄に加える．


### getMarketIndex(t:Long)

時点 $t$ での株価指数を返す．
マーケットは `getTime()` メソッドが返す時点を含む現在までの時系列を保管している．


## getFundamentalIndex(t:Long)

時点 $t$ でのファンダメンタル価格から計算した株価指数を返す．
マーケットは `getTime()` メソッドが返す時点を含む現在までの時系列を保管している．


