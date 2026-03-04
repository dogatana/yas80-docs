# yas80 - Yet Another aSemmbler for z80 and r800

Version. 0.1.0

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

### コメント

- `;` 以降行末までコメントで、アセンブル処理の対象外となる


## 構文要素

### 予約語

- 次のもの予約語のため、ユーザ定義名として利用できない

| カテゴリ | 内容 |
| -- | -- |
| Z80  |命令語、レジスタ名、フラグ名  | 
| ディレクティブ（疑似命令） | IF ELIF ELSE ENDIF<br> MACRO ENDM EXIT<br>REPT ENDR<br>FUNC ENDF FUNCTION<br>CONST EQU VAR |
| システム変数 | _ $ 等|
| 組み込み関数 | $LEN $PUSH $POP $ERROR $WARN $INFO $PRINT $PRINTF |

- システム変数、組み込み関数は全て `$` で始まる
- システム変数は参照のみ可能。
- ユーザ定義名として `$` で始まる名前を定義することはできない。
- `_` はブランク識別子（Go と同じ用途）

### ユーザ定義名

| カテゴリ | 内容 |
| -- | -- |
| グローバル | 英字もしくは `_` で始まり、英数字もしくは `_` が後に続くもの |
| PROC ローカル| 通常名の前に `.` を付けたもの。スコープは PROC 内 |
| MACRO ローカル| 通常名の前に `@` を付けたもの。スコープは マクロ内 |

### リテラル

- 数値リテラル 
    - 10 進数
        -  0-9 を 1 文字以上続けたもの
    - 16 進数
        - 0x もしくは 0X の後に 0-9 a-f A-F を1文字以上続けたもの
        - '$` の後に 0-9 a-f A-F を1文字以上続けたもの
        - 0-9 の後に0-9 a-f A-F を1文字以上続け、末尾に h もしくは H を付けたもの
    - 8 進数
        - 0o もしくは 0O の後に 0-7  を1文字以上続けたもの
    - 2 進数
        - 0b もしくは 0B の後に 0, 1  を1文字以上続けたもの
        - '%' の後に 0, 1  を1文字以上続けたもの
    - いずれの場合も任意の位置に `_` を含むことができる
- 文字列リテラル
    - 二重引用符(`"`) もしくは引用符(`'`)で囲まれた文字列
    - `"` で囲む場合、文字列中に `"` を含めるにはエスケープする(`\"`)。`'` で囲む場合は不要
    - `'` で囲む場合、文字列中に `'` を含めるにはエスケープする(`\'`)。`"` で囲む場合は不要
    - 利用可能なエスケープ文字は次のとおり。これ以外の文字をエスケープした場合、`\` が無視され文字がそのまま出力される

| 表記 | 文字コード |
| :--: | - |
| `\"` | 0x22 |
| `\'` | 0x22 |
| `\\` | 0x5c |
| `\a` | 0x07 |
| `\b` | 0x08 |
| `\t` | 0x09 |
| `\v` | 0x0b |
| `\f` | 0x0c |
| `\n` | 0x0a |
| `\r` | 0x0d |
    

- 真偽値リテラル
    - 特別な真偽値リテラルはなく、次の通り真偽判定する

| 真偽値 | リテラル |
| :-: | -- |
| 偽 | 数値 `0`、空文字列 `""`、空の配列リテラル `[]`
| 真 | 偽以外のリテラル数値、文字列。レジスタ、関数(FUNC)、列挙(ENUM) |

- 配列リテラル
    - `[`, `]` で他の式を囲んだもの
    - 配列リテラルに `[数値式]` を付与することで対応する位置の配列要素を参照することができる
    - 配列リテラルの要素は参照のみ可能で値を更新することはできない。

### 演算子

- 数値演算子
    - 四則演算 `+ - * /`
    - 剰余演算 `%`
    - ビット演算 `| & ^ ~ << >>`
    - 比較演算 `< <= == != > >>`
    - 論理演算 `|| && !（論理否定）`
- 文字列演算子
    - 結合 `+`
- トークン結合演算子
    - ユーザ定義名と数値・文字列を結合し、別のユーザ定義名とするもの
    - ラベルもしくは定数の作成で利用可能

## Z80 命令文

- <opcode> [opran1 [,operand2]]
- Z80 命令文はラベルを持つことが可能

## ユーザ定義名（MACRO 名、FUNC 名、FUNCTION名、ラベル、定数、変数）

- 通常
- .ローカル
    - PROC 内スコープ
    - PROC 外からは 通常.ローカルで参照可能
- @ローカル
    - MACRO、REPT（繰り返し単位）内スコープ
    - MACRO、REPT外からは参照不可

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

### Z80 命令・実行時間

- [Z80 Family CPU User Manual](https://www.zilog.com/docs/z80/z80cpu_um.pdf) に記載の命令・命令実行時間
    - ただし `IN Flag,(C)` を除く
- Z80 非公開命令のうち、`IXH/IXL/IYH/IYL` に関するもの
    - `LD/ADD/ADC/SUB/SBC/AND/AND/OR/XOR/CP/INC/DEC`
    - 命令実行時間は [The Undocumented Z80 Documented](http://www.z80.info/zip/z80-documented.pdf) による

### R800 命令・実行時間

- [R800ユーザーズマニュアル 暫定版](https://d4.princess.ne.jp/msx/datas/R800UM/) に記載の命令・命令実行時間
    - 表記は zilog 表記
    - 乗算命令は `mulub`, `muluw` とも `MUL` を使用する

### 表記の揺らぎ対応

- 本来の書式に加え、次の書式を許容する
- IX の書式は IY も同様

| 本来の書式 | 許容する書式 |
| -- | -- |
| `JP (IX)` | `JP (IX + 0)` |
| `LD (IX + 0), op` | `LD (IX + 0), op` |
| `LD op, (IX + 0)` | `LD op, (IX + 0)` |
| `BIT n, (IX + 0)` | `BIT n, (IX)` |
| `SET n, (IX + 0)` | `SET n, (IX)` |
| `RES n, (IX + 0)` | `RES n, (IX)` |
| `EX HL, DE` | `EX DE, HL` |
| `EX AF, AF'` | `EX AF, AF'` |
| `EX (SP), HL` | `EX HL, (SP)` |
| `EX (SP), IX` | `EX IX, (SP)` |
| `SUB op`| `SUB A, op`   |
| `AND op`| `AND A, op`  |
| `OR op` | `OR A, op`   |
| `XOR op`| `XOR A, op`  |
| `CP op` | `CP A, op`   |

