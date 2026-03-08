# ディレクティブ（疑似命令）

## ORG

## END

## ENTRY

## PROC

- `<name>` PROC \ `<statements>` \ ENDP
- トップレベルでのみ定義可能
- `<name>` の通常ラベルが定義される
- .ローカル名のスコープ管理単位
- .ローカル名は PROC 内で一意であること（通常ラベルとの位置関係は問わない）
- `<statements>` に次のものを含むことはできないできない（定義のネスト不可）
    - PROC
    - MACRO
- 定義のネストは不可

## ALIGN

## CONST/EQU

- CONST `<name>` = `<expr>`
- `<name>` EQU `<expr>`

## VAR

- 定義：VAR `<name>` = `<expr>`
- 更新（代入） VAR = `<expr>`

## = (代入)

## DB/DW/DD

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

## RETURN 文

- 関数の処理を終了し呼出し元に戻る。
- 戻り値なし： RETURN
- 戻り値あり： RETURN `<expr>`

## REPT 文

- REPT `<expr>` \ `<statements>` \ ENDR
- `<expr>` の数だけ `<statements>` を展開する
- ＠名前 は各展開毎のローカル名となる
- `<statements>` の中で EXIT を使用すると、そこから ENDR までの展開をやめる
-

## MACRO 文

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
