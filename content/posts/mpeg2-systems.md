+++
banner = ""
categories = ["Multimedia", "Video", "Audio"]
date = 2019-09-10T23:27:21+09:00
description = "MPEG-2 Systems - MPEG-2 PS と MPEG-2 TS の概要"
images = []
menu = ""
tags = ["MPEG-2 Systems", "MPEG-2 PS", "MPEG-2 TS"]
title = "MPEG-2 Systems"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# MPEG-2 Systems とは ?

## MPEG-2

MPEG-2 は, MPEG-1 を改良し, 放送用として十分と言える程度の高画質を目指して規格化されました. MPEG-2 は, DVD やデジタル BS 放送などのデータ形式として, 幅広く利用されています.

## MPEG-2 Systems

MPEG-2 を多重化して, 伝送するための規格です. 用途別に, <b>MPEG-2 PS</b> (Program Stream) と <b>MPEG-2 TS</b> (Transport Stream) に分類されます.

![MPEG-2 PS と MPEG-2 TS のイメージ](https://user-images.githubusercontent.com/4006693/65369873-c8ddf680-dc8d-11e9-8d5e-196fb76819fc.png)

## ES

MPEG-2 Systems では, 符号化された画像や音声データを, <b>ES</b> (Elementary Stream) と呼びます. そして, MPEG-2 Systes における<b>プログラム</b>とは, たがいに同期関係のある映像や音声で, 1 つの時計で同期制御が可能な ES の集まりのことを指します.

## PES

ES を適切なサイズに分割してパケット化したものを <b>PES</b> (Packetized Elementary Stream) と呼びます. PES パケットのヘッダは再生時刻の情報を含むことができるので, これによって, 映像と音声を同期して再生することが可能になります (詳細は, [多重信号の同期のセクション](./#多重信号の同期)を参照).

![PES](https://user-images.githubusercontent.com/4006693/65368839-1011ba80-dc81-11e9-9667-fa83c7ce8d35.png)

# MPEG-2 PS

単一のプログラム (同期した映像や音声など）を<b>エラーの可能性が低い環境</b>  (光ディスクや HDD など) であつかうことを想定した形式です, 複数の PES パケットを連結して, 先頭にパックヘッダを付与したものを <b>pack</b> と呼び、さらに複数の pack を連結したものがプログラムストリームとなります.

![pack](https://user-images.githubusercontent.com/4006693/65369316-4e5da880-dc86-11e9-9477-1b787546f768.png)

# MPEG-2 TS

同時に 1 つ以上のプログラムを, <b>エラーが発生しうる環境</b> (放送や通信など) であつかうことを想定したデータ形式です. デジタル放送や HLS など, ストリーミングプロトコルで利用されています. MPEG-2 TS は, 伝送を前提としているので, 伝送しやすい固定長で, かつ, 小さいパケットサイズとなっています.

MPEG-2 PS では, <b>PES 複数個をまとめてパケット化</b>するのに対して, MPEG-2 TS では, パケットサイズが小さいので, <b>PES を分解してパケット化</b>します. 具体的には, PES パケットを <b>Transport Packet</b> (TS packet) と呼ばれる <b>188 バイト</b>固定長のパケットへ分割します. そして, この Transport Packet の連続が, トランスポートストリームとなります. そして, この分割処理によって, 多重化しています.

<table>
  <caption>MPEG-2 PS パック / MPEG-2 TS のパケットの比較</caption>
  <thead>
    <tr>
      <th scope="col"></th>
      <th scope="col">パケット長</th>
      <th scope="col">ペイロード</th>
      <th scope="col">クロック</th>
      <th scope="col">プログラム数</th>
      <th scope="col">多重化</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">MPEG-2 PS パック</th>
      <td>可変長</td>
      <td>PES</td>
      <td>時刻の修正<br />ストリームに同期</td>
      <td>単一プログラム</td>
      <td>オーディオ 最大 33<br />ビデオ 最大 17</td>
    </tr>
    <tr>
      <th scope="row">MPEG-2 TS パケット</th>
      <td>固定長<br />(188 bytes)</td>
      <td>PES<br />セクション</td>
      <td>ストリームに同期</td>
      <td>マルチプログラム</td>
      <td>特になし</td>
    </tr>
  </tbody>
</table>

## 同期ワード

Transport Packet のヘッダの先頭バイトは, 必ず <b>0x47</b> (`sync_byte`) になっています. 受信側は, この値があるデータ位置を Transport Packet の先頭として処理します. これによって, 伝送路上でエラーが発生して, 受信側が一時的にパケットの先頭を見失ったとしても 188 バイトごとの `sync_byte` を検出して, 同期しなおすことで比較的早期に同期を回復することができます.

## PID

それぞれの Transport Packet には, <b>PID</b> (Packet Identifier) と呼ばれる <b>13 ビット</b>の情報が含まれます. Transport Packet が何を伝送しているのかを表す識別子です. 同一の画像・音声は, それぞれ同じ PID をもつので, Transport Stream を受信した側はそれを利用して, 元の PES（ES）に戻すことができます.

## 巡回カウンター

MPEG-2 TS はエラーが発生しうる環境であつかうことを想定しているのでエラー検出が必要になります. Transport Packet ヘッダには, 同じ PID の Transport Packet を受信するたびに値が 1 ずつ増加する `continuity_counter` という <b>4 ビット</b>のカウンタがあります. つまり, 受信側でこのカウンタの不連続を検出することで, Transport Packet のロスを検出可能しています.

## 多重信号の同期

### PCR

MPEG-2 TS では, デジタル放送やストリーミングプロトコルで利用されるので, 送信側と受信側の同期をとる必要があります. 基準となるクロックは, <b>42 ビット</b>で, <b>STC</b> (System Time Clock) と呼ばれています. STC の同期は,約 27 h で, STC の校正データを伝送することによって同期をとることができます. そして, この校正データを <b>PCR</b> (Program Clock Reference) と呼びます.

### PTS

同期をとることができたら, 映像や音声といったコンポーネントどうしの同期をとる必要があります. STC を基準にして, 表示する時刻を付加します. この時刻を <b>PTS</b> (Presentation Time Stamp) と呼びます. デコードする時刻を付加することもあります. この時刻を <b>DTS</b> (Decoding Time Stamp) と呼びます.

![Transport Packet (TS packet)](https://user-images.githubusercontent.com/4006693/64762200-a7d42200-d578-11e9-92b3-71939a99afc0.png)

# リファレンス

- [改訂版 デジタル放送教科書（上）](https://www.amazon.co.jp/%E6%94%B9%E8%A8%82%E7%89%88-%E3%83%87%E3%82%B8%E3%82%BF%E3%83%AB%E6%94%BE%E9%80%81%E6%95%99%E7%A7%91%E6%9B%B8%EF%BC%88%E4%B8%8A%EF%BC%89-%E3%82%A4%E3%83%B3%E3%83%97%E3%83%AC%E3%82%B9%E6%A8%99%E6%BA%96%E6%95%99%E7%A7%91%E6%9B%B8%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA-%E4%BA%80%E5%B1%B1-%E6%B8%89/dp/4844395033)
- [第6回　MPEG-2 PS概要 前編](http://www.mpeg.co.jp/libraries/video_it/video_06.html)
- [第7回　MPEG-2 PS概要 後編](http://www.mpeg.co.jp/libraries/video_it/video_07.html)
- [第3回　MPEG-2 TSの概要 前編](http://www.mpeg.co.jp/libraries/video_it/video_03.html)
- [第4回　MPEG-2 TSの概要 後編](http://www.mpeg.co.jp/libraries/video_it/video_04.html)
- [テレビ放送電波はどんな形？(その５・多重化)](http://www.jushin-s.co.jp/michi/download/47_t16.pdf)
