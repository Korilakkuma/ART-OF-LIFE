+++
banner = ""
categories = ["Computer Sience", "Digital Signal Processing", "Mathematics", "Programming"]
date = 2020-05-02T08:52:00+09:00
description = "ピッチを変えずに, 再生速度を変更する"
images = []
menu = ""
tags = ["Digital Signal Processing", "Time Stretch", "Correlation function"]
title = "Time Stretch"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# タイムストレッチ

音波の速度 ($v$), 周波数 ($f$), 波長 ($\lambda$) には以下の関係式があります.

$
\begin{eqnarray}
v=f{\lambda}
\end{eqnarray}
$

速度を速くすると, 周波数も高くなり, 速度を遅くすると, 周波数も低くなります. しかしながら, 波形の周期性に着目して, その繰り返し回数を増減させることによって, 周波数を変化させることなく, 音データを早送り再生したり, スロー再生したりすることが可能です. このアルゴリズムが **タイムストレッチ**です.

## 相関関数

タイムストレッチでは, 波形の周期を求めるために**相関関数**を利用します. 相関関数は以下のように定義されます. $r(m)$ は相関関数, $s(n)$ は音データ, $N$ は相関関数のサイズです.

$
\begin{eqnarray}
r(n)=\sum\_{n=0}^{N-1}s(n)s(n+m)\quad(0 \leqq m \leqq N-1)
\end{eqnarray}
$

つまり, 相関関数の計算は, 本来の音データとそれを $m$ サンプルずらした音データを $N$ 個の区間に限定して乗算して, その結果を加算する処理となっています.

相関関数には, 基本周期とその整数倍で波形のピークを示すという特徴があるので, そのピークを調べることで波形の周期を求めることができます.

<!-- 具体例として, 基本周波数 440 Hz の音データの基本周期は, 2.27 ms (1 / 440) ms となります. -->
## 早送り再生

例えば, 1.5 倍の速度で早送りする場合には, 最初の周期には単調減少の重みづけ, 次の周期には単調増加の重みづけをして, これらの波形をオーバラップアドによって重ね合わせると, 波形の接続部分における不連続的な変化を目立たせることなく, 3 周期の波形を 2 周期に縮めることができます.

![早送り再生のアルゴリズム](https://user-images.githubusercontent.com/4006693/79052665-18f4c880-7c73-11ea-8004-7732326c1b47.png)

このアルゴリズムを, 基準時刻を更新しながら, 逐次実行することで, 音データの持続時間を短くしていきます. 基準時刻の更新に関わるパラメータ $q$ は, 再生速度を $rate$ とすると, 以下のように定義することができます ($p$ は周期).

$
\begin{eqnarray}
q=round\left(\frac{p}{rate-1}\right)\quad(rate > 1) \\\\\
\end{eqnarray}
$

$rate$ が 1.5 の場合, $q$ は $p$ の 2 倍と算出できます.

## スロー再生

例えば, 2 / 3 (0.66) 倍の速度でスロー再生する場合には, 最初の周期には単調増加の重みづけ, 次の周期には単調減少の重みづけをして, これらの波形をオーバラップアドによって重ね合わせると, 波形の接続部分における不連続的な変化を目立たせることなく, 2 周期の波形を 3 周期に伸ばすことができます.

![スロー再生のアルゴリズム](https://user-images.githubusercontent.com/4006693/79052679-37f35a80-7c73-11ea-983e-93d50091ff02.png)

このアルゴリズムを, 基準時刻を更新しながら, 逐次実行することで, 音データの持続時間を長くしていきます. 基準時刻の更新に関わるパラメータ $q$ は, 再生速度を $rate$ とすると, 以下のように定義することができます ($p$ は周期).

$
\begin{eqnarray}
q=round\left(\frac{p{\cdot}rate}{1-rate}\right)\quad(0.5 \leqq rate < 1) \\\\\
\end{eqnarray}
$

$rate$ が 2 / 3 (0.66) の場合, $q$ は $p$ の 2 倍と算出できます.

## 実装

```c++
#include <vector>

void TimeStretch(
  double rate,
  std::vector<double> &in,
  std::vector<double> &out,
  int fs,
  int length
) {
  if ((rate == 1.0) || (rate <= 0.0)) {
    for (int n = 0; n < length; n++) {
      out[n] = in[n];
    }

    return;
  }

  int template_size = static_cast<int>(fs * 0.01);
  int p_min = static_cast<int>(fs * 0.005);
  int p_max = static_cast<int>(fs * 0.02);

  std::vector<double> x(template_size);
  std::vector<double> y(template_size);
  std::vector<double> r(p_max + 1);

  int offset0 = 0;
  int offset1 = 0;

  while ((offset0 + (2 * p_max)) < length) {
    for (int n = 0; n < template_size; n++) {
      x[n] = in[offset0 + n];
    }

    double max_of_r = 0.0;
    int p = p_min;

    for (int m = p_min; m <= p_max; m++) {
      for (int n = 0; n < template_size; n++) {
        y[n] = in[offset0 + m + n];
      }

      r[m] = 0.0;

      // Correlation function
      for (int n = 0; n < template_size; n++) {
        r[m] += x[n] * y[n];
      }

      // Peak of correlation function
      if (r[m] > max_of_r) {
        max_of_r = r[m];
        p = m;
      }
    }

    if (rate < 1.0) {
      for (int n = 0; n < p; n++) {
        out[offset1 + n] = in[offset0 + n];
      }
    }

    for (int n = 0; n < p; n++) {
      if (rate > 1.0) {
        out[offset1 + n] = (in[offset0 + n] * (p - n)) / p;
        out[offset1 + n] += (in[offset0 + p + n] * n) / p;
      } else if (rate < 1.0) {
        out[offset1 + p + n] = (in[offset0 + p + n] * (p - n)) / p;
        out[offset1 + p + n] += (in[offset0 + n] * n) / p;
      }
    }

    int q = 0;

    if (rate > 1.0) {
      q = static_cast<int>((p / (rate - 1.0)) + 0.5);
    } else if (rate < 1.0) {
      q = static_cast<int>(((p * rate) / (1.0 - rate)) + 0.5);
    }

    if (rate > 1.0) {
      for (int n = p; n < q; n++) {
        if ((offset0 + p + n) >= length) {
          break;
        }

        out[offset1 + n] = in[offset0 + p + n];
      }

      offset0 += p + q;
      offset1 += q;
    } else if (rate < 1.0) {
      for (int n = p; n < q; n++) {
        if ((offset0 + n) >= length) {
          break;
        }

        out[offset1 + p + n] = in[offset0 + n];
      }

      offset0 += q;
      offset1 += p + q;
    }
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include "wave.h"
#include "time_stretch.h"

int main(int argc, char **argv) {
  if (argc != 2) {
    std::cerr << "Require rate" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  STEREO_PCM pcm0, pcm1;

  double rate = std::stod(argv[1]);

  if (rate <= 0.0) {
    std::cerr << "rate > 0.0" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = static_cast<int>(pcm0.length / rate) + 1;

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  TimeStretch(rate, pcm0.sL, pcm1.sL, pcm1.fs, pcm0.length);
  TimeStretch(rate, pcm0.sR, pcm1.sR, pcm1.fs, pcm0.length);

  wave_write(&pcm1, "time-stretch.wav");
}
```

## リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
