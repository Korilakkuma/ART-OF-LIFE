+++
banner = ""
categories = ["Programming", "Audio"]
date = 2020-12-12T23:12:00+09:00
description = "The latest AudioWorklet"
images = []
menu = ""
tags = ["JavaScript", "AudioWorklet", "Web Audio API"]
title = "AudioWorklet"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# AudioWorklet とは ?

## 概要

Web Audio API が定義する標準のノード (`GainNode`, `DelayNode`, `BiquadFilterNode` など) の組み合わせでは不可能な音響処理 (ピッチシフターやノイズサプレッサなど) を直接サウンドデータにアクセスして演算することによって実装するためのクラス (`AudioWorkletNode`, `AudioWorkletProcessor` など) です. Web Audio API が誕生してから, AudioWorklet が仕様策定されるまで, **直接サウンドデータにアクセスして音響演算する**という処理は `ScriptProcessorNode` の役割でした.

なぜ AudioWorklet に役割が置き換わるのでしょうか ? `ScriptProcessorNode` は実装当初から 2 つの問題を抱えていました.

## ScriptProcessorNode の問題

<dl>
  <dt>グリッチ</dt>
  <dd>イベントハンドラで非同期に実行されるので, レイテンシ (遅延) に問題を引き起こす</dd>
  <dt>ジャンク</dt>
  <dd>メインスレッドで実行されるので, UI や再生されるサウンドに問題を引き起こす</dt>
</dl>

AudioWorklet はこれらの問題を解決するために, メインスレッドとは別に, **オーディオスレッドで動作**するように仕様策定され, そして, [Chrome 64](https://developers.google.com/web/updates/2017/12/audio-worklet) で実装されました.

# AudioWorklet を構成するクラス

AudioWorklet が実現する機能は, いくつかのクラス (オブジェクト) によって構成されています.

## AudioWorklet

Worklet スクリプトをロードしてインストールする (`addModule` メソッド) という役割を担います. 他の Worklet と同じく, Worklet に関連する API は, セキュアなページ (`https` または, localhost) でのみ動作します. `AudioWorklet` は `AudioContext` インスタンスに定義されています. `addModule` メソッドは, `Promise` を返します.

```JavaScript
const context = new AudioContext();
const promise = context.audioWorklet.addModule('Woklet スクリプトのパス');
```

## AudioWorkletNode

メインスレッドで主役となるクラスです. Global Scope (`window` オブジェクト) で動作します. `AudioNode` を継承しており, 生成したインスタンスは, `connect` / `disconnect` して利用します. コンストラクタの第 1 引数には `AudioContext` インスタンスを, 第 2 引数には, Worklet で登録した (`AudioWorkletGlobalScope` に登録された) 文字列を指定します.

```JavaScript
const context = new AudioContext();
const promise = context.audioWorklet.addModule('Woklet スクリプトのパス');

promise
  .then(() => {
    const worklet = new AudioWorkletNode(context, 'AudioWorkletGlobalScope に登録された文字列');

    worklet.connect(context.destination);
  })
  .catch(console.error);
```

また, `AudioWorkletNode` を継承させて, 独自の `AudioWorkletNode` を定義することも可能です.

```JavaScript
class CustomAudioWorkletNode extends AudioWorkletNode {
  constructor(context) {
    super(context, 'custom-worklet-processor');
  }
}
```

## AudioWorkletGlobalScope

オーディオスレッドにおける (Worklet における) Global Scope です. `AudioWorkletProcessor` が動作して, `registerProcessor` メソッドによって, `AudioWorkletProcessor` と指定された文字列を関連づけて登録します.

## AudioWorkletProcessor

オーディオ処理を担うクラスです (つまり, `ScriptProcessorNode` の役割は, `AudioWorkletProcessor` に移ったと言えます). `process` メソッドで実行され, アプリケーション開発者は, この `process` メソッドを実装する必要があります.

```JavaScript
class CustomAudioWorkletProcessor extends AudioWorkletProcessor {
  constructor() {
    super();
  }

  process(inputs, outputs, parameters) {
    // オーディオを処理を実装する

    // `process` メソッドをコールバックする場合, `true` を返す
    return true;
  }
}

// `AudioWorkletGlobalScope` に登録される
registerProcessor('custom-worklet-processor', CustomAudioWorkletProcessor);
```

![メインスレッドと WebAudio レンダースレッド](https://user-images.githubusercontent.com/4006693/110239596-27368c00-7f8b-11eb-89de-93da23b8f41e.png)

## AudioParamDescriptor

`AudioParamDescriptor` は `AudioParam` で管理される独自パラメータを定義することを可能にします. すなわち, `AudioParam` がもつオートメーションのメソッド (`linearRampToValueAtTime` メソッドなど) を, 独自パラメータにも適用することができます.

```JavaScript
class CustomAudioWorkletProcessor extends AudioWorkletProcessor {
  static get parameterDescriptors() {
    return [{
      name          : 'custom',  // メインスレッドからあつかうための文字列
      defaultValue  : 0,         // `AudioParam#defaultValue` と同じ
      minValue      : 0,         // `AudioParam#minValue` と同じ
      maxValue      : 1,         // `AudioParam#maxValue` と同じ
      automationRate: 'a-rate'   // 'a-rate' または, 'k-rate' (ほとんどの `AudioParam` は 'a-rate')
    }];
  }

  constructor() {
    super();
  }

  process(inputs, outputs, parameters) {
    // オーディオを処理を実装する

    // `process` メソッドをコールバックする場合, `true` を返す
    return true;
  }
}

