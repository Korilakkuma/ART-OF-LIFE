+++
banner = ""
categories = ["Programming", "Audio"]
date = 2020-02-02T17:27:52+09:00
description = "WebAssembly でオーディオ処理を実装する"
images = []
menu = ""
tags = ["JavaScript", "WebAssembly", "Web Audio API", "C"]
title = "Audio Processing by WebAssembly"
+++

# WebAssembly とは

**WebAssembly** とは, C / C++, Go や Rust などで実装され, (WebAssembly のために) コンパイルされたバイナリを実行するための JavaScript API です. WebAssembly を利用することによって, 以下のようなことが可能になります.

- パフォーマンスのネックになっている処理を WebAssembly を利用することで高速化する
- JavaScript の処理系では不可能な処理を実装する

とりわけ, オーディオなどメディアをあつかう処理は, 計算量の多い処理を実装することが多々あります. また, C / C++ にはオーディオなどメディアをあつかうためのライブラリなど, 資産もたくさんあるので, それらを活用できるという点でも, WebAssembly とメディア処理は非常に相性がよいと言えます.

# Web Audio API と WebAssembly

Web Audio API の API 設計は, Web Audio API が定義する AudioNode を接続して高度なオーディオ処理を実装することです. しかし, その設計にしたがうだけでは, 実装できないオーディオ処理も存在します.

- ノイズ
- ボーカルキャンセラ
- ノイズゲート / ノイズサプレッサ
- チャンネルのごとの FFT / IFFT
- ピッチシフター
- タイムストレッチ

例として, 上記にあげたオーディオ処理は, 直接音データにアクセスして, 音響演算を実装する必要があります (そのための AudioNode が **ScriptProcessorNode** (非推奨), **AudioWorklet** です).

その音響演算を JavaScript で実装してもよいのですが, より高速化をはかるのであれば, WebAssembly を利用することで解決できるかもしれません.

# 実装

さっそく, ノイズ / ボーカルキャンセラ / ピッチシフターを WebAssembly で実装してみます.

## WebAssembly の開発環境構築

