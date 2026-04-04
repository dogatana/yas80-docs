# 文法

## 文

- プログラムは文の並びです。
- 文とソースファイルの行は通常 1:1 に対応しますが、中には複数の行をまとめて 1 つの文として扱うものもあります。
- 文は次のいずれかです。
    - 命令文 ･･･ Z80/R800 の命令
    - 疑似命令文 ･･･ 定数定義、`IF など yas80 が用意している命令
    - 空行（コメントのみの行も含む）

## コメント

- `";"` 以降行末まではコメントです。
- 改行記号と同じ扱いです。

## 式

- 文は式を含むことがあります。
- 式は各種リテラル、シンボルなどを必要により演算子で結合したもので評価すると"値"になります。
- yas80 では数値、文字列に他にレジスタ、フラグ、関数および配列も値であり、定数、変数、引数などに利用可能です。

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

- 二重引用符(`"`) もしくは一重引用符(`'`)で囲まれた文字列です。
- `"` で囲む場合、文字列中に `"` を含めるにはエスケープします(`\"`)。`'` で囲む場合はエスケープ不要です。
- `'` で囲む場合、文字列中に `'` を含めるにはエスケープします(`\'`)。`"` で囲む場合はエスケープ不要です。
- 利用可能なエスケープ文字は次表のとおりです。
- 次表に記載以外の文字をエスケープした場合、`\` を含めそのまま文字列リテラルとします。
- 日本語を記載した場合、Shift_JIS コードの文字列リテラルとします。

<table style="width:18em">
<thead>
<tr><th>エスケープ表記</th><th>文字コード</th></tr>
</thead>
<tbody>
<tr><td class="center" style="width:10em"><code> \" </code></td><td class="center"><code> 0x22 </code></td></tr>
<tr><td class="center" style="width:10em"><code> \' </code></td><td class="center"><code> 0x60 </code></td></tr>
<tr><td class="center" style="width:10em"><code> \\ </code></td><td class="center"><code> 0x5c </code></td></tr>
<tr><td class="center" style="width:10em"><code> \a </code></td><td class="center"><code> 0x07 </code></td></tr>
<tr><td class="center" style="width:10em"><code> \b </code></td><td class="center"><code> 0x08 </code></td></tr>
<tr><td class="center" style="width:10em"><code> \t </code></td><td class="center"><code> 0x09 </code></td></tr>
<tr><td class="center" style="width:10em"><code> \v </code></td><td class="center"><code> 0x0b </code></td></tr>
<tr><td class="center" style="width:10em"><code> \f </code></td><td class="center"><code> 0x0c </code></td></tr>
<tr><td class="center" style="width:10em"><code> \n </code></td><td class="center"><code> 0x0a </code></td></tr>
<tr><td class="center" style="width:10em"><code> \r </code></td><td class="center"><code> 0x0d </code></td></tr>
</tbody>
</table>

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
- [継続行](/source/source.md#継続行)を使用し、複数行に渡る文字列リテラルを定義することも可能です。

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

- 真偽値リテラルはありません。
- [IF/ELIF](/directive/directive.md#ifelifelseendif) 等で論理値が必要な場合は次のように真偽を判定します。
- 比較演算、論理演算の結果は真の場合 1、偽の場合 0 となります。

| 真偽値 | リテラル |
| :-:    | --       |
| 偽     | 数値 `0`、空文字列 `""`、空の配列リテラル `[]` |
| 真     | 偽以外の値 |


## レジスタ・フラグリテラル

| リテラル | 表記 |
| -- | -- |
| レジスタ| `A, B, C, D, E, H, L, IXH, IXL, IYH, IYL, I, R, F`|
| フラグ  | `NZ, Z, NC, CY, PO, PE, P, M` |

### キャリーフラグについて

- yas80 では `C` をレジスタリテラル、`CY` をフラグリテラルとして定義しています。<br>
- 条件ジャンプ命令など、フラグが必要な場合は `C` を `CY` に置き換えて評価するため、どちらも利用可能です。

```
carry.asm:
    1  0000 d8   [11]   ret c  ; C はレジスタだが CY に置き換えて評価
    2  0001 d8   [11]   ret cy ; CY を指定することも可
```

- ただし `＝＝, !=` 演算子で同値判定する場合は異なる値となるので、マクロの引数で同値判定する場合などは注意してください。

<table style="width:15em">
<thead>
<tr><th style="width:8kj">式</th><th>値</th><tr>
</thead>
<tbody>
<tr><td class="center"><code>C == CY</code></td><td class="center"><code>0</td></tr>
<tr><td class="center"><code>C != CY</code></td><td class="center"><code>1</td></tr>
</tbody>
</table>


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
    - 剰余演算 `%`&nbsp;&nbsp;__※2 進数リテラルとの誤認識を避けるため`%`の直後は空白文字としてください__
    - ビット演算 `& | ^ ~ << >>`
- 比較演算子 `== != < <= => >`
- 論理演算子 `|| && !（論理否定）`
- 文字列結合 `+`
- シンボル結合演算子 `##`

