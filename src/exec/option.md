## オプション

### -I

- [`INCLUDE`](/directive/directive.md#include),[`INCBIN`](/directive/directive.md#incbin),[`CHARMAP`](/directive/directive.md#charmap) がファイルをロードするフォルダを指定します。
- `-Idir`, `-I dir` のようにフォルダを指定します。
- `-Idir1 -Idir2` のように複数回指定できます
- `-Idir1,dir2` のように `","`で区切って複数のフォルダを指定可能です。
- 検索順は次のとおりです
    1. 疑似命令を含むソースファイルと同じフォルダ
    2. -I で最初に指定したフォルダ
    3. -I で次に指定フォルダ
    4. 以降 -I の指定数まで繰り返し

```
-I dir1 -Idir2,dir3
```
### -D

- シンボルを定義します。ソースファイル内で [`CONST/EQU`](/directive/directive.md#constequ)  を使用したのと同じ効果です。
- `-Dname=value`もしくは`-D name=value`のように名前と値を指定します。
- 値として指定できるのは数値のみです。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- `-Dname` のように名前のみ指定した場合は `-Dname=1` と指定したものとして扱います。
- `-Dname1=1 -Dname2=2` のように複数回指定できます。
- `-Dname1=1,name2=2` のように `","` で区切って複数のシンボルを定義可能です。

```
-D name=1,name2 -Dname3=3
```

### -o --output

- 出力ファイル名を指定します（※）
- 出力ファイル名の拡張子に応じた出力形式で出力します。
    - 拡張子 `.mzt` ･･･ [MZT 形式](/output/output.md#mzt-形式)（`--mzt` 指定）
    - 拡張子 `.t88` ･･･ [T88 形式](/output/output.md#t88-形式)（`--t88` 指定）
    - これ以外の拡張子 ･･･ [BIN 形式](/output/output.md#bin-形式)(デフォルト)
- 出力ファイル名と同時に `--mzt` もしくは `--t88` を指定した場合、拡張子に関わらず、その形式指定が優先されます。

※ アセンブル結果はデフォルトではソースファイル名と同じフォルダに、ソースファイルの拡張子を出力形式に応じたものに変更したファイルへ出力します。

### --mzt

- MZT 形式で出力します。
- -o, --output で指定した拡張子が t88 の場合、エラーとします。

### --t88

- T88 形式で出力します
- -o, --output で指定した拡張子が mzt の場合、エラーとします。

### オプション組み合わせ

<table style="width:40em">
<thead>
<tr><th>-o 拡張子</th><th>--mzt</th><th>-t88</th><th>出力形式</th></tr>
</thead>
<tbody>
<tr><td rowspan="3">.mzt</td><td></td><td></td><td>MZT</td></tr>
<tr></td><td class="center">✓</td><td></td><td>MZT</td></tr>
<tr></td><td class="center">n/a</td><td class="center">✓</td><td>エラー</td></tr>
<tr><td rowspan="3">.t88</td><td></td><td></td><td>T88</td></tr>
<tr></td><td class="center">✓</td><td class="center">n/a</td><td>エラー</td></tr>
<tr></td><td></td><td class="center">✓</td><td>T88</td></tr>
<tr><td rowspan="3">.mzt .t88 以外</td><td></td><td></td><td>BIN</td></tr>
<tr></td><td class="center">✓</td><td></td><td>MZT</td></tr>
<tr></td><td></td><td class="center">✓</td><td>T88</td></tr>
</tbody>
</table>

### -e --entry

- 出力ファイルの実行開始アドレスを指定します。
- ソースコードでは [`END`](/directive/directive.md#END)、[`ENTRY`](/directive/directive.md#entry) で指定しますが、このオプションの指定を優先します。
- 出力ファイルの形式が [MZT 形式](/output/output.md#mzt-形式) の場合のみ有効です。

### -n --load-name 

- 出力ファイルのファイル名を指定します。
- デフォルトではソースファイル名（複数の場合は最初のファイル）を元にファイル名が決定されますが、このオプションの指定を優先します。
- 出力ファイルの形式が [MZT 形式](/output/output.md#mzt-形式) 、[T88 形式](/output/output.md#t88-形式)の場合で有効です。


### -f --fill

- [生成コードの配置](/allocate.md), [`ALIGN`](/directive/directive.md#align), [`DS`](/directive/directive.md#ds) で領域を埋める際の値を指定します。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- このオプションを指定しない場合、255 を使用します。
- [`ALIGN`](/directive/directive.md#align), [`DS`](/directive/directive.md#ds) で個別に値が指定されている場合は、そちらの値が使用されます。

### -R --r800 

R800 をターゲットとしてアセンブルします。

- MUL 命令がアセンブル可能になる
- リストファイルの実行クロック（TStates）が R800 のものに変わる

### -l

- [リストファイル](/output/output.md#リストファイル)を出力します。
- ソースファイル名（複数指定した場合は最初のファイル名）と同じフォルダに、ソースファイルの拡張子を `.lst` としたファイルです。
- ファイル名を指定したい場合は次の --list オプションを使用します。

### --lst

- 出力するリストファイル名を指定します。
- `--list` を指定した場合、`-l` は不要です。

### -s, --sym

- [シンボルファイル](/output/output.md#シンボルファイル)を出力します。
- `-s`, `--sym` の扱いは `-l`, `--lst` と同様です。

### -m, --map

- [マップファイル](/output/output.md#マップファイル)を出力します。
- `-m`, `--map` の扱いは `-l`, `--lst` と同様です。


### -a, --auto-proc

- ailz80asm からソースファイルを移行する際に使用します。
- `"."` で始まるラベルとを自動的に  [`PROC`](/directive/directive.md#proc) に変換してからアセンブルします。
- 動作については次項を参照ください。

## auto-proc 動作

### PROC 範囲開始と判定する文

- ラベルのみの文
- ラベル付きの文
    - Z80/R800 命令文
    - DB/DW/DD 文
    - DS 文
    - MACRO 呼出し文
    - REPT 文

### PROC 範囲終了と判定する文

- `"."` で始まらないラベルのみの文
- `"."` で始まらないラベル付きの文
    - Z80/R800 命令文
    - DB/DW/DD 文
    - DS 文
    - MACRO 呼出し文
    - REPT 文
- PROC 文
- MACRO 文
- FUNC/FUNCTION 文
- ENUM 文

### PROC 変換条件

開始、終了範囲に `.` で始まるラベルを含まない場合は PROC 文への変換は行いません。

### 適用例

<div class="iblock" style="width:15em"><pre><code>addr1: nop
.local1: nop
.local2: nop

addr2:
.local1: nop

addr3:nop </code></pre></div>

<div class="iblock top center"><br><i class="fa fa-arrow-right" aria-hidden="true"></i></div>

<div class="iblock" style="width:15em"><pre><code>add1 proc \ nop
.local1: nop
.local2: nop \ endp

addr2 proc
.local1: nop \ endp

addr3: nop</code></pre></div>