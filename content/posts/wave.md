+++
banner = ""
categories = ["Multimedia", "Audio"]
date = 2019-09-11T17:25:18+09:00
description = "WAVE ファイルの構成を理解して, 読み書きするコードを実装する"
images = []
menu = ""
tags = ["WAVE", "PCM"]
title = "WAVE ファイルの構成"
+++

# WAVE ファイルとは ?

<b>WAVE</b> とは, 音声コーデックの 1 つです. 圧縮をしないので, ファイルサイズは, MP3 や AAC などと比較するとかなり大きくなってしまいますが, 音質の劣化がありません.

# WAVE ファイルの構成

WAVE ファイルは, <b>chunk</b> と呼ばれるブロック構造によって音データを記録します. WAVE ファイルそのものは, <b>RIFF</b> (Resource Interchange File Format) チャンクと呼ばれるブロックになっています. そして, RIFF チャンクのなかに, <b>fmt</b> チャンクと <b>data</b> チャンクが格納されています.

<table>
  <caption>WAVF ファイルの構成</caption>
  <thead>
    <tr><th scope="col">chunk</th><th scope="col">chunk</th><th scope="col">Parameter</th><th scope="col">byte</th><th scope="col">Description</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="14">RIFF チャンク</td><td rowspan="3"></td><td>chunkID</td><td>4</td><td>&apos;RIFF&apos;</td></tr>
    <tr><td>chunkSize</td><td>4</td><td>size + 36</td></tr>
    <tr><td>formType</td><td>4</td><td>&apos;WAVE&apos;</td></tr>
    <tr><td rowspan="8">fmt チャンク</td><td>chunkID</td><td>4</td><td>&apos;fmt &apos;</td></tr>
    <tr><td>chunkSize</td><td>4</td><td>16</td></tr>
    <tr><td>waveFormatType</td><td>2</td><td>プロパティ</td></tr>
    <tr><td>channel</td><td>2</td><td>プロパティ</td></tr>
    <tr><td>samplesPerSec</td><td>4</td><td>プロパティ</td></tr>
    <tr><td>bytesPerSec</td><td>4</td><td>プロパティ</td></tr>
    <tr><td>blockSize</td><td>2</td><td>プロパティ</td></tr>
    <tr><td>bitsPerSample</td><td>2</td><td>プロパティ</td></tr>
    <tr><td rowspan="3">data チャンク</td><td>chunkID</td><td>4</td><td>&apos;data&apos;</td></tr>
    <tr><td>chunkSize</td><td>4</td><td>size</td></tr>
    <tr><td>data</td><td>size</td><td>音データ</td></tr>
  </tbody>
</table>

- waveFormatType
  - 音データの形式で, PCM の場合 1
- channel
  - モノラルの場合 1, ステレオの場合 2
- samplesPerSec
  - サンプリング周波数 (単位は, Hz)
- blockSize
  - 音データの最小単位を記録するのに必要なデータ量 (例:  16 bit ステレオであれば 4 byte)
- bytesPerSec
  - 1 sec の音データを記録するのに必要なデータサイズ. `blockSize` と `samplesPerSec` の積
- bitsPerSample
  - 量子化ビット (8 bit or 16 bit)

data チャンクには, 音データそのものが記録されますが, その値は, 量子化ビットと関係しています.

<table>
  <caption>data チャンク</caption>
  <thead><tr><th scope="col"></th><th scope="col">Min</th><th scope="col">Max</th></tr></thead>
  <tbody>
     <tr><th scope="row">量子化ビット 8 bit</th><td>0</td><td>255</td></tr>
     <tr><th scope="row">量子化ビット 16 bit</th><td>-32768</td><td>32767</td></tr>
  </tbody>
</table>

モノラル場合, 時間の経過にしたがって, 音データが順に記録されます. ステレオの場合, 左チャンネルと右チャンネルの音データが交互に記録されます.