まず, C / C++ などで実装されたソースコードを, WebAssembly のためにコンパイルする開発環境の構築が必要です. 以下は, Mac OS X の場合の手順です (詳細は, [GitHub のリポジトリ](https://github.com/emscripten-core/emsdk)や [MDN のドキュメント](https://developer.mozilla.org/ja/docs/WebAssembly/C_to_wasm)を参考にしてください).

事前にインストールが必要なツールや環境です.

- Xcode
  - git and clang
- CMake
- Python
  - Version 2.7.0 or above.
- Java
  - For running closure compiler (optional)

必要なツールや環境をインストールしたら, 以下のコマンドを実行します. `./emsdk install --build=Release sdk-incoming-64bit binaryen-master-64bit` これは非常に時間がかかるので, 寝る前などに実行するとよいでしょう.

```bash
$ git clone git@github.com:emscripten-core/emsdk.git
$ cd emsdk
$ ./emsdk install --build=Release sdk-incoming-64bit binaryen-master-64bit  # Cost much time ...
$ ./emsdk activate --global --build=Release sdk-incoming-64bit binaryen-master-64bit
$ source ./emsdk_env.sh
```

以上で, WebAssembly の開発環境構築が完了です.

## ノイズ

ホワイトノイズ / ピンクノイズ / ブラウニアンノイズの実装例を解説します.

まずは, オーディオ処理をさせる C 言語 のソースを実装します.

```C
#include <stdlib.h>
#include <time.h>
#include <emscripten/emscripten.h>

int is_init = 0;

double b0 = 0.0;
double b1 = 0.0;
double b2 = 0.0;
double b3 = 0.0;
double b4 = 0.0;
double b5 = 0.0;
double b6 = 0.0;

double last_out = 0.0;

EMSCRIPTEN_KEEPALIVE
double whitenoise(void) {
  if (is_init == 0) {
    srand((unsigned)time(NULL));
    is_init = 1;
  }

  double o = 2 * (((double)rand() / ((double)RAND_MAX + 1.0)) - 0.5);

  return o;
}

EMSCRIPTEN_KEEPALIVE
double pinknoise(void) {
  if (is_init == 0) {
    srand((unsigned)time(NULL));
    is_init = 1;
  }

  double n = 2 * (((double)rand() / ((double)RAND_MAX + 1.0)) - 0.5);

  b0 = (0.99886 * b0) + (n * 0.0555179);
  b1 = (0.99332 * b1) + (n * 0.0750759);
  b2 = (0.96900 * b2) + (n * 0.1538520);
  b3 = (0.86650 * b3) + (n * 0.3104856);
  b4 = (0.55000 * b4) + (n * 0.5329522);
  b5 = (-0.7616 * b5) - (n * 0.0168980);

  double o = 0.11 * (b0 + b1 + b2 + b3 + b4 + b5 + b6 + (n * 0.5362));

  b6 = n * 0.115926;

  return o;
}

EMSCRIPTEN_KEEPALIVE
double browniannoise(void) {
  if (is_init == 0) {
    srand((unsigned)time(NULL));
    is_init = 1;
  }

  double n = 2 * (((double)rand() / ((double)RAND_MAX + 1.0)) - 0.5);
  double o = 3.5 * ((last_out + (0.02 * n)) / 1.02);

  last_out = (last_out + (0.02 * n)) / 1.02;

  return o;
}
```

オーディオ処理の具体的な意味は, 書籍や Web サイトを参考にしていただければと思います. 重要な点は, `emscripten.h` というヘッダファイルを読み込みと, JavaScript 側から呼び出したい処理 (関数) に `EMSCRIPTEN_KEEPALIVE` と宣言していることです.

実装が完了したら, 以下のコマンドを実行して, WebAssembly のためのバイナリを生成します.

```bash
$ emcc noise.c -s WASM=1 -o noise.js
```

次に, WebAssembly を利用して, 生成したバイナリを JavaScript から利用できるようにします.

```JavaScript
async function setupWASM(wasm) {
  try {
    const response = await fetch(`./${wasm}.wasm`)
    const bytes    = await response.arrayBuffer();
    const module   = await WebAssembly.compile(bytes);

    const imports = {};

    imports.env = {};
    imports.env.memoryBase = 0;
    imports.env.tableBase  = 0;

    if (!imports.env.memory) {
      imports.env.memory = new WebAssembly.Memory({ initial: 256, maximum: 256 });
    }

    if (!imports.env.table) {
      imports.env.table = new WebAssembly.Table({ initial: 1, maximum : 1, element: 'anyfunc' });
    }

    const { instance } = await WebAssembly.instantiate(bytes, imports);

    return { module, instance, bytes };
  } catch (e) {
    console.error(e);
  }
}
```

以上で, C 言語で実装した関数を呼び出す準備が実装できました.

あとは, Web Audio API の ScriptProcessorNode を利用して, C 言語で定義した関数を呼び出し, ノイズのためのオーディオ処理を実装します.

```JavaScript
// `processor` は, `ScriptProcessorNode` のインスタンス
processor.onaudioprocess = (event) => {
  const outputLs = event.outputBuffer.getChannelData(0);
  const outputRs = event.outputBuffer.getChannelData(1);

  for (let i = 0; i < bufferSize; i++) {
    let output = 0;

    switch (type) {
      case 'whitenoise':
        output = instance.exports.whitenoise();
        break;
      case 'pinknoise':
        output = instance.exports.pinknoise();
        break;
      case 'browniannoise':
        output = instance.exports.browniannoise();
        break;
      default:
        break;
    }

    outputLs[i] = output;
    outputRs[i] = output;
  }
};
```

WebAssembly (`WebAssembly.instantiate`) によって生成された, `instance` というオブジェクトを参照して, C 言語で定義した関数を呼び出すことができます.

## ボーカルキャンセラ

ボーカルキャンセラも, ノイズと同様, 数値演算だけで実装できるオーディオ処理なので, フローとしては, ノイズと大差ありません.

ボーカルキャンセラの C 言語のソースです.

```C
#include <emscripten/emscripten.h>

EMSCRIPTEN_KEEPALIVE
double vocalcanceler(double l, double r, double d) {
  return l - (d * r);
}
```

ソースをコンパイルします.

```bash
$ emcc vocalcanceler.c -s WASM=1 -o vocalcanceler.js
```

そして, Web Audio API の ScriptProcessorNode を利用して, その関数を呼び出し, ボーカルキャンセラのためのオーディオ処理を実装します. 

```JavaScript
// `processor` は, `ScriptProcessorNode` のインスタンス
processor.onaudioprocess = (event) => {
  const inputLs  = event.inputBuffer.getChannelData(0);
  const inputRs  = event.inputBuffer.getChannelData(1);
  const outputLs = event.outputBuffer.getChannelData(0);
  const outputRs = event.outputBuffer.getChannelData(1);

  for (let i = 0; i < bufferSize; i++) {
    outputLs[i] = instance.exports.vocalcanceler(inputLs[i], inputRs[i], depth);
    outputRs[i] = instance.exports.vocalcanceler(inputRs[i], inputLs[i], depth);
  }
};
```

## ピッチシフター

ノイズやボーカルキャンセラのように, JavaScript から C 言語の関数に渡す引数がない場合や, 数値型である場合は, 比較的単純に実装できます. しかしながら, JavaScript から**参照**として, 言い換えると, C 言語の関数が引数に**配列**や**ポインタ**をとる場合には, 少し実装が煩雑になります.

ピッチシフターのの C 言語のソースです. JavaScript から参照する関数 (`pitchshifter` 関数) がポインタを引数として定義していることに着目してください.

```C
#include <stdlib.h>
#include <math.h>
#include <emscripten/emscripten.h>

int pow2(int n);
void FFT(float *x_real, float *x_imag, int N);
void IFFT(float *x_real, float *x_imag, int N);

EMSCRIPTEN_KEEPALIVE
void pitchshifter(
  double pitch,
  float *reals,
  float *imags,
  float *a_reals,
  float *a_imags,
  int N
) {
  FFT(reals, imags, N);

  for (int k = 0; k < N; k++) {
    int offset = floor(pitch * k);

    int eq = 1;

    if (k > (N / 2)) {
      eq = 0;
    }

    if ((offset >= 0) && (offset < N)) {
      a_reals[offset] += eq * reals[k];
      a_imags[offset] += eq * imags[k];
    }
  }

  IFFT(a_reals, a_imags, N);
}

int pow2(int n) {
  if (n == 0) {
    return 1;
  }

  return 2 << (n - 1);
}

void FFT(float *x_real, float *x_imag, int N) {
  int number_of_stages = log2(N);

  for (int stage = 1; stage <= number_of_stages; stage++) {
    for (int i = 0; i < pow2(stage - 1); i++) {
      int rest = number_of_stages - stage;

      for (int j = 0; j < pow2(rest); j++) {
        int n = i * pow2(rest + 1) + j;
        int m = pow2(rest) + n;
        int r = j * pow2(stage - 1);

        float a_real = x_real[n];
        float a_imag = x_imag[n];
        float b_real = x_real[m];
        float b_imag = x_imag[m];
        float c_real = cos((2.0 * M_PI * r) / N);
        float c_imag = -sin((2.0 * M_PI * r) / N);

        if (stage < number_of_stages) {
          x_real[n] = a_real + b_real;
          x_imag[n] = a_imag + b_imag;
          x_real[m] = (c_real * (a_real - b_real)) - (c_imag * (a_imag - b_imag));
          x_imag[m] = (c_real * (a_imag - b_imag)) + (c_imag * (a_real - b_real));
        } else {
          x_real[n] = a_real + b_real;
          x_imag[n] = a_imag + b_imag;
          x_real[m] = a_real - b_real;
          x_imag[m] = a_imag - b_imag;
        }
      }
    }
  }

  int *index = (int *)calloc(N, sizeof(int));

  for (int stage = 1; stage <= number_of_stages; stage++) {
    int rest = number_of_stages - stage;

    for (int i = 0; i < pow2(stage - 1); i++) {
      index[pow2(stage - 1) + i] = index[i] + pow2(rest);
    }
  }

  for (int k = 0; k < N; k++) {
    if (index[k] <= k) {
      continue;
    }

    float tmp_real = x_real[index[k]];
    float tmp_imag = x_imag[index[k]];

    x_real[index[k]] = x_real[k];
    x_imag[index[k]] = x_imag[k];

    x_real[k] = tmp_real;
    x_imag[k] = tmp_imag;
  }

  free(index);
}

void IFFT(float *x_real, float *x_imag, int N) {
  int number_of_stages = log2(N);

  for (int stage = 1; stage <= number_of_stages; stage++) {
    for (int i = 0; i < pow2(stage - 1); i++) {
      int rest = number_of_stages - stage;

      for (int j = 0; j < pow2(rest); j++) {
        int n = i * pow2(rest + 1) + j;
        int m = pow2(rest) + n;
        int r = j * pow2(stage - 1);

        float a_real = x_real[n];
        float a_imag = x_imag[n];
        float b_real = x_real[m];
        float b_imag = x_imag[m];
        float c_real = cos((2.0 * M_PI * r) / N);
        float c_imag = sin((2.0 * M_PI * r) / N);

        if (stage < number_of_stages) {
          x_real[n] = a_real + b_real;
          x_imag[n] = a_imag + b_imag;
          x_real[m] = (c_real * (a_real - b_real)) - (c_imag * (a_imag - b_imag));
          x_imag[m] = (c_real * (a_imag - b_imag)) + (c_imag * (a_real - b_real));
        } else {
          x_real[n] = a_real + b_real;
          x_imag[n] = a_imag + b_imag;
          x_real[m] = a_real - b_real;
          x_imag[m] = a_imag - b_imag;
        }
      }
    }
  }

  int *index = (int *)calloc(N, sizeof(int));

  for (int stage = 1; stage <= number_of_stages; stage++) {
    int rest = number_of_stages - stage;

    for (int i = 0; i < pow2(stage - 1); i++) {
      index[pow2(stage - 1) + i] = index[i] + pow2(rest);
    }
  }

  for (int k = 0; k < N; k++) {
    if (index[k] <= k) {
      continue;
    }

    float tmp_real = x_real[index[k]];
    float tmp_imag = x_imag[index[k]];

    x_real[index[k]] = x_real[k];
    x_imag[index[k]] = x_imag[k];

    x_real[k] = tmp_real;
    x_imag[k] = tmp_imag;
  }

  for (int k = 0; k < N; k++) {
    x_real[k] /= N;
    x_imag[k] /= N;
  }

  free(index);
}
```

コンパルのコマンドも少し複雑になります.

```bash
$ emcc pitchshifter.c -s WASM=1 -s MODULARIZE=1 -s "EXPORTED_FUNCTIONS=['_pitchshifter']" -s "EXTRA_EXPORTED_RUNTIME_METHODS=['ccall', 'cwrap']" -o pitchshifter.js
```

`EXPORTED_FUNCTIONS` というオプションに, JavaScript から参照する関数に, プレフィックス (`_`) を付加した関数名を指定しています. また, `EXTRA_EXPORTED_RUNTIME_METHODS` というオプションには, `ccall` と `cwrap` を指定しています.


C 言語で実装された関数を呼び出す JavaScript も実装がやや煩雑になります.

```JavaScript
const m = Module({
  wasmBinary: new Uint8Array(bytes),
  onRuntimeInitialized: () => {
    console.log('onRuntimeInitialized');

    const pitchshifter = m.cwrap('pitchshifter', null, ['number', 'Float32Array', 'Float32Array', 'Float32Array', 'Float32Array', 'number']);

    // `processor` は, `ScriptProcessorNode` のインスタンス
    processor.onaudioprocess = (event) => {
      const inputLs  = event.inputBuffer.getChannelData(0);
      const inputRs  = event.inputBuffer.getChannelData(1);
      const outputLs = event.outputBuffer.getChannelData(0);
      const outputRs = event.outputBuffer.getChannelData(1);

      const pointerRealL  = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const pointerRealR  = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const pointerImagL  = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const pointerImagR  = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const apointerRealL = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const apointerRealR = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const apointerImagL = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);
      const apointerImagR = m._malloc(bufferSize * Float32Array.BYTES_PER_ELEMENT);

      const realLs = new Float32Array(inputLs);
      const realRs = new Float32Array(inputRs);
      const imagLs = new Float32Array(bufferSize);
      const imagRs = new Float32Array(bufferSize);

      const arealLs = new Float32Array(m.HEAPF32.buffer, apointerRealL, bufferSize);
      const arealRs = new Float32Array(m.HEAPF32.buffer, apointerRealR, bufferSize);
      const aimagLs = new Float32Array(m.HEAPF32.buffer, apointerImagL, bufferSize);
      const aimagRs = new Float32Array(m.HEAPF32.buffer, apointerImagR, bufferSize);

      m.HEAPF32.set(realLs, pointerRealL / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(realRs, pointerRealR / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(imagLs, pointerImagL / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(imagRs, pointerImagR / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(arealLs, apointerRealL / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(arealRs, apointerRealR / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(aimagLs, apointerImagL / Float32Array.BYTES_PER_ELEMENT);
      m.HEAPF32.set(aimagRs, apointerImagR / Float32Array.BYTES_PER_ELEMENT);

      pitchshifter(pitch, pointerRealL, pointerImagL, apointerRealL, apointerImagL, bufferSize);
      pitchshifter(pitch, pointerRealR, pointerImagR, apointerRealR, apointerImagR, bufferSize);

      outputLs.set(arealLs);
      outputRs.set(arealRs);

      m._free(pointerRealL);
      m._free(pointerImagL);
      m._free(pointerRealR);
      m._free(pointerImagR);
      m._free(apointerRealL);
      m._free(apointerImagL);
      m._free(apointerRealR);
      m._free(apointerImagR);
    };

    audio.play();
  }
});
```

まず, `Module` という変数が定義されていることに着目してください. これは, コンパイル時に生成される JavaScript のファイルに定義されています (したがって, `pitchshifter.js` というファイルを読み込んでおく必要があります.

注意すべき点として, `onRuntimeInitialized` というコールバック関数で, 必要なオーディオ処理を実行します. このイベントが発火する前に, オーディオ処理を実行するとエラーが発生してしまいます.

そして, `Module` インスタンスの `cwrap` メソッドで, JavaScript から呼び出す関数オブジェクトを生成します. 引数には, C 言語で定義した関数名や, その関数が必要とする引数の (JavaScript からみた) 型などを配列で指定します.

最も重要なのは, JavaScript から参照 (C 言語にとっての配列やポインタ) を渡す場合には, **JavaScript でもメモリ管理を実装する必要があります**.

実装手順としては,

- `Module` インスタンスの `_malloc` を利用してメモリを確保 (そのポインタを取得)
- `Module` インスタンスの `HEAPFXX#set` を利用して, ポインタと参照を関連づける
- `Module` インスタンスの `_free` を利用して, メモリを解放する

となります (`HEAPFXX` の `XX` は型に合わせた数値です. 例えば, `Float64Array ` であれば, `HEAPF64` になります).

細かい注意点としては, `Float32Array` など型が明確な場合, C 言語でも型をあわせる必要があります (`Float32Array` であれば `float`, `Float64Array` であれば `double` など).

# リファレンス

かけあしになりましたが, WebAssembly でオーディオ処理を実装するための, 最低限必要な知見は紹介できたと思います. ソースコード全体やデモは, [audio-processing-by-wasm](https://github.com/Korilakkuma/audio-processing-by-wasm) を参照してください.

- [C/C++からWebAssemblyにコンパイルする](https://developer.mozilla.org/ja/docs/WebAssembly/C_to_wasm)
- [JS側で作成したtyped arrayをwasm側に渡す](http://blog.shogonir.jp/entry/2017/05/23/232600a)
