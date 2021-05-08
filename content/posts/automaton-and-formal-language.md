+++
banner = ""
categories = ["Computer Sience", "Automaton", "Language Theory"]
date = 2021-03-07T20:00:00+09:00
description = "オートマトンと形式言語概要と簡易 MML パーサの実装"
images = []
menu = ""
tags = ["Automaton", "Formal Language", "Finite Automaton", "Regular Language", "Pushdown Automaton", "Context Free Language"]
title = "Automaton and Formal Language"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# オートマトンと形式言語とは ?

オートマトンと形式言語は, [基本情報技術者試験](https://www.jitec.ipa.go.jp/1_13download/syllabus_fe_ver7_1.pdf)でもピックアップされる情報科学の基礎理論ですが, 実際のところなんのための理論なのかは説明されていないことも多く, とりあえず, 与えられた記号列が受理できるかどうかを判定して問題が解けるぐらいでとどまっていることも多いかと思います.

この記事では, オートマトンと形式言語について, 基本情報技術者試験などでピックアップされる内容よりは, もう少し深く, その概要を解説します. もっとも, 数学的証明などは, それをピックアップするだけでも 1 つの記事になるので, 詳細はリファレンスを参考にしてください.

ちなみに, この分野における (理論的な理解の) 数学的準備としては, **集合**, **写像**, **数学的帰納法**, **背理法** などが必要になります.

後半では, 理論の実践として, 仕様をかなり限定した **MML** (**Music Macro Language**) パーサの実装をしてみます.

## オートマトンと形式言語が基礎となっている技術

Web マイニングやコンパイラ・文書解析, 果ては, 生命科学におけるタンパク質の構造や遺伝子構造の記述・推定などの基礎になっています. 詳細は, のちのセクションで解説しますが, **正規表現** (正規言語: 有限オートマトンが受理する記号列) や,  C・Java などの**汎用プログラミング言語**や, Tex・HTML をはじめとする**マークアップ言語** (文脈自由言語: プッシュダウンオートマトンが受理する記号列) などは, すべて, オートマトンと形式言語が基礎となって成立している技術です.

情報科学から離れたとこでは, 自然言語 (日本語や英語), 数式などもオートマトンと形式言語で表現することができます.

## オートマトンとは ?

**オートマトン** (automaton) とは, 計算をしたり, ある文が言語に属するかを判断したり, 文を生成したりする抽象的機械で, かつ, **状態機械** (state machine) のことです. 状態機械とは, 以下の性質を有しています.

- **内部に状態をもっており**, 入力に応じて状態を変える
- 入力と状態に応じて出力するかもしれない

オートマトンは状態機械なので, 以下のような, **状態遷移図** (State Transition Diagram) で表現されることが多いです.

![状態遷移図](https://user-images.githubusercontent.com/4006693/110212241-c43aeb80-7edd-11eb-83f4-970987d00d46.png)

形式言語を考える場合は, 出力を考えず, ある特別の状態にたどり着くことのみを目的とします. その特別な状態を**受理状態** (accepting) と言います. そして, 受理状態にたどり着くような記号列のすべての集まりを, オートマトンが**受理する言語** (accepting language) , あるいは, オートマトンの**受理言語**と言います.

![オートマトンの例 (受理状態を 2 重線で表現しています)](https://user-images.githubusercontent.com/4006693/110212240-c309be80-7edd-11eb-9373-376b5ba5a4ce.png)

上記の (有限) オートマトンの場合, `ab` または, `acd` で終了, もしくはその後に, `0` ~ `9` の数字いずれかで終了する入力記号列 (例 `ab0123456789`) は受理する言語となります. 逆に, `a` で開始しない入力記号列や, `b` または, `d`, または, 数字で終了しない入力記号列は受理されない言語ということになります.

## 形式言語とは ?

**形式言語** (formal language) とは, **形式文法**にしたがって生成される記号列の全体集合のことです. **形式文法**とは, 特定の言語に属する文と属さない文を区別する規則の集合です. たとえば,

- 文は, 主部と述部からなる
- 主部は, 名詞句と格助詞からなる
- 述部は, 副詞句と述語からなる

...

などです. 自然言語の文法や計算機言語の文法, さらには, DNA 系列やタンパク質の構造そのものの文法などもこのような規則の集合とみなせます.

## オートマトンと形式言語の関係

オートマトンが受理する言語と形式言語には以下の関係があります.

<table>
  <caption>オートマトンと形式言語の関係</caption>
  <thead><tr><th scope="col">オートマトン</th><th scope="col">形式言語</th></tr></thead>
  <tbody>
  <tr><td>有限オートマトン</td><td>正規言語 (3 型言語)</td></tr>
  <tr><td>プッシュダウンオートマトン</td><td>文脈自由言語 (2 型言語)</td></tr>
  <tr><td>線形拘束オートマトン</td><td>文脈依存言語 (1 型言語)</td></tr>
  <tr><td>チューリングマシン</td><td>句構造言語 (0 型言語)</td></tr>
  </tbody>
</table>

## 受理言語能力

オートマトンの種類によって, 受理できる言語能力に差があります.

### 有限オートマトン

最も低い受理言語能力は, 有限オートマトン (Finite Automaton) が受理する**正規言語** (Regular Language)であり, その代表格が**正規表現** (Regular Expression)です.

正規表現は, **連結** (Concatenation), **選択** (Union), **閉包** (Closure, クリーネ閉包とも呼ばれます) の 3 つで構成されており, これらは, 有限オートマトンで表現可能です. 以下は, その例です.

- 連結 `[abcd][0-9]`
- 選択 `[abcd]`
- 閉包 `[abcd][0-9]*`

プッシュダウンオートマトンやチューリングマシンでは, 状態遷移のほかに補助記憶として, スタックや左右に移動可能な無限長のテープをもつことを考えます. この補助記憶によって, 有限オートマトンよりも受理言語の能力が広がります.

### プッシュダウンオートマトン

**プッシュダウンオートマトン** (Pushdown Automaton) は, **スタック**を補助記憶としてもつことで, **文脈自由言語** (Context Free Language) を受理します (**プッシュダウン**は, スタックに由来します). 汎用プログラミング言語 (C, Java など) や, Tex・HTML といったマークアップ言語, 英語や日本語などの自然言語も, すべて文脈自由言語です.

#### 例: HTML の定義

HTML は大きく, 以下の 6 つのトークン (意味のある記号列のまとまり) に分類することが可能です.

| HTML トークンの種類 | 説明                                                |
|---------------------|-----------------------------------------------------|
|文書型定義 (DTD)     |`<!DOCTYPE html>`                                    |
|開始タグ             |`<head>`, `<body>`, `<main>` など                    |
|終了タグ             |`</head>`, `</body>`, `</main>` など                 |
|テキスト             | タグを含まない任意の記号列                          |
|コメント             | `<!-- -->` で囲まれた記号列. 解析上は意味をもたない |
|EOS                  | End Of String (HTML の終端)                         |

テキスト `<`, `>` そのものは `&lt:` と `&gt:` で表現する (**実体参照**) ので, テキストの中にタグが入ることはありません. これは**正規表現で表現可能**です. しかしながら, 開始タグと終了タグの対が正しい構造を成しているかは, 正規表現 (正規言語) では表現できず, **文脈自由言語であれば表現可能**です (具体的には, **挿入モード** (The insertion mode) などをスタックにプッシュしておくことで, 表現が可能になります).

### チューリングマシン

**チューリングマシン** (Turing Machine) は, **左右にも読み込み可能で, 無限長のテープ**を補助記憶としてもつことで, **句構造言語** (Phrase Structure Language) を受理します. 句構造言語は, 文脈依存言語を含めた, 文脈自由言語, 正規言語すべてを受理します (各言語クラスは, **真の包含関係**にあります).

![言語クラスの包含関係](https://user-images.githubusercontent.com/4006693/110229062-f124e800-7f49-11eb-83f1-f8a6b2408352.png)

#### チャーチの提唱

> 計算できる関数とは, その関数を計算するチューリングマシンが存在する関数のことである

### チューリングマシンの停止問題

#### 決定問題

**決定問題**とは, 解が `true` または `false` のいずれかである問題のことです. 例えば,

$Q$: $n$ 変数の多項方程式 $f(x_{1}, \cdots, x_{n})=0$ を与えたとき, その方程式が整数解をもつか ?

この問題 $Q$ は, 決定問題です.

決定問題の具体的な個々の問題を $Q$ の**インスタンス**と呼びます. 例えば, 「$x^4+3xy+y^2=0$ が整数解をもつか ? 」は, $Q$ のインスタンスと言えます.

#### 決定可能性と決定不能性

決定問題が**決定可能**であるとは, あるチューリングマシン $M$ が存在して, 任意のインスタンス $\omega$ に対して, M は $\omega$ を入力として与えられたとき, $\omega$ の答えを出力してチューリングマシンが停止 (受理) するときのことです.

そうでない場合, **決定不能**であると表現します.

チューリングマシンの停止問題は, **決定不能**です.

証明は背理法によって可能です. 以前, ブログでも紹介しておりますので, 参考までに ([Mathematics for Programmer](https://art-of-life.jp/posts/mathematics-for-programmer/#%E5%81%9C%E6%AD%A2%E5%88%A4%E5%AE%9A%E5%95%8F%E9%A1%8C)).

#### 計算時間

入力 $\omega$ に対する, チューリングマシン $M$ の計算時間とは, $M$ が $\omega$ を受理にかかわらず, $M$ が停止するまでのステップ数 (停止しない場合は, 無限大) として定義されます. 例えば, チューリングマシン $M$ が, 長さ $n$ の任意の語に対する計算時間が, 最大 $T(n)$ であるとき, $M$ の**時間計算量** (Time Complexity) は $T(n)$ と表現します.

#### 多項式時間で解ける問題

問題 $P$ に対し, それを解くチューリングマシン $M$ が存在し, かつ, 多項式 $Q(x)$ が存在し, 長さ $n$ の任意の入力に対して最大 $Q(n)$ のステップで $M$ が停止するとき, $P$ は**多項式時間で解ける**と表現します.

#### クラス $P$ と クラス $NP$

決定性 (遷移先がだた 1 つに決まる) のチューリングマシンで, 多項式時間で解ける問題のクラスを$P$ (Polynomial) と表現します. ある言語 $L$ が, $P$ に属するとは, $L$ を受理する決定性チューリングマシン $M$ と, 多項式 $Q(x)$ が存在して, M は, 長さ $n$ の任意の入力に対して, 最大 $Q(n)$ のステップで停止する場合, **実行可能**であると表現 (そうでない場合, **実行不能**と表現) し, **$P$ に属する**と表現します.

一方で, 非決定性 (遷移先がだた 1 つに決まらない) のチューリングマシンで, 多項式時間で解ける問題のクラスを $NP$ (Non Deterministic Polynomial) と表現します. ある言語 $L$ が, $NP$ に属するとは, $L$ を受理する非決定性チューリングマシン $M$ と, 多項式 $T(n)$ が存在して, M は, 長さ $n$ の任意の入力に対して, 動作系列の長さが $T(n)$ 超えない場合となります.

クラス $NP$ はあきらかに, クラス $P$ を含みますが, $P$ に含まれない $NP$ の要素が存在するか否か ... いわゆる,

**$P$$\neq$$NP$ 問題**は, **未解決問題**です.

![$P$$\neq$$NP$ 問題](https://user-images.githubusercontent.com/4006693/110218531-99608f80-7efd-11eb-91db-7874ba351d93.png)

<!--
## 有限オートマトンと正規言語

### 決定性有限オートマトン

状態機械のうち, 以下の 3 つの制約をつけた状態機械を**決定性有限オートマトン** (Deterministic Finite Automaion, 以下 DFA と表現します) と呼びます.

- 状態の個数が有限である
- 各状態に対して, どの記号が入力として与えられても, 遷移先の状態が一意に決まる
- 状態のうち, 初期状態と受理状態という状態がある. 初期状態は必ず 1 つ存在し, 受理状態は 0 個以上存在する

![DFA の状態遷移図の例]()

数学的に定義すると, DFA $M$ は以下の 5 つ組 (5 - tuple) です.

$M={\langle}Q, {\sum}, {\delta}, q_{0}, F{\rangle}$,

- $Q$ は, 状態の有限集合
- ${\sum}$ は, 入力記号の有限集合
- ${\delta}$ は, **状態遷移関数** (transition function)
- $q_{0}$ は, $Q$ の 1 要素で, **初期状態** (initial state)
- $F$ は, $Q$ のある部分集合で, **受理状態** (accepting state)

記号列 (**語**) ${\omega}$ を左から順に 1 つずつ読んでは, 現在の状態と読んだ文字 (入力) に応じた状態に遷移, ${\omega}$ を読みきったときに受理状態であれば, $M$ (DFA) は ${\omega}$ を受理する. このような語の集合が, DFA が受理する言語となります.

![DFA が受理する言語例]()

### 非決定性有限オートマトン

### 正規表現

### 状態数最小のオートマトン

### 正規言語

### ポンプの補題

## プッシュダウンオートマトンと文脈自由言語

### 決定性プッシュダウンオートマトン

### 非決定性プッシュダウンオートマトン

### 文脈自由言語

#### 簡素化と標準形

### 構文解析

#### 下降型構文解析

#### 上昇型構文解析

#### 文法のあいまいさ

### ポンプの補題

## チューリングマシン

### 線形拘束オートマトン

### 文脈依存言語

### 句構造言語

## チョムスキーの階層

## チューリングマシンの停止問題
-->

# MML パーサの実装

## MML とは ?

**MML** (**Music Macro Language**) とは, 音楽演奏情報を表現する形式言語の 1 種です (同じ略語に, **Music Markup Language** がありますが, これは別物なので注意してください). 例えると, 楽譜を ASCII 文字列で表現した言語と言えます.

MML は, プログラミング言語やマークアップ言語のような厳格な仕様はなく, また, WHATWG のような規格・標準化を図る組織も存在しません. 一応, 以下のような, なんとなくの仕様があります.

- `T` テンポ (BPM) を表現します.
- `O` オクターブを表現します.
- `C` `D` `E` `F` `G` `A` `B` それぞれ, ド・レ・ミ・ファ・ソ・ラ・シ の音を表現します.
- `+` (`#`), `-` シャープ, または, フラット. つまり, 半音の上げ下げを表現します.
- `R` 休符を表現します.
- `0` `1` `2` `3` `4` `5` `6` `7` `8` `9` 音符や休符の長さを表現します. `4` が 4 分音符です.
- `.` 付点音符・付点休符を表現します.
- `&` タイ (2 つの音符を連結) を表現します.

他にも, ボリュームの指定記号 (`V`) があったり, 音色の指定記号 (`@`) があったりします.

## MML の言語仕様

ここで, ミニマムな MML の言語仕様を決定しましょう. 音楽情報として成り立つためには, 音を表現する `C` ~ `B` と半音を制御する, `+` (`#`) と `-`, その長さ `0` - `9` と `.` の記号, そして, 休符の `R` が必要なことが考えられます.

ただし, これだけだと, 時間の算出が不便なので, テンポを表現する `T` もあるとよさそうです. また, 1 オクターブ分の音程だと表現できる音楽に, かなり制限がかかるので, オクターブを制御する `O` も追加します.

方言 (仕様のちがい) が MML において大きいので, **コード (和音) は, 表現不可能**とします.

他にも, スラーやスタッカートなど音楽記号がありますが, なくても, コンピュータに演奏させるにはそれほど不自然でなかったり, 他の用途で代替可能 (スタッカートであれば, 音の長さを半分にして, 休符を追加する) と思われます.

可読性のために, 最終的な構文の間にスペースは (MML パーサのアルゴリズムとして) いくつでも許容します (もちろん, メモリは有限なので, リソース的には無限というわけにはいきませんが ...).

例えば, 以下の MML は (構文的に) 等価です.

`T60O3R16A16B-16O4D16F12A12O5F12.C2C2R4D2.`
`    T60   O3 R16      A16 B-16 O4 D16    F12 A12 O5 F12. C2 C2 R4 D2.    `

逆に, 許容できなスペースの例です.

- BPM 72 を意図してる `T 7 2`
- 付点二分音符を意図している `D2 .`
- C# (C の半音上げ) を意図している `C + 4`

決定した, ミニマムな MML の入力記号列をまとめると, 以下の表のようになります.

|記号                                    | 意味                                            |
|----------------------------------------|-------------------------------------------------|
|`T`                                     | テンポ (BPM)                                    |
|`O`                                     | オクターブ                                      |
|`C` `D` `E` `F` `G` `A` `B`             | ド・レ・ミ・ファ・ソ・ラ・シ (単音のみ)         |
|`+` `-`                                 | 半音制御 (`#` はミニマムにするためなしとします) |
|`R`                                     | 休符                                            |
|`0` `1` `2` `3` `4` `5` `6` `7` `8` `9` | 音の長さ                                        |
|`.`                                     | 音の長さを 1.5 倍にのばす                       |
|スペース                                | 可読性の確保. 構文の間であればいくでも許容する  |

また, 区別をつけるため, この仕様の MML を以降は, **X-MML** と呼ぶことにします.

## X-MML の有限オートマトン

入力記号 (列) が決定しただけでは, 言語としては成立しません. 一定の規則, つまり, 文法にしたがった記号列かどうかを判定する必要があります.

結論から先に述べますと, X-MML は, スタックや無限長のテープなどの補助記憶は必要としません. すなわち, **有限オートマトンで受理できる記号列**であり, 言語クラスの包含関係から, 正規言語であり, 文脈自由言語であり, 文脈依存言語であり, 句構造言語でもあります.

![言語クラスの包含関係](https://user-images.githubusercontent.com/4006693/110229062-f124e800-7f49-11eb-83f1-f8a6b2408352.png)

それでは, X-MML の有限オートマトンを考えます.

まず, 1 つの遷移ルール (状態遷移関数) として, 必ずはじめに `T` が入力記号にくる必要があるとします. 言い換えれば, `T` から開始していない入力記号列は, 受理されない, すなわち, X-MML ではありません.

また, テンポには, 数値が必要なので, `T` の後には必ず `0` ~ `9` の数字が 1 つ以上入力される必要があります.

![X-MML の有限オートマトン テンポ](https://user-images.githubusercontent.com/4006693/110208904-14f61880-7ecd-11eb-91a0-e006ff143b87.png)

次に, 音の高さを決める記号列が続きますが, オクターブが最初に決まっていると, 音の高さが算出しやすいので, `C` ~ `B` が最初に入力されるより前に. `O` の記号が入力されている必要があるという遷移ルール (状態遷移関数) を追加します. さらに,`O` の後には必ず `0` ~ `9` の数字が 1 つ以上入力される必要があります.

![X-MML の有限オートマトン オクターブ](https://user-images.githubusercontent.com/4006693/110208901-132c5500-7ecd-11eb-9c8e-75c9cb414942.png)

そして, 音の高さと音の長さを決定する遷移ルール (状態遷移関数) を追加します. `C` ~ `B` のいずれかで開始して, 必要であれば, `+` or `-` の半音制御の記号, 音の長さを表現する `0` - `9` の記号, または, 付点を表現する `.` が終了するとします.

コードは表現できない仕様 (単音のみの仕様) なので, `C` ~ `B` のいずれかが入力記号にあると, その後には, `+` or `-`, あるいは, `0` - `9` の数字のいずれかが入力記号としてあるはずです.

![X-MML の有限オートマトン 音の高さと音の長さ](https://user-images.githubusercontent.com/4006693/110210905-4bd12c00-7ed7-11eb-8a03-078ea344bbed.png)

休符の場合はよりシンプルで, `R` のあとは, 必ず, `0` - `9` の数字が 1 つ以上, 入力記号になります.

![X-MML の有限オートマトン 休符](https://user-images.githubusercontent.com/4006693/110208903-145d8200-7ecd-11eb-9d75-1c36dabe4033.png)

まとめると, X-MML の有限オートマトンは以下のような状態遷移図となるはずです.
直感的に, **テンポで開始していない, または, 音符か休符で終了していない記号列は, 受理されない**ことがわかるかと思います
.
![X-MML の有限オートマトン (T は `T` と数字, O は `O` と数字, N は `C` ~ `B`, (`+`, `-`) と数字, R は `R` と数字)](https://user-images.githubusercontent.com/4006693/110209824-f1819c80-7ed1-11eb-97c4-fcafb6efe523.png)

## X-MML の実装

まずは, 入力記号となる, MML トークンを列挙体に定義します. `+`, `-` は, `NOTE`, `.` は `NUMBER`  に関わるのでそれらに含めておきます (実装としてどのように落とし込むかのちがいなので別でも問題ありません). MML トークンとして認識できない文字のために `UNKNOWN_TOKEN_ERROR`, また, 字句解析中のためのバッファオーバーフローを識別するために `BUFFER_OVERFLOW_ERROR` を定義しています.

また, 入力記号から, MML トークンを引き出せるように, `TokenMap` という構造体 (辞書, マップ ... etc) を定義しています.

さらに, トークンに関する情報をまとめて格納できるように, `Token` 構造体を定義しています.

X-MML トークンの定義

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TOKEN_LENGTH 64

typedef enum {
  TEMPO,
  OCTAVE,
  NOTE,
  REST,
  NUMBER,
  BUFFER_OVERFLOW_ERROR,
  UNKNOWN_TOKEN_ERROR,
  EOM  // End Of MML
} TokenTypes;

typedef struct {
  char token;
  TokenTypes type;
} TokenMap;

typedef struct {
  TokenTypes type;
  char token[TOKEN_LENGTH];
  int value;
} Token;

extern char *tokenize(char *in, Token *token);
extern void tokenize_print(char *in, Token *tokens);
```

まず, 内部結合の関数 (プライベート関数) `get_token_type` は, 文字を入力としてうけとると, `TokenMap` を参照して, 対応する `TokenTypes` を戻り値として返します.

Tokenization (字句解析の実体) は, `tokenize` 関数です. 入力文字列と, トークン情報を格納する `Token` 構造体へのポインタを引数にとり, 次に字句解析を開始する文字列へのポインタ (文字列の先頭アドレス) を返します.

最初に, トークンの対象とならないスペースを読みとばします (スペースであれば `char *` 1 つぶん, ポインタを進めます).

スペースでなければ, 現在の文字からトークンの種別を取得します.

ここで, `EOM`, つまり, 文字列の終端を表すヌル文字 (`\0`) であれば, `Token` 構造体に情報を格納して字句解析終了です.

簡単なトークンから解説すると, トークンタイプが `TEMPO`, `OCTAVE`, `REST` であれば, `Token` 構造体にその情報を格納するだけです.

`NOTE` の場合は, 次の文字が `+` or `-` であるかどうかの判定をするために, 1 文字先読みします (先読みするためにポインタを進めるので, 対象の文字でなかった場合は, ポインタを戻します).

`NUMBER` の場合も, 同じく先読みして, `NUMBER` であるかぎり, 次の 1 文字を先読みし続けます (ただし, バッファオーバーフローをしないかチェックし, そうであれば, `BUFFER_OVERFLOW_ERROR` を格納して終了します). 数字を読み終えたら, 文字列を `strtol` 関数で, 数値 (`int`) に変換します (ただし, `.` は無視されます).

`tokenize_print` は, 字句解析したトークンの情報を出力する関数です.

X-MML Tokenization (字句解析)
```c++
#include "token.h"

static TokenTypes get_token_type(char c);

static TokenMap token_map[] = {
  {'T', TEMPO},
  {'O', OCTAVE},
  {'C', NOTE},
  {'D', NOTE},
  {'E', NOTE},
  {'F', NOTE},
  {'G', NOTE},
  {'A', NOTE},
  {'B', NOTE},
  {'+', NOTE},
  {'-', NOTE},
  {'R', REST},
  {'0', NUMBER},
  {'1', NUMBER},
  {'2', NUMBER},
  {'3', NUMBER},
  {'4', NUMBER},
  {'5', NUMBER},
  {'6', NUMBER},
  {'7', NUMBER},
  {'8', NUMBER},
  {'9', NUMBER},
  {'.', NUMBER},
  {'\0', EOM}
};

char *tokenize(char *in, Token *token) {
  while (*in == ' ') {
    ++in;
  }

  TokenTypes t = get_token_type(*in);

  if (t == EOM) {
    token->type     = EOM;
    token->token[0] = *in;
    return in;
  }

  switch (t) {
    case TEMPO:
      token->type     = t;
      token->token[0] = *in;
      token->token[1] = '\0';
      break;
    case OCTAVE:
      token->type     = t;
      token->token[0] = *in;
      token->token[1] = '\0';
      break;
    case NOTE:
      token->type     = t;
      token->token[0] = *in;
      token->token[1] = '\0';

      // Look-ahead
      ++in;

      if ((*in == '+') || (*in == '-')) {
        token->token[1] = *in;
        token->token[2] = '\0';
      } else {
        // Look-behind
        --in;
      }

      break;
    case REST:
      token->type     = t;
      token->token[0] = *in;
      token->token[1] = '\0';
      break;
    case NUMBER:
      token->token[0] = *in;
      token->token[1] = '\0';

      // Look-ahead
      while ((*(++in) != '\0') && (get_token_type(*in) == NUMBER)) {
        size_t len = strlen(token->token);

        if (len < (TOKEN_LENGTH - 1)) {
          token->token[len + 0] = *in;
          token->token[len + 1] = '\0';
        } else {
          token->type     = BUFFER_OVERFLOW_ERROR;
          token->token[0] = *in;
          token->token[1] = '\0';

          return in;
        }
      }

      token->type  = NUMBER;
      token->value = (int)strtol(token->token, NULL, 10);

      // Look-behind
      --in;

      break;
    default:
      token->type     = UNKNOWN_TOKEN_ERROR;
      token->token[0] = *in;
      token->token[1] = '\0';
      break;
  }

  return ++in;
}

void tokenize_print(char *in, Token *tokens) {
  while (1) {
    Token token;

    token.type     = UNKNOWN_TOKEN_ERROR;
    token.token[0] = '\0';
    token.value    = -1;

    char *type;

    in = tokenize(in, &token);

    switch (token.type) {
      case BUFFER_OVERFLOW_ERROR:
        fprintf(stderr, "Error: Buffer Overflow\n");
        exit(EXIT_FAILURE);
      case UNKNOWN_TOKEN_ERROR:
        fprintf(stderr, "Error: %s is Unknown Token\n", token.token);
        exit(EXIT_FAILURE);
      default:
        switch (token.type) {
          case TEMPO:
            type = "TEMPO";
            break;
          case OCTAVE:
            type = "OCTAVE";
            break;
          case NOTE:
            type = "NOTE";
            break;
          case REST:
            type = "REST";
            break;
          case NUMBER:
            type = "NUMBER";
            break;
          case BUFFER_OVERFLOW_ERROR:
          case UNKNOWN_TOKEN_ERROR:
            exit(EXIT_FAILURE);
          case EOM:
          default :
            type = "EOM";
            break;
        }

        fprintf(stdout, "%-6s %4s (value=%3d)\n", type, token.token, token.value);

        *tokens++ = token;

        break;
    }

    if (token.type == EOM) {
      break;
    }
  }
}

static TokenTypes get_token_type(char c) {
  if (c == '\0') {
    return EOM;
  }

  for (int i = 0; token_map[i].token != '\0'; i++) {
    if (c == token_map[i].token) {
      return token_map[i].type;
    }
  }

  return UNKNOWN_TOKEN_ERROR;
}
```

## X-MML の構文解析

### 再帰的下向き構文解析 (下降型構文解析)

**再帰的下向き構文解析**とは, (Yacc のような自動生成ツールを使わずに) 構文解析を実装する場合, 主流となっている構文解析の方法です. 主流となっている理由は簡単に説明すると,

- 構文規則と 1 対 1 に対応したコードを実装するのでわかりやすい
- 演算子の優先順位を考慮する必要がない
- 一般文の解析にも使うことができて, 適用範囲が広い

といったことがあげられます.

再帰的下向き構文解析では, 構文図や BNF (バッカス記法) で表現される構文規則をそのまま適用します. したがって, 先ほど定義した, X-MML の有限オートマトンと 1 対 1 に対応するようにコードを実装すれば, 再帰的下向き構文解析となります.

![X-MML の有限オートマトン (構文図)](https://user-images.githubusercontent.com/4006693/110209824-f1819c80-7ed1-11eb-97c4-fcafb6efe523.png)

再帰的下向き構文解析を実装する場合, 構文規則は以下の 2 つの条件を満たす必要があります.

- 次のトークンを取得したときに, 構文規則として何をすべきかが一意に決定される
- 左再帰性をもたない (左再帰性を除去する)

前者の条件が必要な理由は直感的にも理解できるかと思います. オートマトンで遷移する場合, 一意に遷移先が決定できないと, 次の状態を決定できないはずです (一意に決まらない遷移 (**$\epsilon$ 遷移**) をもつオートマトンは, **非決定性オートマトン**と呼ばれます).

後者の条件が必要な理由は簡単で, 無限ループをつくらないためです. X-MML のテンポで説明します. BNF (バッカス記法) で表現すると, 左再帰性をもたないテンポは, 以下のようになります.

```
<TEMPO> ::= T<Digits>
<Digits> ::= <Digits>|0|1|2|3|4|5|6|7|9
```

ここで, 左再帰性をもつようにしてしまうと, 左辺の `<TEMPO>` を右辺で置換したときに, 再び `<TEMPO>` が出現するので, 永遠に終端記号にたどりつくことができずに, 無限ループとなってしまいます.

```
<TEMPO> ::= T<Digits><TEMPO>
<Digits> ::= <Digits>|0|1|2|3|4|5|6|7|9
```

### 構文木

先ほどの BNF のように, 構文規則の左辺を右辺で置換していく処理を繰り返すことが実は構文解析でもあります. 左辺を右辺に変換する処理を, **導出**, **生成**, **書き換え**などと表現します.

そして, このような導出過程の表現形式として**木構造**が適しており, 構文解析過程を保持する木構造を**解析木**と呼びます. ただし, 解析木は, 実際のコード生成には無駄な情報も含んでいるので, 解析木からコード生成に必要な要素のみ抽出した木構造を**構文木** (**抽象構文木**) と呼びます.

結論から見せますと, X-MML の構文木は以下のようになります (EOM は **E**nd **O**f **M**ML で MML の終端を意味します).

<img src="https://user-images.githubusercontent.com/4006693/111919331-d4052300-8acc-11eb-8b43-c3b036f2daca.png" alt="X-MML の構文木 (X-MML の実行結果)" style="max-width: 100%;" />

この構文木を, 行きがけ順 (preorder), つまり,

1. 根
1. 左部分木
1. 右部分木

の順で走査していくと, 入力された MML を構築できることがわかるかと思います.


それでは, 実際の構文解析のコードを解説します.

`Tree` は, 木構造のノード (根, 葉を含む) を表現する構造体です, トークンと, 左の子, 右の子をもつ**二分木**となります.

X-MML 構文解析
```c++
#include "../tokenizer/token.h"

typedef struct Tree {
  Token *token;
  struct Tree *left;
  struct Tree *right;
} Tree;

extern void tree_construct(Token *tokens, Tree *trees);
extern void tree_destruct(Tree *trees);
extern void tree_print(Tree *trees);
```

`tree_construct` が, 構文解析のメインルーチンです. 関数が終了しても, ノードは保持しておく必要があるので, メモリを動的に確保しています. また, 根となるテンポ以外では, 1 つ前のポインタ (ノード) を参照して, 現在のノードを右の子 (右部分木) として連結している処理に着目してください.

そして, 再帰呼び出しで, 根から葉に向かって, つまり, **下向き**に構文木を生成します.

ちなみに, `default` ラベルの処理は, 字句解析が正しく実行されていれば, 実行されることはないはずです.

`tree_destruct` は, `tree_construct` で確保したノードのための動的メモリを解放するルーチンです. 再帰呼び出しで, 右部分木の葉まで, ポインタを進めて, そのあとは, 1 つずつポインタを戻して, 左の子, 右の子のメモリを解放していきます.

`tree_print` は, 構文解析した構文木を出力する関数です.

```c++
#include "tree.h"

void tree_construct(Token *tokens, Tree *trees) {
  Tree *tree  = (Tree *)malloc(sizeof(Tree));
  Tree *left  = (Tree *)malloc(sizeof(Tree));
  Tree *right = (Tree *)malloc(sizeof(Tree));

  switch (tokens->type) {
    case TEMPO:
      // Left leaf
      left->token = tokens + 1;  // Look-ahead
      left->left  = NULL;
      left->right = NULL;

      // Right partial tree
      right->left  = NULL;
      right->right = NULL;

      // Root
      tree->token = tokens;
      tree->left  = left;
      tree->right = right;
      break;
    case OCTAVE:
      // Left leaf
      left->token = tokens + 1;  // Look-ahead
      left->left  = NULL;
      left->right = NULL;

      // Right partial tree
      right->left  = NULL;
      right->right = NULL;

      // Root
      tree->token = tokens;
      tree->left  = left;
      tree->right = right;

      // Concat partial tree
      (trees - 1)->right = tree;
      break;
    case NOTE:
    case REST:
      // Left leaf
      left->token = tokens + 1;  // Look-ahead
      left->left  = NULL;
      left->right = NULL;

      // Right partial tree
      right->left  = NULL;
      right->right = NULL;

      // Root
      tree->token = tokens;
      tree->left  = left;
      tree->right = right;

      // Concat partial tree
      (trees - 1)->right = tree;
      break;
    case EOM:
      // Root and Right leaf
      tree->token = tokens;
      tree->left  = NULL;
      tree->right = NULL;

      // Concat partial tree
      (trees - 1)->right = tree;

      *trees = *tree;
      return;
    default:
      tree->token = tokens;
      tree->left  = NULL;
      tree->right = NULL;

      *trees = *tree;
      return;
  }

  *trees++ = *tree;

  tokens += 2;

  tree_construct(tokens, trees);
}

void tree_destruct(Tree *trees) {
 if (trees->right == NULL) {
   // Bottom up pointer
   --trees;

   free(trees->left);
   free(trees->right);

   if (trees->token->type == TEMPO) {
     free(trees);
   }

   return;
 }

 // Increment until pointing the right leaf by recursive call
 tree_destruct(++trees);
}

void tree_print(Tree *trees) {
  int indent = 0;

  while (trees->right != NULL) {
    for (int i = 0; i < indent; i++) {
      fputs("    ", stdout);
    }

    fprintf(stdout, "   %-2s\n", trees->token->token);

    for (int i = 0; i < indent; i++) {
      fputs("    ", stdout);
    }

    fputs("  / \\\n", stdout);

    for (int i = 0; i < indent; i++) {
      fputs("    ", stdout);
    }

    if ((trees + 1)->right == NULL) {
      fprintf(stdout, "%s", trees->left->token->token);
    } else {
      fprintf(stdout, "%s\n", trees->left->token->token);
    }

    ++trees;
    ++indent;
  }

  fputs("  EOM\n", stdout);
}
```

[MML パーサ ソースコード](https://github.com/Korilakkuma/MML)

# リファレンス

- [例解図説 オートマトンと形式言語入門](https://www.amazon.co.jp/%E4%BE%8B%E8%A7%A3%E5%9B%B3%E8%AA%AC-%E3%82%AA%E3%83%BC%E3%83%88%E3%83%9E%E3%83%88%E3%83%B3%E3%81%A8%E5%BD%A2%E5%BC%8F%E8%A8%80%E8%AA%9E%E5%85%A5%E9%96%80-%E5%B2%A1%E7%95%99-%E5%89%9B/dp/4627852711)
- [明快入門 コンパイラ・インタプリタ開発　C処理系を作りながら学ぶ](https://www.amazon.co.jp/%E6%98%8E%E5%BF%AB%E5%85%A5%E9%96%80-%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%A9%E3%83%BB%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%97%E3%83%AA%E3%82%BF%E9%96%8B%E7%99%BA-C%E5%87%A6%E7%90%86%E7%B3%BB%E3%82%92%E4%BD%9C%E3%82%8A%E3%81%AA%E3%81%8C%E3%82%89%E5%AD%A6%E3%81%B6-%E6%9E%97%E6%99%B4%E6%AF%94%E5%8F%A4%E5%AE%9F%E7%94%A8%E3%83%9E%E3%82%B9%E3%82%BF%E3%83%BC%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA-%E6%9E%97-%E6%99%B4%E6%AF%94%E5%8F%A4-ebook/dp/B00V7Y9MIO)
- [WHATWG Living StandardとHTMLパーサ](https://qiita.com/namusyaka/items/f10ce88e6f8bf322b0ff)
- [XSound MML](https://github.com/Korilakkuma/XSound/tree/main/src/MML)
