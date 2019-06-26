+++
banner = ""
categories = ["Programming", "Multimedia", "Video", "Audio"]
date = "2019-05-29T20:00:59+09:00"
description = "WebRTC による P2P の概要を整理してみた"
images = []
menu = ""
tags = ["JavaScript", "WebRTC"]
title = "WebRTC の概要"
+++

# WebRTC

## Overview

WebRTC (Web Real-Time Communication) は, ブラウザ (ピア) での音声・動画通信, データ共有を可能にする技術です.
したがって, 様々な標準プロトコル (UDP など) や JavaScript API の集合体で定義されています.
... とは言え, ブラウザ API (JavaScript API) の視点では, 大きく 2 つの API に集約されます.

### MediaStream

音声や動画のストリームを取得.

### RTCPeerConnection

音声や動画の通信.

<!--
### RTCDataChannel

アプリケーションデータの通信.

-->

## ストリームの取得

`navigator.mediaDecives.getUserMedia` メソッドを利用して, 音声や動画を取得します
(`navigator.getUserMedia` メソッドは, 古い仕様なので利用しないでください).
引数に渡すプレインオブジェクトには, 様々な[オプション](https://developer.mozilla.org/en-US/docs/Web/API/Media_Streams_API/Constraints#Result)を設定できます (ノイズ除去, 画像補正 ... etc).
戻り値は, `Promise` で, 成功した場合のコールバックの引数には, `MediaStream` インスタンスが渡されます.

また, よくあるコード例がここから `HTMLVideoElement` で表示することですが, 最新の仕様と, 古い仕様で処理が大きく異なる点に注意してください.

```JavaScript
const constraints = {
  audio: true,
  video: true
};

navigator.mediaDecives.getUserMedia(constraints)
  .then((stream) => {
    const video = document.querySelector('video#local');

    if ('srcObject' in video) {
      video.srcObject = stream;
    } else {
      // legacy
      video.src = window.URL.createObjectURL(stream);
    }
  })
  .catch((error)) => {
    console.error(error);
  });
```

WebRTC は, 音声や動画に対して, 音質や画質の最適化, 音声と映像の同期, 出力ビットレートの調整などの複雑な処理を隠蔽しているので, アプリケーション実装側で, それらを意識することなく, 最適化されたストリームを取得できます.

### MediaStream

- `MediaStream` は 1 つ以上のトラック (`MediaStreamTrack`) によって構成される
- `MediaStream` に格納されている複数のトラックは同期されている
- 入力ソースは, マイクロフォンや Web カメラなどの物理デバイス, ディスクファイルなど
- `MediaStream` は, ローカルの動画や音声要素, ポストプロセスを実行する JavaScript, リモートピアなど 1 つ以上の出力先に送信可能

#### getTracks

MediaStream にはいくつかのトラックが含まれています (例えば, 動画と音声なら 2 つ). `MediaStream#getTracks` は, それらを `MediaStreamTrack` として個別に取得します. また, `MediaStream#getAudioTracks`, `MediaStream#getVideoTracks` メソッドでは, それぞれ, 音声のみ, 映像のみの `MediaStreamTrack` を取得することが可能です.

また, 後述する `RTCPeerConnection` には `addTrack` というメソッドがあるので, これを利用してトラックを自身に設定します.

```JavaScript
const constraints = {
  audio: true,
  video: true
};

navigator.mediaDecives.getUserMedia(constraints)
  .then((stream) => {
    for (const track of stream.getTracks()) {
      peerConnection.addTrack(track, stream);
    }

    const video = document.querySelector('video#local');

    if ('srcObject' in video) {
      video.srcObject = stream;
    } else {
      // legacy
      video.src = window.URL.createObjectURL(stream);
    }
  })
  .catch((error)) => {
    console.error(error);
  });
```

## RTCPeerConnection

### UDP

WebRTC は, データの転送に UDP (User Datagram Protocol) を利用します.
その理由は, データの信頼性よりも適時性が要求されるからです.
UDP は, TCP とは対照的です.

<dl>
  <dt>メッセージの到着を保証しない</dt>
  <dd>通知, 再送, タイムアウトは存在しない</dd>
  <dt>到着順序を保証しない</dt>
  <dd>パケットのシーケンス番号, 再構成, HoL ブロッキングは存在しない</dd>
  <dt>接続状況の追跡をしない</dt>
  <dd>接続確立や中継のステートマシンは存在しない</dd>
  <dt>輻輳制御をしない</dt>
  <dd>ビルトインのクライアントやネットワークフィードバックの仕組みをもたない</dd>
</dl>

実際には, DTLS (Datagram Transport Layer Security), SRTP (Secure Real-Time Transport Protocol) や SCTP (Stream Control Transmission Protocol) が追加で利用されており, UDP でありながら, TCP ライクなデータ転送を実現します.

### RTCPeerConnection API

しかしながら, 上記にあげたプロトコルのほとんどは, `RTCPeerConnection` という JavaScript API に隠蔽されており, アプリケーション開発者が意識することはほとんどありません.

```JavaScript
const options = {
};

const peerConnection = new RTCPeerConnection(options);
```

まずは, オプションを引数にインスタンスを生成する必要がありますが, このオプションしてには, ICE を理解しておく必要があります.

#### ICE

ICE (Interactive Connective Establishment) とは, WebRTC の通信経路候補を収集します.
デフォルトでは, プライベートネットワーク限定で経路を収集しますが, 多くの場合, インターネット間で通信をしたいはずです.

そのためには, ホストの IP アドレスが, インターネット側からどうなっているのかを, サーバーに問い合わせます.
このサーバーを STUN (Session Traversal Utilities for NAT) と呼びます.

また, ファイアウォールなど, 制約の多いネットワークでは, 自由に通信経路を選択できないので, 中継サーバーを利用します.
このサーバーを TURN (Traversal Using Relays around NAT) と呼びます.

#### シグナリング

シグナリングとは, P2P 通信を確立するための手順です.  一般的には, シグナリングサーバーを構築します. 通信開始時点では, 通信相手となるホストの情報がわからないので, シグナリングサーバーに探してもらう必要があります.

以下のコードは, WebSocket を利用した簡易的なシグナリングサーバーの実装例です.

```JavaScript
const WebSocketServer = require('ws').Server;
const port = (process.argv[2] > 0) && (process.argv[2] <= 65535) ? process.argv[2] : 3000;

const ws = new WebSocketServer({ port });

console.log(`Wait port (${port}) ...`);

const sockets = [];

ws.on('connection', (socket) => {
  sockets.push(socket);

  socket.on('message', (message) => {
    ws.clients.forEach((client) => {
      if (socket !== client) {
        client.send(message);
      } else {
        console.log('--- Skip Sender ---');
      }
    });
  });
});
```

P2P 通信を確立するためには, 通信要件をネゴシエーションする必要があります.

- 送受信の対象 (動画 ? 音声 ? データ ?, コーデックは ? ... etc)
- 通信経路 (IP アドレスとポート番号)

これらの通信要件は, SDP (Session Description Protocol) というフォーマットに記述され, P2P の確立のために送受信されます.

まずは, オファーの SDP を生成してシグナリングします.
そのためには, `RTCPeerConnection#createOffer` メソッドを利用します.
戻り値は, `Promise` で成功時のコールバックに `RTCSessionDescription` が渡されます (古い仕様では `Promise` は返らないので注意してください).
また, シグナリングと同時に, 自身の設定として `RTCPeerConnection` に反映するために, `RTCPeerConnection#setLocalDescription` メソッドを利用します.

```JavaScript
const ice = {
  iceServers: [
    { url: 'stun:stunserver.com:12345' },
    { url: 'turn:user@turnserver.com'. creadentials: 'pass' }
  ]
};

const signalingChannel = new SignalingChannel('ws://localhost:3000/');
const peerConnection = new RTCPeerConnection(ice);

peerConnection.createOffer()
  .then((offer) => {
    peerConnection.setLocalDescription(offer);
    signalingChannel.send(offer);
  });
  .catch((error) => {
    console.error(error);
  });
```

`SignalingChannel` は, クライアントにおけるシグナリング処理をまとめたクラスです (以下は, 実装例です).

```JavaScript
class SignalingChannel {
  constructor(url, onopen, onerror) {
    this.websocket = new WebSocket(url);

    this.websocket.onopen  = Object.prototype.toString.call(onopen) === '[object Function]' ? onopen  : () => {};
    this.websocket.onerror = Object.prototype.toString.call(onopen) === '[object Function]' ? onerror : () => {};
  }

  send(sessionDescription) {
    this.websocket.send(JSON.stringify(sessionDescription));
  }

  message(offer, answer, candidate) {
    this.websocket.onmessage = (event) => {
      const message = JSON.parse(event.data);

      switch (message.type) {
        case 'offer':
          if (Object.prototype.toString.call(offer) === '[object Function]') {
            offer(message);
          }

          break;
        case 'answer':
          if (Object.prototype.toString.call(answer) === '[object Function]') {
            answer(message);
          }

          break;
        default:
          if (message.candidate && (Object.prototype.toString.call(candidate) === '[object Function]')) {
            candidate(message);
          }

          break;
      }
    };
  }
}
```

#### トランスポートの候補

SDP を送信しても, トランスポート (IP アドレスとポート番号) の候補がないので通信できません.
そこで, `RTCPeerConnection#onicecandidate` イベントを監視することで取得できます.

```JavaScript
peerConnection.onicecandidate = (event) => {
  if (event.candidate) {
    signalingChannel.send(event.candidate);
  }
};
```

#### アンサー側の処理

シグナリングやトランスポート候補の監視は, 対となるピア (アンサー側と呼ぶことにします) でも実行する必要があります.
対となるので, やっていることはほぼ同じなのですが, 対となるがゆえに, メソッドが異なる部分 (`RTCPeerConnection#setRemoteDescription`, `RTCPeerConnection#createAnswer`) や, `WebSocket#onmessage` のイベントハンドラで処理を開始する部分などがオファー側と異なってきます.

また, `RTCPeerConnection#addIceCandidate` メソッドは, 経路候補を即時に取り込みます.

```JavaScript
const ice = {
  iceServers: [
    { url: 'stun:stunserver.com:12345' },
    { url: 'turn:user@turnserver.com'. creadentials: 'pass' }
  ]
};

const signalingChannel = new SignalingChannel('ws://localhost:3000/');

let peerConnection = null;

const offer = (message) => {
  peerConnection = new RTCPeerConnection(ice);
  peerConnection.setRemoteDescription(message);

  peerConnection.onicecandidate = (event) => {
    if (event.candidate) {
      signalingChannel.send(event.candidate);
    }
  };

  navigator.mediaDevices.getUserMedia({ audio: true, video: true })
    .then((stream) => {
      peerConnection.createAnswer()
        .then((answer) => {
          peerConnection.setLocalDescription(answer);
          signalingChannel.send(answer);
        })
        .catch((error) => {
          console.error(error);
        });
    })
    .catch((error) => {
      console.error(error);
    });
};

const answer = (message) => peerConnection.setRemoteDescription(message);

const candidate = (message) => peerConnection.addIceCandidate(message);

signalingChannel.message(offer, answer, candidate);
```

#### ontrack

最後に ... メディアが送受信されたときのイベントの監視です. そのためには, `RTCPeerConnection#ontrack` イベントを監視します (古い仕様では, `RTCPeerConnection#onaddstream` となっているので注意してください).

```JavaScript
peerConnection.ontrack = (event) => {
  const remote = document.querySelector('video#remote');
  const stream = event.streams[0] ? event.streams[0] : null;

  if ('srcObject' in remote) {
    remote.srcObject = stream;
  } else {
    // legacy
    remote.src = window.URL.createObjectURL(stream);
  }

  remote.play()
    .then(() => {})
    .catch(() => {});
};
```

## コード例 (全体像)

記事にした WebRTC の概要を, ローカルでデモできるリポジトリを作成したので, よかったら参考にしてみてください !

[Signaling](https://github.com/Korilakkuma/Signaling)


## 参考記事

- [シグナリングサーバーを動かそう ーWebRTC入門2016](https://html5experts.jp/mganeko/20013/)
- [WebRTCやるのに最低限必要なJavaScriptのAPIについて](https://lealog.hateblo.jp/entry/2019/04/22/095422)
