+++
banner = ""
categories = ["Programming", "Audio"]
date = 2020-10-25T23:50:24+09:00
description = "モジュールベースの XSound"
images = []
menu = ""
tags = ["JavaScript", "Web Audio API", "XSound"]
title = "How to Use XSound as Module Base"
+++

# XSound とは ?

[@Korilakkuma](https://github.com/Korilakkuma/XSound) が実装している Web Audio API ライブラリです. 以下のような豊富な機能が利用可能で, **オーディオ技術のないエンジニアにも利用しやすいライブラリ**をコンセプトにしています.

- サウンドの生成
- ワンショットオーディオの再生
- オーディオデータの再生
- HTMLMediaElement の再生
- WebRTC による外部オーディオインターフェースへのアクセス
- Web MIDI API による, MIDI メッセージング
- MML (Music Macro Language) による自動再生
- エフェクター
- ビジュアライゼーション
- マルチトラックレコーダー
- WebSocket によるバイナリメッセージング
- Media Sound Extensions によるオーディオストリーミング
- オーディオスプライト

[9 libraries to kickstart your Web Audio stuff](https://dev.to/areknawo/9-libraries-to-kickstart-your-web-audio-stuff-460p) にも,

<blockquote>XSound is a batteries-included library for everything audio. From basic management and loading through streaming, effects, ending with visualizations and recording, this libraries provides almost everything! It also has nice, semi-chainable API with solid documentation.</blockquote>

と記載されています.

# モジュールベースの XSound とは ?

## フルスタックベースの API

v2.20.0 までは, フルスタックベースのみの API が実装されていました. フルスタックベースと言っても, 伝わりにくいので, コードで説明すると, 例えば, コーラスエフェクターをかけてオシレーターでサウンド生成する場合,

```JavaScript
const params = {
  time : 0.025,
  depth: 0.5,
  rate : 2.5,
  mix  : 0.5
};

X('oscillator').setup(true);
X('oscillator').module('chorus').param(params);
X('oscillator').module('chorus').state(true);
X('oscillator').start(440);
```

このように, オシレーターの管理からエフェクターの管理まで, Web Audio API が関わる処理はすべて XSound がめんどうをみる ... という感じの API でした.

しかしながら, フルスタックベースの API だと, **バニラな Web Audio API のコードと組み合わせにくい**, あるいは, **既存のプロダクトに XSound を導入しにくい** ... といった問題点がありました.

## モジュールベースの API

v2.20.0 以降は, これらの問題を解決するために, モジュールベースの API も併用可能にしました. 例えば, 先ほどのコードを, モジュールベースで実装する場合,

```JavaScript
const params = {
  time : 0.025,
  depth: 0.5,
  rate : 2.5,
  mix  : 0.5
};

const context = X.get();

const chorus = new X.Chorus(context, 0);

const oscillator = context.createOscillator();

oscillator.connect(chorus.INPUT);
chorus.OUTPUT.connect(context.destination);

chorus.param(params);
chorus.state(true);

oscillator.start(0);
```

このように, バニラな Web Audio API のコードに XSound のモジュールを組み込んでいることがわかります. さらに, このような API であれば, 既存のプロダクトに XSound を導入するということも, これまでよりはハードルが下がると思われます.

## 背景

XSound をローンチした 2012 年は, まだ jQuery が当然のように利用されている時代でした. そこで, XSound も jQuery ライクな API にすれば利用しやすいかもという考えで, フルスタックベースの API が実装されました (また, 実装も jQuery の影響を受けています).

2015 年以降, React に代表される Virtual DOM を備えた UI Component Library の普及とともに, 直接 DOM を操作することは少なくなり, 脱 jQuery が進みました. さらに, npm によるエコシステムの発展によって, パッケージを組み合わせてプロダクトを実装することが普及しました.

遅ればせながら, XSound もこの時代背景に追従しました.

# モジュールベースの実装

モジュールベースで利用可能なモジュールは, [Connectable](https://github.com/Korilakkuma/XSound/blob/master/src/interfaces/Connectable.js) インタフェースを実装したクラスです.

... といっても, それはコードを知っている必要があるので, [v2.20.0](https://github.com/Korilakkuma/XSound/releases/tag/v2.20.0) の時点でモジュールベースで利用可能なモジュールと, コンストラクタのシグネチャは以下のとおりです.

```TypeScript
type BufferSize = 0 | 256 | 512 | 1024 | 2048 | 4096 | 8192 | 16384;

X.Autopanner(context: AudioContext, size: BufferSize);
X.Chorus(context: AudioContext, size: BufferSize);
X.Compressor(context: AudioContext, size: BufferSize);
X.Delay(context: AudioContext, size: BufferSize);
X.Distortion(context: AudioContext, size: BufferSize);
X.Equalizer(context: AudioContext, size: BufferSize);
X.Filter(context: AudioContext, size: BufferSize);
X.Flanger(context: AudioContext, size: BufferSize);
X.Listener(context: AudioContext, size: BufferSize);
X.Panner(context: AudioContext, size: BufferSize);
X.Phaser(context: AudioContext, size: BufferSize);
X.PitchShifter(context: AudioContext, size: BufferSize);
X.Reverb(context: AudioContext, size: BufferSize);
X.Ringmodulator(context: AudioContext, size: BufferSize);
X.Stereo(context: AudioContext, size, size: BufferSize);
X.Tremolo(context: AudioContext, size: BufferSize);
X.Wah(context: AudioContext, size: BufferSize);

X.Analyser(context: AudioContext);

X.Recorder(context: AudioContext, size: BufferSize, numberOfInputs: number, numberOfOutputs: number);

X.Session(context: AudioContext, size: BufferSize, numberOfInputs: number, numberOfOutputs, analyser: X.Analyser);
```

`Connectable` インターフェースを実装したクラスは, コネクターとなる `INPUT` と `OUTPUT` プロパティが使えます (どちらも実体は `AudioNode` のゲッターです). これらが, バニラな Web Audio API のコードと組み合わせるときに重要となり, `AudioNode` を `INPUT` に接続して `OUTPUT` から `AudioNode` に接続することで, XSound との組み合わせが可能になります.

## エフェクター

モジュールベースで利用可能なエフェクターは以下のとおりです.

- オートパン
- コーラス
- コンプレッサー
- ディレイ
- ディストーション
- イコライザ
- フィルタ
- フランジャー
- パンナー / リスナー
- ピッチシフター
- リバーブ
- リングモジュレーター
- 擬似ステレオ
- トレモロ
- ワウ

コンストラクタのシグネチャはすべて同じです (第 1 引数に `AudioContext` インスタンス, 第 2 引数に `ScriptProcessorNode` のバッファサイズを指定します (`0` 指定で問題ないでしょう)).

```TypeScript
type BufferSize = 0 | 256 | 512 | 1024 | 2048 | 4096 | 8192 | 16384;

X.Autopanner(context: AudioContext, size: BufferSize);
X.Chorus(context: AudioContext, size: BufferSize);
X.Compressor(context: AudioContext, size: BufferSize);
X.Delay(context: AudioContext, size: BufferSize);
X.Distortion(context: AudioContext, size: BufferSize);
X.Equalizer(context: AudioContext, size: BufferSize);
X.Filter(context: AudioContext, size: BufferSize);
X.Flanger(context: AudioContext, size: BufferSize);
X.Listener(context: AudioContext, size: BufferSize);
X.Panner(context: AudioContext, size: BufferSize);
X.Phaser(context: AudioContext, size: BufferSize);
X.PitchShifter(context: AudioContext, size: BufferSize);
X.Reverb(context: AudioContext, size: BufferSize);
X.Ringmodulator(context: AudioContext, size: BufferSize);
X.Stereo(context: AudioContext, size, size: BufferSize);
X.Tremolo(context: AudioContext, size: BufferSize);
X.Wah(context: AudioContext, size: BufferSize);
```

モジュールベースでエフェクターを利用する場合, 必要なメソッドは, `param` と `state` です. それぞれ, エフェクターのパラメータの設定 or 取得, エフェクターのバイパスを切り替えるメソッドです.

```JavaScript
const params = {
  time    : 0.025,
  depth   : 0.5,
  rate    : 2.5,
  mix     : 0.5,
  feedback: 0.9
};

const context = X.get();

const flanger = new X.Flanger(context, 0);

const oscillator = context.createOscillator();

oscillator.connect(flanger.INPUT);
flanger.OUTPUT.connect(context.destination);

flanger.param(params);
flanger.state(true);

oscillator.start(0);
```

上記のコードは, (モジュールベースで) フランジャーを利用する例ですが, 先ほどのコーラスを利用する場合とほとんど同じコードであることがわかるかと思います (`param` メソッドで設定可能なパラメータは, エフェクターごとで大きく異なるので, 詳細は, [API Documentation](https://xsound.dev) を参考にしていください).

## ビジュアライゼーション

コンストラクタの引数は `AudioContext` インスタンスです.

```TypeScript
X.Analyser(context: AudioContext);
```

### domain

ビジュアライゼーションの対象となるドメインを第 1 引数に指定します. 第 2 引数は, チャンネルです.

```TypeScript
interface IFAnalyser {
  domain(domain: 'timeoverview' | 'time' | 'fft', channel: 0 | 1): Visualizer;
}
```

### Visualizer#setup

`HTMLCanvasElement` または, `SVGElement` を指定します.

```TypeScript
interface IFVisualizer {
  setup(element: HTMLCanvasElement | SVGElement): Visualizer;
}
```

### Visualizer#param

描画のスタイルなどを設定します.

```TypeScript
type GradientParams = {
  offset: number,
  color: string
};

type FontParams = {
  family: string,
  size: string,
  style: string
};

interface VisualizerParams {
  shape: 'line' | 'rect';
  grad: GradientParams[];
  wave: string;
  grid: string;
  text: string;
  font: FontParams;
  width: number;
  cap: 'round' | 'butt' | 'square';
  join: 'miter' | 'bevel' | 'round';
  top: number;
  right: number;
  bottom: number;
  left: number;
}

interface IFVisualizer {
  param(
    key: string | VisualizerParams,
    value?: number | string | GradientParams[] | FontParams
  ): number | string | GradientParams[] | FontParams | Visualizer;
}
```

### Visualizer#state

ビジュアライゼーションの状態を取得, または, ビジュアライゼーションの有効 or 無効を切り替えます.

```TypeScript
interface IFVisualizer {
  state(isActive?: boolean): boolean | Visualizer;
}
```

## レコーダー

コンストラクタの第 1 引数には `AudioContext` インスタンス, 第 2 引数には `ScriptProcessorNode` のバッファサイズ, 第 3 引数には `ScriptProcessorNode` の入力チャンネル数, 第 4 引数には `ScriptProcessorNode` の出力チャンネル数を指定します.

```TypeScript
X.Recorder(context: AudioContext, size: BufferSize, numberOfInputs: number, numberOfOutputs: number);
```

### setup

レコーディングに利用する, トラック数を指定します.

```TypeScript
interface IFRecorder {
  setup(numberOfTracks: number): Recorder;
}
```

### ready

レコーディング対象のトラック番号 (`0` ~) を指定します.

```TypeScript
interface IFRecorder {
  ready(track: number): Recorder;
}
```

### start

レコーディングを開始します. 引数はありません.

```TypeScript
interface IFRecorder {
  start(void): Recorder;
}
```

### stop

レコーディングを停止します. 引数はありません.

```TypeScript
interface IFRecorder {
  stop(void): Recorder;
}
```

### param

レコーディングのパラメータを指定します. 現状は, 左右のゲインのみです.

```TypeScript
interface IFRecorder {
  param(key: string, value: number): number | Recorder;
}
```

### clear

引数に指定したトラック (`0` ~) に格納されているデータをクリアします. `'all'` を指定するとすべてのトラックをクリアします.

```TypeScript
interface IFRecorder {
  clear(track: number | 'all'): Recorder;
}
```

### create

レコードしたデータを, WAVE ファイルとしてエクスポートします. 第 1 引数は, 対象のトラック (`0` ~, `'all'` を指定するとすべてのトラックのデータをミックスします), 第 2 引数は, モノラル (`1`), または, ステレオ (`2`), 第 3 引数は, 量子化ビット (8 bit or 16 bit), 第 4 引数は, エクスポートするデータ形式を指定します.

```TypeScript
interface IFRecorder {
  create(track: number | 'all', numberOfChannels: 1 | 2, qbit: 8 | 16, type: 'base64' | 'dataurl' | 'blob' | 'objecturl'): string | Blob;
}
```

### getActiveTrack

レコード対象となっているトラック番号 (`0` ~) を取得します. 対象がなければ `-1` を返します.

```TypeScript
interface IFRecorder {
  getActiveTrack(void): number;
}
```

## セッション

コンストラクタの第 1 引数には `AudioContext` インスタンス, 第 2 引数には `ScriptProcessorNode` のバッファサイズ, 第 3 引数には `ScriptProcessorNode` の入力チャンネル数, 第 4 引数には `ScriptProcessorNode` の出力チャンネル数, 第 5 引数には XSound が実装する `Analyser` インスタンスを指定します.

```TypeScript
X.Session(context: AudioContext, size: BufferSize, numberOfInputs: number, numberOfOutputs: number, analyser: X.Analyser);
```

### setup

第 1 引数には `wss` (TLS) を利用する場合に `true` を指定します. 第 2 引数には WebSocket サーバーのホスト名, 第 3 引数にはポート番号, 第 4 引数にはパス名, 第 5, 6, 7 引数にはイベントハンドラを指定します (WebSocket の `onopen`, `onclose`, `onerror` に対応します).

```TypeScript
interface IFSession {
  setup(
    tls: boolean,
    host: string,
    port: number,
    path: string,
    open?: Function,
    close?: Function,
    error?: Function
  ): Session;
}
```

### start

セッション (WebSocket によるバイナリメッセージング) を開始します. 引数はありません.

```TypeScript
interface IFSession {
  start(void): Session;
}
```

### close

セッションのコネクションをクローズします. 引数はありません.

```TypeScript
interface IFSession {
  close(void): Session;
}
```

### get

WebSocket インスタンスを取得します.

```TypeScript
interface IFSession {
  get(void): WebSocket;
}
```

### isConnected

セッションのコネクションが存在していれば, `true` を返します.

```TypeScript
interface IFSession {
  isConnected(void): boolean;
}
```

### state

セッションの状態を取得, または, セッションの有効 or 無効を切り替えます.

```TypeScript
interface IFSession {
  state(isActive?: boolean): boolean | Session;
}
```

# 関連リンク

- [API Documentation](https://xsound.dev)
- [XSound Playground](https://xsound.jp/playground/)
- [XSound.app](https://xsound.app)
