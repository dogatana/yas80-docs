# 疑似命令（ディレクティブ）

## ORG

#### 書式
```
ORG address [,ABS|,REL]
```

#### 説明

- 次の命令のベースアドレスを `address` で指定します。
- この`ORG`から次の`ORG, END`もしくはファイル末尾までをセグメントとして出力ファイル内での配置方法を指定します。
   - `ABS`- `ORG`から始まるセグメントを"絶対セグメント"として出力ファイルでの配置を決定します。 
   - `REL`- `ORG`から始まるセグメントを"相対セグメント"として出力ファイルでの配置を決定します。 
- 配置方法については"生成コードの配置"を参照ください。

#### 関連

- [生成コードの配置](/allocate.md)
- [`END`](#end)

## END

#### 書式

```
END [address]
```

#### 説明

- トップレベルにある場合、アセンブルを終了し、以降の命令、以降のファイルはアセンブル対象外とします。
- `address` を指定した場合、MZT 形式の実行開始アドレスとなります。
- 複数の `END address` が指定された場合、最後に指定されたものが有効です。
- `-e --entry`オプションが指定された場合、そちらが有効です。

#### 関連
- [MZT 形式](/output/output.md#mzt-形式)
- [`ENTRY`](#entry)
- [`-e --entry`](/exec/exec.md#e---entry)オプション

## ENTRY

#### 書式

```
ENTRY address
```

#### 説明

- MZT 形式の実行開始アドレスを指定します。
- 複数指定された場合、最後に指定されたものが有効です。
- `-e --entry`オプションが指定された場合、そちらが有効です。
- アセンブルを中止しない点を除き、"`END address`" と同じ機能です。

#### 関連
- [MZT 形式](/output/output.md#mzt-形式)
- [`END`](#end)
- [`-e --entry`](/exec/exec.md#e---entry)オプション

## ALIGN

#### 書式
```
ALIGN size [,fill]
```

#### 説明
- ローケンションカウンタを `size` の倍数となるまで進めます。
- ローケンションカウンタを既に `size` の倍数であれば何も行いません。
- 進める場合、`fill` が指定されている場合、その値でギャップを埋めます。
- 進める場合、`fill` が指定されていない場合、デフォルト(255)もしくは `-f --fill`オプションで指定した値で埋めます。

#### 関連

- [`-f --fill`](/exec/exec.md#f---fill)オプション
- [システム変数](syntax/syntax.md#システム変数)`$FILL`

## CHECK256

#### 書式

```
CHECK256 expression
```

#### 説明

- 現在のローケンションカウンタ(`$`)が`expression`で示す256バイトの境界をまたいでいるかどうかを検査し、またいでいる場合はエラーとします。
- 次のマクロと同等の処理です。

```
check256m macro base
  if $stage2 && $ > ($h(base) +1) * 256
    error $fmt("256 バイト境界をまたいでいる。基準 0x%x, $ 0x%x", base, $)
  endif
endm
```


## PROC

#### 書式

```
name    PROC
.local_label1:
        statements
label:
        stataments
.local_name2:
        statements
        ENDP

        call name
        call label
        call name.local_label1
        call name.local_label2
```

#### 説明

- 複数の文を一つにまとめ、ローカルラベルを定義可能なスコープを形成します。
- トップレベルでのみ定義できます。
- `name`はラベルとして参照可能
- `label` はグローバルラベルで`PROC`外部からそのまま参照可能です。
- `.local_label1` を外部から参照する場合、`name.local_label1` とします。
- `.local_label1` は同一`PROC` 内で一意であることが必要ですが、他の`PROC`内の名前と同じでもかまいません。
- `statements` として次のものを含むことはできません。
    - `PROC`
    - `MACRO`

#### 関連

- [スコープ](/scope.md)
- [ユーザ定義名とスコープ](/syntax/syntax.md#ユーザ定義名とスコープ)
- [`PROC`ローカルラベル](/syntax/syntax.md#proc-ローカルラベル)

## 匿名ラベル{#anon-symbol}

#### 説明

- `PROC`内では匿名ラベルとして`@@, @0-@9`が使用可能です。
- `@@, @0-@9`は定義専用のラベルで`PROC`内で複数定義可能です。
- `@F, @0F-@9F`は参照専用のラベルで、記述位置より前方（下行方向）にある直近の`@@, @0-@9`を参照します。
- `@B, @0B-@9B`も参照専用のラベルで、記述位置より前方（上行方向）にある直近の`@@, @0-@9`を参照します。
- 参照位置は物理行で判定します。参照している行と同一物理行で定義されている匿名ラベルは参照できません。
- `@0, @0F, @0B` は視認性の観点から利用は非推奨です。


```
    1                                           anon proc
    2  0000 c3 06 00                 [10]            jp @f
    3  0003 c3 08 00                 [10]            jp @1f
    4
    5  0006 3e 00                    [ 7]       @@:  ld a, 0
    6  0008 3e 01                    [ 7]       @1:  ld a, 1
    7  000a c3 16 00                 [10]            jp @f
    8  000d c3 18 00                 [10]            jp @1f
    9
   10  0010 c3 06 00                 [10]            jp @b
   11  0013 c3 08 00                 [10]            jp @1b
   12  0016 3e 10                    [ 7]       @@:  ld a, $10
   13  0018 3e 11                    [ 7]       @1:  ld a, $11
   14  001a c3 16 00                 [10]            jp @b
   15  001d c3 18 00                 [10]            jp @1b
   16                                           endp
```

#### 関連

- [`PROC`](#proc)
- [ユーザ定義名とスコープ](/syntax/syntax.md#ユーザ定義名とスコープ)

## CONST/EQU

#### 書式

```
CONST name = expression
name EQU expression
```

#### 説明

- 名前が`name`、値が `expression`の定数を定義します。
- 定数のため再定義できません。再定義しようとするとエラーになります。
- `CONST`と`EQU`は書式が異なるだけで同等です。（`EQU`は内部では`CONST`として処理）
- `expression` を評価する際、値が確定できない場合、そのパスでの評価は行わず、評価を次のパスに回します（前方参照可）。
- パスの評価状況によって、値が変わる可能性があります。


#### 関連

- [マルチパスアセンブラ](/README.md#マルチパスアセンブラ) 

## VAR 

#### 書式

```
VAR name = expression ; 定義・初期化
name = expression     ; 再代入
```

#### 説明

- 名前が`name`、値が `expression`の変数を定義します。
- `"="=を使用して再代入可能です。
- 定義・初期化、再代入の際、`expression`の値が確定している必要があります（前方参照不可）。
- パスの評価状況によって、値が変わる可能性があります。

#### 関連

- [マルチパスアセンブラ](/README.md#マルチパスアセンブラ) 

## DB/DEFB{#db}

#### 書式

```
[name] DB   expression [, expression...]
[name] DEFB expression [, expression...]
```

#### 説明

- `expression`の値をバイトデータ（複数指定可）として定義します。
- ラベル `name` を付加することができます。
- `DEFB` は `DB` のエイリアスです。
- `expression` の評価結果が次のものが有効です。

<table>
<thead>
<tr><th>値</th><th>定義</th></tr>
</thead>
<tbody>
<tr><td>数値</td><td class="p0"><ul class="m0">
  <li>バイトの範囲（-128 <= x <= 255）にない場合、警告が出ます。</li>
  <li>バイトの範囲を外れる場合、組み込み関数 <code>$H, $HI, $L, $LO</code> を使ってバイト範囲に変換してください。</li>
</ul>
</td>
</tr>
<tr>
<td class="nowrap">文字列</td>
<td class="p0"><ul class="m0">
<li>文字列をバイト列として解釈し定義します。</li>
<li>2 バイト文字の場合、Shift_JIS コードの上位バイト、下位バイトの順にの定義します。</li>
</ul></td>
</tr>
<tr>
<td>配列</td>
<td class="p0"><ul class="m0"><li>配列要素が数値、文字列の場合、上に記載のとおり定義します。</li></td>
</tr>
</tbody>
</table>

<br>
```
    1                                           charmap mz700,"mz700.json"
    2
    3  0000 01 02 03                 [  ]       db 1, 2, 3
    4  0003 61 62                    [  ]       db "ab"
    5  0005 82 a0 82 a2              [  ]       db "あい"
    6  0009 61 82 a0                 [  ]       db "aあ"
    7  000c 04 35 06                 [  ]       db [4, "5", 6]
    8  000f 0d 1a 2a 27 20 20        [  ]       db mz700("mz-700")
```

#### 関連

- [組み込み関数](/syntax/syntax.md#組み込み関数)`$H, $HI, $L, $LO`
- [`CHARMAP`](#charmap)

## DW/DEFW{#dw}

#### 書式

```
[name] DW   expression [, expression...]
[name] DEFW expression [, expression...]
```

#### 説明

- `expression`の値をワードデータ（リトルエンディアン、複数指定可）として定義します。
- ラベル `name` を付加することができます。
- `DEFW` は `DW` のエイリアスです。
- `expression` の評価結果が次のものが有効です。

<table>
<thead>
<tr><th>値</th><th>定義</th></tr>
</thead>
<tbody>
<tr><td>数値</td><td class="p0"><ul class="m0">
  <li>ワードの範囲（-32768 <= x <= 65535）にない場合、警告が出ます。</li>
  <li>ワードの範囲を外れる場合、組み込み関数 <code>$W, $WORD</code> を使ってワード範囲に変換してください。</li>
</ul>
</td>
</tr>
<tr>
<td class="nowrap">文字列</td>
<td class="p0"><ul class="m0">
<li>文字列をバイト列として解釈し定義します。</li>
<li>2 バイト文字の場合、Shift_JIS コードをワードとして定義します。</li>
</ul></td>
</tr>
<tr>
<td>配列</td>
<td class="p0"><ul class="m0"><li>配列要素が数値、文字列の場合、上に記載のとおり定義します。</li></td>
</tr>
</tbody>
</table>

```
    1                                           charmap mz700,"mz700.json"
    2
    3  0000 01 00 02 00 03 00        [  ]       dw 1, 2, 3
    4  0006 61 00 62 00              [  ]       dw "ab"
    5  000a a0 82 a2 82              [  ]       dw "あい"
    6  000e 61 00 a0 82              [  ]       dw "aあ"
    7  0012 04 00 35 00 06 00        [  ]       dw [4, "5", 6]
    8  0018 0d 00 1a 00 2a 00 27 00  [  ]       dw mz700("mz-700")
       0020 20 00 20 00              [  ]
```

#### 関連

- [組み込み関数](/syntax/syntax.md#組み込み関数)`$W $WORD`
- [`CHARMAP`](#charmap)

## DD

#### 書式
```
[name] DD   expression [, expression...]
```

#### 説明

- `expression` の値に従ってバイトデータ、もしくはワードデータとして定義します。
- ラベル `name` を付加することができます。
- `expression` の評価結果が次のものを有効とします。

<table>
<thead>
<tr><th>値</th><th>定義</th></tr>
</thead>
<tbody>
<tr><td>数値</td><td class="p0"><ul class="m0">
  <li>バイトの範囲の場合、バイトとして定義します。</li>
  <li>バイト範囲のデータをワードとして格納したい場合は、組み込み関数<code>$W, $WORD</code> を使用します。</li>
  <li>バイトの範囲にない場合、ワードとして定義しますが、ワードの範囲（-32768 <= x <= 65535）にない場合、警告が出ます。</li>
  <li>ワードの範囲を外れる場合、組み込み関数 <code>$W, $WORD</code> を使ってワード範囲に変換してください。</li>
</ul>
</td>
</tr>
<tr>
<td class="nowrap">文字列</td>
<td class="p0"><ul class="m0">
<li>文字列をバイト列として解釈し定義します。（<code>DB</code>と同じ）</li>
</ul></td>
</tr>
<tr>
<td>配列</td>
<td class="p0"><ul class="m0"><li>配列要素が数値、文字列の場合、上に記載のとおり定義します。</li></td>
</tr>
</tbody>
</table>

```
    1                                           charmap mz700,"mz700.json"
    2
    3  0000 01 02 03                 [  ]       dd 1, 2, 3
    4  0003 01 02 00 03              [  ]       dd 1, $W(2), 3
    5  0007 61 62                    [  ]       dd "ab"
    6  0009 82 a0 82 a2              [  ]       dd "あい"
    7  000d 61 82 a0                 [  ]       dd "aあ"
    8  0010 04 35 06                 [  ]       dd [4, "5", 6]
    9  0013 0d 1a 2a 27 20 20        [  ]       dd mz700("mz-700")
```

#### 関連

- [組み込み関数](/syntax/syntax.md#組み込み関数)`$W $WORD`
- [`CHARMAP`](#charmap)

## DS/DSB{#ds}

#### 書式

```
[name] DS  length [, fill]
[name] DSB length [, fill]
```

#### 説明

- `legth`数のバイト領域を確保します。
- `fill`を指定した場合、その値で領域を埋めます。
- `fill`を指定しない場合デフォルトの値（255）で埋めます。`-f --fill`オプションを指定した場合はその値で埋めます。
- ラベル `name` を付加することができます。
- `DSB`は`DS`のエイリアスです。

#### 関連

- [`-f --fill`](/exec/exec.md#f---fill) オプション
- [システム変数](/syntax/syntax.md#システム変数)`$FILL`


## DSW

#### 書式

```
[name] DSW length [, fill]
```

#### 説明

- `legth`数のバイト領域を確保します。（`length * 2`バイトの領域を確保）
- `fill`を指定した場合、その値をワードデータ（リトルエンディアン）とし領域を埋めます。
- `fill`を指定しない場合デフォルトの値（255）で埋めます。`-f --fill`オプションを指定した場合はその値で埋めます。
- ラベル `name` を付加することができます。

```
    1  0000 ff ff ff ff              [  ]       dsw 2
    2  0004 34 12 34 12              [  ]       dsw 2, $1234
```

#### 関連

- [`-f --fill`](/exec/exec.md#f---fill) オプション
- [システム変数](/syntax/syntax.md#システム変数)`$FILL`

### ENUM

#### 書式

```
name ENUM
  element_name [= expression]
ENDE
```

#### 説明

- `element_name`要素を持つ列挙体`name`を定義します。
- `name.element_name`として定義した値を参照します。
- 数値以外の値も利用可能です。
- `expression`がない場合、最初は数値 0 を、以降は前回定義した数値要素に +1 した値を定義します。
- `expression`では前方参照できません。
- `expression`で定義済みの`element_name`を参照刷る場合、名前の前に `"."` を付けます

```
RGB enum
  black     ; 0
  blue      ; 1
  red       ; 2
  magenta   ; 3
  green     ; 4
  cyan      ; 5
  yellow    ; 6
  white     ; 7
ende
```
<br>
```
    1                                           ATTR enum
    2                                             WHITE   = $70
    3                                             YELLOW  = $60
    4                                             CYAN    = $50
    5                                             GREEN   = $40
    6                                             MAGENTA = $30
    7                                             RED     = $20
    8                                             BLUE    = $10
    9                                             BLACK   = $00
   10                                           ende
   11
   12  0000 3e 10                    [ 7]       ld a, ATTR.BLUE
```
<br>
```
    1                                           msgs enum
    2                                             m0 = "0"
    3                                             m1 = .m0 + "1"
    4                                             m2 = .m1 + "2"
    5                                             m3 = .m2 + "3"
    6                                           ende
    7
    8  0000 30                       [  ]       db msgs.m0
    9  0001 30 31                    [  ]       db msgs.m1
   10  0003 30 31 32                 [  ]       db msgs.m2
   11  0006 30 31 32 33              [  ]       db msgs.m3
```

## INCLUDE
#### 書式

```
INCLUDE filename
```

#### 説明

- ファイル`filename`をソースファイルとして現在の位置に読み込みます。
- 読み込むファイルは次の順で検索します。
   - ソースファイルと同じディレクトリ
   - `-I`オプションで指定されたディレクトリ
- ファイル読み込みは構文解析工程で行うため`filename`は文字列リテラルのみ有効です。式は使用できません。
- 指定したファイルが存在しないとエラーになります。
- 極端な例ですが次のケースでは`somefile.inc`が存在しないとエラーになります。

<pre style="font-size:90%"><code>if 0                     ; 0 なので次の include の（展開）内容は評価対象外
  include "somefile.inc" ; ここは構文解析で展開されるため somefile.incが存在しないとエラーとなる
endif
</code></pre>

#### 関連

- [`-I`](/exec/exec.md#i)オプション

## INCBIN/BINCLUDE{#incbin}

#### 書式

```
INCBIN   filename [, offset [, length]]
BINCLUDE filename [, offset [, length]]
```

#### 説明

- ファイル`filename`をバイトデータとして現在の位置に埋め込みます。
- 読み込むファイルは次の順で検索します。
   - ソースファイルと同じディレクトリ
   - `-I`オプションで指定されたディレクトリ
- `offset`が指定された場合、`filename`を指定された位置から読み込みます。
- `offset, length`が指定された場合、`filename` を指定された位置から`length`バイト分読み込みます。
- 実際に読み込めるサイズが `length`より小さい場合でもエラーにはならず、読める分だけ読み込みます。
- 読み込んだバイト数はシステム変数 `$RSIZE` に設定されます。（エラーの場合は -1）
- `BINCLUDE`は`INCBIN`のエイリアスです。

```
; b16.bin
00000000: 30 31 32 33 34 35 36 37  38 39 41 42 43 44 45 46  |0123456789ABCDEF
```

```
sample.asm:
    1  0000 30 31 32 33 34 35 36 37  [  ]       incbin "b16.bin"
       0008 38 39 41 42 43 44 45 46  [  ]
    2  0010 3e 10                    [ 7]       ld a, $rsize
    3  0012 38 39 41 42 43 44 45 46  [  ]       incbin "b16.bin", 8
    4  001a 3e 08                    [ 7]       ld a, $rsize
    5  001c 38 39 41 42 43 44 45 46  [  ]       incbin "b16.bin", 8, 16
    6  0024 3e 08                    [ 7]       ld a, $rsize
```


#### 関連

- [`-I`](/exec/exec.md#i)オプション
- [システム変数](/syntax/syntax.md#システム変数)`$RSIZE`

## CHARMAP

#### 書式

```
CHARMAP name, json-filename|json-string [, option]
```

#### 説明

- json ファイル `json-filename`もしくは json 文字列`json-string` を指定して charmap `name`を定義します。
- 定義した charmap は文字列を引数とし、要素数が 1 か 2 のバイト配列を出力する関数となります。
- charmap を適用した結果を数値とする場合は、組み込み関数 `$CHR` を使用します。
- json ファイルは次の順で検索します。
   - ソースファイルと同じディレクトリ
   - `-I`オプションで指定されたディレクトリ
- `option` によって定義した charmap にない文字の扱いを指定できます。
- 有効な json のフォーマットは次のとおりです。
  - 単1オブジェクト `{}`
  - キーは `string`（1 文字）
  - 値は `number`の`array`

<br>
__mz700.json の冒頭__
```json
{" ": [0], "!": [97], "\"": [98], "#": [99], "$": [100], "%": [101], 以降省略 
```
<br>
<table>
<thead>
<tr>
<th style="width:5em"><code>option</code></th>
<th style="width:5em">値</th><th>説明</th></tr>
</thead>
<tbody>

<tr>
<td>指定なし</td>
<td></td>
<td rowspan="2">未定義の文字をエラーとします。 </td>
</tr>

<tr>
<td rowspan="3">指定あり</td>
<td><code>$CMAP_ERR (-1)</code></td>
</tr>
<tr>
<td class="nowrap"><code>$CMAP_THRU (-2)</code></td>
<td>未定義の文字をそのまま出力します。</td>
</tr>
<tr>
<td class="nowrap">数値 (<code>0-65535</code>)</td>
<td>指定された数値を文字コードとし出力します。</li>
</tbody>
</table>

<br>

```
    1                                           charmap map,  '{"a":[1]}'
    2                                           charmap mapb, '{"a":[1]}', 255
    3                                           charmap mapw, '{"a":[1]}', $1234
    4                                           charmap mapt, '{"a":[1]}', $cmap_thru
    5
    6  0000 01 3f                    [  ]       db map("ab")
    *  [ERR] CHARMAP に文字 'b' の定義がない
    7  0002 01 ff                    [  ]       db mapb("ab")
    8  0004 01 12 34                 [  ]       db mapw("ab")
    9  0007 01 62                    [  ]       db mapt("ab")
   10
   11                                           charmap vmap, '{"a":[1], "b":[1,2]}'
   12  0009 3e 01                    [ 7]       ld a,  $chr(vmap("a"))
   13  000b 21 02 01                 [10]       ld hl, $chr(vmap("b"))
```

#### 関連

- [システム変数](/syntax/syntax.md#システム変数) `$CMAP_ERR, $CMAP_THRU`
- [組み込み関数](/syntax/syntax.md#組み込み関数) `$CHR`


## SETMAP

#### 書式

```
SETMAP name, char, array
```

#### 説明

- 定義済みの charmap を修正します。
- `name` は charmap 名、`char`は 1 文字の文字列、`array`は要素数が 1 か 2 のバイトデータ配列です。
- 定義の追加、更新は可能ですが、削除はできません。

```
    1                                           charmap map,  '{}'       ; 空の charmap
    2       "a":[1, 2]                          setmap  map, "a", [1, 2] ; 追加
    3
    4  0000 01 02                    [  ]       db map("a")
    5
    6                                           charmap map2, '{"a": [1, 2]}'
    7       "a":[1]                             setmap map2, "a", [1]    ; 更新
    8       "b":[2]                             setmap map2, "b", [2]    ; 追加
    9
   10  0002 01 02                    [  ]       db map2("ab")
```


#### 関連

- [`CHARMAP`](#charmap)

## ERROR/WARN/INFO

#### 書式

```
ERROR expression
WARN  expression
INFO  expression
```

#### 説明

- `expression`（文字列）をそれぞれ、エラー/警告/情報として出力します。
- `expression` の評価結果として文字列、数値、レジスタリテラル、フラグリテラルのいずれかが有効です。
- 所定の書式で出力したい場合は組み込み関数`$FMT, $FORMAT`を使用します。

```
temp/sample.asm:
    1       03e7(999)               const num = 999
    2
    3                               error "エラー"
    *  [ERR] エラー
    4                               warn "警告"
    *  [WARN] 警告
    5                               info $fmt("pass: %d, num: %d", $pass, num)
    *  [INFO] pass: 1, num: 999
```

#### 関連

- [エラー・警告・情報](/error.md#エラー・警告・情報)
- [重複出力の集約](/error.md#重複出力の集約)
- [組み込み関数](/syntax/syntax.md#組み込み関数)`$FMT, $FORMAT`

## IF/ELIF/ELSE/ENDIF

#### 書式

```
IF expression
[ELIF expression...]
[ELSE expression]
ENDIF
```
#### 説明

- 条件アセンブルです。
- ネストも可能です。
- `ELIF`は複数定義できます。
- `ELIF`は評価時にネストした`IF`のシンタックスシュガーです。

<div class="iblock top" style="width:45%;"><pre style="height:25em"><code>IF 1
  ; statment
ELIF 2
  ; statment
ELIF 3
  ; statment
ELSE
  ; statment
ENDIF</code><br><br><br><br><br></pre></div>
<div class="iblock top center" style="width:2em"><br><i class="fa fa-arrow-right"></i></div>
<div class="iblock" style="width:45%"><pre style="height:25em"><code>IF 1
  ; statment
ELSE
  IF 2
  ; statment
  ELSE
    IF 3
  ; statment
    ELSE
  ; statment
    ENDIF
  ENDIF
ENDIF
</code></pre></div>



#### 関連

- [真偽値リテラル](/syntax/syntax.md#真偽値リテラル)

## FUNC/RETURN/ENDF
#### 書式

```
; 定義
name FUNC [parameter [, parameter...]]
    [statement...]
    RETURN [expression]
    [statement...]
ENDF

; 呼出し ※yas80 には式文がないため、ブランク識別子への代入としている
_ = name ( [argument [, argument...]] )
```

#### 関数定義

- 関数`name`を定義します。
- 任意数（0 個以上）の仮引数を定義可能です

#### 関数呼出し

- `name` の後に引数のリストを`()` で囲ったものです。
- 引数を持たない関数でも呼出しの際に `()` が必要です。
- 関数定義の仮引数の数と、関数呼出しの実引数の数が同じでない場合エラーになります。
- 可変長引数はありませんが、配列を使用することで代替できる場合があります。

#### 戻り値

- `RETURN expression`の`expression`を評価した値を戻り値とし、呼出し元へ戻ります。
- `RETURN`で`expression`を省略した場合、もしくは関数内の文が`ENDF`に到達した場合、"不定な値" を戻り値として呼出し元へ戻ります。
- "不定な値" を値として使用するとエラーになります。
- `name`は定数、変数として定義できる"関数値"として評価します。

#### クロージャ

- 関数呼出し毎に新しいスコープを作成します。
- 関数呼出しから関数を返す場合、クロージャとなります。


```
    1                                           ; フィボナッチ数列の n 項(0-) を返す関数
    2                                           fib func n
    3                                             if n == 0
    4                                                return 0
    5                                             elif n == 1
    6                                                return 1
    7                                             else
    8                                                return fib(n - 1) + fib(n - 2) ; 再帰
    9                                             endif
   10                                           endf
   11
   12                                           rept 5
   13                                             db $i, fib($i)
   14                                           endr
   14                                         + $COUNT = 5(0x5)
   14                                         + $I = 0(0x0)
   14  0000 00 00                    [  ]     +   db $i, fib($i)
   14                                         + $COUNT = 5(0x5)
   14                                         + $I = 1(0x1)
   14  0002 01 01                    [  ]     +   db $i, fib($i)
   14                                         + $COUNT = 5(0x5)
   14                                         + $I = 2(0x2)
   14  0004 02 01                    [  ]     +   db $i, fib($i)
   14                                         + $COUNT = 5(0x5)
   14                                         + $I = 3(0x3)
   14  0006 03 02                    [  ]     +   db $i, fib($i)
   14                                         + $COUNT = 5(0x5)
   14                                         + $I = 4(0x4)
   14  0008 04 03                    [  ]     +   db $i, fib($i)
```
<br>
```
    1                                           ; "n から始まる数値を返す関数"を返す関数
    2                                           mk_counter func n
    3                                             var value = n - 1
    4                                             fn func
    5                                               value = value + 1
    6                                               return value
    7                                             endf
    8                                             return fn ; 戻り値は"関数"
    9                                           endf
   10
   11                                           var counter = mk_counter(0)
   12
   13  0000 3e 00                    [ 7]       ld a, counter()
   14  0002 3e 01                    [ 7]       ld a, counter()
   15  0004 3e 02                    [ 7]       ld a, counter()
   16  0006 3e 03                    [ 7]       ld a, counter()
```

#### 関連

- [`FUNCTION`](#function)

## FUNCTION

#### 書式

```
FUNCTION name ( [parameter [, parameter...]] ) expression
```
&nbsp;<i class="fa fa-arrow-down">
```
name FUNC [parameter [, parameter...]]
   RETURN expression
ENDF
```

#### 説明

- `expression`を戻り値とする関数`name`を定義します。
- これは書式の下側の `FUNC` を使用した定義のシンタックスシュガーです。
- 引数は `()` で囲みます。引数がない(0 個）場合でも `()` が必要です

```
    1                         ; 配列要素の合計を返す関数
    2                         function sum(vals) sum_iter(0, 0, vals)
    3 
    4                         sum_iter func total, index, vals
    5                           if index == $len(vals)
    6                             return total
    7                           else
    8                             return sum_iter(total + vals[index], index + 1, vals)
    9                           endif
   10                         endf
   11
   12  0000 3e 00     [ 7]    ld a, sum([])
   13  0002 3e 01     [ 7]    ld a, sum([1])
   14  0004 3e 06     [ 7]    ld a, sum([1, 2, 3])
```

#### 関連

- [`FUNC`](#funcreturnendf)

## MACRO/ENDM
#### 書式

```
; 定義
name MACRO [parameter [, parameter...]]
  statements
ENDM

; 呼出し
name [argument [, argument]]
```
#### マクロ定義

- マクロ`name`を定義します。
- 任意個（0 個以上）の仮引数を定義可能です。
- マクロ定義内で`"@"`で始まる名前を使用した場合、`MACRO`ローカルの名前になります。
- `MACRO`ローカル名は定義されているマクロ内でユニークである必要があります。
- マクロ定義はネストできません（マクロ定義の中でマクロを定義することはできません）
- マクロ定義の中に`PROC`を含むことはできません。
- マクロ定義の中から定義中のマクロそのものを呼び出す（再帰呼び出し）ことはできません。

#### マクロ呼出し

- マクロ定義の仮引数の数と、マクロ呼出しの実引数の数が同じでない場合エラーになります。
- 可変長引数はありませんが、配列を使用することで代替できる場合があります。
- マクロ呼出し中に`EXITM`を評価した場合、そのマクロの評価を終了し、マクロ呼出し元へ戻ります。

```
    1       1000(4096)                          const addr1 = $1000
    2       2000(8192)                          const addr2 = $2000
    3
    4                                           check macro n, addr
    5                                                 cp n
    6                                                 jp nz, @ret
    7                                                 call addr
    8                                           @ret: ret
    9                                           endm
   10
   11                                           check 1, addr1
   11  0000 fe 01                    [ 7]     +       cp n
   11  0002 c2 08 00                 [10]     +       jp nz, @ret
   11  0005 cd 00 10                 [17]     +       call addr
   11  0008 c9                       [10]     + @ret: ret
   11                                         + endm(CHECK)
   12                                           check 2, addr2
   12  0009 fe 02                    [ 7]     +       cp n
   12  000b c2 11 00                 [10]     +       jp nz, @ret
   12  000e cd 00 20                 [17]     +       call addr
   12  0011 c9                       [10]     + @ret: ret
   12                                         + endm(CHECK)
```
<br>

```
sample.asm:
    1                                           pushreg macro reg
    2                                             if $isary(reg)
    3                                               rept reg ; 配列なら展開して push
    4                                                 push $v
    5                                               endr
    6                                             else
    7                                               push reg ; そうでなければそのまま push
    8                                             endif
    9                                           endm
   10
   11                                           pushreg hl
   11  0000 e5                       [11]     +     push reg ; そうでなければそのまま push
   11                                         + endm(PUSHREG)
   12                                           pushreg [hl, de]
   12                                         + $COUNT = 2(0x2)
   12                                         + $I = 0(0x0)
   12                                         + $V = HL
   12  0001 e5                       [11]     +       push $v
   12                                         + $COUNT = 2(0x2)
   12                                         + $I = 1(0x1)
   12                                         + $V = DE
   12  0002 d5                       [11]     +       push $v
   12                                         + endm(PUSHREG)
```
<br>
__sample.lst__
```
sample.asm
    1                                           defmsg macro num, msg
    2                                           data ## $fmt("_%02d", num) db msg
    3                                           endm
    4
    5                                           defmsg 1, "one"
    5  0000 6f 6e 65                 [  ]     + data ## $fmt("_%02d", num) db msg
    5                                         + endm(DEFMSG)
    6                                           defmsg 2, "two"
    6  0003 74 77 6f                 [  ]     + data ## $fmt("_%02d", num) db msg
    6                                         + endm(DEFMSG)
    7                                           defmsg 3, "three"
    7  0006 74 68 72 65 65           [  ]     + data ## $fmt("_%02d", num) db msg
    7                                         + endm(DEFMSG)
```
__sample.sym__
```
0000 DATA_01
0003 DATA_02
0006 DATA_03
```
#### 関連

- [`REPT/ENDR`](#reptendr)
- [`EXITM`](#exitm)

## REPT/ENDR

#### 書式

```
REPT expression
  statements
ENDR
```

#### 説明

- `expressioin`の評価結果が数値の場合、その回数だけ `REPT-ENDR` 間の`statement`を展開します。
- `expressioin`の評価結果が配列の場合、その配列要素の数だけ `REPT-ENDR` 間の`statement`を展開します。
- 展開の際、次のシステム変数の値が設定されます。

| 変数名       | 型   | 内容 |
| --           | :--:   | --   |
| `$COUNT`     | 数値 | "`REPT`数値" の場合はその数値（展開回数）<br>"`REPT`配列" の場合は配列要素数 |
| `$I`         | 数値 | 展開毎の序数（`0`から`$COUNT - 1`まで）|
| `$V`         | -   | "`REPT`配列" の展開毎の値（配列要素の値）|

<br>

```
push_regs macro lst
  rept lst
    push $v
  endr
endm

pop_regs  macro lst
  rept $rev(lst)         ; 逆順にする
    pop $v
  endr
endm

process proc
  const .regs = [hl, de] ; 保存レジスタの定義
  push_regs .regs        ; 保存レジスタ push
  ; do something
  pop_regs .regs         ; 保存レジスタ pop
  ret
endp
```

#### 関連

- [`MACRO/ENDM`](#macroendm)
- [`EXITM`](#exitm)
- [システム変数](/syntax/syntax.md#システム変数)


## EXITM

#### 書式

```
1) EXITM
```
<br>

```
2) EXITM IF expression
```
&nbsp;<i class="fa fa-arrow-down">
```
IF expression
  EXITM
ENDIF
```

#### 説明

- `EXITM`を評価するとマクロ展開を中止します。
- 後置`IF`は`IF expression\EXITM\ENDIF` のシンタックスシュガーです。
- 次の例では`REPT`の最後の展開時のみ`inc hl`が評価されないよう`EXITM`を使用しています。


```
    1                                           rept 4
    2                                             ld (hl), a
    3                                             exitm if $i == $count - 1
    4                                             inc hl
    5                                           endr
    5                                         + $COUNT = 4(0x4)
    5                                         + $I = 0(0x0)
    5  0000 77                       [ 7]     +   ld (hl), a
    5  0001 23                       [ 6]     +   inc hl
    5                                         + $COUNT = 4(0x4)
    5                                         + $I = 1(0x1)
    5  0002 77                       [ 7]     +   ld (hl), a
    5  0003 23                       [ 6]     +   inc hl
    5                                         + $COUNT = 4(0x4)
    5                                         + $I = 2(0x2)
    5  0004 77                       [ 7]     +   ld (hl), a
    5  0005 23                       [ 6]     +   inc hl
    5                                         + $COUNT = 4(0x4)
    5                                         + $I = 3(0x3)
    5  0006 77                       [ 7]     +   ld (hl), a
```

#### 関連

- [`MACRO/ENDM`](#macroendm)
- [`REPT/ENDR`](#reptendr)

## LIST

#### 書式

```
LIST expression
```

#### 説明

- リスト出力を制御します。
- `expression`が真(1) の場合、次の行からリスト出力します。
- `expression`が偽(0) の場合、次の行からリスト出力しません。
- `LIST` 行はリスト出力対象外です。


```
ld a, 1
list 0
ld a, 2
list $on
ld a, 3
list $false
ld a, 4
list 1
ld a, 5
```
&nbsp;<i class="fa fa-arrow-down"</i>
```
    1  0000 3e 01                    [ 7]       ld a, 1
    5  0004 3e 03                    [ 7]       ld a, 3
    9  0008 3e 05                    [ 7]       ld a, 5
```
        
#### 関連

- [オプション](/exec/exec.md#l)`-l, --lst`
- [リストファイル](/output/output.md#リストファイル)