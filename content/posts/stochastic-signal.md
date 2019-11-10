+++
banner = ""
categories = ["Digital Signal Processing", "Mathematics"]
date = 2019-10-20T17:02:31+09:00
description = ""
images = []
menu = ""
tags = ["Digital Signal Processing", "Mathematics", "Stochastic Signal"]
title = "確率信号"
+++

音声信号などにおいて, ある時刻における振幅を知りたい場合, 実際に発音してからでないと知ることはできません. しかしながら, ある程度の時間をかけて, 音声信号を観測すると, 統計的な性質がつかめます.

# 確定信号

<b>確定信号</b> (Deterministic Signal) とは, 時刻や位置の関数として表される信号で, 時刻や位置を指定すると, 特定の値が求められます. 典型的な例としては, 正弦波信号があります.

# 確率信号

<b>確率信号</b> (Stochastic Singnal) とは, <b>確率過程</b> (Stochastic Process) としてあつかうことによって, 過去や未来の信号の統計量を特定する信号のことです. したがって, 確定信号のように, 時間や位置の関数として表すことはできません. そして, 身の回りで発生している音のほとんどは, 確率信号です.

## 実現値

確率信号は, その定義上, 実際に発生した値のみが数値として表すことができます. この値を, <b>実現値</b> (Value) と呼びます.

試行回数 $k$ 依存しない確率信号の場合, 実現値を多数観測することで, 値の傾向を知ることができます.

## ヒストグラム

未来の確率信号の値を特定することはできませんが, どのような傾向で実現値が発生するかを知ることはできます. そのためには, 確率信号の実現値の傾向をすることが有効です. 一般的には, <b>ヒストグラム</b>を利用して, その傾向を視覚化します.

