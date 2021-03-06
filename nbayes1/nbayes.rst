.. _nbayes1-nbayes:

単純ベイズ：カテゴリ特徴の場合
==============================

実装の前に，特徴がカテゴリ変数である場合の単純ベイズ法による分類について，ごく簡単に復習します．

.. index:: ! naive Bayes

.. index::
   single: naive Bayes; multinomial

変数を次のように定義します．

* 特徴量 :math:`\mathbf{x}_i=(x_{i1}, \ldots, x_{iK})` の要素 :math:`x_{ij}` はカテゴリ変数で， :math:`F_j` 個の値のうちの一つをとります．
  ただし， :math:`K` は特徴の種類数です．
* クラス :math:`y` は， :math:`C` 個の値のうちの一つをとります．

ここで，特徴 :math:`\mathbf{X}` は，クラス :math:`Y` が与えられたとき条件付き独立であるとする単純ベイズの仮定を導入すると， :math:`\mathbf{X}` と :math:`Y` の同時分布は次式で与えられます．

.. math::
   :label: nbayes1-joint-prob

   \Pr[\mathbf{X}, Y] = \Pr[Y] \prod_{j=1}^K \Pr[X_j | Y]

:math:`\Pr[Y]` と :math:`\Pr[X_j|Y]` の分布がカテゴリ分布（離散分布）である場合，学習すべき単純ベイズのパラメータは次のとおりです．

.. math::
   :label: nbayes1-param

   \Pr[y],&\quad y=1, \ldots, C\\
   \Pr[x_j | y],&\quad y=1,\ldots,C,\;x_j=1,\ldots,F_j,\; j=1,\ldots,K

さらに，ここでは実装を容易にするために， :math:`C` や :math:`F_j` は全て 2 に固定します．
すなわち，クラスや各特徴量は全て2値変数となり，これにより :math:`\Pr[Y]` と :math:`\Pr[X_j|Y]` は，カテゴリ分布の特殊な場合であるベルヌーイ分布に従うことなります．

ここで，大きさ :math:`N` のデータ集合 :math:`\mathcal{D}=\{\mathbf{x}_i, y_i\},\,i=1,\ldots,N` が与えられると，対数尤度は次式になります．

.. math::
   :label: nbayes1-likelihood

   \mathcal{L}(\mathcal{D}; \{\Pr[y]\}, \{\Pr[x_j | y]\}) = \sum_{(\mathbf{x}_i, y_i)\in\mathcal{D}} \ln\Pr[\mathbf{x}_i, y_i]

この対数尤度を最大化する最尤推定により 式 :eq:`nbayes1-param` のパラメータを求めます．
クラスの分布のパラメータ群 :math:`\Pr[y]` は次式で計算できます．

.. math::
   :label: nbayes1-pY

   \Pr[y]=\frac{N[y_i=y]}{N},\quad y\in\{0,1\}

ただし， :math:`N[y_i=y]` は，データ集合 :math:`\mathcal{D}` のうち，クラス :math:`y_i` が値 :math:`y` である事例の数です．
もう一つのパラメータ群 :math:`\Pr[x_{ij}=x_{ij}|y_i=y]` は次式となります．

.. math::
   :label: nbayes1-pXgY

   \Pr[x_j | y]=\frac{N[x_{ij}=x_j, y_i=y]}{N[y_i=y]},
   \quad y\in\{0,1\},\;x_j\in\{0,1\},\;j=1,\ldots,K

ただし， :math:`N[x_{ij}=x_j, y_i=y]` は，データ集合 :math:`\mathcal{D}` のうち，クラス :math:`y_i` の値が :math:`y` であり，かつ特徴 :math:`x_{ij}` の値が :math:`x_j` である事例の数です．
以上のパラメータの計算に必要な値 :math:`N` ， :math:`\Pr[x_{ij}=x_{ij}|y_i=y]` ，および  :math:`N[x_{ij}=x_j, y_i=y]` は，データ集合 :math:`\mathcal{D}` に対する分割表を作成すれば計算できます．

予測をするときには，入力ベクトル :math:`\mathbf{x}^\mathrm{new}` が与えられたときのクラス事後確率を最大にするクラスを，次式で求めます．

.. math::
   :label: nbayes1-inferance

   \hat{y} &= \arg\max_y \Pr[y|\mathbf{x}^\mathrm{new}] \\
           &= \arg\max_y \frac{\Pr[y]\Pr[\mathbf{x}^\mathrm{new}|y]}{\sum_{y'} \Pr[y']\Pr[\mathbf{x}^\mathrm{new} | y']} \\
           &= \arg\max_y \Pr[y]\Pr[\mathbf{x}^\mathrm{new}|y] \\
           &= \arg\max_y \Big(\Pr[y]
              \prod_j \Pr[x_j^\mathrm{new}|y]\Big) \\
           &= \arg\max_y \Big(\log\Pr[y] +
              \sum_j \log\Pr[x_j^\mathrm{new}|y]\Big)

この式は，式 :eq:`nbayes1-pY` と :eq:`nbayes1-pXgY` で求めたパラメータを利用して計算できます．
最後に対数をとっているのは，浮動小数点計算では小さな値のかけ算を繰り返すことにより計算結果が不安定になる場合がありますが，この不安定さを避けるためです．