// `AudioWorkletGlobalScope` に登録される
registerProcessor('custom-worklet-processor', CustomAudioWorkletProcessor);
```

```JavaScript
const context = new AudioContext();
const promise = context.audioWorklet.addModule('Woklet スクリプトのパス');

promise
  .then(() => {
    const worklet = new AudioWorkletNode(context, 'AudioWorkletGlobalScope に登録された文字列');

    worklet.connect(context.destination);

    // `AudioParam` を取得
    const audioParam = worklet.parameters.get('custom');  // Worklet で指定した文字列

    // `AudioParam` のオートメーション機能が利用できる
    audioParam.setValueAtTime(0, context.currentTime);
    audioParam.linearRampToValueAtTime(0.5, context.currentTime + 0.5);
  })
  .catch(console.error);
```

## MessagePort

`AudioParamDescriptor` で定義できる値は, 数値 (`number`, つまり, 64 bit 浮動小数点数) のみです. したがって, メインスレッドで変更した任意のデータをオーディオスレッドに送信する, 逆に, オーディオスレッドで変更した任意のデータをメインスレッドで受信するというケース ...

すなわち, `AudioWorkletNode` と `AudioWorkletProcessor` の双方向で任意のデータを送受信可能にするために, `MessagePort` が定義されています.
![MessagePort](https://user-images.githubusercontent.com/4006693/110239595-269df580-7f8b-11eb-94f0-1a2b9d7486fc.png)

```JavaScript
const context = new AudioContext();
const promise = context.audioWorklet.addModule('Woklet スクリプトのパス');

promise
  .then(() => {
    const worklet = new AudioWorkletNode(context, 'AudioWorkletGlobalScope に登録された文字列');

    worklet.connect(context.destination);

    worklet.onmessage = (event) => {
      console.log(event.data); // 'custom'
    });

    worklet.postMessage('sawtooth');
  })
  .catch(console.error);
```

```JavaScript
class CustomAudioWorkletProcessor extends AudioWorkletProcessor {
  constructor() {
    super();

    this.custom = 'custom';

    this.port.onmessage = (event) => {
      this.custom = event.data;

      console.log(this.custom);  // 'sawtooth'
    };

    this.port.postMessage(this.custom);
  }