![data チャンク (8 bit モノラル, 8 bit ステレオ, 16 bit モノラル, 16 bit ステレオ](https://user-images.githubusercontent.com/4006693/66471174-d8e73a00-eac5-11e9-977a-fe4e11071ccb.png)

# クリッピング

ところで, 音データが, 最大値を上回ってしまうと<b>オーバーフロー</b>が発生 (量子化ビット 16 なら, 32768 であれば -32768, -32769 なら 0 になってしまう) してしまい, 波形が大きく変形してしまいます.

これを防止するために, それらの値 (量子化ビットが 16 なら, 32767, -32768) を上限, 下限として振幅をうちきります. この処理を<b>クリッピング</b>と呼びます.

# 実装

C++ による, WAVE ファイルの読み書きの実装です.

```c++
#include <iostream>
#include <fstream>
#include <string>
#include <vector>

typedef struct {
  int fs;
  int bits;
  int length;
  std::vector<double> s;
} MONO_PCM;

typedef struct {
  int fs;
  int bits;
  int length;
  std::vector<double> sL;
  std::vector<double> sR;
} STEREO_PCM;

void wave_read(MONO_PCM *pcm, std::string filename) {
  int number_of_channels = 1;

  char riff_chunk_id[4];
  long riff_chunk_size;
  char riff_form_type[4];
  char fmt_chunk_id[4];
  long fmt_chunk_size;
  short fmt_wave_format_type;
  short fmt_channel;
  long fmt_samples_per_sec;
  long fmt_bytes_per_sec;
  short fmt_block_size;
  short fmt_bits_per_sample;
  char data_chunk_id[4];
  int data_chunk_size;
  short data;

  std::fstream file(filename, std::ios::binary | std::ios::in);

  file.read(reinterpret_cast<char *>(riff_chunk_id), 1 * 4);
  file.read(reinterpret_cast<char *>(&riff_chunk_size), 4 * 1);
  file.read(reinterpret_cast<char *>(riff_form_type), 1 * 4);
  file.read(reinterpret_cast<char *>(fmt_chunk_id), 1 * 4);
  file.read(reinterpret_cast<char *>(&fmt_chunk_size), 4 * 1);
  file.read(reinterpret_cast<char *>(&fmt_wave_format_type), 2 * 1);
  file.read(reinterpret_cast<char *>(&fmt_channel), 2 * 1);
  file.read(reinterpret_cast<char *>(&fmt_samples_per_sec), 4 * 1);
  file.read(reinterpret_cast<char *>(&fmt_bytes_per_sec), 4 * 1);
  file.read(reinterpret_cast<char *>(&fmt_block_size), 2 * 1);
  file.read(reinterpret_cast<char *>(&fmt_bits_per_sample), 2 * 1);
  file.read(reinterpret_cast<char *>(data_chunk_id), 1 * 4);
  file.read(reinterpret_cast<char *>(&data_chunk_size), 4 * 1);

  pcm->fs = fmt_samples_per_sec;
  pcm->bits = fmt_bits_per_sample;
  pcm->length = data_chunk_size / (number_of_channels * (pcm->bits / 8));

  pcm->s.resize(pcm->length);

  for (int n = 0; n < pcm->length; n++) {
    file.read(reinterpret_cast<char *>(&data), 2 * 1);
    pcm->s[n] = static_cast<double>(data) / 32768.0;
  }

  file.close();
}

void wave_write(MONO_PCM *pcm, std::string filename) {
  int number_of_channels = 1;

  char riff_chunk_id[4];
  long riff_chunk_size;
  char riff_form_type[4];
  char fmt_chunk_id[4];
  long fmt_chunk_size;
  short fmt_wave_format_type;
  short fmt_channel;
  long fmt_samples_per_sec;
  long fmt_bytes_per_sec;
  short fmt_block_size;
  short fmt_bits_per_sample;
  char data_chunk_id[4];
  int data_chunk_size;
  short data;
  double s;

  riff_chunk_id[0] = 'R';
  riff_chunk_id[1] = 'I';
  riff_chunk_id[2] = 'F';
  riff_chunk_id[3] = 'F';

  riff_chunk_size = 36 + (number_of_channels * (16 / 8) * pcm->length);

  riff_form_type[0] = 'W';
  riff_form_type[1] = 'A';
  riff_form_type[2] = 'V';
  riff_form_type[3] = 'E';

  fmt_chunk_id[0] = 'f';
  fmt_chunk_id[1] = 'm';
  fmt_chunk_id[2] = 't';
  fmt_chunk_id[3] = ' ';

  fmt_chunk_size = 16;
  fmt_wave_format_type = 1;
  fmt_channel = number_of_channels;
  fmt_samples_per_sec = pcm->fs;
  fmt_bytes_per_sec = pcm->fs * (number_of_channels * (pcm->bits / 8));
  fmt_block_size = number_of_channels * (pcm->bits / 8);
  fmt_bits_per_sample = pcm->bits;

  data_chunk_id[0] = 'd';
  data_chunk_id[1] = 'a';
  data_chunk_id[2] = 't';
  data_chunk_id[3] = 'a';

  data_chunk_size = (number_of_channels * (16 / 8)) * pcm->length;

  std::fstream file(filename, std::ios::binary | std::ios::out);

  file.write(reinterpret_cast<char *>(riff_chunk_id), 1 * 4);
  file.write(reinterpret_cast<char *>(&riff_chunk_size), 4 * 1);
  file.write(reinterpret_cast<char *>(riff_form_type), 1 * 4);
  file.write(reinterpret_cast<char *>(fmt_chunk_id), 1 * 4);
  file.write(reinterpret_cast<char *>(&fmt_chunk_size), 4 * 1);
  file.write(reinterpret_cast<char *>(&fmt_wave_format_type), 2 * 1);
  file.write(reinterpret_cast<char *>(&fmt_channel), 2 * 1);
  file.write(reinterpret_cast<char *>(&fmt_samples_per_sec), 4 * 1);
  file.write(reinterpret_cast<char *>(&fmt_bytes_per_sec), 4 * 1);
  file.write(reinterpret_cast<char *>(&fmt_block_size), 2 * 1);
  file.write(reinterpret_cast<char *>(&fmt_bits_per_sample), 2 * 1);
  file.write(reinterpret_cast<char *>(data_chunk_id), 1 * 4);
  file.write(reinterpret_cast<char *>(&data_chunk_size), 4 * 1);

  for (int n = 0; n < pcm->length; n++) {
    s = ((pcm->s[n] + 1.0) / 2.0) * 65536.0;

    if (s > 65535.0) {
        s = 65535.0;
    }

    if (s < 0.0) {
        s = 0.0;
    }

    data = static_cast<short>((s + 0.5) - 32768);

    file.write(reinterpret_cast<char *>(&data), 2 * 1);
  }

  file.close();
}

void wave_read(STEREO_PCM *pcm, std::string filename) {
  int number_of_channels = 2;

  char riff_chunk_id[4];
  long riff_chunk_size;
  char riff_form_type[4];
  char fmt_chunk_id[4];
  long fmt_chunk_size;
  short fmt_wave_format_type;
  short fmt_channel;
  long fmt_samples_per_sec;
  long fmt_bytes_per_sec;
  short fmt_block_size;
  short fmt_bits_per_sample;
  char data_chunk_id[4];
  int data_chunk_size;
  short dataL;
  short dataR;

  std::fstream file(filename, std::ios::binary | std::ios::in);

  file.read(reinterpret_cast<char *>(riff_chunk_id), 1 * 4);
  file.read(reinterpret_cast<char *>(&riff_chunk_size), 4 * 1);
  file.read(reinterpret_cast<char *>(riff_form_type), 1 * 4);
  file.read(reinterpret_cast<char *>(fmt_chunk_id), 1 * 4);
  file.read(reinterpret_cast<char *>(&fmt_chunk_size), 4 * 1);
  file.read(reinterpret_cast<char *>(&fmt_wave_format_type), 2 * 1);
  file.read(reinterpret_cast<char *>(&fmt_channel), 2 * 1);
  file.read(reinterpret_cast<char *>(&fmt_samples_per_sec), 4 * 1);
  file.read(reinterpret_cast<char *>(&fmt_bytes_per_sec), 4 * 1);
  file.read(reinterpret_cast<char *>(&fmt_block_size), 2 * 1);
  file.read(reinterpret_cast<char *>(&fmt_bits_per_sample), 2 * 1);
  file.read(reinterpret_cast<char *>(data_chunk_id), 1 * 4);
  file.read(reinterpret_cast<char *>(&data_chunk_size), 4 * 1);

  pcm->fs = fmt_samples_per_sec;
  pcm->bits = fmt_bits_per_sample;
  pcm->length = data_chunk_size / (number_of_channels * (pcm->bits / 8));

  pcm->sL.resize(pcm->length);
  pcm->sR.resize(pcm->length);

  for (int n = 0; n < pcm->length; n++) {
    file.read(reinterpret_cast<char *>(&dataL), 2 * 1);
    file.read(reinterpret_cast<char *>(&dataR), 2 * 1);

    pcm->sL[n] = static_cast<double>(dataL) / 32768.0;
    pcm->sR[n] = static_cast<double>(dataR) / 32768.0;
  }

  file.close();
}

void wave_write(STEREO_PCM *pcm, std::string filename) {
  int number_of_channels = 2;

  char riff_chunk_id[4];
  long riff_chunk_size;
  char riff_form_type[4];
  char fmt_chunk_id[4];
  long fmt_chunk_size;
  short fmt_wave_format_type;
  short fmt_channel;
  long fmt_samples_per_sec;
  long fmt_bytes_per_sec;
  short fmt_block_size;
  short fmt_bits_per_sample;
  char data_chunk_id[4];
  int data_chunk_size;
  short dataL;
  short dataR;
  double sL;
  double sR;

  riff_chunk_id[0] = 'R';
  riff_chunk_id[1] = 'I';
  riff_chunk_id[2] = 'F';
  riff_chunk_id[3] = 'F';

  riff_chunk_size = 36 + (number_of_channels * (16 / 8) * pcm->length);

  riff_form_type[0] = 'W';
  riff_form_type[1] = 'A';
  riff_form_type[2] = 'V';
  riff_form_type[3] = 'E';

  fmt_chunk_id[0] = 'f';
  fmt_chunk_id[1] = 'm';
  fmt_chunk_id[2] = 't';
  fmt_chunk_id[3] = ' ';

  fmt_chunk_size = 16;
  fmt_wave_format_type = 1;
  fmt_channel = number_of_channels;
  fmt_samples_per_sec = pcm->fs;
  fmt_bytes_per_sec = pcm->fs * (number_of_channels * (pcm->bits / 8));
  fmt_block_size = number_of_channels * (pcm->bits / 8);
  fmt_bits_per_sample = pcm->bits;

  data_chunk_id[0] = 'd';
  data_chunk_id[1] = 'a';
  data_chunk_id[2] = 't';
  data_chunk_id[3] = 'a';

  data_chunk_size = (number_of_channels * (16 / 8)) * pcm->length;

  std::fstream file(filename, std::ios::binary | std::ios::out);

  file.write(reinterpret_cast<char *>(riff_chunk_id), 1 * 4);
  file.write(reinterpret_cast<char *>(&riff_chunk_size), 4 * 1);
  file.write(reinterpret_cast<char *>(riff_form_type), 1 * 4);
  file.write(reinterpret_cast<char *>(fmt_chunk_id), 1 * 4);
  file.write(reinterpret_cast<char *>(&fmt_chunk_size), 4 * 1);
  file.write(reinterpret_cast<char *>(&fmt_wave_format_type), 2 * 1);
  file.write(reinterpret_cast<char *>(&fmt_channel), 2 * 1);
  file.write(reinterpret_cast<char *>(&fmt_samples_per_sec), 4 * 1);
  file.write(reinterpret_cast<char *>(&fmt_bytes_per_sec), 4 * 1);
  file.write(reinterpret_cast<char *>(&fmt_block_size), 2 * 1);
  file.write(reinterpret_cast<char *>(&fmt_bits_per_sample), 2 * 1);
  file.write(reinterpret_cast<char *>(data_chunk_id), 1 * 4);
  file.write(reinterpret_cast<char *>(&data_chunk_size), 4 * 1);

  for (int n = 0; n < pcm->length; n++) {
    sL = ((pcm->sL[n] + 1.0) / 2.0) * 65536.0;
    sR = ((pcm->sR[n] + 1.0) / 2.0) * 65536.0;

    if (sL > 65535.0) {
        sL = 65535.0;
    }

    if (sL < 0.0) {
        sL = 0.0;
    }

    if (sR > 65535.0) {
        sR = 65535.0;
    }

    if (sR < 0.0) {
        sR = 0.0;
    }

    dataL = static_cast<short>((sL + 0.5) - 32768);
    dataR = static_cast<short>((sR + 0.5) - 32768);

    file.write(reinterpret_cast<char *>(&dataL), 2 * 1);
    file.write(reinterpret_cast<char *>(&dataR), 2 * 1);
  }

  file.close();
}
```

# リファレンス

- [C言語ではじめる音のプログラミング―サウンドエフェクトの信号処理](https://www.amazon.co.jp/C%E8%A8%80%E8%AA%9E%E3%81%A7%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E9%9F%B3%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E2%80%95%E3%82%B5%E3%82%A6%E3%83%B3%E3%83%89%E3%82%A8%E3%83%95%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E4%BF%A1%E5%8F%B7%E5%87%A6%E7%90%86-%E9%9D%92%E6%9C%A8-%E7%9B%B4%E5%8F%B2/dp/4274206505)
