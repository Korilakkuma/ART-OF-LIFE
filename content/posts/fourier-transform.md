+++
banner = ""
categories = ["Mathematics", "Programming"]
date = 2019-09-21T17:27:49+09:00
description = "フーリエ変換・離散フーリエ変換・変形離散コサイン変換"
images = []
menu = ""
tags = ["Mathematics", "Fourier Transform", "Discrete Fourier Transform", "DFT", "Modified Discrete Cosine Transform", "MDCT"]
title = "Fourier Transform"
+++

# フーリエ変換

周波数という視点 (ドメイン) から, 音や光の特徴を調べる方法が<b>フーリエ変換</b>です. 音の場合であれば, 基本周波数と倍音の比率などを分析することができます.

![フーリエ変換](https://user-images.githubusercontent.com/4006693/63822772-69d9da00-c98c-11e9-9870-ce3c5805cf24.gif)

フーリエ変換は, $-\infty$ ~ $\infty$ の時刻まで観測したアナログ信号の周波数特性を調べるための数学的処理で, フーリエ変換と, その逆変換である, <b>逆フーリエ変換</b>は以下のように定義されています.

$
\begin{eqnarray}
X(f)=\int\_{-\infty}^{\infty}x(t)\exp(-j2{\pi}ft)dt \quad (-\infty \leqq f \leqq \infty)
\end{eqnarray}
$

$
\begin{eqnarray}
x(t)=\int\_{-\infty}^{\infty}X(t)\exp(j2{\pi}ft)df \quad (-\infty \leqq t \leqq \infty)
\end{eqnarray}
$

$x(t)$ は, 時間 $t$ を変数とするアナログ信号, $X(f)$ は周波数 $f$ を変数とする $x(t)$ の周波数特性を表しています.

# 離散フーリエ変換

アナログ信号のフーリエ変換では, 無限の長さの連続信号となりますが, コンピュータでは<b>無限大</b>や<b>連続</b>した信号をあつかうことができません. したがって, コンピュータでフーリエ変換を実装する場合には, <b>離散フーリエ変換</b> (<b>DFT</b> : Discrete Fourier Transform) と, その逆変換である<b>逆離散フーリエ変換</b> (<b>IDFT</b> : Inverse Discrete Fourier Transform) を利用します.

$
\begin{eqnarray}
t=nt\_{s} \quad (1 \leqq n \leqq N-1)
\end{eqnarray}
$

$
\begin{eqnarray}
f=\frac{kf\_{s}}{N} \quad (f \leqq k \leqq N - 1)
\end{eqnarray}
$

と定義できるので,

$
\begin{eqnarray}
X(k)=\sum\_{n=0}^{N-1}x(n)\exp\left(\frac{-j2{\pi}kn}{N}\right)
\end{eqnarray}
$

$
\begin{eqnarray}
x(n)=\frac{1}{N}\sum\_{k=0}^{N-1}X(k)\exp\left(\frac{j2{\pi}kn}{N}\right)
\end{eqnarray}
$

音の場合, $x(n)$ は実数となります. 一方で, $X(k)$ は複素数となります. $X(k)$ を極座標形式で表すと, 以下のようになります.

$
\begin{eqnarray}
X(k)=A(k)\exp(j{\theta}(k))
\end{eqnarray}
$

$
\begin{cases}
\begin{eqnarray}
A(k)     &=&\sqrt{real(X(k))^{2}+imag(X(k))^{2}} \\\\\
\theta(k)&=&\tan^{-1}\left(\frac{imag(X(k))}{real(X(k))}\right) \\\\\
\end{eqnarray}
\end{cases}
$

ここで, $X(k)$ の逆離散フーリエ変換を考えます.

$
\begin{eqnarray}
x(n)=\frac{1}{N}\sum\_{n=0}^{N-1}A(k)exp\left(\frac{-j2{\pi}kn}{N}+j\theta(k)\right) \quad (0 \leqq n \leqq N - 1)
\end{eqnarray}
$

オイラーの公式を適用すると,

$
\begin{eqnarray}
x(n)=\frac{1}{N}\sum\_{n=0}^{N-1}A(k)\left[\cos\left(\frac{2{\pi}kn}{N}+\theta(k)\right)+j\sin\left(\frac{2{\pi}kn}{N}+\theta(k)\right)\right] \quad (0 \leqq n \leqq N - 1)
\end{eqnarray}
$

音の場合, $x(n)$ は, 虚数になるので, 虚数部を消去すると, 以下のようになります.

$
\begin{eqnarray}
x(n)=\frac{1}{N}\sum\_{n=0}^{N-1}A(k)\cos\left(\frac{2{\pi}kn}{N}+\theta(k)\right) \quad (0 \leqq n \leqq N - 1)
\end{eqnarray}
$

この式は, $N$ 個の $\cos$ 波の重ねあわせによって $x(n)$ を合成できることを表しています. したがって, 離散フーリエ変換によって計算される周波数特性 (スペクトル) は, $\cos$ 波の振幅と位相の情報です ($A(k)$ は, <b>振幅スペクトル</b>, $\theta(k)$は, <b>位相スペクトル</b> です).

周波数特性は, $k=\frac{N}{2}$ を中心に, <b>振幅スペクトルは線対称</b>, <b>位相スペクトルは点対称</b>になります. さらに, $k=\frac{N}{2}$ は, $\frac{f\_{s}}{2}$, つまり, サンプリング周波数の $\frac{1}{2}$ に対応し, <b>サンプリング定理</b>にも関係しています.

人間の聴覚は, <b>位相の違いに鈍感</b> なので, 音の周波数特性といえば振幅スペクトルを意味していることがほとんどです (ただし, ホワイトノイズとインパルス音のように位相の違いを分析する必要がある場合もあります).

## 窓関数

離散フーリエ変換では, $x(n)$, $x(k)$ が, 離散フーリエ変換のサイズ $N$ の周期信号という前提があります. つまり, $N$ が $sin$ 波の整数倍となっていれば, 離散フーリエ変換によって求められる周波数特性は, 基本周波数をのみを含んだものになります.

ところが, そうでない場合, 実際の音にはない周波数成分が含まれてしまいます.

この問題を緩和するのが, <b>窓関数</b>です. 窓関数を乗算することで, 端部の不連続点をあいまいにすることができ, 周波数成分の広がりをある程度おさえることができます.

窓関数にはいくつか種類がありますが, 代表的な窓関数として, <b>ハニング窓</b>があり, 以下のように定義されています(ほかには, ブラックマン窓などがあります).

($N$ が偶数の場合)

$
w(n)=\begin{cases}
\begin{eqnarray}
&0.5-0.5\cos\left(\frac{2{\pi}n}{N}\right)& \quad (0 \leqq n \leqq N - 1) \\\\\
&0& \quad (otherwise) \\\\\
\end{eqnarray}
\end{cases}
$

($N$ が奇数の場合)

$
w(n)=\begin{cases}
\begin{eqnarray}
&0.5-0.5\cos\left(\frac{2{\pi}(n+0.5)}{N}\right)& \quad (0 \leqq n \leqq N - 1) \\\\\
&0& \quad (otherwise) \\\\\
\end{eqnarray}
\end{cases}
$

## 実装

C++ で実装した, 離散フーリエ変換の関数です.

```c++
#include <vector>
#include <cmath>
#include <complex>

void DFT(std::vector<std::complex<double>> &x, std::vector<std::complex<double>> &X, int N) {
  for (int k = 0; k < N; k++) {
    double real_sum = 0.0;
    double imag_sum = 0.0;

    for (int n = 0; n < N; n++) {
      double real =  std::cos((2 * M_PI * k * n) / N);
      double imag = -std::sin((2 * M_PI * k * n) / N);

      real_sum += (real * x[n].real()) - (imag * x[n].imag());
      imag_sum += (real * x[n].imag()) + (imag * x[n].real());
    }

    X[k] = std::complex<double>(real_sum, imag_sum);
  }
}

void IDFT(std::vector<std::complex<double>> &x, std::vector<std::complex<double>> &X, int N) {
  for (int n = 0; n < N; n++) {
    double real_sum = 0.0;
    double imag_sum = 0.0;

    for (int k = 0; k < N; k++) {
      double real = std::cos((2 * M_PI * k * n) / N);
      double imag = std::sin((2 * M_PI * k * n) / N);

      real_sum += ((real * X[k].real()) - (imag * X[k].imag()));
      imag_sum += ((real * X[k].imag()) + (imag * X[k].real()));
    }

    x[n] = std::complex<double>((real_sum / N), (imag_sum / N));
  }
}
```

ハニング窓

```c++
#include <vector>
#include <cmath>

void hanning_window(std::vector<double> &w, int N) {
  if ((N % 2) == 0) {
    for (int n = 0; n < N; n++) {
      w[n] = 0.5 - (0.5 * std::cos((2.0 * M_PI * n) / N));
    }
  } else {
    for (int n = 0; n < N; n++) {
      w[n] = 0.5 - (0.5 * std::cos((2.0 * M_PI * (n + 0.5)) / N));
    }
  }
}
```

実行例

```c++
#include <iostream>
#include <vector>
#include <cmath>
#include <complex>
#include "wave.h"
#include "window_functions.h"
#include "dft.h"

enum {
  N = 8
};

int main() {
  MONO_PCM pcm;

  std::vector<std::complex<double>> x(N);
  std::vector<std::complex<double>> X(N, 0.0);
  std::vector<double> w(N);

  WAVE::wave_read(&pcm, "sample.wav");

  hanning_window(w, N);

  for (int n = 0; n < N; n++) {
    double real = w[n] * pcm.s[n];
    x[n] = std::complex<double>(real, 0.0);
  }

  std::cout << std::fixed;

  std::cout << "x(n)" << std::endl;

  for (int n = 0; n < N; n++) {
    std::cout << "[" << n << "]" << " " << x[n].real() << std::endl;
  }

  DFT(x, X, N);

  std::cout << "X(k)" << std::endl;

  for (int k = 0; k < N; k++) {
    std::cout << "[" << k << "]" << " " <<  X[k].real() << " + j" << X[k].imag() << std::endl;
  }

  x.assign(N, 0.0);

  IDFT(x, X, N);

  std::cout << "x(n)" << std::endl;

  for (int n = 0; n < N; n++) {
    std::cout << "[" << n << "]" << " " << x[n].real() << std::endl;
  }
}
```

# 離散コサイン変換

フーリエ変換を利用することで, 周波数解析が可能になります. しかし, フーリエ変換では, その演算結果が複素数になってしまうというデメリットがあります. 実数である時間領域の信号を実数の周波数信号に変換できれば, コンピュータで処理することが容易になります. このセクションでは, MPEG などで利用されている<b>離散コサイン変換</b> (Discrete Cosine Transform) について解説します.

$
\begin{eqnarray}
X(k)=\sqrt{\frac{2}{N}}a\_{K}\sum\_{n=0}^{N-1}x(n)\cos\left(\frac{\left(2n+1\right)k{\pi}}{2N}\right)
\end{eqnarray}
$

$
\begin{eqnarray}
a\_{k}=
\begin{cases}
&1&\quad(k=1,2,\cdots,N-1) \\\\\
&\frac{1}{\sqrt{2}}&\quad(k=0)\\\\\
\end{cases}
\end{eqnarray}
$

また, その逆変換である, 逆離散コサイン変換は以下のように定義されます.

$
\begin{eqnarray}
x(n)=\sqrt{\frac{2}{N}}\sum\_{k=0}^{N-1}a\_{K}X(k)\cos\left(\frac{\left(2n+1\right)k{\pi}}{2N}\right)
\end{eqnarray}
$

$
\begin{eqnarray}
a\_{k}=
\begin{cases}
&1&\quad(k=1,2,\cdots,N-1) \\\\\
&\frac{1}{\sqrt{2}}&\quad(k=0)\\\\\
\end{cases}
\end{eqnarray}
$

# 変形離散コサイン変換

離散コサイン変換では, $N$ 個ずつデータを取得して, 周波数変換, 信号処理, そして, 逆変換を実行します. しかしながら, それでは, データのまとまりに不連続な点が発生してしまいます.

そこで, 信号を窓関数を利用してオーバラップ (隣接するデータのまとまりを重ねて連結していく処理) をしながら, 信号を取得して, 周波数解析をする処理として, <b>変形離散コサイン変換</b> (<b>修正離散コサイン変換</b> (Modified Discrete Cosine Transform)) が利用されます.

![オーバーラップ (赤い矩形の重なった部分がオーバーラップしている部分](https://user-images.githubusercontent.com/4006693/67263894-640ff900-f4e4-11e9-91a7-ce280737e1e0.png)

$
\begin{eqnarray}
X(k)&=&\sum\_{n=0}^{2N-1}x(n)\cos\left(\frac{\left(2n+N+1\right)\left(2k+1\right){\pi}}{4N}\right) \\\\\
    &=&\sum\_{n=0}^{2N-1}x(n)\cos\left(\frac{4\left(n+\frac{N}{2}+\frac{1}{2}\right)\left(k+\frac{1}{2}\right){\pi}}{4N}\right) \\\\\
    &=&\sum\_{n=0}^{2N-1}x(n)\cos\left(\frac{{\pi}}{N}\left(n+\frac{N}{2}+\frac{1}{2}\right)\left(k+\frac{1}{2}\right)\right) \quad (k=0,1,\cdots,N-1) \\\\\
\end{eqnarray}
$

また, 逆変形離散コサイン変換は, 以下のように定義されます.

$
\begin{eqnarray}
x(n)&=&\frac{1}{N}\sum\_{k=0}^{N-1}X(k)\cos\left(\frac{\left(2n+N+1\right)\left(2k+1\right){\pi}}{4N}\right) \\\\\
    &=&\frac{1}{N}\sum\_{k=0}^{N-1}X(k)\cos\left(\frac{4\left(n+\frac{N}{2}+\frac{1}{2}\right)\left(k+\frac{1}{2}\right){\pi}}{4N}\right) \\\\\
    &=&\frac{1}{N}\sum\_{k=0}^{N-1}X(k)\cos\left(\frac{{\pi}}{N}\left(n+\frac{N}{2}+\frac{1}{2}\right)\left(k+\frac{1}{2}\right)\right) \quad (n=0,1,\cdots,2N-1) \\\\\
\end{eqnarray}
$

## 実装

C++ による, 変形離散コサイン変換の実装例です.

```c++
#include <vector>
#include <cmath>

void MDCT(std::vector<double> &x, std::vector<double> &X, int N) {
  for (int k = 0; k < N; k++) {
    double s = 0.0;

    for (int n = 0; n <= ((2 * N) - 1); n++) {
      s += x[n] * std::cos((M_PI / N) * (n + (N / 2.0) + (1.0 / 2.0)) * (k + (1.0 / 2.0)));
    }

    X[k] = s;
  }
}

void IMDCT(std::vector<double> &x, std::vector<double> &X, int N) {
  for (int n = 0; n <= ((2 * N) - 1); n++) {
    double t = 0.0;

    for (int k = 0; k < N; k++) {
      t += X[k] * std::cos((M_PI / N) * (n + (N / 2.0) + (1.0 / 2.0)) * (k + (1.0 / 2.0)));
    }

    x[n] = t / N;
  }
}
```

実行例

```c++
#include <iostream>
#include <vector>
#include <cmath>
#include "wave.h"
#include "window_functions.h"
#include "mdct.h"

enum {
  N = 8
};

int main() {
  MONO_PCM pcm;

  std::vector<double> x(2 * N);
  std::vector<double> X(N, 0.0);
  std::vector<double> w(2 * N);

  wave_read(&pcm, "sample.wav");

  std::cout << std::fixed;

  sine_window(w, (2 * N));

  std::cout << "x(n)" << std::endl;

  for (int n = 0; n < (2 * N); n++) {
    x[n] = w[n] * pcm.s[n];
    std::cout << "[" << n << "]" << " " << x[n] << std::endl;
  }

  MDCT(x, X, N);

  std::cout << "X(k)" << std::endl;

  for (int k = 0; k < N; k++) {
    std::cout << "[" << k << "]" << " " << X[k] << std::endl;
  }

  x.assign((2 * N), 0.0);

  IMDCT(x, X, N);

  std::cout << "x(n)" << std::endl;

  for (int n = 0; n < (2 * N); n++) {
    x[n] *= w[n];
    std::cout << "[" << n << "]" << " " << x[n] << std::endl;
  }
}
```

注意点としては, MDCT / IMDCT を実行するまえに, 窓関数をかける必要があります. 上記の例では, MP3 などで利用される, <b>サイン窓</b>を利用しています.

```c++
#include <vector>
#include <cmath>

void sine_window(std::vector<double> &w, int N) {
  for (int n = 0; n < N; n++) {
    w[n] = std::sin(M_PI * (static_cast<double>(n) / N));
  }
}
```

# リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
- [ディジタル音声&画像の圧縮/伸長/加工技術: 大容量化するマルチメディア・データを転送・保存・活用するために (ディジタル信号処理シリーズ)](https://www.amazon.co.jp/%E3%83%87%E3%82%A3%E3%82%B8%E3%82%BF%E3%83%AB%E9%9F%B3%E5%A3%B0-%E7%94%BB%E5%83%8F%E3%81%AE%E5%9C%A7%E7%B8%AE-%E5%8A%A0%E5%B7%A5%E6%8A%80%E8%A1%93-%E5%A4%A7%E5%AE%B9%E9%87%8F%E5%8C%96%E3%81%99%E3%82%8B%E3%83%9E%E3%83%AB%E3%83%81%E3%83%A1%E3%83%87%E3%82%A3%E3%82%A2%E3%83%BB%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E8%BB%A2%E9%80%81%E3%83%BB%E4%BF%9D%E5%AD%98%E3%83%BB%E6%B4%BB%E7%94%A8%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AB-%E3%83%87%E3%82%A3%E3%82%B8%E3%82%BF%E3%83%AB%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA/dp/4789831450)
