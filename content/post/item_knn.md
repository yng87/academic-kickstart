---
title: "ItemKNN を使って MovieLens 学習"
date: 2020-09-05
lang: en-us
draft: false    
tags: ["machine learning", "recommend"]
---

# 概要
---
近年推薦の分野ではディープラーニングを使った手法がよく提案されている。しかし実は、それらは古典的な手法に比べて特別優れているわけではないという指摘が相次いでいる[^1] [^2]。
ディープラーニング全盛の現代でも、レコメンド分野においては古典的なベースライン手法を勉強しておくのは重要そうである。

ということで、ここではそのようなベースライン手法の勉強として、初期の Amazon でも使われていた由緒正しい手法である[^3]、アイテムベースの協調フィルタリング (**ItemKNN 協調フィルタリング**) を実装し、MovieLens データセット[^4]に適用してみた。

# 手法
---

ItemKNN はアイテムベースの協調フィルタリングであり、アイテムの類似度を元に似ているアイテムを $K$ 件抽出して推薦を行うアルゴリズムである。具体的には

1. アイテム $i$ と アイテム $j$ の (コサイン) 類似度を次で定義する：
$$
s_{ij} = \frac{\vec{r}_i\cdot\vec{r}_j}{|\vec{r}_i| |\vec{r}_j| + h}.
$$
$\vec{r}_i = (r_{i1}, \dots, r_{iN_U})$ はアイテム $i$ の特徴量ベクトルで、ここでは $r_{iu}$ がアイテム $i$ に対するユーザー $u$ の評価値を表すとする ($u = 1,\dots,N_U$)。
分母の $h$ は shrink parameter と呼ばれ、$\vec{r}_i$ のノルムが小さい (=評価数が少ない) 時に類似度を下方修正する役割をする[^1]。

2. (Optional) TF-IDF によって多くのアイテムに評価をつけるユーザーの寄与を抑える[^5]。計算は
$$
r_{iu} \to r_{iu}\times \left(\log\left( \frac{N_I}{\sum_{i: r_{ui}\neq0}1}\right) + 1\right)
$$
とする。$\log$の中の分母はユーザー $u$ が評価をつけたアイテムの総数を、$N_I$はアイテムの総数を表す。各ユーザーは各アイテムに一回しか評価をつけないので、TF項は入れていない。

3. ユーザー $u$ のアイテム $i$ に対する選好度のスコアは
$$
P_{ui} = \sum_{j \in kNN(i)} s_{ij}r_{uj} .
$$
$j$の和はアイテム $i$ の近傍 $k$ 件だけ取ってくる。
そして、ユーザー $u$ への推薦は、$P_{ui}$ が大きい順に $n$ 個のアイテムを取ってくることで行う。

# 実装
---
今回は Julia で以上のアルゴリズムを実装した
https://github.com/yng87/kNNRecommenders.jl

ユーザー・アイテムの評価データ $r_{iu}$ は $N_I\times N_U$ の行列として表せるが、0が多いので dense 行列として持たない方が、スケーラビリティの観点では良い。
類似度の計算は Amazon の論文[^3]にもあるように $r_{iu}$ のスパース性を利用して

1. アイテム $i$ に評価をつけているユーザー $u_i$ の一覧を抽出 ($r_{iu_i}\neq0$)
2. 各 $u_i$ が評価をつけているアイテム $j$ を抽出 ($r_{ju_i}\neq0$)
3. $r_{iu_i}r_{ju_i}$ を計算し、$u_i$ で和をとることで $i$ と 各 $j$ との類似度 $s_{ij}$ を計算

とやるのが効率が良い。



# 実験結果
---

MovieLens 1M[^6] で実験をした。データセットは (user_id, item_id, rating) という形で与えられており、rating = 1, ..., 5 という値をとる。
ここでは、rating=5 を positive ($r_{iu}=1$)、rating<5 を negative ($r_{iu}=0$) として上の ItemkNN 協調フィルタリングを実行した。適当にパラメータチューニングをしたところ、best score として以下の表のような結果が得られた[^7]。
評価指標は各ユーザーにアイテムを $n$ 件推薦した際の recall の平均値 (Recall@n) である。

| Method                | Recall＠20  | Recall＠60|Recall＠100|
| -------------------   | ----------- |---       |---|
| k=200, h=0, w/ TF-IDF |0.2773       |0.4641|0.5647|
| k=200, h=0, w/o TF-IDF|0.2704       |0.4537|0.5533|
| 参考値[^1] |0.2819|0.4712|0.5737|

TF-IDF はスコアを系統的に良い方向に引き上げている。
"参考値"は論文から持ってきたものであり、上二つの段とはデータセットの分割が違うので、数字をそのまま比較することはできないが、大体近い値になっている。
もっときちんとチューニングすればこちらの結果ももう少しよくなるかもしれない。


[^1]: https://arxiv.org/abs/1907.06902
[^2]: http://arxiv.org/abs/2005.09683
[^3]: http://ieeexplore.ieee.org/document/1167344/
[^4]: ユーザーが映画の評価を5段階でつけたデータセット: https://grouplens.org/datasets/movielens/
[^5]: https://link.springer.com/article/10.1007/s10791-008-9060-1
[^6]: https://grouplens.org/datasets/movielens/1m/
[^7]: 実験ノートブック: https://github.com/yng87/kNNRecommenders.jl/blob/master/jupyter/experiment1.ipynb