  process(inputs, outputs, parameters) {
    // オーディオを処理を実装する

    // `process` メソッドをコールバックする場合, `true` を返す
    return true;
  }
}

// `AudioWorkletGlobalScope` に登録される
registerProcessor('custom-worklet-processor', CustomAudioWorkletProcessor);
```

# 実装例

AudioWorklet を構成するクラスの概要をもとに, 実装例をとおして AudioWorklet の理解を深めます.

## Bypass

まずは, ウォーミングアップです. 特に意味のない処理ですが, 入力されたオシレーターをそのまま出力するだけです.

[Bypass デモ](https://korilakkuma.github.io/audio-worklet-samples/samples/bypass)

メインスレッドの処理は, 概要を把握できていればそれほど難しいことはないと思います.

main-scripts/bypass.js

```JavaScript
'use strict';

const context = new AudioContext();

const promise = context.audioWorklet.addModule('./worklet-scripts/bypass.js');

promise
  .then(() => {
    const bypass = new AudioWorkletNode(context, 'bypass');

    let oscillator = null;

    document.querySelector('button').addEventListener('click', async (event) => {
      // for Autoplay Policy
      if (context.state !== 'running') {
        await context.resume();
      }

      const button = event.target;

      if (button.textContent === 'START') {
        oscillator = context.createOscillator();

        oscillator.connect(bypass);
        bypass.connect(context.destination);

        oscillator.start(0);

        button.textContent = 'STOP';
      } else {
        oscillator.stop(0);

        button.textContent = 'START';
      }
    }, false);
  })
  .catch(console.error);
```

重要なのは, オーディオ処理の実装, すなわち `AudioWorkletProcessor` の `process` メソッドです.

`process` メソッドには, 第 1 引数に入力となる配列, 第 2 引数に出力となる配列が引数としてわたされます. これら引数の配列の構造は同じとなっており, 多次元配列の要素として `Float32Array` が格納されています. これが, 入力, または, 出力のサウンドデータとなります. 実は, オーディオ処理の実装の理解としては `ScriptProcessorNode` と大差ありません.

まずは, `0` 番目の要素にアクセスして `Float32Array` が各要素となっている配列を取得します (この処理に関しては, 特に理屈なく, こうするものだと理解してだいじょうぶでしょう).

そして, 取得した配列は, チャンネルごとに **128 サンプル**の `Float32Array` が格納されています.

![128 samples の Float32Array がチャンネル順に格納されている](https://user-images.githubusercontent.com/4006693/110239593-256cc880-7f8b-11eb-879b-f8af1726af29.png)

したがって, チャンネルごとの `Float32Array` を走査して, 出力となる `Float32Array` の要素を格納していきます.

1 つ注意点としては, 入力データは必ずしも格納されているわけではないので, 条件判定で `undefined` でないか判定しています.

worklet-scripts/bypass.js

```JavaScript
'use strict';

class Bypass extends AudioWorkletProcessor {
  constructor() {
    super();
  }

  process(inputs, outputs) {
    const input  = inputs[0];
    const output = outputs[0];

    const numberOfChannels = output.length;

    // Traverse channels
    for (let channel = 0; channel < numberOfChannels; channel++) {
      // `Float32Array` ?
      if (input[channel]) {
        output[channel].set(input[channel]);
      }
    }

    return true;
  }
}

registerProcessor('bypass', Bypass);
```

## オシレーター

[オシレーター デモ](https://korilakkuma.github.io/audio-worklet-samples/samples/oscillator)

メインスレッドで注目したいのは,

- `AudioParamDescriptor` によって定義された (独自パラメータの) `AudioParam` を `AudioParamMap` (`AudioWorkletNode` の `parameters` プロパティ) として参照している
- `MessagePort` を利用して, オシレーターの波形 (文字列) をオーディオスレッドに送信している

`AudioParamMap` は, `AudioParam` を要素にもつ `Map` なので, `Map` がもつメソッドを利用して, `AudioParam` を取得可能です.

main-scripts/oscillator.js

```JavaScript
'use strict';

