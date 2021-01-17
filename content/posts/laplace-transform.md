+++
banner = ""
categories = ["Mathematics", "Digital Signal Processing"]
date = 2019-10-05T18:00:22+09:00
description = "ラプラス変換 / Z 変換の概要"
images = []
menu = ""
tags = ["Digital Signal Processing", "Mathematics", "Laplace Transform", "Z-Transform"]
title = "Laplace Transform"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# ラプラス変換

<b>ラプラス変換</b>を数式を使わずに説明するのであれば, <b>フーリエ変換の拡張</b>です.

実用的には, フーリエ変換が音や光の周波数特性を分析するために利用されるのに対して, ラプラス変換は, 電気回路やシステム制御などの<b>過渡現象</b>を扱うドメインにおいて重要な道具です.

過渡現象とは, 簡単に表現すると, <b>ある入力を切り替えた場合に, それに対する出力が少し遅延する現象</b>のことです. 実生活にたとえると, 自転車や自動車を運転している場合, ブレーキ (入力) をかけたときに, すぐにそれが停止 (出力) することが理想ではありますが, 実際には, 遅延が発生して停止します.

## ラブラス変換の定義

以下のような, マイナスの時間において 0 をとるような連続信号 (<b>因果信号</b>) において,

$
\begin{eqnarray}
x(t)=0 \quad (t < 0)
\end{eqnarray}
$

$
\begin{eqnarray}
X(s)=\int\_{0}^{\infty}x(t)\exp\left(-st\right)dt
\end{eqnarray}
$

上記の式がラプラス変換の定義です (ただし, <b>$s$ は $\delta+j\omega$ で定義される複素数</b>です).

$s$ によって定義される複素平面を, <b>$s$ 平面</b>と呼びます. ラプラス変換は, 時間領域の連続信号 $x(t)$ を <b>$s$ 領域</b> $X(s)$ に変換する処理ということです.

ところで, $x(t)$ のラブラス変換が存在するには, 右辺の積分が収束する必要があり, $x(t)$ に対して,

$
\begin{eqnarray}
\left|x(t)\right| \leqq M\exp\left(\alpha t \right)
\end{eqnarray}
$

を満たす $M$, $\alpha$ が存在する必要があります. この条件を翻訳すると, <b>$t$ が増加しても, ある指数関数 $\exp\left(\alpha t \right)$ よりも, $x(t)$ が急速に増大しない</b>ことを意味しています (具体的には, $\exp\left(t^{2}\right)$ のような信号の場合, ラプラス変換は存在しません).

## フーリエ変換との関係

ラプラス変換は, フーリエ変換の拡張と説明しましたが, もう少し詳細を追います.

実は, フーリエ変換にも存在条件があり,

$
\begin{eqnarray}
\lim\_{t \to \pm{\infty}}x(t)=0
\end{eqnarray}
$

が成立する必要があります. そして, この条件は, ラプラス変換の存在条件よりもより制限のきびしい条件になります. この条件を, ラプラス変換において翻訳すると, フーリエ変換は, <b>$s$ 平面の虚軸 (周波数軸) におけるラプラス変換</b>という意味になります.

![$s$ 平面とフーリエ変換](https://user-images.githubusercontent.com/4006693/66258366-5fa6c900-e7df-11e9-838f-401717a5debd.png)

また, フーリエ変換においては, 変換領域が周波数という物理的に明確な意味がありますが, ラブラス変換の変換領域 ($s$ 平面) には物理的に明確な意味はありません.

![ラプラス変換とフーリエ変換](https://user-images.githubusercontent.com/4006693/66257803-77c71a00-e7d8-11e9-8ae4-605e940511ee.png)

# Z 変換

<b>Z 変換</b>を数式を使わずに説明するのであれば, <b>離散フーリエ変換の拡張</b>です. 離散信号に対するラプラス変換を Z 変換と呼んでいるだけなので, 本質的にはラプラス変換と同じです.

$
\begin{eqnarray}
X(z)=\sum\_{n=-\infty}^{\infty}x(n)z^{-n}
\end{eqnarray}
$

![アナログ信号, ディジタル信号, s 平面, z 平面の関係](https://user-images.githubusercontent.com/4006693/66258379-82d17880-e7df-11e9-9181-ef3d00bd0d50.png)

より正確には, 離散フーリエ変換は, $z$ 変換の特殊な場合であり, $z$ 平面と呼ばれる, 複素平面内の $|z|=1$ という単位円上の $z$ 変換は, 離散フーリエ変換に一致します.

![$z$ 平面と離散フーリエ変換](https://user-images.githubusercontent.com/4006693/66827341-7ee4e980-ef89-11e9-8587-f7fd93d6c402.png)

# リファレンス

- [信号・システム理論の基礎―フーリエ解析、ラプラス変換、z変換を系統的に学ぶ](https://www.amazon.co.jp/%E4%BF%A1%E5%8F%B7%E3%83%BB%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E7%90%86%E8%AB%96%E3%81%AE%E5%9F%BA%E7%A4%8E%E2%80%95%E3%83%95%E3%83%BC%E3%83%AA%E3%82%A8%E8%A7%A3%E6%9E%90%E3%80%81%E3%83%A9%E3%83%97%E3%83%A9%E3%82%B9%E5%A4%89%E6%8F%9B%E3%80%81z%E5%A4%89%E6%8F%9B%E3%82%92%E7%B3%BB%E7%B5%B1%E7%9A%84%E3%81%AB%E5%AD%A6%E3%81%B6-%E8%B6%B3%E7%AB%8B-%E4%BF%AE%E4%B8%80/dp/433903214X)
