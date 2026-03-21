# 構文

## コメント

- `";"` 以降行末まではコメントです。
- 改行記号と同じ扱いとなります。


## 数値リテラル

###  10 進数

-  `0-9` を 1 文字以上続けたもの

###  16 進数

- `0x` もしくは `0X` の後に `0-9a-fA-F` を 1 文字以上続けたもの
- `$` の後に `0-9a-fA-F` を 1 文字以上続けたもの
- `0-9` の後に `0-9a-fA-F` を 1 文字以上続け、末尾に `h` もしくは `H` を付けたもの

### 8 進数

- `0o` もしくは `0O` の後に `0-7` を 1 文字以上続けたもの

###  2 進数

- `0b` もしくは `0B` の後に `0,1` を 1 文字以上続けたもの
- `%` の後に `0,1`  を 1 文字以上続けたもの

### _ 区切り

- 数値リテラルに `"_"` を含むことができます。
- ただしプレフィックス(`$`, `%`)の直後には含められません。

```
num.asm:
    1       04d2(1234)                          const d1 = 1234
    2       04d2(1234)                          const d2 = 1_234
    3       1234(4660)                          const h1 = $1234
    4       1234(4660)                          const h2 = 0x12_34
    5       1234(4660)                          const h3 = 1234h
    6       00ff(255)                           const o1 = 0o377
    7       000f(15)                            const b1 = 0b1111
    8       00aa(170)                           const b2 = %1010_1010
```

## 文字列リテラル

- 二重引用符(`"`) もしくは引用符(`'`)で囲まれた文字列です。
- `"` で囲む場合、文字列中に `"` を含めるにはエスケープします(`\"`)。`'` で囲む場合は不要です。
- `'` で囲む場合、文字列中に `'` を含めるにはエスケープします(`\'`)。`"` で囲む場合は不要です。
- 利用可能なエスケープ文字は次表のとおりです。
- 次表に記載以外の文字をエスケープした場合、`\` を含めそのまま文字列リテラルとします。
- 日本語を記載した場合、Shift_JIS コードの文字列リテラルとします。

| エスケープ表記 | 文字コード |
| :--: |:-: |
| `\"` | 0x22 |
| `\'` | 0x60 |
| `\\` | 0x5c |
| `\a` | 0x07 |
| `\b` | 0x08 |
| `\t` | 0x09 |
| `\v` | 0x0b |
| `\f` | 0x0c |
| `\n` | 0x0a |
| `\r` | 0x0d |
    
<br>
```
str.asm
    1  0000 6d 65 73 73 61 67 65     [  ]       db "message"
    2  0007 22                       [  ]       db '"'
    3  0008 27                       [  ]       db "'"
    4  0009 22 27 5c 07 08 09 0b 0c  [  ]       db "\"'\\\a\b\t\v\f\n\r"
       0011 0a 0d                    [  ]
    5  0013 82 a0 82 a2 82 a4        [  ]       db "あいう"
    6  0019 5c 78 5c 79 5c 7a        [  ]       db "\x\y\z"
```

## 文字列リテラルの結合

- 連続する文字リテラルは結合され、1 つの 文字リテラルとして扱います。

```
; db "hello, world!" 
db "hello, " "world" "!" 

; charmap cmap, '{"あ":[65]}'
charmap cmap, '{' \      
  '"あ":[65]' \
  '}'                    
db cmap("あ")            ;
```

## 真偽値リテラル

- 特別な真偽値リテラルはなく、[IF/ELIF](/directive/directive.md#ifelifelseendif) 等で論理値が必要な場合は次のように真偽を判定します。
- 論理演算は真の場合 1 を、偽の場合 0 を返します。

| 真偽値 | リテラル |
| :-:    | --       |
| 偽     | 数値 `0`、空文字列 `""`、空の配列リテラル `[]` |
| 真     | 偽以外のリテラル |

##  配列リテラル

- `[`, `]` で式を囲んだもの
- 配列リテラルに `[数値式]` を付与することで対応する位置の配列要素を参照することができます。
- 配列リテラルの要素は参照のみ可能で値を更新することはできません。
- [`MACRO`](/directive/directive.md#macroendm)呼出し、[`REPT`](/directive/directive.md#reptendr)展開の引数に指定することもできます。


```
array.asm:
    1                                           const msgs = ["one", "two", "three"]
    2
    3                                           rept msgs
    4                                             db $v
    5                                           endr
    5                                         + $COUNT = 3(0x3)
    5                                         + $I = 0(0x0)
    5                                         + $V = "one"
    5  0000 6f 6e 65                 [  ]     +   db $v
    5                                         + $COUNT = 3(0x3)
    5                                         + $I = 1(0x1)
    5                                         + $V = "two"
    5  0003 74 77 6f                 [  ]     +   db $v
    5                                         + $COUNT = 3(0x3)
    5                                         + $I = 2(0x2)
    5                                         + $V = "three"
    5  0006 74 68 72 65 65           [  ]     +   db $v
    6
    7  000b 74 77 6f                 [  ]       message db msgs[1]  ; "two"
```

## 演算子

- 数値演算子
    - 四則演算 `+ - * /`
    - 剰余演算 `%`
    - ビット演算 `& | ^ ~ << >>`
- 比較演算子 `== != < <= => >`
- 論理演算子 `|| && !（論理否定）`
- 文字列結合 `+`
- シンボル結合演算子 `##`

### 演算子の優先順位

<style> 
    #prec {
        width:35em;
    }
    #prec thead tr th:first-child { width:6em; }
    #prec thead tr th:last-child { width:5em; }
    #prec tbody tr td:first-child ,
    #prec tbody tr td:last-child {
     text-align: center; 
    }
