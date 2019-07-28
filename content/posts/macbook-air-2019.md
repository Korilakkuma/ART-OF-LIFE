+++
banner = ""
categories = ["Development"]
date = 2019-07-28T17:27:12+09:00
description = "MacBook Air 2019 の使用感とセットアップメモ"
images = []
menu = ""
tags = ["Mac", "dotfiles"]
title = "Macbook Air 2019"
+++

# MacBook Air 2019 を買いました

開封の儀 🎉

![MacBook Air 2019](https://user-images.githubusercontent.com/4006693/62010053-cc2e9780-b1a0-11e9-9056-e1ad5b550c59.JPG)
![MacBook Air 2019](https://user-images.githubusercontent.com/4006693/62010137-bc638300-b1a1-11e9-8e00-33feae71d71c.JPG)

## 使用感

久々に, MacBook Air に復帰したこともあり, やはりボディの薄さとバッテリーのもちはよく, 神出鬼没にコーディングしたり, ドキュメント書いたりするわたしには, やはり, MacBook Pro よりも Air のほうがあっているのかもしれません !

### スペック

店頭で購入可能な最大スペックの MacBook Air 2019 です.

- MacBook Air (Retina, 13-inch, 2019)
- Processor 1.6 GHz Intel Core i5
- Memory 8 GB 2133 MHz LPDDR3
- SSD 256 GB

以前は, Illustrator や Photoshop, また, 動画編集などの頻度が多かったですが, 最近はほとんどしないので, スペックはこれぐらいで十分という感じです (`ffmpeg` を利用するときは, まあ ... という感じもありますが, どのみち, スクリーンショット用の動画変換であることが多いので, そこまで気にならないです).

カラーはやはり, スペースグレイですね !

### キーボード

最も懸念される使用感ですね ... MacBook Pro 2017 の使用経験がありますが, そのキーボードよりさらに浅いタッチ感ですね. まあ ... 悪くはないといったところです.

### Touch ID

Touch ID (指紋認証) が可能です. これは, 思っていた以上に便利だったので, 設定しておくことをおすすめします.

## セットアップ

### Xcode のインストール

Xcode をインストールしないと, なにかと不便なのは変わりません. 時間がかかるので, 最初にインストール作業をします.

### Source Code Pro のインストール

Xcode のインストールをまっている間に可能な作業を進めます. ターミナル (iTerm2) などでは, わたしは Source Code Pro を利用するので, フォントをインストールします.

```bash
$ curl -LO https://github.com/adobe-fonts/source-code-pro/archive/release.zip
$ unzip release.zip
$ cp -a source-code-pro-release/TTF/* ~/Library/Fonts
```

iTerm2 を使いますが, しばらくは, ターミナルでも作業をするので, ターミナルの見た目を使いやすくカスタマイズします.

### Builtin Apache の設定

macOS のバージョンごとに微妙に異なってくるのが, この Builtin Apache の設定です. 業務用に利用している場合などは必要ないかもしれませんが, 趣味でも利用する場合, ちょっとした動作を確認するのに, Docker や webpack-dev-server を使うのはさすがに大げさなので設定しておきます.

```bash
$ sudo vim /etc/apache2/httpd.conf
```

L174 - L177 をコメントアウトします (まあ, PHP も使えるようにしておきます).

```
LoadModule userdir_module libexec/apache2/mod_userdir.so
LoadModule alias_module libexec/apache2/mod_alias.so
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
LoadModule php7_module libexec/apache2/libphp7.so
```

L511 をコメントアウトします.

```
Include /private/etc/apache2/extra/httpd-userdir.conf
```

最後に, 以下の設定を追加します.

```
Include /private/etc/apache2/users/*.conf
```

次に (`${whoami}` は, ユーザー名です),

```bash
$ sudo vim /etc/apache2/users/${whoami}.conf
```

```
<Directory /Users/${whoami}/Sites>
    Options Indexes MultiViews FollowSymLinks Includes
    AllowOverride All
    Require all granted
    Order allow,deny
    Allow from all
</Directory>
```

最後に,

```bash
$ sudo /usr/sbin/apachectl start
```

`http://localhost/~${whoami}/` でアクセスできれば OK です.

### SSH 公開鍵の生成

```bash
$ ssh-keygen
$ cat ~/.ssh/id_rsa.pub | pbcopy
```

生成した公開鍵を, GitHub に登録して,

```bash
$ ssh -T git@github.com
```

で, 成功メッセージが表示されれば OK です.

もし, git clone などで毎回パスフレーズをきかれる場合は,

```bash
$ sudo vim ~/.ssh/config
```

```
Host *
  UseKeychain yes
  AddKeysToAgent yes
```

という内容のファイルを作成しておきます.

### dotfiles でワンライナーインストール

Xcode のインストールが完了したら, はじめます.

利用する Shell の設定や Vim の設定, CLI / GUI アプリケーションのインストールなどはすべて, dotfiles でワンライナーインストールします.

... と言いつつ, わたし自身, dotfiles の作成・運用をはじめて間もないので, 非常にしょぼい [dotfiles](https://github.com/Korilakkuma/dotfiles) となっています. もっとよりよい開発環境をワンライナーでインストールしたい方は, 有名な dotfiles を git clone してください.

しかしながら, 有名な dotfiles はシンプルとは言え, dotfiles を学びはじめたビギナーの方には, 理解しずらいというデメリットもあります. 一応, わたしの dotfiles は, dotfiles のワンライナーインストールの仕組みを最低限理解できるようにはなっていると思います (dotfiles の流儀はおさえていると思います).

必要に応じて選択してください.

わたしの, dotfiles だと ...

```bash
$ git clone git@github.com:Korilakkuma/dotfiles.git
$ make deploy  # アプリケーションのインストールなど
$ make init  # ホームディレクトリにシンボリックリンクをはる
```

ちなみに, `brew cask install` で GUI (デスクトップ) アプリケーションをインストールできます. 古い記事では, `brew cask` は, 別途インストールが必要と記載されているものもありますが, 現在は, 最新の Homebrew をインストールすれば, `brew cask install` も可能なようです.

`brew cask install` でインストールできない, GUI アプリケーションは手動でインストールしましょう.

あとは, iTerm2 もターミナルと同様に外観をカスタマイズしておきます.
