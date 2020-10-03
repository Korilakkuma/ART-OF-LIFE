+++
banner = ""
categories = ["Computer Sience", "Digital Signal Processing", "Mathematics", "Programming"]
date = 2020-06-07T02:57:12+09:00
description = "サンプリング周波数を変更する"
images = []
menu = ""
tags = ["Digital Signal Processing", "Resampling", "Linear Interpolation", "Sampling Theorem"]
title = "Resampling"
+++

# リサンプリング

**リサンプリング**とは, デジタル信号をアナログ信号に戻し, サンプリング周波数を (本来のサンプリング周波数から) 変更して, 再度, サンプリングを実行するアルゴリズムです. リサンプリングを実行したあと, 本来のサンプリング周波数で再生すると, ピッチを変化させることができます.

![リサンプリング (上は, リサンプリングでピッチを低くする. 下は, リサンプリングでピッチを高くする](https://user-images.githubusercontent.com/4006693/95013068-9b85a000-0678-11eb-9aa3-792fa23bb86b.gif)

## 線形補間

リサンプリングでは, **サンプルとサンプルの中間値を求める処理が必要となります**. そのための, 比較的, 簡単なアルゴリズムとして, **線形補間**があります.

線形補間は, 以下の数式のように, サンプルとサンプルの中間値を比例配分によって算出します.

$
\begin{eqnarray}
s(t)={\delta}s(m+1)+(1-{\delta})s(m) \\\\\
\end{eqnarray}
$

ここで, ${\delta}$ は以下のように定義されます.

$
\begin{eqnarray}
{\delta}=t-m\quad(m \leqq t < m+1) \\\\\
\end{eqnarray}
$

![線形補間](https://user-images.githubusercontent.com/4006693/94989437-d1ab1d00-05af-11eb-8149-1baf9dcb0c59.gif)

## 線形補間とサンプリング定理

もっとも, リサンプリングを正確に実行するには, サンプリング定理も考慮する必要があります.

サンプリング定理は, 以下の式によって, アナログ信号とデジタル信号を関連づけています.

$
\begin{eqnarray}
s_\{a}(t)=\sum\_{n=-\infty}^{\infty}s_\{d}(n)sinc({\pi}(t-n))
\end{eqnarray}
$

線形補間は, それぞれのサンプルを頂点として, 三角形のハット関数を配置し, これらを加算することで, サンプルとサンプルの間を補間します.

一方で, サンプリング定理による補間は, それぞれのサンプルを頂点として, シンク関数を配置し, これらを加算することで, サンプルとサンプルの間を補間します (コンピュータで実装する場合, シンク関数を有限のサイズにうちきる必要があります).

線形補間は, 言うなれば, アナログ信号を折れ線で近似した処理にすぎないので, 厳密には, サンプリング定理による補間が, 本来のアナログ信号を, (可能な限り) 正確に求めるための唯一のアルゴリズムと言えます (ただし, アルゴリズムが複雑なほど, 計算量を要するので, アプリケーションによっては, 線形補間で十分なケースもあります).

## 実装

```c++
#include <vector>
#include <cmath>

enum {
  J = 24
};

double sinc(double x) {
  if (x == 0.0) {
    return 1.0;
  }

  return std::sin(x) / x;
}

void Resampling(
  double pitch,
  std::vector<double> &in,
  std::vector<double> &out,
  int length
) {
  if (pitch <= 0) {
    return;
  }

  for (int n = 0; n < (length / pitch); n++) {
    double t = pitch * n;
    int offset = static_cast<int>(t);

    for (int m = (offset - (J / 2)); m <= (offset + (J / 2)); m++) {
      if ((m >= 0) && (m < length)) {
        out[n] += in[m] * sinc(M_PI * (t - m));
      }
    }
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include "wave.h"
#include "resampling.h"

int main(int argc, char **argv) {
  if (argc != 2) {
    std::cerr << "Require pitch" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  STEREO_PCM pcm0, pcm1;

  double pitch = std::stod(argv[1]);

  if (pitch <= 0.0) {
    std::cerr << "pitch > 0.0" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = static_cast<int>(pcm0.length / pitch);

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  Resampling(pitch, pcm0.sL, pcm1.sL, pcm0.length);
  Resampling(pitch, pcm0.sR, pcm1.sR, pcm0.length);

  wave_write(&pcm1, "resampling.wav");
}
```

## リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
