+++
banner = ""
categories = ["Computer Sience", "Digital Signal Processing", "Mathematics", "Programming"]
date = 2019-09-28T17:02:03+09:00
description = "FIR Filter の概要"
images = []
menu = ""
tags = ["Digital Signal Processing", "Mathematics", "Z-Transform", "Digital Filter"]
title = "FIR Filter"
+++

# FIR フィルタとは ?

デジタルフィルタは, <b>乗算器</b>, <b>加算器</b>, <b>遅延器</b>で構成されています. これらの構成によって, <b>FIR (Finite Impulse Response) フィルタ</b>と, <b>IIR (Infinite Impulse Response) フィルタ</b>の 2 種類に分類できます.

![デジタルフィルタの構成 (FIR フィルタの例)](https://user-images.githubusercontent.com/4006693/65827477-74113000-e2cc-11e9-9df2-920f834bd8e5.png)

FIR フィルタの定義は以下のようになります. この式は, <b>コンボリューション</b> (<b>畳み込み</b>) と呼ばれる計算の定義です.

$
\begin{eqnarray}
y(n)=\sum\_{m=0}^{J}b(m)x(n-m)
\end{eqnarray}
$

$x(n)$ は入力信号, $y(n)$ は出力信号, $b(m)$ は乗算器のフィルタ係数, $J$ は遅延器の数を表しています ($m=0$ から開始しているので, フィルタ係数の数は $J+1$ となります).

 遅延器とは, デジタル信号を 1 サンプルだけ格納することができるメモリです. 音データを遅延器に入力すると, 遅延器に 1 時刻の間だけ蓄積されたあと次の時刻に出力されます. 過去の音データをどのくらい記憶できるかは, 遅延器の数によって決定されます. 例えば, サンプリング周波数 44.1 kHz の場合, 1 秒間の音データを記憶するのには, 44100 個の遅延器が必要になります.

# Z 変換

フィルタの特性というのは, <b>Z 変換</b> によって分析することが可能です.

$x(n)$ と $y(n)$ の Z 変換は, 以下のように定義されます.

$
\begin{eqnarray}
X(z)=\sum\_{n=-\infty}^{\infty}x(n)z^{-n}
\end{eqnarray}
$

$
\begin{eqnarray}
Y(z)=\sum\_{n=-\infty}^{\infty}y(n)z^{-n}
\end{eqnarray}
$

この定義をもとに, FIR フィルタの定義式の Z 変換を求めます.

$
\begin{eqnarray}
Y(z)&=&\sum\_{m=0}^{J}b(m)\sum\_{n=-\infty}^{\infty}x(n-m)z^{-n} \\\\\
    &=&\sum\_{m=0}^{J}b(m)\sum\_{n=-\infty}^{\infty}x(n-m)z^{-(n-m)}z^{-m} \\\\\
    &=&\sum\_{m=0}^{J}b(m)X(z)z^{-m} \\\\\
\end{eqnarray}
$

ここで, フィルタ係数の定義というのは,

$
\begin{cases}
\begin{eqnarray}
&b(m)\neq0&\quad(0 \leqq m \leqq J) \\\\\
&b(m)=0&\quad(otherwise) \\\\\
\end{eqnarray}
\end{cases}
$

となるので,

$
\begin{eqnarray}
Y(z)&=&\sum\_{m=-\infty}^{\infty}b(m)X(z)z^{-m} \\\\\
    &=&\sum\_{m=-\infty}^{\infty}b(m)z^{-m}X(z) \\\\\
\end{eqnarray}
$

と書き換えることができます.

また, $b(m)$ の Z 変換は,

$
\begin{eqnarray}
B(z)=\sum\_{m=\infty}^{\infty}b(m)z^{-n}
\end{eqnarray}
$

となるので, FIR フィルタの Z 変換は以下のように定義できます.

$Y(z)=B(z)X(z)$

この式は, $z$ を変数として信号を表すと, FIR フィルタは入力信号とフィルタ係数の乗算によって出力信号が生成できることを意味します.

また, 入力信号と出力信号の比を<b>伝達関数</b>と呼び, FIR フィルタの伝達関数は以下のように定義できます.

$
\begin{eqnarray}
H(z)=\frac{Y(z)}{X(z)}=B(z)
\end{eqnarray}
$

## Z 変換とフーリエ変換

Z 変換は, デジタル信号に対するフーリエ変換と関係しています. $z$ を $\exp(j2{\pi}f)$ に置き換えると,

$
\begin{eqnarray}
X(f)=\sum\_{n=-\infty}^{\infty}x(n)\exp\left(-j2{\pi}fn\right)
\end{eqnarray}
$

$
\begin{eqnarray}
Y(f)=\sum\_{n=-\infty}^{\infty}y(n)\exp\left(-j2{\pi}fn\right)
\end{eqnarray}
$

$
\begin{eqnarray}
B(f)=\sum\_{m=-\infty}^{\infty}b(m)\exp\left(-j2{\pi}fm\right)
\end{eqnarray}
$

これらは, サンプリング周波数 $f\_{s}$ を 1 に正規化したデジタル信号のフーリエ変換です.

つまり, FIR フィルタは, <b>時間領域では畳み込み</b>, <b>周波数領域では乗算</b>として定義できるということです.

$Y(f)=B(f)X(f)$

# 窓関数法

FIR フィルタをそのまま利用するのは, フィルタの性能があまりよくありません. 周波数成分をはっきりと分離するには, フィルタの設計に工夫が必要となります.

そのための 1 つの方法に, <b>窓関数法</b>があります. 窓関数法を適用することで, <b>LPF</b> (Low-Pass Filter), <b>HPF</b> (High-Pass Filter), <b>BPF</b> (Band-Pass Filter), <b>BEF</b> (Band-Eliminate Filter) という, 基本的なフィルタを設計することが可能になります.

フィルタが通過させる帯域は, <b>通過域</b>, 減衰させる帯域は, <b>阻止域</b>と呼びます. また, $f\_{e}$ は, <b>エッジ周波数</b>と呼ばれ, 理想的なフィルタは, この周波数を境にして, 通過域と阻止域をはっきりと分離する事ができますが, コンピュータではこのような理想フィルタを実装することはできません.

![FIR フィルタ (LPF, HPF, BPF, BEF)](https://user-images.githubusercontent.com/4006693/66262353-cf857580-e818-11e9-8ce5-653461d69fc8.png)

では, LPF を例にして, 窓関数法による FIR フィルタを設計してみます.

理想的な LPF は, エッジ周波数より低域の周波数成分は 1, それより高域の周波数成分では 0 となるので, 理想特性は以下のように定義できます.

$
B(f)=
\begin{cases}
\begin{eqnarray}
&1&\quad(0 \leqq \left|f\right| \leqq f\_{e}) \\\\\
&0&\quad(f\_{e} < \left|f\right| \leqq \frac{1}{2}) \\\\\
\end{eqnarray}
\end{cases}
$

$B(f)$ から, フィルタ係数を求めるには, デジタル信号に対する逆フーリエ変換を利用します.

$
\begin{eqnarray}
b(m)&=&\int\_{-f\_{e}}^{f\_{e}}\exp\left(j2{\pi}fm\right)df \quad (-\infty \leqq m \leqq \infty) \\\\\
    &=&2f\_{e}sinc\left(2{\pi}f_{e}m\right) \\\\\
\end{eqnarray}
$

$
sinc(x)=
\begin{cases}
\begin{eqnarray}
&1& \quad (x=0) \\\\\
&\frac{\sin\left(x\right)}{x}& \quad (otherwise) \\\\\
\end{eqnarray}
\end{cases}
$

$sinc\left(x\right)$ は, <b>シンク関数</b>と呼ばれています. シンク関数は, $-\infty$ から $+\infty$ の区間で定義されているので, フィルタ係数が無限に存在します. コンピュータではこのようなフィルタをあつかえません. したがって, フィルタ係数を有限でうちきるために窓関数を利用します.

窓関数法は, 0 を中心として対称となるようにフィルタをうちきりますが, このままだと, フィルタ係数のインデックスが負数からはじまるので, インデックスが 0 ~ $J$ になるように書き換えます.

ただし, 窓関数で有限にうちきった結果, 通過域と阻止域の境界が広がるという問題が発生します (理想特性をもつフィルタが, コンピュータでは実装できない理由です).

通過域と阻止域の間に $\delta$ で定義される帯域を, <b>遷移帯域幅</b> と呼びます. $\delta$ は, フィルタ係数の数 $J+1$ に反比例します. つまり, 理想特性に近づけるほど, 乗算器の数が大きくなってしまいます.

実は, フィルタ係数をうちきる窓関数として, ハニング窓を適用する場合, $\delta$ からフィルタ係数の数を決定できます.

$
J+1=
\begin{eqnarray}
round\left(\frac{3.1}{\delta}\right)
\end{eqnarray}
$

窓関数法では 0 を中心として対称になるようにフィルタをうちきる必要があるので, フィルタ係数の数は奇数にする必要があります.

LPF と同様の設計で, HPF, BPF, BEF を実装することができます.

## HPF

$
B(f)=
\begin{cases}
\begin{eqnarray}
&0&\quad(0 \leqq \left|f\right| \leqq f\_{e}) \\\\\
&1&\quad(f\_{e} < \left|f\right| \leqq \frac{1}{2}) \\\\\
\end{eqnarray}
\end{cases}
$

$
\begin{eqnarray}
b\left(m\right)=sinc\left({\pi}m\right)-2f\_{e}sinc\left(2{\pi}f\_{e}m\right)
\end{eqnarray}
$

## BPF

$
B(f)=
\begin{cases}
\begin{eqnarray}
&0&\quad(0 \leqq \left|f\right| \leqq f\_{e1}) \\\\\
&1&\quad(f\_{e1} < \left|f\right| \leqq f\_{e2}) \\\\\
&0&\quad(f\_{e2} < \left|f\right| \leqq \frac{1}{2}) \\\\\
\end{eqnarray}
\end{cases}
$

$
\begin{eqnarray}
b\left(m\right)=2f\_{e2}sinc\left(2{\pi}f\_{e2}m\right)-2f\_{e1}sinc\left(2{\pi}f\_{e1}m\right)
\end{eqnarray}
$

## BEF

$
B(f)=
\begin{cases}
\begin{eqnarray}
&1&\quad(0 \leqq \left|f\right| \leqq f\_{e1}) \\\\\
&0&\quad(f\_{e1} < \left|f\right| \leqq f\_{e2}) \\\\\
&1&\quad(f\_{e2} < \left|f\right| \leqq \frac{1}{2}) \\\\\
\end{eqnarray}
\end{cases}
$

$
\begin{eqnarray}
b\left(m\right)=sinc\left({\pi}m\right)-2f\_{e2}sinc\left(2{\pi}f\_{e2}m\right)+2f\_{e1}sinc\left(2{\pi}f\_{e1}m\right)
\end{eqnarray}
$

# 実装

FIR フィルタによる LPF, HPF, BPF, BEF の実装 (C++) です.

```c++
#include <vector>
#include <cmath>

double sinc(double n) {
  if (n == 0) {
    return 1.0;
  }

  return std::sin(n) / n;
}

void FIR_LPF(double fe, int J, std::vector<double> &b, std::vector<double> &w) {
  int offset = J / 2;

  for (int m = -(J / 2); m <= (J / 2); m++) {
    b[offset + m] = 2.0 * fe * sinc(2.0 * M_PI * fe * m);
  }

  for (int m = 0; m < (J + 1); m++) {
    b[m] *= w[m];
  }
}

void FIR_HPF(double fe, int J, std::vector<double> &b, std::vector<double> &w) {
  int offset = J / 2;

  for (int m = -(J / 2); m <= (J / 2); m++) {
    b[offset + m] = sinc(M_PI * m) - (2.0 * fe * sinc(2.0 * M_PI * fe * m));
  }

  for (int m = 0; m < (J + 1); m++) {
    b[m] *= w[m];
  }
}

void FIR_BPF(double fe1, double fe2, int J, std::vector<double> &b, std::vector<double> &w) {
  int offset = J / 2;

  for (int m = -(J / 2); m <= (J / 2); m++) {
    b[offset + m] = (2 * fe2 * sinc(2 * M_PI * fe2 * m)) - (2.0 * fe1 * sinc(2.0 * M_PI * fe1 * m));
  }

  for (int m = 0; m < (J + 1); m++) {
    b[m] *= w[m];
  }
}

void FIR_BEF(double fe1, double fe2, int J, std::vector<double> &b, std::vector<double> &w) {
  int offset = J / 2;

  for (int m = -(J / 2); m <= (J / 2); m++) {
    b[offset + m] = sinc(M_PI * m) - (2 * fe2 * sinc(2 * M_PI * fe2 * m)) + (2.0 * fe1 * sinc(2.0 * M_PI * fe1 * m));
  }

  for (int m = 0; m < (J + 1); m++) {
    b[m] *= w[m];
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include <string>
#include <vector>
#include "wave.h"
#include "window_functions.h"
#include "fir_filters.h"

int main(int argc, char **argv) {
  if ((argc != 4) && (argc != 5)) {
    std::cerr << "Require type, fe, delta" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  std::string type = argv[1];

  STEREO_PCM pcm0, pcm1;
  std::vector<double> b;
  std::vector<double> w;

  // Read WAVE
  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = pcm0.length;

  pcm1.sL.resize(pcm0.length);
  pcm1.sR.resize(pcm0.length);

  if (argc == 4) {
    double fe    = std::stod(argv[2]) / pcm0.fs;
    double delta = std::stod(argv[3]) / pcm0.fs;
    int J        = std::round(3.1 / delta);

    if ((J % 2) == 1) {
      J++;
    }

    b.resize(J);
    w.resize(J);

    hanning_window(w, (J + 1));

    if (type == "lpf") {
      FIR_LPF(fe, J, b, w);
    } else if (type == "hpf") {
      FIR_HPF(fe, J, b, w);
    }
  } else if (argc == 5) {
    double fe1   = std::stod(argv[2]) / pcm0.fs;
    double fe2   = std::stod(argv[3]) / pcm0.fs;
    double delta = std::stod(argv[4]) / pcm0.fs;
    int J        = std::round(3.1 / delta);

    if ((J % 2) == 1) {
      J++;
    }

    b.resize(J);
    w.resize(J);

    hanning_window(w, (J + 1));

    if (type == "bpf") {
      FIR_BPF(fe1, fe2, J, b, w);
    } else if (type == "bef") {
      FIR_BEF(fe1, fe2, J, b, w);
    }
  }

  for (int n = 0; n < pcm1.length; n++) {
    for (int m = 0; m <= J; m++) {
      if ((n - m) >= 0) {
        pcm1.sL[n] += b[m] * pcm0.sL[n - m];
        pcm1.sR[n] += b[m] * pcm0.sR[n - m];
      }
    }
  }

  // Write WAVE
  wave_write(&pcm1, "fir_filter.wav");
}
```

# ディレイ

<b>ディレイ</b>とは, 現在の時刻の音データとそれよりも過去の音データを同時に再生することで, やまびこのような効果を生み出すサウンドエフェクトの 1 種です. デジタル信号処理のなかでも, 最も基本的な処理として位置づけられています (実際, <b>ディレイ系</b>と呼ばれるエフェクター群があります).

$
\begin{eqnarray}
s\_{1}(n)=\sum\_{m=0}^{J}b(m)s\_{0}(n-m)
\end{eqnarray}
$

ディレイの処理は, FIR フィルタと同様です.

現在の時刻の音データに, 過去の音データをどのようにミックスするかは, 乗算器の係数 $b(m)$ によって決まります.
やまびこをつくりだすには, 一般的に以下のように定義します.

$
b(m)=
\begin{cases}
\begin{eqnarray}
&1&\quad(m=0) \\\\\
&wet^{i}&\quad(m=i\*time, 1 \leqq i \leqq feedback) \\\\\
&0&\quad(otherwise) \\\\\
\end{eqnarray}
\end{cases}
$

$wet$ はやまびこの減衰率を表しています. やまびこがしだいに小さくなるようにするには 1 より小さくする必要があります. $time$ は, やまびこの遅延時間です. $feedback$ は, やまびこの繰り返し回数です.

実際のディレイエフェクターでも, ほぼ同様にパラメータを設定可能になっています.

![ディレイとパラメータ](https://user-images.githubusercontent.com/4006693/66262413-28a1d900-e81a-11e9-916d-98886dc00e1b.png)

ここで, ピストル音のような, ごく短時間に非常に大きな音が発生する音データ $s\_{0}(n)$ を仮定します. このような音は, <b>インパルス</b>と呼ばれます (数学的には, <b>デルタ関数</b>で定義されます).

$
s\_{0}(n)=
\begin{cases}
\begin{eqnarray}
&1&\quad(n=0) \\\\\
&0&\quad(otherwise) \\\\\
\end{eqnarray}
\end{cases}
$

![デルタ関数 (アナログ信号なので, $\infty$ となっています)](https://user-images.githubusercontent.com/4006693/65945730-8d061680-e46f-11e9-95c7-9f264cf55848.png)

インパルスを入力として, FIR フィルタを計算すると, 出力信号は以下のように, $b(n)$ となります.

$
\begin{eqnarray}
s\_{1}(n)&=&\sum\_{m=0}^{J}b(m)s\_{0}(n-m) \\\\\
         &=&\sum\_{m=0}^{m=n-1}b(m)s\_{0}(n-m)+b(n)s\_{0}(0)+\sum\_{m=n+1}^{m=J}b(m)s\_{0}(n-m)  \\\\\
         &=&b(n) \\\\\
\end{eqnarray}
$

インパルスを入力信号としたとき, その応答として得られる出力信号を, <b>インパルス応答</b>と呼びます.
<b>FIR フィルタのインパルス応答は, 乗算器の係数</b>です (FIR フィルタの特徴の 1 つ).

![インパルス応答](https://user-images.githubusercontent.com/4006693/65945811-bd4db500-e46f-11e9-9ce5-0bf595f4dc75.png)

```c++
#include <vector>
#include <cmath>

void Delay(
  std::vector<double> &dry,
  std::vector<double> &mix,
  int length,
  double wet,
  double time,
  int feedback) {
  for (int n = 0; n < length; n++) {
    mix[n] = dry[n];

    for (int i = 1; i <= feedback; i++) {
      int m = static_cast<int>(static_cast<double>(n) - static_cast<double>(i * time));

      if (m >= 0) {
        mix[n] = mix[n] + std::pow(wet, static_cast<double>(i)) * dry[m];
      }
    }
  }
}
```

# フレーム単位のフィルタリング

コンピュータのメモリは有限なので, 音データがあまりに長いと, 音データを一度にメモリに読み込んでフィルタリングすることが難しくなります. そこで, 音データを小さなフレームに分割して, フレーム単位でフィルタリングする必要があります. フレームに分割するので, フレームとフレームの接続を考慮する必要があります.

FIR フィルタの場合は, 直前のフレームの音データを, 現在のフレームの最初に連結してフィルタリングします (ただし, 最初のフレームに関しては, 直前のフレームが存在しないので, 無音 (0) のフレームを連結します).

![フレーム単位のフィルタリング](https://user-images.githubusercontent.com/4006693/65952621-2ee03000-e47d-11e9-8985-527417790d03.png)

## 巡回畳み込み

FIR フィルタは, <b>時間領域では畳み込み</b>, <b>周波数領域では乗算</b>として定義できました. つまり, FIR フィルタの計算は, 時間領域で畳み込み演算する方法と, 周波数領域で乗算をしてその結果を時間領域に戻す方法の 2 つがあるということです.

ところで, 離散フーリエ変換では, $x(n)$ が周期 $N$ の周期信号であることを仮定しているので, 畳み込みの結果も周期 $N$ の周期信号になります. これを, <b>巡回畳み込み</b>と呼びます.

![巡回畳み込み](https://user-images.githubusercontent.com/4006693/65955726-c21c6400-e483-11e9-8092-a0aa6d845011.png)

音のデータの長さを $L$, フィルタ係数の数を $J+1$ とすると, 通常の畳み込みの結果は $L+J$ サンプルになります. したがって, 巡回畳み込みの結果から, 通常の畳み込みの結果を取得するには, $N$ を $L+J$ 以上になるように設定する必要があります.

$
\begin{eqnarray}
L+J \leqq N
\end{eqnarray}
$

離散フーリエ変換による畳み込みの計算は, <b>0 埋め処理</b>をして, $N$ サンプルに拡張してから, 離散フーリエ変換を実行します (高速フーリエ変換の場合は, $N$ が 2 のべき乗になるようにします).

## オーバーラップアド

フレーム単位でフィルタリングをする場合, 隣接するフレームの畳み込みの結果を重ね合わせながら連結していく必要があります. この処理を<b>オーバーラップアド</b>と呼びます.

## 実装

```c++
#include <vector>
#include <complex>
#include "window_functions.h"
#include "fft.h"

void fir_frame_filter(
  std::vector<double> &s0,
  std::vector<double> &s1,
  std::vector<double> &b0,
  int length,
  int J,
  int L,
  int N) {
  int number_of_frames = length / L;

  std::vector<std::complex<double>> b(N);
  std::vector<std::complex<double>> x(N);
  std::vector<std::complex<double>> y(N);

  for (int frame = 0; frame < number_of_frames; frame++) {
    int offset = L * frame;

    for (int n = 0; n < N; n++) {
      x[n] = std::complex<double>(0.0, 0.0);
    }

    for (int n = 0; n < L; n++) {
      x[n] = std::complex<double>(s0[offset + n], 0.0);
    }

    FFT(x, N);

    for (int m = 0; m < N; m++) {
      b[m] = std::complex<double>(0.0, 0.0);
    }

    for (int m = 0; m < J; m++) {
      b[m] = std::complex<double>(b0[m], 0.0);
    }

    FFT(b, N);

    for (int k = 0; k < N; k++) {
      y[k] = std::complex<double>(((b[k].real() * x[k].real()) - (b[k].imag() * x[k].imag())), ((b[k].real() * x[k].imag()) + (b[k].imag() * x[k].real())));
    }

    IFFT(y, N);

    for (int n = 0; n < (2 * L); n++) {
      if ((offset + n) < length) {
        s1[offset + n] += y[n].real();
      }
    }
  }
}
```

実行例

```c++
#include <iostream>
#include <cstdlib>
#include <cmath>
#include <vector>
#include "wave.h"
#include "fir_filters.h"
#include "frame_filters.h"

enum {
  L = 256,
  N = 512
};

int main(int argc, char **argv) {
  if (argc != 3) {
    std::cerr << "Require fe, delta" << std::endl;
    std::exit(EXIT_FAILURE);
  }

  STEREO_PCM pcm0, pcm1;
  std::vector<double> b;
  std::vector<double> w;

  wave_read(&pcm0, "stereo.wav");

  pcm1.fs     = pcm0.fs;
  pcm1.bits   = pcm0.bits;
  pcm1.length = pcm0.length;

  double fe    = std::stod(argv[1]) / pcm0.fs;
  double delta = std::stod(argv[2]) / pcm0.fs;
  int J        = std::round(3.1 / delta);

  if ((J % 2) == 1) {
    J++;
  }

  b.resize(J + 1);
  w.resize(J + 1);

  hanning_window(w, (J + 1));

  FIR_LPF(fe, J, b, w);

  pcm1.sL.resize(pcm1.length);
  pcm1.sR.resize(pcm1.length);

  fir_frame_filter(pcm0.sL, pcm1.sL, b, pcm0.length, J, L, N);
  fir_frame_filter(pcm0.sR, pcm1.sR, b, pcm0.length, J, L, N);

  wave_write(&pcm1, "fir_frame_filter.wav");
}
```

# リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