![ヒストグラムの例](https://user-images.githubusercontent.com/4006693/67201823-26f32a80-f442-11e9-8a33-c8bcd351dbcd.png)

## 確率密度関数

確率信号の値の傾向がわかっている場合, 「<b>確率分布</b> があたえられている」と表現します. 微小区間の値の確率を表した関数を<b>確率密度関数</b> (PDF: Probability Density Function) と呼びます.

確率信号 $x$ の PDF を $p(x)$ とすると, 以下の式が成立します.

$
\begin{eqnarray}
\int\_{-\infty}^{\infty}p(x)dx=1
\end{eqnarray}
$

これは, 確率信号 $x$ のいずれかの実現値が必ず発生することを意味しています

### ガウス分布

代表的な PDF に, <b>ガウス分布</b> (Gaussian Distribution) (<b>正規分布</b> (Normal Distribution)) と呼ばれる関数があります. 確率信号を $x$ とすると, ガウス分布は以下の関数で表されます.

$
\begin{eqnarray}
p(x)=\frac{1}{\sqrt{2{\pi}\delta^{2}}}\exp\left(-\frac{(x-{\mu})^{2}}{2\delta^{2}}\right)
\end{eqnarray}
$

$\mu$ はガウス分布の中心を決定するパラメータで, <b>平均</b> (Mean) と呼びます. $\delta^2$ は分布の広がりを決定するパラメータで, <b>分散</b> (Variance) と呼びます. ガウス分布では, この 2 つのパラメータ (平均と分散) を指定すると, その分布が一様に定まるという特徴があります.

ガウス分布の実装 (C++)

```c++
#include <vector>
#include <cmath>

void Gaussian(std::vector<double> &dist, double mean, double variance, int N) {
  for (int n = 0; n < N; n++) {
    dist[n] = (1 / std::sqrt(2 * M_PI * variance)) * std::exp(-1 * (std::pow((n - mean), 2) / (2 * variance)));
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include <vector>
#include "gaussian.h"

enum {
  N = 10
};

int main(int argc, char **argv) {
  if (argc != 3) {
    std::cerr << "Require mean, variance" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  std::vector<double> dist(N);

  double mu     = std::stod(argv[1]);
  double delta2 = std::stod(argv[2]);

  Gaussian(dist, mu, delta2, N);

  for (int n = 0; n < N; n++) {
    std::cout << dist[n] << std::endl;
  }
}
```

## 確率分布関数

確率分布があたえられている場合, 負の方向から積分した関数を<b>確率分布関数</b> (Probability Distribution Function) と呼びます.

確率信号 $x$ がしたがう PDF を $p(x)$ とすると, 確率分布関数 $F(A)$ は,

$
\begin{eqnarray}
f(A)=\int\_{-\infty}^{A}p(x)dx=1
\end{eqnarray}
$

さらに, 確率密度関数の性質より,

$
\begin{eqnarray}
\int\_{-\infty}^{\infty}p(x)dx=1
\end{eqnarray}
$

であるので, $F(A)$ にも以下の性質があります.

$
\begin{eqnarray}
F(\infty)=\int\_{-\infty}^{\infty}p(x)dx=1
\end{eqnarray}
$

また, 確率分布関数 $F(A)$ が既知であれば, 微分によって PDF を求めることが可能です.

$
\begin{eqnarray}
p(x)=\frac{dF(x)}{dx}
\end{eqnarray}
$

## 結合確率密度関数

複数の確率信号が同時に発生する場合の PDF を考えます. 複数の確率信号 $x\_{1}, x\_{2}, \cdots, x\_{M}$ が同時に発生する場合の PDF を $p(x\_{1}, x\_{2}, \cdots, x\_{M})$ と表す場合, これを<b>結合確率密度関数</b> (Join PDF) と呼びます.

同時に $M$ 個の確率信号が発生すると, いずれかの実現値が, ほかの実現値に影響を与える可能性があるので, ここの PDF が既知でも, $p(x\_{1}, x\_{2}, \cdots, x\_{M})$ を求めることは困難です.

## 期待値

実現値以外に, 確率信号を記述する方法としては, <b>期待値</b> (Expectation Value) が有効です.

確率信号を $x$, その PDF を $p(x)$ とすると, その期待値 $E\left[x\right]$ は,

$
\begin{eqnarray}
E\left[x\right]=\int\_{-\infty}^{\infty}xp(x)dx
\end{eqnarray}
$

また, $x\_{k}$ を $k$ 回目の試行で得られた実現値であるとすると, 以下のように求めることもできます.

$
\begin{eqnarray}
E\left[x\right]=\lim\_{N \to \infty}\frac{1}{N}\sum\_{k=1}^{N-1}x\_{k}
\end{eqnarray}
$

実は, 期待値 $E\left[x\right]$ は, 実現値の平均を表しています. , 分散は以下の式で定義できます.

$
\begin{eqnarray}
E\left[(x-{\mu})^{2}\right]=\int\_{-\infty}^{\infty}(x-{\mu})^{2}p(x)dx
\end{eqnarray}
$

## 時間平均とエルゴード過程

1 回の試行で 1 つの値を観測するのではなく, 1 回の試行で複数個の値を順に取得する場合を考えます (配列を取得するイメージ).

時刻を $n$, $k$ 回目の試行における, 時刻 $n$ の実現値を $x\_{k}(n)$ とすると, <b>時間平均</b> (Time Average) は以下のように定義できます.

$
\begin{eqnarray}
\bar{x\_{k}}=\lim\_{N \to \infty}\frac{1}{N}\sum\_{n=0}^{N-1}x\_{k}(n)
\end{eqnarray}
$

期待値と似ていますが, 期待値と時間平均はほとんどの場合において異なります. 具体的には, 確率分布が時間とともに変化する確率信号の場合です (逆に, 確率分布が不変である確率信号に限って, 時間平均と期待値は一致します).

![期待値と時間平均のちがい](https://user-images.githubusercontent.com/4006693/67203147-57889380-f445-11e9-8efd-51009b6ab70b.png)

音声信号をあつかう場合には, 試行回数が 1 回しかないことが多く, 期待値を求めることはできません. そこで, 観測信号の<b>期待値と時間平均が一致すると過程</b>して, 期待値を時間平均で代用します.

$
\begin{eqnarray}
E\left[x(n)\right]=\lim\_{N \to \infty}\frac{1}{N}\sum\_{n=0}^{N-1}x(n)
\end{eqnarray}
$

上記の式が成立するような場合で, このような性質をもつ信号系列を<b>エルゴード過程</b> (Ergodic Process) と呼びます.

## 共分散と無相関

確率信号 $x\_{1}$ の平均を $\mu\_{1}$ として, 確率信号 $x\_{2}$ の平均を $\mu\_{2}$ とすると, $x\_{1}$ と $x\_{2}$ の平均からのずれの積の期待値, すなわち, 下記の式は,

$
\begin{eqnarray}
E\left[\left(x\_{1}-\mu\_{1}\right)\left(x\_{2}-\mu\_{2}\right)\right]=\int\_{-\infty}^{\infty}\int\_{-\infty}^{\infty}\left(x\_{1}-\mu\_{1}\right)\left(x\_{2}-\mu\_{2}\right)p\left(x\_{1},x\_{2}\right)dx\_{1}dx\_{2}
\end{eqnarray}
$

分散との対比から, $x\_{1}$ と $x\_{2}$ の<b>共分散</b> (Covariance) と呼びます. 共分散は $x\_{1}$ と $x\_{2}$ が同じ値になるとき最も大きくなるので, <b>信号の関連性</b>を調べるときに有用になります.

共分散が 0 の場合,

$
\begin{eqnarray}
E\left[\left(x\_{1}-\mu\_{1}\right)\left(x\_{2}-\mu\_{2}\right)\right]=0
\end{eqnarray}
$

$x\_{1}$ と $x\_{2}$ は, <b>無相関</b> (Uncorrelated) であると呼びます. 無相関である場合, 以下の式が成立します.

$
\begin{eqnarray}
E\left[x\_{1}x\_{2}\right]=E\left[x\_{1}\right]E\left[x\_{2}\right]
\end{eqnarray}
$

上記の式は, 無相関である場合, 2 つの確率信号の期待値は, それぞれの期待値の積で求めることができるということを意味しています.

確率信号をあつかう場合, あらかじめ期待値を差し引いて, 期待値を 0 とすることが多いです. この場合, 無相関の条件は, 以下のように定義できます.

$
\begin{eqnarray}
E\left[x\_{1}x\_{2}\right]=0
\end{eqnarray}
$

## 独立

複数の確率信号をあつかう場合, それらの和の期待値は, それぞれの期待値の和に等しくなります.

$
\begin{eqnarray}
E\left[x\_{1}+x\_{2}+\cdots+x\_{M}\right]=E\left[x\_{1}\right]+E\left[x\_{2}\right]+\cdots+E\left[x\_{M}\right]
\end{eqnarray}
$

しかしながら, 複数の確率信号の積の期待値は, それぞれにあつかうことができません.

$
\begin{eqnarray}
E\left[x\_{1}x\_{2}{\cdots}x\_{M}\right]=\int\_{-\infty}^{\infty}\int\_{-\infty}^{\infty}\cdots\int\_{-\infty}^{\infty}x\_{1}x\_{2}{\cdots}x\_{M}p\left(x\_{1},x\_{2},{\cdots},x\_{M}\right)dx\_{1}dx\_{2}{\cdots}dx\_{M}
\end{eqnarray}
$

もし, それぞれの PDF の積として定義できる場合, 確率信号 $x\_{1},x\_{2},{\cdots},x\_{M}$ は互いに<b>独立</b> (Independent) であると呼びます. 独立である場合, それぞれの期待値の積として定義できます.

$
\begin{eqnarray}
E\left[x\_{1}x\_{2}{\cdots}x\_{M}\right]=E\left[x\_{1}\right]E\left[x\_{2}\right]{\cdots}E\left[x\_{M}\right]
\end{eqnarray}
$

独立である場合, 下記の式も成立します.

$
\begin{eqnarray}
E\left[x\_{1}x\_{2}\right]=E\left[x\_{1}\right]E\left[x\_{2}\right]
\end{eqnarray}
$

<b>独立であれば無相関でもあります</b>. その<b>逆</b>は, 必ずしも真ではありません. すなわち, 無相関であっても, その積の期待値が, それぞれの PDF の積として定義できるはかぎらないということです.

## 独立と無相関のちがい

![無相関](https://user-images.githubusercontent.com/4006693/68540983-29abc480-03dd-11ea-9ae7-cb3f8809ee54.png)

無相関の場合, $x\_{1}$ と $x\_{2}$ のいずれかが正か負の大きな値をとる場合, 他方は小さな値しか撮りません. このように, 一方の実現値が他方の実現値に影響をあたえているので, 2 つの確率信号は独立ではありません.

![独立](https://user-images.githubusercontent.com/4006693/68543656-7a331a00-03fd-11ea-900f-7cb55a109541.png)

独立の場合, $x\_{1}$ と $x\_{2}$ のいずれかが正か負の大きな値をとる場合, 他方は影響をうけていません. このように, 一方の実現値が他方の実現値に影響をあたえていないので, 2 つの確率信号は独立となります.

## 定常と非定常

ある確率信号の統計的性質が, 時刻 $n$ とともに変化しない場合, その確率信号は, <b>定常</b> (Stationary) と呼びます. 定常はさらに, 2 つの状態, <b>弱定常</b> (Wide Sense Stationary) と<b>強定常</b> (Strictly Sense Stationary) に分類されます.

$
\begin{eqnarray}
E\left[x(n)\right]=\mu
\end{eqnarray}
$

$
\begin{eqnarray}
E\left[x(n)x(n-k)\right]=r(k)
\end{eqnarray}
$

上記の 2 式で定義されるように, 期待値が時間に関わらず一定で, 時間差 $k$ のみの関数となるとき, 確率信号は弱定常と呼ばれます.

高次の結合確率密度関数が,

$
\begin{eqnarray}
p\left(x\left(n\_{1}\right),x\left(n\_{2}\right),{\cdots},x\left(n\_{M}\right)\right)=p\left(x\left(n\_{1}+t\right),x\left(n\_{2}+t\right),{\cdots},x\left(n\_{M}+t\right)\right)
\end{eqnarray}
$

上記の式のように, 時間によって変化しない場合は, 強定常と呼ばれます.

確率信号の統計的性質が時間的に変化する場合は, <b>非定常</b> (Non-stationary) と呼びます.

## 中心極限定理

互いに独立で, 同一の確率分布にしたがう複数の確率信号を <b>i.i.d</b> (Independent Identically Distributed) な確率信号と呼びます.

そのような確率信号の場合, <b>中心極限定理</b> (Central Limit Theorem) が成立します.

平均 $\mu$, 分散 $\delta^{2}$ の i.i.d. な確率信号 $x\_{1},x\_{2},\cdots$に対して, それらの和としてあたえられる確率信号は,

$
\begin{eqnarray}
s\_{n}=\frac{1}{n}\sum\_{k=1}^{n}x\_{k}
\end{eqnarray}
$

$n$ が十分に大きい場合, 近似的に, 平均 $\mu$, 分散 $\frac{\delta^{2}}{n}$ のガウス分布にしたがいます. つまり, 確率信号の和がしたがう PDF は, もとの PDF よりもガウス分布に近づくということになります.

# リファレンス

- [ディジタル音声&画像の圧縮/伸長/加工技術: 大容量化するマルチメディア・データを転送・保存・活用するために (ディジタル信号処理シリーズ)](https://www.amazon.co.jp/%E3%83%87%E3%82%A3%E3%82%B8%E3%82%BF%E3%83%AB%E9%9F%B3%E5%A3%B0-%E7%94%BB%E5%83%8F%E3%81%AE%E5%9C%A7%E7%B8%AE-%E5%8A%A0%E5%B7%A5%E6%8A%80%E8%A1%93-%E5%A4%A7%E5%AE%B9%E9%87%8F%E5%8C%96%E3%81%99%E3%82%8B%E3%83%9E%E3%83%AB%E3%83%81%E3%83%A1%E3%83%87%E3%82%A3%E3%82%A2%E3%83%BB%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E8%BB%A2%E9%80%81%E3%83%BB%E4%BF%9D%E5%AD%98%E3%83%BB%E6%B4%BB%E7%94%A8%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AB-%E3%83%87%E3%82%A3%E3%82%B8%E3%82%BF%E3%83%AB%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA/dp/4789831450)
