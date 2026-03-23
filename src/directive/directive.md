# ディレクティブ

## ORG

#### 書式
```
ORG addr [, ABS|REL]
```

#### 説明

- 次の命令のベースアドレスを `addr` で指定します。
- この`ORG`から（あれば）次の`ORG`までのセグメントについて、出力ファイル内での配置方法を指定します。
   - `ABS` この`ORG`から始まるセグメントを"絶対セグメント"として出力ファイルでの配置を決定します。 
   - `REL` この`ORG`から始まるセグメントを"相対セグメント"として出力ファイルでの配置を決定します。 

#### 関連

- [生成コードの配置](/allocate.md)

## END

#### 書式

```
END [start_addr]
```

#### 説明

- トップレベルにある場合、アセンブルを終了し、以降の命令、以降のファイルはアセンブル対象外とします。
- `start_addr` を指定した場合、MZT 形式の実行開始アドレスとなります。
- 複数の `start_addr` が指定された場合、最後に指定されたものが有効です。
- `-e --entry`オプションが指定された場合、そちらが有効です。

#### 関連
- [MZT 形式](/output/output.md#mzt-形式)
- [`ENTRY`](#entry)
- [`-e --entry`](/exec/option.md#e---entry)オプション

## ENTRY

#### 書式
```
ENTRY start_addr
```

#### 説明

- MZT 形式の実行開始アドレスを指定します。
- 複数指定された場合、最後に指定されたものが有効です。
- `-e --entry`オプションが指定された場合、そちらが有効です。
- アセンブルを中止しない点を除き、機能的には "`END start_addr`" と同じです。

#### 関連
- [MZT 形式](/output/output.md#mzt-形式)
- [`END`](#end)
- [`-e --entry`](/exec/option.md#e---entry)オプション

## ALIGN

#### 書式
```
ALIGN size [,fill]
```

#### 説明
- ローケンションカウンタを `size` の倍数となるまで進めます。
- `fill` が指定されている場合、その値で（あれば）ギャップを埋めます。
- `fill` が指定されていない場合、デフォルトもしくは `-f --fill`オプションで指定した値で埋めます。

#### 関連

- [`-f --fill`](/exec/option.md#f---fill)オプション
- [システム変数](syntax/syntax.md#システム変数)`$FILL`


## PROC

#### 書式

```
name    PROC
.local_label1:
        statements
inner_label:
        stataments
.local_name2:
        statements
        ENDP

        call name
        call inner_label
        call name.local_label1
        call name.local_label2
```

#### 説明

- トップレベルでのみ定義でき、複数の文を一つにまとめます。
- `name` はラベルとして参照可能
- `inner_label` はグローバルラベルで `PROC` 外部からそのまま参照できます。
- `.local_name` を外部から参照する場合、`name.local_name` とします。
- `.local_name` は同一`PROC` 内で一意であることが必要ですが、他の `PROC` の名前と同じでもかまいません。
- `statements` として次のものを含むことはできません。
    - `PROC`
    - `MACRO`

#### 関連

- [スコープ](/scope.md)
- [ユーザ定義名とスコープ](/syntax/syntax.md#ユーザ定義名とスコープ)
- [PROC ローカルラベル](/syntax/syntax.md#proc-ローカルラベル)

## CONST/EQU

#### 書式

```
CONST name = expression
name EQU expression
```

#### 説明

- 名前が`name`、値が `expression`の定数を定義します。定数のため再定義できません。
- `CONST`と`EQU`は書式が異なるだけで同等です。（`EQU`は yas80 内部では `CONST` 扱い）
- `expression` を評価する際、値が確定できない場合、そのパスでの評価は行わず、評価を次のパスに回します。（前方参照可）
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

- 名前が`name`、値が `expression`の変数を定義します。変数のため再代入可能です。
- 定義・初期化、再代入の際、`expression`の値が確定している必要があります。（前方参照は不可）
- パスの評価状況によって、値が変わる可能性があります。

#### 関連

- [マルチパスアセンブラ](/README.md#マルチパスアセンブラ) 

## DB

#### 書式

```
[name] DB   expression [, expression...]
[name] DEFB expression [, expression...]
```

#### 説明

- `expression` をバイト列として定義します。
- `expression` の評価結果が次のものを有効とします。
- ラベル `name` を付加することができます。
- `DEFB` は `DB` のエイリアスです。

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

## DW

#### 書式

```
[name] DW   expression [, expression...]
[name] DEFW expression [, expression...]
```

#### 説明

- `expression` をワード（リトルエンディアン）列として定義します。
- `expression` の評価結果が次のものを有効とします。
- ラベル `name` を付加することができます。
- `DEFW` は `DW` のエイリアスです。

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

- `expression` の値に従ってバイト、もしくはワードとして定義します。
- `expression` の評価結果が次のものを有効とします。
- ラベル `name` を付加することができます。

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

## DS/DSB

#### 書式

```
[name] DS  length [, fill]
[name] DSB length [, fill]
```

#### 説明

- `legth`数のバイト領域を確保します。
- `fill`を指定した場合、その値で領域を埋めます。
- `fill`を指定しない場合デフォルトの値（255）で埋めます。`-f --fill`オプションを指定した場合はその値で埋めます。
- `DSB`は`DS`のエイリアスです。
- ラベル `name` を付加することができます。

#### 関連

- [`-f --fill`](/exec/option.md#f---fill) オプション
- [`システム変数`](/syntax/syntax.md#システム変数)`$FILL`


## DSW

#### 書式

```
[name] DSW length [, fill]
```

#### 説明

- `legth`数のバイト領域を確保します。
- `fill`を指定した場合、その値をワードデータとし領域を埋めます。
- `fill`を指定しない場合デフォルトの値（255）で埋めます。`-f --fill`オプションを指定した場合はその値で埋めます。
- ラベル `name` を付加することができます。

```
    1  0000 ff ff ff ff              [  ]       dsw 2
    2  0004 34 12 34 12              [  ]       dsw 2, $1234
```

#### 関連

- [`-f --fill`](/exec/option.md#f---fill) オプション
- [`システム変数`](/syntax/syntax.md#システム変数)`$FILL`

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

#### 関連

- [`-I`](/exec/option.md#i)オプション

## INCBIN

#### 書式

```
INCBIN filename [, offset [, length]]
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

- [`-I`](/exec/option.md#i)オプション
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
- `name` は charmap 名、`char`は 1 文字の文字列、`array`は要素数が 1 or 2 のバイト配列です。
- 追加、更新は可能ですが、削除はできません。

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
- `ELIF` は複数定義できます。
- `ELIF` は評価時にネストした `IF` として評価しています。

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
#### 説明
#### 関連

- 定義：`<name>` FUNC [param1 [, param2...]] \ `<statements>` \ ENDF
- 呼出: `<name>` ( [arg1 [, arg2...]])
- 関数呼出しは`<expr>`
- 引数がない場合でも () は必要
- `<statements>` の中で RETURN 文が評価されると、その時点で呼出し元へ戻る
- RETURN 文に値が指定されていると、その値が呼出し元に返される
- `<statements>` 内に RETURN 文がない場合、全ての文を実行後、呼び出し元に戻る。値は返さない
- `<name>` は関数値を持つため、他の定数・変数に代入したり、関数を値として返すことも可能
- 関数呼出しはクロージャを形成する

### RETURN 文
#### 書式
#### 説明
#### 関連
- 関数の処理を終了し呼出し元に戻る。
- 戻り値なし： RETURN
- 戻り値あり： RETURN `<expr>`

```
フィボナッチ関数の例
フィボナッチ数列生成関数の例
```

## FUNCTION
#### 書式
#### 説明
#### 関連

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
## MACRO/ENDM
#### 書式
#### 説明
#### 関連

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

## REPT/ENDR
#### 書式
#### 説明
#### 関連

- REPT `<expr>` \ `<statements>` \ ENDR
- `<expr>` の数だけ `<statements>` を展開する
- ＠名前 は各展開毎のローカル名となる
- `<statements>` の中で EXIT を使用すると、そこから ENDR までの展開をやめる

## EXITM
#### 書式
#### 説明
#### 関連