const context = new AudioContext();

const promise = context.audioWorklet.addModule('./worklet-scripts/oscillator.js');

promise
  .then(() => {
    const oscillator = new AudioWorkletNode(context, 'oscillator');

    document.querySelector('button').addEventListener('click', async (event) => {
      // for Autoplay Policy
      if (context.state !== 'running') {
        await context.resume();
      }

      const button = event.target;

      if (button.textContent === 'START') {
        oscillator.connect(context.destination);

        button.textContent = 'STOP';
      } else {
        oscillator.disconnect(0);

        button.textContent = 'START';
      }
    }, false);

    document.querySelector('form').addEventListener('change', (event) => {
      const form = event.currentTarget;

      for (let i = 0, len = form.elements['radio-wave-type'].length; i < len; i++) {
        if (form.elements['radio-wave-type'][i].checked) {
          const type = form.elements['radio-wave-type'][i].value;

          oscillator.port.postMessage(type);

          break;
        }
      }
    }, false);

    document.querySelector('[type="range"]').addEventListener('change', (event) => {
      const frequency = event.currentTarget.valueAsNumber;

      const currentTime = context.currentTime;
      const audioParam  = oscillator.parameters.get('frequency');

      audioParam.setValueAtTime(audioParam.value, currentTime);
      audioParam.linearRampToValueAtTime(frequency, currentTime + 0.5);
    }, false);
  })
  .catch(console.error);
```

オーディオスレッドで注目したいのは,

- `parameterDescriptors` メソッドで, 独自パラメータを `AudioParam` として定義している
- `process` メソッドの第 3 引数に `parameters` がわたされている

`process` メソッドの第 3 引数の `parameters` は, **オートメーションが実行されている場合**, 値の変化を **128 サンプル**ごとに格納しています. ただし, そうでない場合は, `0` 番目に固定値が格納されているだけです. したがって, そのサイズを判定することで, オートメーションが実行されているかを判定できます.

ちなみに, 変数 `sampleRate` は, `AudioWorkletGlobalScope` に定義されている変数です.

worklet-scripts/oscillator.js

```JavaScript
'use strict';

class Oscillator extends AudioWorkletProcessor {
  static get parameterDescriptors() {
    return [{
      name          : 'frequency',
      defaultValue  : 440,
      minValue      : 20,
      maxValue      : sampleRate / 2,
      automationRate: 'a-rate'
    }];
  }

  constructor() {
    super();

    this.type = 'sine';

    this.n = 0;

    this.port.onmessage = (event) => {
      this.type = event.data;
    };
  }

  process(inputs, outputs, parameters) {
    const output = outputs[0];

    const numberOfChannels = output.length;

    // Traverse channels
    for (let channel = 0; channel < numberOfChannels; channel++) {
      const outputChannel = output[channel];

      // Traverse `Float32Array`
      for (let i = 0, len = outputChannel.length; i < len; i++) {
        // Has automation ?
        const frequency = parameters.frequency.length > 1 ? parameters.frequency[i] : parameters.frequency[0];
        const t0        = sampleRate / frequency;

        let output = 0;
        let s      = 0;

        switch (this.type) {
          case 'sine':
            output = Math.sin((2 * Math.PI * frequency * this.n) / sampleRate);
            break;
          case 'square':
            output = (this.n < (t0 / 2)) ? 1 : -1;
            break;
          case 'sawtooth':
            s = 2 * this.n / t0;

            output = s - 1;
            break;
          case 'triangle':
            s = 4 * this.n / t0;

            output = (this.n < (t0 / 2)) ? (-1 + s) : (3 - s);
            break;
          default:
            break;
        }

        outputChannel[i] = output;

        this.n++;

        if (this.n >= t0) {
          this.n = 0;
        }
      }
    }

    return true;
  }
}

