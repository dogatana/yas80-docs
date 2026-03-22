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

- `expression` をバイトデータ（の列）として定義します。
- `expression` を評価したとき、バイトの範囲（-128 <= x <= 255）にない場合、警告が出ます。
- バイトの範囲を外れる場合、組み込み関数 `$H, $HI, $L, $LO` を使ってバイト範囲に変換してください。
- ラベル `name` を付加することができます。
- `DEFB` は `DB` のエイリアスです。

#### 関連

-  [組み込み関数](syntax/syntax.md#組み込み関数)`$H, $HI, $L, $LO`

## DW

#### 書式

```
[name] DW   expression [, expression...]
[name] DEFB expression [, expression...]
```

- `expression` をバイトデータ（の列）として定義します。
- `expression` を評価したとき、バイトの範囲（-128 <= x <= 255）にない場合、警告が出ます。
- バイトの範囲を外れる場合、組み込み関数 `$H, $HI, $L, $LO` を使ってバイト範囲に変換してください。
- ラベル `name` を付加することができます。
- `DEFB` は `DB` のエイリアスです。

#### 関連

-  [組み込み関数](syntax/syntax.md#組み込み関数)`$H, $HI, $L, $LO`


## DS

## INCLUDE

## INCBIN

## CHARMAP

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

## SETMAP

## IF/ELIF/ELSE/ENDIF

- IF `<expr>` \ `<statements>` \ ENDIF
- IF `<expr>` \ `<statements>` \ ELSE \ `<statements>` \ ENDIF
- IF `<expr>` \ `<statements>` \ ELIF `<expr>`ELSE \ `<statements>` \ ENDIF
- IF 文のネストは可能
- ELIF は IF ELSE IF ELSE ENDIF ENDIF のシンタックスシュガー

## FUNC/RETURN/ENDF

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
- 関数の処理を終了し呼出し元に戻る。
- 戻り値なし： RETURN
- 戻り値あり： RETURN `<expr>`

```
フィボナッチ関数の例
フィボナッチ数列生成関数の例
```

## FUNCTION

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

- REPT `<expr>` \ `<statements>` \ ENDR
- `<expr>` の数だけ `<statements>` を展開する
- ＠名前 は各展開毎のローカル名となる
- `<statements>` の中で EXIT を使用すると、そこから ENDR までの展開をやめる

## EXITM
