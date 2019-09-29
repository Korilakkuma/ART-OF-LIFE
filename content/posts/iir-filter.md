+++
banner = ""
categories = ["Computer Sience", "Digital Signal Processing", "Mathematics", "Programming"]
date = 2019-09-29T17:51:06+09:00
description = "IIR Filter の概要"
images = []
menu = ""
tags = ["Digital Signal Processing", "Mathematics", "Z-transform", "Digital Filter"]
title = "IIR Filter"
+++

# IIR フィルタとは ?

<b>IIR (Infinite Impulse Response) フィルタ</b>は, FIR フィルタと異なって, 現在の時刻の出力信号に対して, それよりも過去の出力信号がフィードバックされます. したがって, インパルス応答が無限に続きます.

$
\begin{eqnarray}
y(n)=-\sum\_{m=1}^{I}a(m)y(n-m)+\sum\_{m=0}^{J}b(m)x(n-m)
\end{eqnarray}
$

![IIR フィルタ](https://user-images.githubusercontent.com/4006693/66048249-29114a00-e564-11e9-8f5d-839c8ebe9b37.png)

FIR フィルタと比較して, よりよい周波数特性をもつ (つまり, 理想特性に近い) フィルタを実装しやすいというメリットがあります. 逆に, 出力信号がフィードバックされるので, 設計・実装が複雑になるというデメリットがあります.

# Z 変換

FIR フィルタと同様に, IIR フィルタの周波数特性も Z 変換で調べます.

$
\begin{eqnarray}
Y(z)&=&-\sum\_{n=-\infty}^{\infty}\sum\_{m=1}^{I}a(m)y(n-m)z^{-n}+\sum\_{n=-\infty}^{\infty}\sum\_{m=0}^{J}b(m)x(n-m) \\\\\
    &=&-\sum\_{m=1}^{I}a(m)\sum\_{n=-\infty}^{\infty}y(n-m)z^{-n}+\sum\_{m=0}^{J}b(m)\sum\_{n=-\infty}^{\infty}x(n-m)z^{-n} \\\\\
    &=&-\sum\_{m=1}^{I}a(m)\sum\_{n=-\infty}^{\infty}y(n-m)z^{-(n-m)}z^{-m}+\sum\_{m=0}^{J}b(m)\sum\_{n=-\infty}^{\infty}x(n-m)z^{-(n-m)}z^{-m} \\\\\
    &=&-\sum\_{m=1}^{I}a(m)Y(z)z^{-m}+\sum\_{m=0}^{J}b(m)X(z)z^{-m} \\\\\
\end{eqnarray}
$

ここで, $a(m)$ は, $1 \leqq m \leqq I$, $b(m)$ は, $0 \leqq m \leqq J$ で値をもつので,

$
\begin{eqnarray}
Y(z)&=&-\sum\_{m=-\infty}^{\infty}a(m)Y(z)z^{-m}+\sum\_{m=-\infty}^{\infty}b(m)X(z)z^{-m}
\end{eqnarray}
$

フィルタ係数, $a(m)$ と $b(m)$ の Z 変換は以下のように定義されます.

$
\begin{eqnarray}
A(z)&=&\sum\_{m=-\infty}^{\infty}a(m)z^{-m} \\\\\
B(z)&=&\sum\_{m=-\infty}^{\infty}b(m)z^{-m} \\\\\
\end{eqnarray}
$

したがって, IIR フィルタの Z 変換は以下のように定義できます.

$
\begin{eqnarray}
Y(z)&=&-A(z)Y(z)+B(z)X(z)
\end{eqnarray}
$

よって, 伝達関数は,

$
\begin{eqnarray}
H(z)&=&\frac{Y(z)}{X(z)}=\frac{B(z)}{1+A(z)}
\end{eqnarray}
$

となります.

# 双 1 次変換法

ある周波数特性をもつ IIR フィルタを設計する方法として, アナログフィルタの伝達関数を利用する方法があります.

ここで, 伝達関数が以下のように定義されるアナログフィルタの LPF を例に, IIR フィルタを設計します.

$
\begin{eqnarray}
H(s)&=&\frac{4{\pi}^{2}f\_{c}^{2}}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

ここで, $f\_{c}$ は, <b>遮断周波数</b> (または, <b>カットオフ周波数</b>) と呼ばれ, 通過域と阻止域の境界を表します. $Q$ は, <b>クオリティファクタ</b>と呼ばれ, 遮断周波数における増幅率を表します (一般的に, 遮断周波数は, 増幅率が $\frac{1}{\sqrt{2}}$ となる周波数として定義されるので, $Q$ は $\frac{1}{\sqrt{2}}$ となります.

ところで, 上記の伝達関数は $s$ を変数として定義されています. $s$ を $z$ におきかえて, アナログフィルタの伝達関数を IIR フィルタの伝達関数に変換する方法として利用される 1 つの方法が, <b>双 1 次変換法</b>です. (<b>s - z 変換</b>とも呼ばれます. 数学的観点で簡単に解説すると, アナログ信号 $x(t)$ を<b>ラプラス変換</b>した信号を $X(s)$ とし, デジタル信号 $x(n)$ を Z 変換した信号を $X(z)$ とすると, $X(s)$ から $X(z)$ に変換するための処理です).

$
\begin{eqnarray}
s=\frac{1-z^{-1}}{1+z^{-1}}
\end{eqnarray}
$

このおきかえを適用すると,

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

これらの式に, $f\_{c}$ と $Q$ を指定すると, それに応じた周波数特性の IIR フィルタを設計することができます. ただし, これらの式の $f\_{c}$ はアナログフィルタの周波数なので変換が必要です. そこで, 以下の関係を利用します.

$
\begin{eqnarray}
f\_{a}=\frac{1}{2{\pi}}\tan\left(\frac{{\pi}f\_{d}}{f\_{s}}\right)
\end{eqnarray}
$

この関係は,

$
\begin{eqnarray}
s=\frac{1-z^{-1}}{1+z^{-1}}
\end{eqnarray}
$

$s=j2{\pi}f\_{a}$, $z=\exp\left(j2{\pi}f\_{a}\right)$ とおきかえると導出できます.

LPF フィルタと同様に, 双 1 次変換法を適用することで, HPF, BPF, BEF も設計することができます.

## HPF

$
\begin{eqnarray}
H(s)&=&\frac{s^{2}}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{1}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{1}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

## BPF

$
\begin{eqnarray}
H(s)&=&\frac{2{\pi}\left(f\_{c2}-f\_{c1}\right)s}{s^{2}+2{\pi}\left(f\_{c2}-f\_{c1}\right)s+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{2{\pi}\left(f\_{c2}-f\_{c1}\right)}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&0
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{-2{\pi}\left(f\_{c2}-f\_{c1}\right)}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c1}f\_{c2}-2}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-2{\pi}\left(f\_{c2}-f\_{c1}\right)+4{\pi}^{2}f\_{c1}f\_{c2}}{1+2{\pi}\left(f\_{c2}-f\_{c1}\right)+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

## BEF

$
\begin{eqnarray}
H(s)&=&\frac{s^{2}+4{\pi}^{2}f\_{c1}f\_{c2}}{s^{2}+2{\pi}\left(f\_{c2}-f\_{c1}\right)s+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{4{\pi}^{2}f\_{c1}f\_{c2}+1}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{8{\pi}^{2}f\_{c1}f\_{c2}-2}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{4{\pi}^{2}f\_{c1}f\_{c2}+1}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c1}f\_{c2}-2}{1+{2{\pi}\left(f\_{c2}-f\_{c1}\right)}+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-2{\pi}\left(f\_{c2}-f\_{c1}\right)+4{\pi}^{2}f\_{c1}f\_{c2}}{1+2{\pi}\left(f\_{c2}-f\_{c1}\right)+4{\pi}^{2}f\_{c1}f\_{c2}}
\end{eqnarray}
$

