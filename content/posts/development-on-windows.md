+++
banner = ""
categories = ["Development", "Operating System"]
date = 2019-06-28T17:27:34+09:00
description = "Windows Subsystem for Linux による Windows 開発環境構築"
images = []
menu = ""
tags = ["Windows", "Linux"]
title = "Development on Windows"
+++

# まえおき

諸事情でしばらく Mac が使えなくなったので, しぶしぶ Windows でしばらく開発することにしました.
ハードウェア・OS は以下のとおりです.

## ideapad 710S Plus - 13lSK

<dl>
  <dt>プロセッサ</dt>
  <dd>Intel(R) Core(TM) i3-6100U CPU @ 2.30 GHz</dd>
  <dt>RAM</dt>
  <dd>4.00 GB</dd>
  <dt>OS</dt>
  <dd>Windows 10 Home</dd>
</dl>

もともとは, 2017 年の末に母親へのクリスマスプレゼントとして購入した PC でしたが, フラットデザインが気に入らなかったのか放置されていたので使うことにしました.

# Windows Subsystem for Linux (WSL) とは ?

これまで, Windows で UNIX ライクな開発環境を導入するには, MinGW や Cygwin を導入するのが一般的でした. しかし, これらのツールは, <b>Linux 環境のプログラムの動作や互換性</b>に問題がありました. もう少し簡単に表現すると, Linux コマンドを似た役割の Windows コマンドに置き換えて実行する (例えば, `ls` であれば `dir`) ので, 利用可能なコマンドを簡単に増やせないという欠点がありました. また, VirtualBox などの仮想環境では, Windows と連携できない, さらに, リソース (CPU やメモリ) を余分に消費してしまいます.

Windows Subsystem for Linux (WSL) はこれらのデメリットを解決するツールで, Linux のバイナリ実行ファイルを Windows 10 でネイティブ実行するための互換レイヤー (エミュレータ) です. つまり, macOS のコンソールのようなものを, Windows 10 でも利用可能になるということです.

<table>
  <thead>
    <tr>
      <th scope="col"></th>
      <th scope="col">MinGW</th>
      <th scope="col">Cygwin</th>
      <th scope="col">仮想化</th>
      <th scope="col">WSL</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Windows 環境との親和性, Windows アプリとの連携</th><td>〇</td><td>〇</td><td>X</td><td>〇</td></tr>
    <tr><th scope="row">実行速度・メモリ使用効率</th><td>〇</td><td>〇</td><td>X</td><td>〇</td></tr>

    <tr><th scope="row">Linux 環境プログラムの動作・互換性</th><td>X</td><td>X</td><td>〇</td><td>〇</td></tr>

  </tbody>
</table>

# インストール

## 機能の有効化

Windows で Linux Subsystem を有効化します.

スタートボタンを右クリックして, 「アプリと機能」を選択します.

![アプリと機能](https://user-images.githubusercontent.com/4006693/60387358-a5673d80-9adc-11e9-8d28-51eb0d353a73.png)
そして, 「プログラムと機能」を選択します.

![プログラムと機能](https://user-images.githubusercontent.com/4006693/60391031-7377ca80-9b20-11e9-8f5b-8a09dc5dab5f.png)

左側の一覧から, 「Windows の機能の有効化または無効化」を選択します.

![Windows の機能の有効化または無効化](https://user-images.githubusercontent.com/4006693/60391037-8db1a880-9b20-11e9-9ec8-7c5cad50969e.png)

Windows Subsystem for Linux にチェックをして, 「OK」を選択します.

![Windows Subsystem for Linux](https://user-images.githubusercontent.com/4006693/60389409-8deb7d00-9afb-11e9-81fb-ede83eaf97fd.png)

インストールが開始されるので完了するまで待ちます. 完了したら再起動します.

再起動したら, スタートメニューを右クリックして, 「設定」>「更新とセキュリティ」を選択し, 左側のメニューから「開発者向け」を選択して「開発者モード」にチェックをします (警告が表示されますが, そのまますすめて大丈夫です).

![開発者モード](https://user-images.githubusercontent.com/4006693/60387421-dc8a1e80-9add-11e9-94d1-7f93f50899f7.png)
## bash のインストール

### 古い WSL の削除

もし, インストールしている場合, 安全のために古い WSL は削除しておきます.

コマンドプロンプト, または PowerShell で以下のコマンドを実行します (ただし, このときすべてのデータが消去されるので, 必要なデータはバックアップを取得しておきましょう).

```bash
> lxrun /uninstall
```

### ストアから WSL をインストール

Microsoft Store を開いて, 検索窓で「Ubuntu」と検索します (お気に入りのディストリビューションがあれば, そちらで検索してください).

![Microsoft Store](https://user-images.githubusercontent.com/4006693/60387535-a0f05400-9adf-11e9-87f3-434919eeb31c.png)

![Ubuntu](https://user-images.githubusercontent.com/4006693/60387536-a51c7180-9adf-11e9-9f7f-07423fd0a858.png)

最後に, 「入手」ボタンをクリックします.

インストール作業が完了したら, スタートに追加される「Ubuntu」を起動して, インストールの完了を待ちます. 完了すると, 内部で利用する, ユーザー名とパスワードの設定を要求されるので入力します.

![Ubuntu Install](https://user-images.githubusercontent.com/4006693/60387629-f4af6d00-9ae0-11e9-859e-0da4aa7602d8.png)

### 初期設定

#### リポジトリの変更

デフォルトでは, 海外サーバーにリポジトリのデータを取得しにいくので, 日本のサーバーに切り替えます.

```bash
$ sudo sed -i -e 's%http://.*.ubuntu.com%http://ftp.jaist.ac.jp/pub/Linux%g' /etc/apt/sources.list
```

#### GitHub SSH 公開鍵の登録

デフォルトで git が利用可能ですが, そのままでは git から clone したり, push したりすることができないので, 公開鍵を登録します.

```bash
$ ssh-keygen
$ cat ~/.ssh/id_rsa.pub
```
`cat` コマンドで表示される公開鍵の内容をコピーして, GitHub の「Settings」>「SSH and GPG keys」>「New SSH key」を選択してペーストします (Title は, 任意で OK です).

最後に, SSH 接続します.

```bash
$ ssh -T git@github.com
```

`Hi Korilakkuma! You've successfully authenticated, but GitHub does not provide shell access.` のようなメッセージが表示されれば, 完了です.

#### git config

`git config` でいくつかの設定を入力・変更します.

```bash
$ git config --global user.email rilakkuma.san.xjapan@gmail.com
$ git config --global user.name "Tomohiro IKEDA"
$ git config --global core.editor vim
```

### パッケージのアップデート

パッケージのアップデートは定期的におこなう必要があります.

```bash
$ sudo apt update
$ sudo apt upgrade
```

### 任意設定

ここからは, 個人的な設定 (インストール) なので読み飛ばしていただいても大丈夫です.

#### gcc, g++, make のインストール

C/C++ を書くプログラマにとってはおそらくあったほうがよいかと思います.

```bash
$ sudo apt install gcc g++ make
```

#### nodebrew のインストール

```bash
$ curl -L git.io/nodebrew | perl - setup
```

`export PATH=$HOME/.nodebrew/current/bin:$PATH` を `~/.bashrc` に追記して,

```bash
$ source ~/.bashrc
```

で読み込みます.

```bash
$ nodebrew ls-remote
$ nodebrew install-binary v12.0.0  # インストールしたいバージョン
$ nodebrew use v12.0.0
```

```bash
$ node -v
v12.0.0
$ npm -v
6.9.0
```

のように表示されれば完了です.
