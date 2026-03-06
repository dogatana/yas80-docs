# yas80 - Yet Another aSemmbler for Z80 and R800

Version. 0.1.0

## はじめに
## マルチパスアセンブラ

yas80 では前方参照、マクロ、条件アセンブルの評価のため

- 字句解析
- 構文解析
- 評価（意味解析）
  - 1st stage - 未定義シンボルの解決。最大256回
  - 2nd stage - 生成コードの安定度評価。最大16回
- コード出力

の流れで実行するマルチパスアセンブルとしている。
現在評価中のパスはシステム変数 `$PASS` で参照可能。

## シンボルとスコープ

yas80 は次のスコープを持つ

| シンボル | スコープ | 説明 |
| - | - | - |
| 英字, `_` 始まり | グローバル | どのスコープでも定義、参照可能 |
| `.` 始まり | PROC ローカル | シンボルを含む PROC 内ローカル<br>PROC 外部からは `PROC-name.symbol-name` で参照可能
| `@` 始まり | MACRO/REPT ローカル| シンボルを含む MACRO/REPT 内ローカル<br>外部からは参照不可<br>MACRO 呼出しがネストしている場合は別スコープ扱い<br>REPT の場合は展開の都度、別スコープ扱い|

- MACRO/REPT ローカルはアセンブル時、他の名前にマングリングすることで実現

## ORG による生成コードの配置指定

- ORG から次の ORG までを "セグメント" としてまとめる
- セグメントは ORG の第1オペランド（アドレス）でソートして配置
- ORG の第2オペランドが REL の場合、該当セグメントは先行セグメントの従属データとして続けて配置

## 実行

### コマンド書式

```
> yas80 -h
Usage: yas80 [options] file [file...]
Options:
  -I, --I strings       directories to search for iclude
  -D, --D strings       define constants
  -o, --output string   output file name
  -f, --fill int        filler for DS and Segment Gap (default 255)
      --mzt             output with MZT format
      --t88             output with T88 format
  -l, --l               generate list file
      --list string     list filename
  -m, --m               generate map file
      --map string      map filename
  -s, --s               generate symbol file
      --sym string      symbol filename
      --auto-proc       generate PROC from normal label
  -R, --R800            assmble for R800
  -v, --version         print version
      --stdin           assemble stdin
      --astdebug int    debug level for Parse
      --yydebug int     debug level for go-yacc
      --evaldebug int   debug level for Evaluator
      --listdebug int   debug level for Lister
  -a, --arg             assemble Args[0]
```

### オプションとその意味

### ソースファイル

- ソースファイルは複数指定可能
- 複数指定した場合、それらを結合した1つのファイルとしてアセンブルする

## ソースファイル

### エンコーディング

- エンコーディングは utf-8（BOM の有無は問わない）
- utf-8 として不正な文字を検出した場合、ShiftJISとして扱い、utf-8 へのエンコディーング変換を試みる。
- 変換が失敗した場合、その旨表示するが、アセンブルは継続する

### 行

- ソースファイルは行単位でアセンブル処理を進める
- 物理行（行頭から改行記号までのテキスト）と論理行（アセンブラが1行として処理するもの）の対応
    - 1:1 
    - 1:n マルチステートメント
    - n:1 継続行


### マルチステートメント

