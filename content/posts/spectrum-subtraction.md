+++
banner = ""
categories = ["Computer Sience", "Digital Signal Processing", "Mathematics", "Programming"]
date = 2020-02-11T17:27:52+09:00
description = "ノイズサプレッサの実装から, スペクトルサブトラクションを理解する"
images = []
menu = ""
tags = ["Digital Signal Processing", "Spectrum Subtraction", "Noise Suppressor"]
title = "Spectrum Subtraction"
+++

# スペクトルサブトラクションとは

**スペクトルサブトラクション**とは, その名のとおり, 周波数領域で減算処理をすることで, 背景雑音を除去するアルゴリズムです. 音データを $x(n)$, 背景雑音を ${\omega}(n)$ とすると, 背景雑音が混入した音データ $y(n)$ は以下の式で定義できます.

$
\begin{eqnarray}
y(n)=x(n)+{\omega}(n)
\end{eqnarray}
$

つまり, 背景雑音を除去するには, 以下の式で定義できます.

$
\begin{eqnarray}
x(n)=y(n)-{\omega}(n)
\end{eqnarray}
$

この減算処理を, 周波数領域で実行するのが, スペクトラムサブトラクションです.

$x(n)$, ${\omega}(n)$, $y(n)$ の周波数領域の信号をそれぞれ, $X(k)$, $W(k)$, $Y(k)$ と定義すると,

$
\begin{eqnarray}
X(k)=Y(k)-W(k)
\end{eqnarray}
$

ここで, $Y(k)$, $W(k)$ をそれぞれ極座標形式で定義すると,

$
\begin{eqnarray}
Y(k)=A\_{Y}(k)\exp(j{\theta}\_{Y}(k))
\end{eqnarray}
$

$
\begin{cases}
\begin{eqnarray}
A\_{Y}(k)     &=&\sqrt{real(Y(k))^{2}+imag(Y(k))^{2}} \\\\\
\theta\_{Y}(k)&=&\tan^{-1}\left(\frac{imag(Y(k))}{real(Y(k))}\right) \\\\\
\end{eqnarray}
\end{cases}
$

$
\begin{eqnarray}
W(k)=A\_{W}(k)\exp(j{\theta}\_{W}(k))
\end{eqnarray}
$

$
\begin{cases}
\begin{eqnarray}
A\_{W}(k)     &=&\sqrt{real(W(k))^{2}+imag(W(k))^{2}} \\\\\
\theta\_{W}(k)&=&\tan^{-1}\left(\frac{imag(W(k))}{real(W(k))}\right) \\\\\
\end{eqnarray}
\end{cases}
$

したがって, スペクトルサブトラクションの定義式は以下のように定義できます.

$
\begin{eqnarray}
X(k)=A\_{Y}(k)\exp(j{\theta}\_{Y}(k))-A\_{W}(k)\exp(j{\theta}\_{W}(k))
\end{eqnarray}
$

さらに, 人間の聴覚は, **位相スペクトルのちがいに鈍感** という特性を考慮すると, 位相スペクトルを無視することが可能で,

$
\begin{eqnarray}
X(k)=(A\_{Y}(k)\-A\_{W}(k))\exp(j{\theta}\_{Y}(k))
\end{eqnarray}
$

と定義できます.

ところで, 一般的に, 背景雑音の振幅は未知です. そこで, 音データが 0 (無音) の部分から背景雑音の振幅スペクトルを推定した, 一般的な, スペクトラムサブトラクションの定義は以下のようになります.

$
X(k)=\begin{cases}
\begin{eqnarray}
&(A\_{Y}(k)-\overline{A\_{W}(k)})\exp(j{\theta}\_{Y}(k))& \quad (A\_{Y}(K)-\overline{A\_{W}(k)} \geqq 0) \\\\\
&0& \quad (A\_{Y}(K)-\overline{A\_{W}(k)} < 0) \\\\\
\end{eqnarray}
\end{cases}
$

一般的に, 音データの周波数特性は時間の経過とともに変化するので, スペクトラムサブトラクションではフレーム単位で周波数特性を加工します.

![フレーム単位のフィルタリング](https://user-images.githubusercontent.com/4006693/65952621-2ee03000-e47d-11e9-8985-527417790d03.png)

# ノイズサプレッサ

**ノイズサプレッサ**は, 背景雑音を除去するアルゴリズム (あるいは, エフェクター) の 1 つです. 一般的に, スペクトルサブトラクションを利用して実装されることが多いです.

## ノイズゲートの問題点

背景雑音を除去するアルゴリズムには, ノイズサプレッサだけではなく, **ノイズゲート** もあります. 非常に単純なアルゴリズムで, ある閾値以下の振幅は, 0 (無音) にしてしまうというものです. しかしながら, このアルゴリズムでは, 例えば, 楽器音にかさなった背景雑音を除去することはできません.

ノイズサプレッサは, この問題点を解決するアルゴリズムです.

## 実装

```c++
#include <cmath>
#include <complex>
#include "window_functions.h"
#include "fft.h"

enum {
  N = 1024
};

void NoiseSuppressor(
  double threshold,
  std::vector<double> &in,
  std::vector<double> &out,
  int fs,
  int length
) {
  int number_of_frame = (length - (N / 2)) / (N / 2);

  std::vector<std::complex<double>> x(N);
  std::vector<std::complex<double>> y(N);

  std::vector<double> w(N);
  std::vector<double> x_real(N);
  std::vector<double> x_imag(N);
  std::vector<double> A(N);
  std::vector<double> T(N);
  std::vector<double> y_real(N);
  std::vector<double> y_imag(N);

  hanning_window(w, N);

  for (int frame = 0; frame < number_of_frame; frame++) {
    int offset = (N / 2) * frame;

    for (int n = 0; n < N; n++) {
      x_real[n] = in[offset + n] * w[n];
      x_imag[n] = 0.0;

      x[n] = std::complex<double>(x_real[n], x_imag[n]);
    }

    FFT(x, N);

    for (int k = 0; k < N; k++) {
      A[k] = std::sqrt((x[k].real() * x[k].real()) + (x[k].imag() * x[k].imag()));

      if ((x[k].imag() != 0.0) && (x[k].real() != 0.0)) {
        T[k] = std::atan2(x[k].imag(), x[k].real());
      }
    }

    for (int k = 0; k < N; k++) {
      A[k] -= threshold;

      if (A[k] < 0.0) {
        A[k] = 0.0;
      }
    }

    for (int k = 0; k < N; k++) {
      y_real[k] = A[k] * cos(T[k]);
      y_imag[k] = A[k] * sin(T[k]);

      y[k] = std::complex<double>(y_real[k], y_imag[k]);
    }

    IFFT(y, N);

    for (int n = 0; n < N; n++) {
      out[offset + n] += y[n].real();
    }
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include "wave.h"
#include "noise_suppressor.h"

int main(int argc, char **argv) {
  if (argc != 2) {
    std::cerr << "Require threshold" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  STEREO_PCM pcm0, pcm1;

  double threshold = std::stod(argv[1]);

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = pcm0.length;

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  NoiseSuppressor(threshold, pcm0.sL, pcm1.sL, pcm1.fs, pcm1.length);
  NoiseSuppressor(threshold, pcm0.sR, pcm1.sR, pcm1.fs, pcm1.length);

  wave_write(&pcm1, "noise-suppressor.wav");
}
```

## リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