## 演算子の優先順位

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
<tr><td>高</td><td><code>()</code>関数呼出し <code>[]</code>配列参照</code></td><td>なし</td></tr>
<tr><td>  </td><td><code>-</code>（単項）<code> - ~ !              </code></td><td>右</td></tr>
<tr><td>  </td><td><code>* / % & &lt;&lt; &gt;&gt;    </code></td><td>左</td></tr>
<tr><td>  </td><td><code>+ - | ^                      </code></td><td>左</td></tr>
<tr><td>  </td><td><code>== != &lt; &lt;= &gt;= &gt;</code></td><td>左</td></tr>
<tr><td>  </td><td><code>&&                         </code></td><td>左</td></tr>
<tr><td>  </td><td><code>||                         </code></td><td >左</td></tr>
<tr><td>低</td><td><code>##</code></td><td >左</td></tr>
</tbody>
</table>

## 中置演算子と型混合演算

- '"+"' 等の数値中置演算子は数値と文字列、配列の演算が可能です。
- 文字列は長さ 1 の文字列をShift_JIS 文字コードへ変換した数値となります。
- 文字列の長さが 1 以外の場合はエラーになります。
- 配列は要素が数値で要素数が 1 のときは先頭の要素が、要素数が 2 のときは先頭の要素が下位バイト、次の要素が上位バイトとした数値となります。
- 配列の要素数が 1 でも 2 でもない場合はエラーになります。

```
    1       0000(0)                             var v1 = 0
    2                                           
    3       "a"                                 v1 = 'a'        ; 文字列
    4       0061(97)                            v1 = 'a' + 0    ; 数値
    5       0062(98)                            v1 = 'a' + 1    ; 数値
    6       0062(98)                            v1 = 1 + 'a'    ; 数値
    7       "あ"                                v1 = 'あ'       ; 文字列
    8       82a1(33441)                         v1 = 'あ' + 1   ; 数値
    9       82a1(33441)                         v1 = 1 + 'あ'   ; 数値
   10                                           v1 = [1]        ; 配列
   11       0003(3)                             v1 = [1] + 2    ; 数値
   12       0105(261)                           v1 = [1, 2] + 3 ; 数値
   13       0003(3)                             v1 = 1 + [2]    ; 数値
   14       0204(516)                           v1 = 1 + [2, 3] ; 数値
```

## シンボル結合演算子

- シンボルと数値もしくは文字列を結合し、別のユーザ定義名とするものです。
- 左辺がシンボル、右辺が数値もしくは文字列です。
- シンボル結合演算子は結合しません。
- 結合対象のシンボルは、既定義、未定義のどちらでも利用できます。

__concat.lst__
```
concat.asm
    1  0000 01                       [  ]       data ## 1                db 1
    2  0001 02                       [  ]       data ## "_2"             db 2
    3  0002 03                       [  ]       data ## $fmt("_%03d", 3) db 3
```

__concat.sym__
```
0000 DATA1
0001 DATA_2
0002 DATA_003
```

## 名前

- yas80 で扱う"名前"として英字もしくは `"_"` で始まり、英数字もしくは`"_"`を続けたものと、`"$"`で始まるシステムで定義した名前があります。
- 構文解析の工程で全て大文字に変換するため、大文字、小文字の違いは区別されません。


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
<tr><td colspan="2">定数名</td><td><a href="/directive/directive.html#constequ"><code>CONST/EQU</code></a>で定義</td></tr>
<tr><td rowspan="2">変数名</td><td>ユーザ定義</td><td><a href="/directive/directive.html#var"><code>VAR</code></a>で定義</td></tr>
<tr><td><a href="/syntax/syntax.html#システム変数">システム変数</a></td><td><code>$</code>で始まるシステム定義の変数で、参照のみ可能</td></tr>
<tr><td colspan="2"><a href="/syntax/syntax.html#組み込み関数">組み込み関数</a></td><td><code>$</code>で始まるシステム定義の関数</td></tr>
<tr><td colspan="2"><code>PROC</code> 名</td><td><a href="/directive/directive.html#proc"><code>PROC</code></a> の名前。ラベルと同じ扱い</td></tr>
<tr><td colspan="2"><code>ENUM</code> 名、<code>ENUM</code> 要素名</td><td><a href="/directive/directive.html#enum"><code>ENUM</code></a>で定義</td></tr>
<tr><td colspan="2">関数名</td><td><a href="/directive/directive.html#funcendfunc"><code>FUNC</code></a>、<a href="/directive/directive.html#function"><code>FUNCION</code></a>で定義したユーザ定義関数</td></tr>
<tr><td colspan="2"><code>MACRO</code> 名</td><td><a href="/directive/directive.html#macro"><code>MACRO</code></a>で定義</td></tr>
</tbody>
</table>

- 名前のうち、ラベル、定数名、変数名は"シンボル"とし式で利用可能です。
- `PROC`名、関数名も式で利用可能です。


## ユーザ定義名とスコープ