- `\` でマルチステートメント記載可能
- エラー表示等は物理行単位で行う

例）
`ld a,(hl) \ inc hl` は

```
ld a,(hl)
inc hl
```
と同等。


### 継続行

- 物理行の行末が `\` の場合（`\` の直後が改行） の場合、次行と合わせて1行として扱う

例）
```
ld \
a \
, \
b \
```
は `ld a, b` と同等

### 行の種類

- 行は次のいずれか
    - 空白行
    - コメント行（コメントのみ含む）
    - Z80 / R800 命令行
    - 疑似命令（アセンブラディレクティブ）行


## ディレクティブ

### 定数定義文 CONST/EQU

- CONST `<name>` = `<expr>`
- `<name>` EQU `<expr>`

### 変数定義文 VAR

- 定義：VAR `<name>` = `<expr>`
- 更新（代入） VAR = `<expr>`

### IF 文

- IF `<expr>` \ `<statements>` \ ENDIF
- IF `<expr>` \ `<statements>` \ ELSE \ `<statements>` \ ENDIF
- IF `<expr>` \ `<statements>` \ ELIF `<expr>`ELSE \ `<statements>` \ ENDIF
- IF 文のネストは可能
- ELIF は IF ELSE IF ELSE ENDIF ENDIF のシンタックスシュガー

### REPT 文

- REPT `<expr>` \ `<statements>` \ ENDR
- `<expr>` の数だけ `<statements>` を展開する
- ＠名前 は各展開毎のローカル名となる
- `<statements>` の中で EXIT を使用すると、そこから ENDR までの展開をやめる
-

### MACRO 文

- 定義： `<name>` MACRO [param1 [, param2...]] \ `<statements>` \ ENDM
- トップレベルでのみ定義可能
- 呼出： `<name>` [arg1 [, arg2...]]
- 0 個以上の引数をとれる
- ＠名前 は各展開毎のローカル名となる
- `<statements>` の中で EXIT を使用すると、そこから ENDR までの展開をやめる
- トップレベルのみで定義可能
- `<statements>` に次のものを含むことはできないできない（定義のネスト不可）
    - MACRO
    - PROC
- `<statements>` の中で他のマクロを呼び出すことは可能
- `<statements>` の中で自分自身を呼び出すことは不可（再帰呼び出しは不可）

### PROC 文

- `<name>` PROC \ `<statements>` \ ENDP
- トップレベルでのみ定義可能
- `<name>` の通常ラベルが定義される
- .ローカル名のスコープ管理単位
- .ローカル名は PROC 内で一意であること（通常ラベルとの位置関係は問わない）
- `<statements>` に次のものを含むことはできないできない（定義のネスト不可）
    - PROC
    - MACRO
- 定義のネストは不可

### FUNC 文

- 定義：`<name>` FUNC [param1 [, param2...]] \ `<statements>` \ ENDF
- 呼出: `<name>` ( [arg1 [, arg2...]])
- 関数呼出しは`<expr>`
- 引数がない場合でも () は必要
- `<statements>` の中で RETURN 文が評価されると、その時点で呼出し元へ戻る
- RETURN 文に値が指定されていると、その値が呼出し元に返される
- `<statements>` 内に RETURN 文がない場合、全ての文を実行後、呼び出し元に戻る。値は返さない
- `<name>` は関数値を持つため、他の定数・変数に代入したり、関数を値として返すことも可能
- 関数呼出しはクロージャを形成する

```
フィボナッチ関数の例
フィボナッチ数列生成関数の例
```

### FUNCTION 文

```
FUNCTION name ([param1 [, param2...]]) expr
```

- 1 行で定義できる関数
- 次の FUNC 文のシンタックスシュガーで

```
name FUNC [param1 [, param2...]]
statement(s)
ENDF
```

### RETURN 文

- 関数の処理を終了し呼出し元に戻る。
- 戻り値なし： RETURN
- 戻り値あり： RETURN `<expr>`


### CHARMAP 文

- 定義
```
CHARMAP name, filename | json-string [, option ]
```

- 適用

```
  name("文字列")
```

| 項目 | 説明 |
| -- | -- |
| name | CHARMAP 名。この名前を関数として、文字列に適用する |
| filename | CHARMAP を定義した JSON ファイル名。アセンブル対象ファイルと同じフォルダか、-I オプションで指定したフォルダからロードする |
| json-string | 文字列の先頭が '{' の場合、ファイル名でなく、JSON 文字列と解釈して CHARMAP を定義する |
| option | $CMAP_ERR - CHARMAP を適用する文字列の文字が、CHARMAP で定義されていない場合、エラーとする<br>$CMAP_THRU - CHARMAP で未定義の文字の文字を、元の文字のまま出力する<br>数値(0-65535) - CHARMAPで未定義の文字を、この数値で指定された文字コードに置き換える。文字コードが 256 以上の場合、上位バイト、下位バイトにの順に出力する |



## 対応 CPU