</style>

<table id="prec">
<thead>
<tr><th>優先順位</th><th>演算子</th><th>結合</th></tr>
</thead>
<tbody>
<tr><td>高</td><td><code>関数呼出し()   配列参照[]   </code></td><td>なし</td></tr>
<tr><td>  </td><td><code>単項- ~ !              </code></td><td>右</td></tr>
<tr><td>  </td><td><code>* / % & &lt;&lt; &gt;&gt;    </code></td><td>左</td></tr>
<tr><td>  </td><td><code>+ - | ^                      </code></td><td>左</td></tr>
<tr><td>  </td><td><code>== != &lt; &lt;= &gt;= &gt;</code></td><td>左</td></tr>
<tr><td>  </td><td><code>&&                         </code></td><td>左</td></tr>
<tr><td>低</td><td><code>||                         </code></td><td >左</td></tr>
</tbody>
</table>

### シンボル結合演算子

- シンボルと数値もしくは文字列を結合し、別のユーザ定義名とするものです。
- 結合対象のシンボルは、既定義、未定義のどちらでも利用できます。

__concat.lst__
```
concat.asm
    1  0000 01                       [  ]       data ## 1               db 1
    2  0001 02                       [  ]       data ## "_2"            db 2
    3  0002 03                       [  ]       data ## $fmt("%03d", 3) db 3
    4
```

__concat.sym__
```
0000 DATA1
0001 DATA_2
0002 DATA_003
```


## 組み込み関数

yas80 の組み込み関数を次表に示します。
関数名が 2 つ記載されているものはエイリアスで、違いはありません。


