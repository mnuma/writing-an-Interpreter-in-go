# writing-an-interpreter-in-go

## はじめに

- インタプリタは魔法のようだ
- インタプリタがどのように動作するのかを理解する
- 900ページにも及ぶコンパイラについて書籍と、50行のRubyコードでLispインタプリタを実装する方法に関するブログ記事との間にあるような
- インタプリタをゼロから書いていく
- インタプリタとコンパイラ
- JITインタプリタ

> JITコンパイラ: (Just-in-time Compiler)
> プログラムの実行時に、あらかじめ用意された（実行環境に依存しない汎用的な）中間コードを、プログラムの実行時点でプロセッサが実行可能な機械語（ネイティブコード）にコンパイルすること。
> JITコンパイラはJava、Microsoft .NETなどで採用されている技術である

- tree-walking型

> ソースコードを構文解析し、抽象構文木(Abstract Syntax Tree; AST)を構築し、この木を評価するインタプリタ

## Monkey プログラミング言語とインタプリタ

- C言語風の構文
- 変数束縛
- 整数と真偽値
- 算術式
- 組み込み関数
- 第一級の高階関数
- クロージャ
- 文字列データ型
- 配列データ型
- ハッシュデータ型

## どうしてGoか?

- 簡潔
- 素晴らしいツール

## 本書の使い方

```
https://interpreterbook.com/waiig_code_1.4.zip
```
```
https://github.com/mmyoji/go-monkey
```

# 1章 字句解析

## 1.1 字句解析

- テキストを解釈する
- 取扱のしやすい他の形式でソースコードを表現する
- ソースコード -> トークン列 -> 抽象構文木
- ソースコードからトークン列への変換を「字句解析」という

- 例:

```
let x = 5 + 5;
```

- 字句解析器から出てくるのは次のようなもの

```
[
  LET,
  IDENTIFIER("x"),
  EQUAL_SIGN,
  INTEGER(5),
  PLUS_SIGN,
  INTEGER(5),
  SEMICOLON
]
```

- ※ホワイトスペースはMonkey言語では意味を持たない
- ※プロダクションで使うような字句解析器では行番号や列番号、ファイル名をトークンに付加する

## 1.2 トークンを定義する

```
let five = 5;
let ten = 10;

let add = fn(x, y) {
  x + y;
};

let result = add(five, ten);
```

- どんな種類のトークンがあるだろうか？
 - 数 5,10
 - 変数名 x, y, add, result
 - let, fn
 - 記号 「(」、「)」、「{」、「}」、「=」、「,」、「;」

- タイプを定義する
 - 識別子 ・・・ 変数名、整数など
 - キーワード ・・・言語の一部である単語

 ### tokenパッケージ

```
 type Token struct {
 	Type    TokenType
 	Literal string
 }

 type TokenType string
 ```

 - Monkey言語におけるトークンタイプの種類は有限
 - 2つ特別なタイプ
   - ILLEGAL・・・ILLEGALはトークンや文字が未知であることを表E
   - EOF ・・・ファイル終端(end of file)

## 1.3 字句解析機(レキサー)

- テストケースを拡張
  - 英字判定 isLetter() ・・・インタプリタの解釈でき る言語に広範な影響をもたらす
  - let、fn、foobar readIdentifier() ・・・ トークンリテラル
  - 渡された識別子の判定 LookupIdent()
  - ホワイトスペースの対応 skipWhitespace ・・・空白行読み飛ばし
  - 数字の判定 isDigit() ・・・ あとはisLetter()と同じ
  - readNumber
  - readNumberは整数しか読めない
  - さあ、シャンパンを開けてお祝いをしよう!

## 1.4 トークン集合の拡充と字句解析器の拡張

- トークンの拡張

```
「==」、「!」、「!=」、「-」、「/」、「*」、「<」、「>」、それにキーワード true、false、if、else、 return
```

- 2文字のトークンを認識 「==」、「!=」

> switch文は現在の文字l.chを比較の対象にしているので、単にcase "=="というケースを追加 するわけにはいかない。

- 先読み peekChar()の追加

> ほとんどの字句解析器や構文解析器は、このような peek 関数を持っていて、先読みを行う。大抵の場合は直後の文字を返すだけだ。言語におけるパースの難 易度の違いは、ソースコードを解釈する際に、どの程度先まで読む(もしくは戻って読む!)必要があ るかによるところが大きい。

## 1.5 REPL のはじまり

> REPL は「Read(読み込み)、Eval(評価)、Print(表示)、Loop (繰り返し)」の略
