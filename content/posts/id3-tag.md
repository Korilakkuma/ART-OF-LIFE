+++
banner = ""
categories = ["Multimedia", "Audio", "Computer Sience"]
date = 2020-11-29T17:25:45+09:00
description = "ID3 タグのコードリーディング"
images = []
menu = ""
tags = ["MP3", "ID3 Tag", "hls.js"]
title = "ID3 Tag"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# ID3 タグとは ?

**ID3 タグ**とは, オーディオデータの圧縮形式である MP3 に書き込まれている, 楽曲タイトルやアーティスト名などのメタデータのことです.

ID3 タグにはいくつかのバージョンが存在し, 大きくは バージョン 1.x とバージョン 2.x に分類されます.

# ID3 タグの構造

ID3 タグの構造はバージョンによって異なるため, この記事では, [hls.js の demuxer](https://github.com/video-dev/hls.js/tree/master/src/demux) で利用されている, バージョン 2.x.x について記載します.

## ヘッダー

注意すべき点はフレームのサイズで, 4 bytes のバイト列ですが, それぞれ 1 byte の先頭ビット (MSB: Most Significant Bit) は常に 0 で, 意味をもっていません (つまり, 先頭以外の 7 ビットが意味をもっています).

<table>
  <caption>ヘッダーの構造</caption>
  <thead>
    <tr><th scope="col">offset</th><th scope="col">Size</th><th scope="col">Description</th></tr>
  </thead>
  <tbody>
    <tr><td>0 - 2</td><td>3 bytes</td><td><b>ID3</b> の固定文字列</td></tr>
    <tr><td>3 - 4</td><td>2 bytes</td><td>バージョン</td></tr>
    <tr><td>5</td><td>1 bytes</td><td>フラグ</td></tr>
    <tr><td>6 - 9</td><td>4 bytes</td><td>フレームのサイズ</td></tr>
  </tbody>
</table>

### フラグ

フラグは, 1 byte の構造で, それぞれビットの意味は以下のようになります.

<table>
  <caption>フラグの構造</caption>
  <thead>
    <tr><th scope="col">offset</th><th scope="col">Size</th><th scope="col">Description</th></tr>
  </thead>
  <tbody>
    <tr><td>0 - 3</td><td>4 bytes</td><td>未使用</td></tr>
    <tr><td>4</td><td>1 bytes</td><td>フッターが存在するか (v2.4.x 以降)</td></tr>
    <tr><td>5</td><td>1 bytes</td><td>タグが試験的なものか (v2.3.x 以降)</td></tr>
    <tr><td>6</td><td>1 bytes</td><td>タグが圧縮されているか (v2.2.x 以前)<br />拡張ヘッダが存在するか (v2.3.x 以降)</td></tr>
    <tr><td>7</td><td>1 bytes</td><td>非同期処理がされているか</td></tr>
  </tbody>
</table>

## フレームデータ

ここでは, hls.js の demuxer で利用されている, v2.3.x 以降のフレームデータ構造に関して記載します.

<table>
  <caption>フレームデータの構造</caption>
  <thead>
    <tr><th scope="col">offset</th><th scope="col">Size</th><th scope="col">Description</th></tr>
  </thead>
  <tbody>
    <tr><td>0 - 3</td><td>4 bytes</td><td>フレーム ID</td></tr>
    <tr><td>4 - 7</td><td>4 bytes</td><td>フレームサイズ</td></tr>
    <tr><td>8 - 9</td><td>2 bytes</td><td>フラグ</td></tr>
    <tr><td>10 -</td><td>可変</td><td>フレームデータ</td></tr>
  </tbody>
</table>

## フッター

フレーム領域のあとに追加されます. 0 - 2 ビット目の固定文字列が異なる以外, ヘッダーと同じ構造です.

<table>
  <caption>フッターの構造</caption>
  <thead>
    <tr><th scope="col">offset</th><th scope="col">Size</th><th scope="col">Description</th></tr>
  </thead>
  <tbody>
    <tr><td>0 - 2</td><td>3 bytes</td><td><b>3DI</b> の固定文字列</td></tr>
    <tr><td>3 - 4</td><td>2 bytes</td><td>バージョン</td></tr>
    <tr><td>5</td><td>1 bytes</td><td>フラグ</td></tr>
    <tr><td>6 - 9</td><td>4 bytes</td><td>フレームのサイズ</td></tr>
  </tbody>
</table>

# コードリーディング

[hls.js の /src/demux/id3.js](https://github.com/video-dev/hls.js/blob/master/src/demux/id3.js) のコードをリーディングしてみましょう.

ID3 タグのヘッダーが存在するかどうかを判定するメソッドです. 第 1 引数は, `Uint8Array`, 第 2 引数はオフセットですが, オフセットはとりあえず `0` と考えるとリーディングしやすいでしょう.

`offset` とヘッダーのサイズである 10 bytes を加算して, ID3 タグ全体のサイズを超えていないか判定して, ヘッダーの解析をします.

0 - 3 ビットまでをチェックして, `'ID3'` の文字列が含まれているか判定します (`0x49`, `0x44`, `0x33` はそれぞれ, `I`, `D`, `3` の文字コードです).

次に, バージョンが, `65535` (`(0xFF << 8) | (0xFF << 0)`) 未満かどうかを判定します.

フラグの判定はスキップして, 最後に 6 - 9 bytes までの 4 bytes のバリデーションを実行します. 先ほども記載したように, MSB は常に `0` で意味をもっていないので, 1 byte が `127` (`0x80`) 未満かどうかを判定します (また, このコードからビッグエンディアンで格納されていることがわかります)..

すべてのバリデーションが `true` であれば, ヘッダーが正常であることが判定できます.

```JavaScript
static isHeader (data, offset) {
  /*
  * http://id3.org/id3v2.3.0
  * [0]     = 'I'
  * [1]     = 'D'
  * [2]     = '3'
  * [3,4]   = {Version}
  * [5]     = {Flags}
  * [6-9]   = {ID3 Size}
  *
  * An ID3v2 tag can be detected with the following pattern:
  *  $49 44 33 yy yy xx zz zz zz zz
  * Where yy is less than $FF, xx is the 'flags' byte and zz is less than $80
  */
  if (offset + 10 <= data.length) {
    // look for 'ID3' identifier
    if (data[offset] === 0x49 && data[offset + 1] === 0x44 && data[offset + 2] === 0x33) {
      // check version is within range
      if (data[offset + 3] < 0xFF && data[offset + 4] < 0xFF) {
        // check size is within range
        if (data[offset + 6] < 0x80 && data[offset + 7] < 0x80 && data[offset + 8] < 0x80 && data[offset + 9] < 0x80) {
          return true;
        }
      }
    }
  }

  return false;
}
```

## フレームデータ

ここで, 汎用的に利用されている, `_readSize` メソッドについて簡単に説明しておきます (`offset` は `0` で考えます).

`& 0x7f` のマスク処理は, `128` をオーバーしていれば `0` にしてしまう処理です. MSB は意味をもたないので, 各 byte の最大値は `127` となるからです.

サイズはビッグエンディアンで格納されているので, 1 byte 目は `21 (7 bits * 3)` bits 左算術シフト, 同様に, 2 byte 目は, `14 (7 bits * 2)` bits 左算術シフト ... とシフト演算して, 最後に, ビットごとの OR 演算子を適用するとサイズが取得できます (7 bits ごとのシフトなのは, MSB が意味をもっていないからです).

```JavaScript
static _readSize (data, offset) {
  let size = 0;
  size = ((data[offset] & 0x7f) << 21);
  size |= ((data[offset + 1] & 0x7f) << 14);
  size |= ((data[offset + 2] & 0x7f) << 7);
  size |= (data[offset + 3] & 0x7f);
  return size;
}
```

フレームデータをスライスして返すメソッドです. フレーム ID とフレームサイズの取得処理も含まれています. フレームデータが格納されているのは, 10 bytes 目からなので, `Uint8Array#subarray` の第 1 引数に `10`, 第 2 引数に `10` + `フレームデータのサイズ` を指定すれば, ID3 タグのフレームデータの `Uint8Array` が取得できます.

```JavaScript
static _getFrameData (data) {
  /*
  Frame ID       $xx xx xx xx (four characters)
  Size           $xx xx xx xx
  Flags          $xx xx
  */
  const type = String.fromCharCode(data[0], data[1], data[2], data[3]);
  const size = ID3._readSize(data, 4);

  // skip frame id, size, and flags
  let offset = 10;

  return { type, size, data: data.subarray(offset, offset + size) };
}
```

## フッター

フッターの判定も, ヘッダーの判定と本質的に同じです (`'ID3'` ではなく, `'3DI'` となるだけです).

```JavaScript
static isFooter (data, offset) {
  /*
  * The footer is a copy of the header, but with a different identifier
  */
  if (offset + 10 <= data.length) {
    // look for '3DI' identifier
    if (data[offset] === 0x33 && data[offset + 1] === 0x44 && data[offset + 2] === 0x49) {
      // check version is within range
      if (data[offset + 3] < 0xFF && data[offset + 4] < 0xFF) {
        // check size is within range
        if (data[offset + 6] < 0x80 && data[offset + 7] < 0x80 && data[offset + 8] < 0x80 && data[offset + 9] < 0x80) {
          return true;
        }
      }
    }
  }

  return false;
}
```

# リファレンス

- [ID3 tag version 1.x.x](http://www.datavoyage.com/mpgscript/mpeghdr.htm)
- [ID3 tag version 2.3.0](https://id3.org/id3v2.3.0#Declared_ID3v2_frames)