registerProcessor('oscillator', Oscillator);
```

## ボーカルキャンセラ

[ボーカルキャンセラ デモ](https://korilakkuma.github.io/audio-worklet-samples/samples/vocal-canceler)

独自パラメータである, ボーカルキャンセラの `depth` も `AudioParamDescriptor` で定義することによって, `AudioWorkletNode` インスタンスから `parameters` プロパティとして参照可能となります.

main-scripts/vocal-canceler.js

```JavaScript
'use strict';

const context = new AudioContext();

const promise = context.audioWorklet.addModule('./worklet-scripts/vocal-canceler.js');

promise
  .then(() => {
    const vocalCanceler = new AudioWorkletNode(context, 'vocal-canceler');

    let source = null;

    document.querySelector('[type="file"]').addEventListener('change', async (event) => {
      // for Autoplay Policy
      if (context.state !== 'running') {
        await context.resume();
      }

      const file = event.target.files[0];

      if (file && file.type.includes('audio')) {
        const objectURL = window.URL.createObjectURL(file);

        const audioElement = document.querySelector('audio');

        audioElement.src = objectURL;

        audioElement.addEventListener('loadstart', () => {
          if (source === null) {
            source = context.createMediaElementSource(audioElement);
          }

          source.connect(vocalCanceler);
          vocalCanceler.connect(context.destination);

          audioElement.play(0);
        }, false);
      }
    }, false);

    document.querySelector('[type="range"]').addEventListener('change', (event) => {
      const currentTime = context.currentTime;
      const audioParam  = vocalCanceler.parameters.get('depth');

      audioParam.setValueAtTime(audioParam.value, currentTime);
      audioParam.linearRampToValueAtTime(event.currentTarget.valueAsNumber, currentTime + 5);
    }, false);
  })
  .catch(console.error);
```

入力データを利用するので, 入力となる `Float32Array` が存在しているか判定していることに注意してください.

worklet-scripts/vocal-canceler.js

```JavaScript
'use strict';

class VocalCanceler extends AudioWorkletProcessor {
  static get parameterDescriptors() {
    return [{
      name          : 'depth',
      defaultValue  : 0,
      minValue      : 0,
      maxValue      : 1,
      automationRate: 'a-rate'
    }];
  }

  constructor() {
    super();
  }

  process(inputs, outputs, parameters) {
    const input  = inputs[0];
    const output = outputs[0];

    const numberOfChannels = output.length;

    if (numberOfChannels === 2) {
      const inputLs  = input[0];
      const inputRs  = input[1];
      const outputLs = output[0];
      const outputRs = output[1];

      // Traverse `Float32Array`
      for (let i = 0, len = outputLs.length; i < len; i++) {
        const depth = parameters.depth.length > 1 ? parameters.depth[i] : parameters.depth[0];

        // `Float32Array` ?
        outputLs[i] = inputLs ? (inputLs[i] - (depth * inputRs[i])) : 0;
        outputRs[i] = inputRs ? (inputRs[i] - (depth * inputLs[i])) : 0;
      }
    } else if (input[0]) {
      output[0].set(input[0]);
    }

    return true;
  }
}

registerProcessor('vocal-canceler', VocalCanceler);
```

# リファレンス

- [W3C](https://www.w3.org/TR/webaudio/#audioworklet)
- [Enter Audio Worklet](https://developers.google.com/web/updates/2017/12/audio-worklet)
- [AudioWorklet の導入](https://qiita.com/ryoyakawai/items/1160586653330ccbf4a4)
- [AudioWorklet で遊ぶ](https://weblike-curtaincall.ssl-lolipop.jp/blog/?p=2027) (著者が以前に投稿したブログをリニューアルしています)
- [サンプルプログラムリポジトリ](https://github.com/Korilakkuma/audio-worklet-samples)
