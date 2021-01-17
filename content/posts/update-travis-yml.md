+++
banner = ""
categories = ["Programming"]
date = 2019-07-26T05:27:00+09:00
description = ""
images = []
menu = ""
tags = ["CI"]
title = ".travis.yml の更新"
draft = true
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# Travis CI でエラー

昨日, Travis CI でビルドしたら, ソースコード・テストコードとも何も変更していないのに, 2 つのエラーが発生するようになりました ...

- `Can't open /etc/init.d/xvfb`
- テストの実行でタイムアウトエラー

ちなみに, それまでパスしていた .travis.yml は以下のような記述です.

```yml
sudo: true
language: node_js
node_js:
  - 10.0
before_install:
  - export CHROME_BIN=chromium-browser
  - export DISPLAY=:99.0
  - sh -e /etc/init.d/xvfb start
script:
  - karma start --single-run
```

簡単に説明すると, テストランナーに Karma を利用して, Chromium 環境でブラウザ JavaScript をテストするというものです.

## Can't open /etc/init.d/xvfb`

このエラーを解決するには, .travis.yml の記述を以下のように修正します.

```yml
sudo: true
language: node_js
node_js:
  - 10.0
services:
  - xvfb
before_install:
  - export CHROME_BIN=chromium-browser
  - export DISPLAY=:99.0
script:
  - karma start --single-run
```

`services:` に `xvdb` を指定することで解決できます. 原因は, あくまで推測ですが, Travis CI のアップデートによるものでしょうか ... ?

## テストの実行でタイムアウトエラー

この原因は, 環境変数 `CHROME_BIN` に設定している, `chromium-browser` のパスが見つからなくなっていることにより発生していました (URL が削除されたのでしょうか ... ?).

このエラーを解決するには, .travis.yml の記述を以下のように修正します.

```yaml
sudo: true
language: node_js
node_js:
  - 10.0
services:
  - xvfb
addons:
  chrome: stable
before_install:
  - export CHROME_BIN=google-chrome-stable
  - export DISPLAY=:99.0
script:
  - karma start --single-run
```

取得可能な Chrome の `stable` 版 (または, `beta` 版) をインストールして, `CHROME_BIN` にそのパスを指定します.

以上で, 無事 CI がパスしました.