| 関数名   | 動作内容 |
| --       |  -- |
|`$W`<br>`$WORD` | 引数（数値）を WORD（2 バイト整数）としてマークします。[`DD`](/directive/directive.html#dbdwdd) で使用します。|
|`$H`<br>`$HI`   | 引数（数値）の上位バイトを返します。 |
|`$L`<br>`$LO`   | 引数（数値）の下位バイトを返します。 |
|`$LEN`      | 引数が文字列の場合は文字数を、配列リテラルの場合は要素数を返します。|
|`$REV`      | 引数（配列）の並びを逆転した配列を新たに作くり返します。引数の配列は元のまま変化しません。|
|`$FMT` `$FORMAT` | 最初の引数を書式文字列、残りの引数（数値もしくは文字列）を書式文字列に従って文字列化した文字列を返します。書式指定文字は Go 言語の fmt パッケージの [Printing](https://pkg.go.dev/fmt@go1.26.1#hdr-Printing) を参照ください（C 言語等とほぼ同じです）。|
|`$ISARY` `$ISARRAY` | 引数が配列の場合 1 を、配列でない場合 0 を返します。|
|`$CHR`   | 引数（配列）が 1 要素の場合はその値を、2 要素の場合は最初の値を上位バイト、次の値を下位バイトとした数値を返します。1 文字の文字列リテラルに[`CHARMAP`](/directive/directive.md#charmap) を適用した場合、結果が配列になるため、それを「コード」としたい場合に使用します。|
|`$DEFINED` | 引数（シンボル）が現在のパスでこの関数の呼出し時点で値が確定してれば 1 をそうでない場合 0 を返します。[`IF`](/directive/directive.md#ifelifelseendif)での使用を想定したものですが、[マルチパスアセンブラ](/README.md#マルチパスアセンブラ)のため利用機会があるかどうか。。 |


### 組み込み関数使用例

__sample.lst__
```
sample.asm:
    1  0000 01 01 00   [  ]    dd 1, $w(1)                     ; db 1 \ dw 1
    2  0003 30 30 31   [  ]    db $fmt("%03d", 1)              ; db "001"
    3  0006 00         [ 4]    data ## $fmt("%03d", 12): nop   ; data012: nop
```

__sample.sym__
```
0006 DATA012
```


## シンボル{#symbol}

yas80 のシンボルには次の種類があります。
構文解析の工程で全て大文字に変換するため、大文字、小文字の区別はありません。

命令、疑似命令以外のシンボルは「値」として定数、変数、引数、式などで使用可能です。

<table>
<thead>
<tr><th colspan="2">種別</th><th>説明</th></tr>
</thead>
<tbody>
<tr><td rowspan="3">予約語</td><td>命令</td><td>Z80/R800 の命令</td></tr>
<tr><td>レジスタ</td><td>Z80/R800 のレジスタ</td></tr>
<tr><td>フラグ</td><td>Z80/R800 のフラグ （※）</td></tr>
<tr><td colspan="2">疑似命令</td><td>命令</td></tr>
<tr><td colspan="2">ラベル</td><td>命令もしくは疑似命令のアドレス</td></tr>
<tr><td colspan="2">定数</td><td><a href="/directive/directive.html#constequ"><code>CONST/EQU</code></a>で定義</td></tr>
<tr><td rowspan="2">変数</td><td>ユーザ定義</td><td><a href="/directive/directive.html#var"><code>VAR</code></a>で定義</td></tr>
<tr><td><a href="/syntax/syntax.html#システム変数">システム変数</a></td><td><code>$</code>で始まるシステム定義の変数で、参照のみ可能</td></tr>
<tr><td colspan="2"><a href="/syntax/syntax.html#組み込み関数">組み込み関数</a></td><td><code>$</code>で始まるシステム定義の関数</td></tr>
<tr><td colspan="2"><code>PROC</code> 名</td><td><a href="/directive/directive.html#proc"><code>PROC</code></a> の名前。ラベルと同じ扱い</td></tr>
<tr><td colspan="2"><code>ENUM</code> 名、<code>ENUM</code> 要素名</td><td><a href="/directive/directive.html#enum"><code>ENUM</code></a>で定義</td></tr>
<tr><td colspan="2">関数名</td><td><a href="/directive/directive.html#funcendfunc"><code>FUNC</code></a>、<a href="/directive/directive.html#function"><code>FUNCION</code></a>で定義したユーザ定義関数</td></tr>
<tr><td colspan="2"><code>MACRO</code> 名</td><td><a href="/directive/directive.html#macro"><code>MACRO</code></a>で定義</td></tr>
</tbody>
</table>

### キャリーフラグについて

シンボル `C` はレジスタです。
条件ジャンプ命令などでキャリーフラグを指定し、表記は同じ `C` となりますが、
命令の評価の際にキャリーフラグに置き換えて解釈しています。

yas80 では `CY` をキャリーフラグリテラルとして定義しているため `CY` を使用することも可能です。

```
carry.asm:
    1  0000 d8   [11]   ret c  ; C はレジスタだが CY に置き換えて評価
    2  0001 d8   [11]   ret cy ; CY を指定することも可
```

## ラベル

## システム変数

yas80 では次のシステム変数を定義しています。
システム変数は参照のみ可能です。書込みはできません。

| 変数名   | 型   | 内容 |
| --       | --   | --   |
| `$`      | 数値 | 現在のロケーションカウンタ（アドレス）|
| `$R800`  | 数値 | `-R800` オプションを指定したとき 1、指定しないとき 0 |
| `$PASS`  | 数値 | アセンブラのパス（1 ～）|
| `$RSIZE` | 数値 | `INCBIN` で読み込んだバイトサイズ。エラーの場合は -1  |
| `$CMAP_ERR`  | 数値 | `CHARAMAP` を適用する際、未定義の文字であればエラーにする (-1)|
| `$CMAP_THRU` | 数値 | `CHARAMAP` を適用する際、未定義の文字で適用前の文字とする (-2)|
| `$COUNT` | 数値 | `REPT 展開回数` の場合はその展開回数を、`REPT 配列` の場合は配列要素数 |
| `$I`     | 数値 | `REPT` の展開毎の順序数（0 から `$COUNT - 1` まで変化）|
| `$V`     | 値   | `REPT 配列` の展開毎の値（配列要素の値）|
| `_`（下線）  | -    | 代入文の左辺値として使用できるブランク識別子（Go, Python と同様） |

## ユーザ定義名

| カテゴリ | 内容 |
| -- | -- |
| グローバル | 英字もしくは `_` で始まり、英数字もしくは `_` が後に続くもの |
| PROC ローカル| 通常名の前に `.` を付けたもの。スコープは PROC 内 |
| MACRO ローカル| 通常名の前に `@` を付けたもの。スコープは マクロ内 |

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

