---
title: "CatBoost の feature importance"
date: 2020-09-11
lang: en-us
draft: true    
tags: ["machine learning"]
---


CatBoost が計算してくれる feature importance の定義がドキュメントを見てもよくわからなかったので調べた。
いくつか importance の定義があるが、ここでは `PredictionValuesChange` についてまとめる。
これはその feature が最終的な予測値にどれだけ影響を持つかという量を計算している

## 定義
公式ドキュメントによると、feature F の feature importance は
$$
importance_F=\sum_{tree, leaf} c_1(v_1 - avr)^2 + c_2(v_2 - avr)^2
$$
で計算される。ここで、和の範囲は各 tree の 各 leaf node であり、$c_1$, $c_2$ は同じ parent node を持つ二つの leaf node それぞれの持つサンプル数（木を辿ってその leaf に属することになるデータの数）、$v_1$, $v_2$ はそこでの予測値の値である。
また、$avr$ は以下のような $v_1$ と $v_2$ の加重平均である
$$
avr = \frac{c_1v_1 + c_2 v_2}{c_1 + c_2}.
$$
つまり、その feature を使った分岐によって最終的な予測値がどれだけ変わるかという量になっている。

実際の具体例で見てみる。iris dataset を使い、以下のようなモデルパラメータで CatBoost を訓練した
```python
iris = datasets.load_iris()
X, y = iris.data, iris.target

model = CatBoostClassifier(iterations=1, depth=2, random_seed=42)
model.fit(X, y)
```
深さ 2 の tree が一つという非常にシンプルなモデルである。
得られたモデルを可視化すると以下のようになる
```python
model.plot_tree(tree_idx=0)
```
{{< figure library="true" src="tree_iris.png" title="A caption" >}}


{{< figure src="tree_iris.png">}}

なぜ