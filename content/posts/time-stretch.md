+++
banner = ""
categories = ["Computer Sience", "Digital Signal Processing", "Mathematics", "Programming"]
date = 2020-02-24T15:22:57+09:00
description = ""
images = []
menu = ""
tags = ["Digital Signal Processing", "Time Stretch", "Correlation function"]
title = "Time Stretch"
draft = true
+++

# タイムストレッチとは

## リサンプリング

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

      for (int n = 0; n < template_size; n++) {
        r[m] += x[n] * y[n];
      }

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