## 共振フィルタとノッチフィルタ

BPF と BEF に関しては, それぞれ, <b>共振フィルタ</b>と<b>ノッチフィルタ</b>と呼ばれる定義があります. これらのフィルタでは, $f\_{c}$ と $Q$ を以下のように定義します.

$
\begin{eqnarray}
\frac{f\_{c}}{Q}=f\_{c2}-f\_{c1}
\end{eqnarray}
$

$
\begin{eqnarray}
f\_{c}^{2}=f\_{c1}f\_{c2}
\end{eqnarray}
$

このように定義した場合, $f\_{c}$ は<b>中心周波数</b>と呼ばれ, $Q$ によって帯域幅を調整することが可能になります ($Q$ が大きいほど帯域幅は小さくなり, $Q$ が小さいほど帯域幅は大きくなります).

### 共振フィルタ

$
\begin{eqnarray}
H(s)&=&\frac{\frac{2{\pi}f\_{c}}{Q}s}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{\frac{2{\pi}f\_{c}}{Q}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&0
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{-\frac{2{\pi}f\_{c}}{Q}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

### ノッチフィルタ

$
\begin{eqnarray}
H(s)&=&\frac{s^{2}+4{\pi}^{2}f\_{c}^{2}}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{4{\pi}^{2}f\_{c}^{2}+1}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{4{\pi}^{2}f\_{c}^{2}+1}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

![IIR フィルタ (LPF, HPF, BPF, BEF)](https://user-images.githubusercontent.com/4006693/66048644-e00dc580-e564-11e9-9056-4dc98c761d47.png)

# 実装

IIR フィルタによる LPF, HPF, BPF, BEF の実装 (C++) です.

```c++
#include <vector>
#include <cmath>

void IIR_LPF(double fd, double Q, std::vector<double> &a, std::vector<double> &b) {
  double fa = std::tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - (2.0 * M_PI * fa) / Q + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
  b[1] = (8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
  b[2] = (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
}

void IIR_HPF(double fd, double Q, std::vector<double> &a, std::vector<double> &b) {
  double fa = tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - (2.0 * M_PI * fa) / Q + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = 1 / numerator;
  b[1] = -2 / numerator;
  b[2] = 1 / numerator;
}

void IIR_BPF(double fd, double Q, std::vector<double> &a, std::vector<double> &b) {
  double fa = tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - (2.0 * M_PI * fa) / Q + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = (2 * M_PI * fa / Q) / numerator;
  b[1] = 0;
  b[2] = -(2 * M_PI * fa / Q) / numerator;
}

void IIR_BEF(double fd, double Q, std::vector<double> &a, std::vector<double> &b) {
  double fa = tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - (2.0 * M_PI * fa) / Q + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = (4 * std::pow(M_PI, 2) * std::pow(fa, 2) + 1) / numerator;
  b[1] = (8 * std::pow(M_PI, 2) * std::pow(fa, 2) - 2) / numerator;
  b[2] = (4 * std::pow(M_PI, 2) * std::pow(fa, 2) + 1) / numerator;
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include <string>
#include <cmath>
#include "wave.h"
#include "iir_filters.h"

enum {
  I = 2,
  J = 2
};

int main(int argc, char **argv) {
  if (argc != 3) {
    std::cerr << "Require type, fd" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  std::string type = argv[1];

  STEREO_PCM pcm0, pcm1;
  std::vector<double> a(3);
  std::vector<double> b(3);

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = pcm0.length;

  double fd = std::stod(argv[2]) / pcm0.fs;
  double Q  = 1.0 / std::sqrt(2.0);

  if (type == "lpf") {
    IIR_LPF(fd, Q, a, b);
  } else if (type == "hpf") {
    IIR_LPF(fd, Q, a, b);
  } else if (type == "bpf") {
    IIR_BPF(fd, Q, a, b);
  } else if (type == "bef") {
    IIR_BEF(fd, Q, a, b);
  }

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  for (int n = 0; n < pcm1.length; n++) {
    for (int m = 0; m <= J; m++) {
      if ((n - m) >= 0) {
        pcm1.sL[n] += b[m] * pcm0.sL[n - m];
        pcm1.sR[n] += b[m] * pcm0.sR[n - m];
      }
    }

    for (int m = 1; m <= I; m++) {
      if ((n - m) >= 0) {
        pcm1.sL[n] += -a[m] * pcm1.sL[n - m];
        pcm1.sR[n] += -a[m] * pcm1.sR[n - m];
      }
    }
  }

  wave_write(&pcm1, "iir_filter.wav");
}

```

# フレーム単位のフィルタリング

コンピュータのメモリは有限なので, 音データがあまりに長いと, 音データを一度にメモリに読み込んでフィルタリングすることが難しくなります. そこで, 音データを小さなフレームに分割して, フレーム単位でフィルタリングする必要があります. フレームに分割するので, フレームとフレームの接続を考慮する必要があります.

IIR フィルタの場合には, 直前のフレームの音データだけでなく, 直前のフィルタリング結果も考慮して, フィルタリングする必要があります.

![フレーム単位のフィルタリング](https://user-images.githubusercontent.com/4006693/66061100-b4e1a100-e579-11e9-9f68-e7622b17688e.png)

## 実装

```c++
#include <vector>

void iir_frame_filter(
  std::vector<double> &s0,
  std::vector<double> &s1,
  std::vector<double> &a,
  std::vector<double> &b,
  int length, 
  int J,
  int I,
  int L) {
  int number_of_frames = length / L;

  std::vector<double> x(L + J);
  std::vector<double> y(L + I);

  for (int frame = 0; frame < number_of_frames; frame++) {
    int offset = L * frame;

    for (int n = 0; n < L + J; n++) {
      if ((offset - J + n) < 0) {
        x[n] = 0.0;
      } else {
        x[n] = s0[offset - J + n];
      }
    }

    for (int n = 0; n < L + I; n++) {
      if ((offset - I + n) < 0) {
        y[n] = 0.0;
      } else {
        y[n] = s1[offset - I + n];
      }
    }

    for (int n = 0; n < L; n++) {
      for (int m = 0; m <= J; m++) {
        y[I + n] += b[m] * x[J + (n - m)];
      }

      for (int m = 1; m <= I; m++) {
        y[I + n] += -a[m] * y[I + (n - m)];
      }
    }

    for (int n = 0; n < L; n++) {
      s1[offset + n] = y[I + n];
    }
  }
}
```

実行例

```c++
#include <cstdio>
#include <cstdlib>
#include <cmath>
#include <vector>
#include "wave.h"
#include "iir_filters.h"
#include "frame_filters.h"

enum {
  L = 256,
  J = 2,
  I = 2
};

int main(int argc, char **argv) {
  if (argc != 2) {
    std::cerr << "Require fd" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  STEREO_PCM pcm0, pcm1;
  std::vector<double> a(3);
  std::vector<double> b(3);

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = pcm0.length;

  double fd = std::stod(argv[1]) / pcm0.fs;
  double Q  = 1.0 / std::sqrt(2.0);

  IIR_LPF(fd, Q, a, b);

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  iir_frame_filter(pcm0.sL, pcm1.sL, a, b, pcm0.length, J, I, L);
  iir_frame_filter(pcm0.sR, pcm1.sR, a, b, pcm0.length, J, I, L);

  wave_write(&pcm1, "iir_frame_filter.wav");
}
```

# イコライザ

<b>イコライザ</b>は, 周波数特性を加工して, 好みの音色をつくりだすためのエフェクターです. イコライザは, 低域の周波数成分を強調する<b>ローシェルビングフィルタ</b>, 高域の周波数成分を強調する<b>ハイシェルビングフィルタ</b>, 特定の周波数帯域を強調する<b>ピーキングフィルタ</b>の 3 種類のフィルタによって構成されます.

![ローシェルビングフィルタ, ハイシェルビングフィルタ, ピーキングフィルタ](https://user-images.githubusercontent.com/4006693/66130390-a0121580-e62c-11e9-9ae6-ad5e3c6e7d53.png)

## ローシェルビングフィルタ

$
\begin{eqnarray}
H(s)&=&\frac{s^{2}+\sqrt{1+g}\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}\left(1+g\right)}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{1+\sqrt{1+g}\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}\left(1+g\right)}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}\left(1+g\right)-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{1-\sqrt{1+g}\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}\left(1+g\right)}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

## ハイシェルビングフィルタ

$
\begin{eqnarray}
H(s)&=&\frac{\left(1+g\right)s^{2}+\sqrt{1+g}\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{\left(1+g\right)+\sqrt{1+g}\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2\left(1+g\right)}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{\left(1+g\right)-\sqrt{1+g}\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

## ピーキングフィルタ

$
\begin{eqnarray}
H(s)&=&\frac{s^{2}+\frac{2{\pi}f\_{c}}{Q}\left(1+g\right)s+4{\pi}^{2}f\_{c}^{2}}{s^{2}+\frac{2{\pi}f\_{c}}{Q}s+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
H(z)&=&\frac{b(0)+b(1)z^{-1}+b(2)z^{-2}}{1+a(1)z^{-1}+a(2)z^{-2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(0)&=&\frac{1+\frac{2{\pi}f\_{c}}{Q}\left(1+g\right)+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
b(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}\left(1+g\right)+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(1)&=&\frac{8{\pi}^{2}f\_{c}^{2}-2}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

$
\begin{eqnarray}
a(2)&=&\frac{1-\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}{1+\frac{2{\pi}f\_{c}}{Q}+4{\pi}^{2}f\_{c}^{2}}
\end{eqnarray}
$

これらのフィルタには, $g$ という所定の帯域の増幅率を設定できるパラメータ (一般的に, <b>ゲイン</b>と呼ばれています) があります. ローシェルビングフィルタ, および, ハイシェルビングフィルタでは, $Q$ を $\frac{1}{\sqrt{2}}$ に設定することが一般的です. ピーキングフィルタでは, $Q$ によって帯域幅を調整することが可能です($Q$ が大きいほど帯域幅は小さくなり, $Q$ が小さいほど帯域幅は大きくなります).

いずれのフィルタでも, 帯域を減衰させる (<b>カット</b>と呼びます) のは, $-1 \leqq g < 0$ の範囲に設定します. 一方で, 帯域を増幅させる (<b>ブースト</b>と呼びます) には, $0 < g$ に設定します.

![3 バンドイコライザ](https://user-images.githubusercontent.com/4006693/66130511-c59f1f00-e62c-11e9-88e6-4c839ac7e0cc.png)

## 実装

```c++
#include <vector>
#include <cmath>

void IIR_LOW_SHELVING(double fd, double Q, double g, std::vector<double> &a, std::vector<double> &b) {
  double fa = std::tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = (1 + std::sqrt(1 + g) * ((2 * M_PI * fa) / Q) + 4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
  b[1] = (8.0 * std::pow(M_PI, 2) * std::pow(fa, 2) * (1 + g) - 2) / numerator;
  b[2] = (1 - std::sqrt(1 + g) * ((2 * M_PI * fa) / Q) + 4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
}

void IIR_HIGH_SHELVING(double fd, double Q, double g, std::vector<double> &a, std::vector<double> &b) {
  double fa = std::tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = ((1 + g) + std::sqrt(1 + g) * ((2 * M_PI * fa) / Q) + 4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
  b[1] = (8 * std::pow(M_PI, 2) * std::pow(fa, 2) - 2 * (1 + g)) / numerator;
  b[2] = ((1 + g) - std::sqrt(1 + g) * ((2 * M_PI * fa) / Q) + 4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
}

void IIR_PEAKING(double fd, double Q, double g, std::vector<double> &a, std::vector<double> &b) {
  double fa = std::tan(M_PI * fd) / (2.0 * M_PI);
  double numerator = 1.0 + ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2));

  a[0] = 1.0;
  a[1] = ((8.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) - 2.0) / numerator;
  a[2] = (1.0 - ((2.0 * M_PI * fa) / Q) + (4.0 * std::pow(M_PI, 2) * std::pow(fa, 2))) / numerator;
  b[0] = (1 + ((2 * M_PI * fa) / Q) * (1 + g) + 4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
  b[1] = (8.0 * std::pow(M_PI, 2) * std::pow(fa, 2) - 2) / numerator;
  b[2] = (1 - ((2 * M_PI * fa) / Q) * (1 + g) + 4.0 * std::pow(M_PI, 2) * std::pow(fa, 2)) / numerator;
}
```

イコライザ

```c++
#include <vector>
#include "iir_filters.h"

enum {
  NUMBER_OF_BANDS = 3,
  BASS   = 500,
  MIDDLE = 1000,
  TREBLE = 2000
};

void Equalizer(
  std::vector<double> &s0,
  std::vector<double> &s1,
  double fs,
  int length,
  int I,
  int J,
  double bass,
  double middle,
  double treble,
  double Q) {
  std::vector<std::vector<double>> A(NUMBER_OF_BANDS, std::vector<double>(3));
  std::vector<std::vector<double>> B(NUMBER_OF_BANDS, std::vector<double>(3));
  std::vector<double> a(3);
  std::vector<double> b(3);

  IIR_LOW_SHELVING(static_cast<double>(BASS) / fs, Q, bass, a, b);

  for (int m = 0; m <= I; m++) {
    A[0][m] = a[m];
  }

  for (int m = 0; m <= J; m++) {
    B[0][m] = b[m];
  }

  IIR_PEAKING(static_cast<double>(MIDDLE) / fs, Q, middle, a, b);

  for (int m = 0; m <= I; m++) {
    A[1][m] = a[m];
  }

  for (int m = 0; m <= J; m++) {
    B[1][m] = b[m];
  }

  IIR_HIGH_SHELVING(static_cast<double>(TREBLE) / fs, Q, treble, a, b);

  for (int m = 0; m <= I; m++) {
    A[2][m] = a[m];
  }

  for (int m = 0; m <= J; m++) {
    B[2][m] = b[m];
  }

  for (int i = 0; i < NUMBER_OF_BANDS; i++) {
    s1.assign(length, 0.0);

    for (int n = 0; n < length; n++) {
      for (int m = 0; m <= J; m++) {
        if ((n - m) >= 0) {
          s1[n] = s1[n] + (B[i][m] * s0[n - m]);
        }
      }

      for (int m = 1; m <= I; m++) {
        if ((n - m) >= 0) {
          s1[n] = s1[n] + (-A[i][m] * s1[n - m]);
        }
      }
    }

    s0 = s1;
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include <cmath>
#include "wave.h"
#include "equalizer.h"

enum {
  I = 2,
  J = 2
};

int main(int argc, char **argv) {
  if (argc != 4) {
    std::cerr << "Require bass, middle, treble" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  STEREO_PCM pcm0, pcm1;

  double bass   = std::stod(argv[1]);
  double middle = std::stod(argv[2]);
  double treble = std::stod(argv[3]);
  double Q      = 1.0 / std::sqrt(2.0);

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = pcm0.length;

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  Equalizer(pcm0.sL, pcm1.sL, pcm1.fs, pcm1.length, I, J, bass, middle, treble, Q);
  Equalizer(pcm0.sR, pcm1.sR, pcm1.fs, pcm1.length, I, J, bass, middle, treble, Q);

  wave_write(&pcm1, "equalizer.wav");
}
```

# リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
