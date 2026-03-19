# ソースファイル

- ソースファイルは複数指定可能です。
- 複数のソースファイルを指定した場合、それらを結合した1つのファイルとしてアセンブルします。
- アセンブル中、トップレベルに [`END`](/directive/directive.md#END) 文があった場合、アセンブル中のソースファイルと以降のソースファイルのアセンブルを終了します。

## エンコーディング

- 利用可能なエンコーディングは UTF-8 および Shift_JIS です。
- UTF-8 の場合、BOM の有無は問いません（あってもなくても可）
- UTF-8 としてとして不正な文字を検出した場合、Shift_JIS から UTF-8 へのエンコディーング変換を試みます。
- 変換に失敗した場合、アセンブルを中止します。

## 行

- アセンブル処理は行単位で実行します。
- 通常、物理行（行頭から行末改行記号）と論理行（アセンブラの処理単位）は同じですが、1 物理行に複数の論理行行を記述する[マルチステートメント](#マルチステートメント)、逆に 1 論理行に複数の物理行が対応する[継続行](#継続行)が可能です。


## マルチステートメント

<style>
.center { text-align:center;}
</style>

- `"\"` で区切ることで １物理行の中に複数の論理行を記載することができます。
- エラー、警告等の表示は物理行単位になります。

<div class="iblock top" style="width:15em"><pre><code>ld a, (hl) \ inc hl</code></pre></div>
<div class="iblock top"><i class="fa fa-arrow-left"></i><i class="fa fa-arrow-right"></i></div>
<div class="iblock" style="width:15em"><pre><code>ld a, (hl)
inc hl</code></pre></div>


## 継続行

- 行末が `"\"` の場合（`"\"` の直後が改行） の場合、次行と合わせて1行として扱います。
- `"\"` の直後に改行以外の文字がある場合は継続行になりません。

```
msg db "hello, " \
       "world" \
       "!"
```
&nbsp;<i class="fa fa-arrow-down" aria-hidden="true"> 継続行
```
msg db "hello, " "world" "!"
```
&nbsp;<i class="fa fa-arrow-down" aria-hidden="true"> [文字列リテラルの結合](/syntax/syntax.md#文字列リテラルの結合)
```
msg db "hello, world!"
```

## 行の種類

- 空白行
- コメント行（コメントのみ含む）
- Z80 / R800 命令行
- 疑似命令行