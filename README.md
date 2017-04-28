# new lang 2017

## new lang 2017 とは

  新しい言語を1年に1回作ってみようと言う企画です。
  仕様をしっかり BNF で書いて操作的意味論を書いて SWI-Prolog [[1]<a name="r1"></a>](#1) で動かし、 OCaml [[2]<a name="r2"></a>](#2) などの言語でも実装してみようというものです。
  プログラミング言語を趣味で作っている人は結構いるのですが、グループで実装することが苦手なのでこんなことやれたらなぁと思い2016年から始めてみたものです。
  まだ軌道に乗っていないので試行錯誤の段階ですが徐々に形にしていければと考えています。
  Shen [[3]<a name="r3"></a>](#3) が思想的に近いかもしれません。

## 今年のお題

  今年のお題は OCaml の不満を修整した言語を作ろうです。
  Twitter で OCaml の文法気持ち悪いとかあまり聞かないとか言う話から、どこが気持ち悪いかなぁみたいな話題があったので、 OCaml をこうしたいというような言語を作ってみようと思います。

## 名前

  とりあえず、 Postocaml にしておきます。

# 構文 grammer

  Postocaml の文法は大きくわけて字句解析パートと構文解析パートに別れます。
  字句解析パートではテキスト文字列をトークンに分割します。
  構文解析パートではトークン列から構文木への構文規則を示します。

# 構文サマリ syntax summary

    p  ::=                                プログラム
         | d (;;? d)*                     宣言列
    d  ::=                                宣言
         | def x1...xn = e                再帰宣言
         | let x1...xn = e                let宣言
         | e                              式
    e  ::=                                式
         | {| x1 -> e1 | ... | xn -> en } 部分関数
         | x                              変数
         | i                              整数
         | e + e                          加算
         | e - e                          減算
         | if e then e else e             条件式
         | e1 e2                          関数適用
         | e2 |> e1                       連結式
         | let x=e1 ; e2                  let 式
         | def x=e1 ; e2                  def 式
         | e1 ; e2                        文
         | ( e )                          括弧
         | {l1=e1;...;ln=en}              レコード
         | [e1;...;en]                    リスト
         | [|e1;...;en|]                  配列

## 構文 syntax

## プログラム program

    p  ::=                                プログラム
         | d (;;? d)*                     宣言列

  プログラムは１つ以上の宣言(Declare)の列です。 宣言と宣言は連続して記述します。
  式と式を連続して記述したい場合等で必要な場合 `;;` を２つの式の間に書きます。

  例

    add 1 2
    ;;
    sub 2 3

## 宣言 declare

  文法

    d  ::=                                宣言
         | def x1...xn = e                再帰宣言
         | let x1...xn = e                let宣言
         | e                              式


### let 宣言 let declare

  構文

    let x = e

  OCaml の `let` 宣言です。

### def 宣言 def declare

  構文

    def x = e

  OCaml の `let rec` 宣言です。

    def add x y = x + y

  は

    def add = {|x->{|y->x+y}}

  のシンタックスシュガーです。

### 式宣言 expression declare

  構文

    e

  例

    printf "test\n"

## 式 expression

  構文

    e  ::=                                式
         | {| x1 -> e1 | ... | xn -> en } 部分関数
         | x                              変数
         | i                              整数
         | e + e                          加算
         | e - e                          減算
         | if e then e else e             条件式
         | e1 e2                          関数適用
         | e2 |> e1                       連結式
         | let x=e1 ; e2                  let 式
         | def x=e1 ; e2                  def 式
         | e1 ; e2                        文
         | ( e )                          括弧
         | {l1=e1;...;ln=en}              レコード
         | [e1;...;en]                    リスト
         | [|e1;...;en|]                  配列

### 部分関数式 partial function expression

  構文

    {| x1 -> e1 |...| xn -> en }

  例

    val x = 1 |> {| 0->0 | x -> x + 1}

    def fib = {
      | 0 -> 0
      | 1 -> 1
      | x -> (fib (x - 2)) + (fib (x - 1))
    }

  OCamlのパターンマッチ構文はネストした場合の不満がありました。
  実のところ `match with` 式は F# 由来の `|>` 演算子を用いれば以下のように記述可能です。

  例えば以下の式は

    match 1 with
    | 0 -> 0
    | x -> x + 1

  次の式に書き換えることが出来ます。

    1 |> function
      | 0 -> 0
      | x -> x + 1

  これはぶら下がりのマッチをする際に `()` 又は `begin end` で囲う必要があります。

    1 |> function
      | x when x < 1 ->
        begin x |> function
          | 0 -> 0
          | 1 -> 1
        end
      | x -> x + 1


  そこで Coq [[4]<a name="r4"></a>](#4) の `match` には `end` を追加されました。
  今度は冗長になり、 Ruby に近い方向性になります。
  一方 Scala [[5]<a name="r5"></a>](#5) の `match` 式は以下のように記述します。

    1 match {
      case 0 => 0
      case x => x + 1
    }

  Scala の `match` 式および部分関数は `case` や `match` のキーワードが冗長です。
  そこで Postocaml では、 OCaml の 配列は `[|` から始まるように `{|` から始まる式を部分関数とすることでパターンマッチ式を記述しました。

### 文式 statement expression

  文は式と式を2項演算子 `;` を用いて結合した式です。 文は文であると同時に式であるので式として使うことが出来ます。

    e ; e

# 参照

[[1]<a name="1"></a>](#r1) SWI-Prolog

[[2]<a name="2"></a>](#r2) OCaml

[[3]<a name="3"></a>](#r3) Shen

[[4]<a name="4"></a>](#r4) Coq

[[5]<a name="5"></a>](#r5) Scala
