+++
banner = ""
categories = ["Multimedia", "Audio", "Computer Sience", "Mathematics"]
date = 2019-09-09T17:02:55+09:00
description = "AAC の圧縮符号化方式の概要をとおして, 音声コーデックの高度な技術を学ぶ"
images = []
menu = ""
tags = ["MPEG-2 Audio", "AAC", "Advanced Audio Coding"]
title = "AAC の圧縮符号化方式"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# MPEG-2 オーディオ

MPEG-2 オーディオには, <b>MPEG-2/BC</b> (Backward Compatible) と <b>MPEG-2/AAC</b> (Advanced Audio Coding) の 2 つの規格があります. MPEG-2/BC の基本的な仕組みは MPEG-1 オーディオと変わりません.

MPEG-2/BC には, サンプリング周波数を半分に落とした, <b>MPEG-2/LFS</b> (Low Sampling Frequency) と, 5.1 チャンネルを可能とした, <b>MPEG-2/MC</b> (Multi Channel) があります. 2 つの規格は, たまたま同じ仕様に含まれているだけで, LSF はインターネットでの利用を想定した低ビットレート, MC は 5.1 チャンネルというまったく異なる目的の規格であり, 同時に利用するためのものではありません. ただし, 共通点として, <b>MPEG-1 オーディオと互換性をもっています</b>.

MPEG-2/AAC は, 高音質・高圧縮を求めるために, MPEG-2/BC のあとに制定された規格です. MPEG-2/AAC は. 大きな音の部分に雑音を集中させたり, 一定の時間内の次のデータを予測したりすることで, データサイズ大幅に削減します (MPEG-1 オーディオ Layer I と比較して, データサイズを約 70 % にできます). ただし, その代償として, <b>MPEG-1 オーディオと互換性はありません</b>.

# AAC の圧縮

![AAC の圧縮フロー](https://user-images.githubusercontent.com/4006693/68105796-f96aae80-ff22-11e9-93dd-a62d3fb3d253.png)

## 1. 入力データ

オーディオ信号で, AAC の場合, 2 チャンネルのステレオ信号や 5.1 チャンネルの信号などです.

## 2. ゲイン・コントロール

ゲイン・コントロールは, アタック音の鋭い音に対する性能を少し向上させる処理ですが, オプションとなっています.
## 3. フィルタ・バンク

フィルタ・バンクは, MDCT を利用して, 信号を <b>128</b> 分割, または, <b>1024</b> 分割します.

## 4. TNS

Temporal Noise Shaping は, 瞬間ノイズ整形とも呼ばれ, 圧縮率を向上させるための処理です.

## 5. インテンシティー/カップリング

ステレオ信号を効率的に符号化するための処理です.

## 6. 予測器

継続する音を, 前の信号を利用して効率よく圧縮するための処理です.

## 7. M/S

Main/Side 処理とは, ステレオ信号を左右の和と差にわけて処理する符号化処理です.

## 8. スケール・ファクタ

スケール・ファクタは, MP3 と同様, 信号の大きさを表す指数を算出する処理です.

## 9. 量子化

圧縮した信号を符号化 (数値化) します.

## 10. ノイズレス・コーディング

量子化した信号を, 統計的な性質を利用してさらに圧縮します.

# AAC で採用された代表的な技術

以下のの技術により, MP3 と比較すると, 20 % - 50 % 圧縮率が向上します.

## 適応ブロック長 MDCT

1024 サンプルのブロックと 128 サンプルのブロックの組み合わせ.

## TNS

量子化ノイズの最適化.

## ノイズレス・コーディング

効率のよいロスレス符号化.

## 4 次元ハフマン

MP3 より, さらに 10 % 程度効率のよいハフマン符号化.

## ゲイン・コントロール

プリエコー (オーディオ信号が急に大きくなる直前に発生するノイズ) を抑制.

## 予測器

周波数ごとの成分の予測.

<table>
  <caption>AAC の圧縮技術とプロファイル</b>
  <thead>
    <tr>
      <th scope="col"></th>
      <th scope="col">メイン</th>
      <th scope="col">LC<br />(AAC の簡易アルゴリズム)</th>
      <th scope="col">SSR<br />(AAC の拡張)</th>
  </thead>
  <tbody>
    <tr>
      <th scope="row">適応ブロック長 MDCT</th>
      <td>◯</td>
      <td>◯</td>
      <td>◯</td>
    </tr>
    <tr>
      <th scope="row">TNS</th>
      <td>◯</td>
      <td>◯</td>
      <td></td>
    </tr>
    <tr>
      <th scope="row">ノイズレス・コーディング</th>
      <td>◯</td>
      <td>◯</td>
      <td></td>
    </tr>
    <tr>
      <th scope="row">4 次元ハフマン</th>
      <td>◯</td>
      <td>◯</td>
      <td>◯</td>
    </tr>
    <tr>
      <th scope="row">ゲイン・コントロール</th>
      <td></td>
      <td></td>
      <td>◯</td>
    </tr>
    <tr>
      <th scope="row">予測器</th>
      <td>◯</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>

# リファレンス

- [改訂版 デジタル放送教科書（上）](https://www.amazon.co.jp/%E6%94%B9%E8%A8%82%E7%89%88-%E3%83%87%E3%82%B8%E3%82%BF%E3%83%AB%E6%94%BE%E9%80%81%E6%95%99%E7%A7%91%E6%9B%B8%EF%BC%88%E4%B8%8A%EF%BC%89-%E3%82%A4%E3%83%B3%E3%83%97%E3%83%AC%E3%82%B9%E6%A8%99%E6%BA%96%E6%95%99%E7%A7%91%E6%9B%B8%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA-%E4%BA%80%E5%B1%B1-%E6%B8%89/dp/4844395033)
