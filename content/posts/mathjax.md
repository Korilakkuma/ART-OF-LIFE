+++
banner = ""
categories = ["Programming", "Mathematics"]
date = "2019-05-28T19:46:00+09:00"
description = "Web ページで数式を表示するために, MathJax を導入してみた"
images = []
menu = ""
tags = ["JavaScript", "Mathematics"]
title = "MathJax で数式を表示する"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# MathJax

## Overview

Web ページ (HTML) で数式を表示するための JavaScript ライブラリです. 同様のものに, [jsLatex](https://plugins.jquery.com/jsLaTeX/) がありますが, こちらは jQuery のプラグインであるので, jQuery を利用しないことが多い, SPA (Single Page Application) では使えません. また, [KaTex](https://katex.org/) は, 単体で利用可能ですが, 個人的な感想として使いにくかったです ... (他のライブラリなら正しい LaTex の数式がエラーになったりとかで ...).

そこで, [MathJax](https://github.com/mathjax/MathJax) を導入してみました ... と言いますか, Hugo の  hugo-icarus-theme ではデフォルトで採用されていました.

## Usage

```HTML
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']]}
  });
</script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

必要な JavaScript ファイルの読み込みと, デリミタの設定をするだけです (hugo-icarus-theme では既に記述されています).

あとは, 表示したい箇所で, LaTex の数式をデリミタで囲むだけです.

## Example

### フーリエ変換

$X(f) = \int\_{-\infty}^{\infty}x(t)\exp(-j2{\pi}ft)dt\ \ (-\infty \leqq f \leqq \infty)$

```HTML
<div>$X(f) = \int_{-\infty}^{\infty}x(t)\exp(-j2{\pi}ft)dt\ \ (-\infty \leqq f \leqq \infty)$</div>
```

### 離散フーリエ変換

$X(k) = \sum\_{n=0}^{N-1}x(n)\exp\left(\frac{-j2{\pi}kn}{N}\right)\ \ (0 \leqq k \leqq N-1)$

```HTML
<div>$X(k) = \sum_{n=0}^{N-1}x(n)\exp\left(\frac{-j2{\pi}kn}{N}\right)\ \ (0 \leqq k \leqq N-1)$</div>
```

### コンボリューション積分

$s\_1(n) = \sum\_{m=0}^{J}b(m)s\_0(n-m)$

```HTML
<div>$s_1(n) = \sum_{m=0}^{J}b(m)s_0(n-m)$</div>
```

以上です. 初回は, 簡単に, そして, プログラミングと数学ということをテーマにしてみました.