| カテゴリ | 内容 |
| -- | -- |
| グローバル | 英字もしくは `"_"` で始まり、英数字もしくは `"_"` が後に続くもの |
| `PROC`ローカル| グローバル名の前に `"."` を付けたもの。スコープは`PROC`内 |
| `MACRO`ローカル| グローバル名の前に `"@"` を付けたもの。スコープは マクロ内 |

## ラベルとコロン`:`{#colon}

ラベルに対するコロンの要否についてはアセンブラによって異なっていて、
有無どちらでも許容していますが、yas80 では次の仕様としています。

- ラベル文のラベルにはコロンが必要
- 命令、疑似命令のラベルにはコロンが必要
- `DB/DW/DS/PROC` の名前はラベル扱いであるが、名前なのでコロン不要
- コロンが必要な場合の省略、コロンが不要な場合の追加はエラー
                

```
check   macro val, name
        cp val
        jr nz, @next
        call name
@next:                      ; 必要 マクロローカル
        endm

check1: check 'a', process  ; 必要 マクロ呼出し

        call disp
        jr skip
skip:                       ; 必要 ラベル行
disp:   ret                 ; 必要 命令

all:    rept 3              ; 必要 疑似命令
        nop
        endr


message db "hello"          ; 不要 名前

process proc                ; 不要 名前
        or a
        jr z, .ret
        
.ret:                       ; 必要 PROCローカル
        ret
        endp
```

## システム変数

yas80 では次のシステム変数を定義しています。
システム変数は参照のみ可能です。書込みはできません。

| 変数名       | 型   | 内容 |
| --           | --   | --   |
| `$`          | 数値 | 現在のロケーションカウンタ（アドレス）|
| `$R800`      | 数値 | [`-R --r800`](/exec/option.md#r---r800) オプションを指定したとき 1、指定しないとき 0 |
| `$FILL`      | 数値 | [`-f --fill`](/exec/option.md#f---fill) オプションで指定した値。指定しないとき 255 |
| `$PASS`      | 数値 | アセンブラのパス（1 ～）|
| `$RSIZE`     | 数値 | [`INCBIN`](/directive/directive.md#incbin) で読み込んだバイト数。エラーの場合は -1  |
| `$CMAP_ERR`  | 数値 | [`CHARMAP`](/directive/directive.md#charmap) を適用する際、未定義の文字であればエラーにする (-1)|
| `$CMAP_THRU` | 数値 | [`CHARMAP`](/directive/directive.md#charmap) を適用する際、未定義の文字で適用前の文字とする (-2)|
| `$COUNT`     | 数値 | "[`REPT`](/directive/directive.md#reptendr) 数値（展開回数）" の場合は展開回数。"`REPT` 配列" の場合は配列要素数 |
| `$I`         | 数値 | [`REPT`](/directive/directive.md#reptendr) の展開毎の序数（0 から `$COUNT - 1` まで変化）|
| `$V`         | 値   | "[`REPT`](/directive/directive.md#reptendr) 配列" の展開毎の値（配列要素の値）|
| `_`（下線）  | -    | 代入文の左辺値として使用できるブランク識別子（Go, Python と同様） |


## 組み込み関数

yas80 の組み込み関数を次表に示します。
関数名が 2 つ記載されているものはエイリアスで、違いはありません。


| 関数名   | 動作内容 |
| --       |  -- |
|`$W`<br>`$WORD` | 引数（数値）を WORD（2 バイト整数）としてマークします。[`DD`](/directive/directive.html#dbdwdd) で使用します。|
|`$H`<br>`$HI`   | 引数（数値）の上位バイトを返します。 |
|`$L`<br>`$LO`   | 引数（数値）の下位バイトを返します。 |
|`$LEN`<br>`$LENGTH`  | 引数が文字列の場合は文字数を、配列リテラルの場合は要素数を返します。|
|`$REV`<br>`$REVERSE` | 引数（配列）の並びを逆転した配列を新たに作り返します。<br>引数の配列は元のまま変化しません。|
|`$FMT` `$FORMAT` | 最初の引数を書式文字列、残りの引数（数値もしくは文字列）を書式文字列に従って文字列化した文字列を返します。書式指定文字は Go 言語の fmt パッケージの [Printing](https://pkg.go.dev/fmt@go1.26.1#hdr-Printing) を参照ください（C 言語等とほぼ同じです）。|
|`$ISARY` `$ISARRAY` | 引数が配列の場合 1 を、配列でない場合 0 を返します。|
|`$CHR`   | 引数（配列）が 1 要素の場合はその値を、2 要素の場合は最初の値を上位バイト、次の値を下位バイトとした数値を返します。1 文字の文字列リテラルに[`CHARMAP`](/directive/directive.md#charmap) を適用した場合、結果が配列になるため、それを"コード（数値）"としたい場合に使用します。|
|`$DEFINED` | 引数（シンボル）が現在のパスでこの関数の呼出し時点で値が確定してれば 1 をそうでない場合 0 を返します。[`IF`](/directive/directive.md#ifelifelseendif)と組み合わせて使用する場合は[マルチパスアセンブルと`#DEFINED`](/README.md#defined)の記載内容に注意ください。|


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