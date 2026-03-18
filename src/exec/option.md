## オプション

### -I

- [`INCLUDE`](/directive/directive.md#include),[`INCBIN`](/directive/directive.md#incbin),[`CHARMAP`](/directive/directive.md#charmap) がファイルをロードするフォルダを指定します。
- `-Idir`, `-I dir` のようにフォルダを指定します。
- `-Idir1 -Idir2` のように複数回指定できます
- `-Idir1,dir2` のように "`,`"で区切って複数のフォルダを指定可能です。
- 検索順は次のとおりです
    1. 疑似命令を含むソースファイルと同じフォルダ
    2. -I で最初に指定したフォルダ
    3. -I で次に指定フォルダ
    4. 以降 -I の指定数まで繰り返し

```
-I dir1 -Idir2,dir3
```
### -D

- シンボルを定義します。ソースファイル内で [CONST/EQU](directive/directive.md#constequ)  を使用したのと同じ効果です。
- `-Dname=value`もしくは`-D name=value`のように名前と値を指定します。
- 値として指定できるのは数値のみです。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- `-Dname` のように名前のみ指定した場合は `-Dname=1` と指定したものとして扱います。
- `-Dname1=1 -Dname2=2` のように複数回指定できます。
- `-Dname1=1,name2=2` のように "`,`2 で区切って複数のシンボルを定義可能です。

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

### -f --fill

- [ORG](directive/directive.md#org) - [生成コードの配置](allocate.md), [ALIGN](directive/directive.md#align), [DS](directive/directive.md#ds) で領域を埋める際の値を指定します。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- このオプションを指定しない場合、255 を使用します。
- [ALIGN](directive/directive.md#align), [DS](directive/directive.md#ds) で個別に値が指定されている場合は、そちらの値が使用されます。

### -l

- [リストファイル](output/output.md#リストファイル)を出力します。
- ソースファイル名（複数指定した場合は最初のファイル名）と同じフォルダに、ソースファイルの拡張子を `.lst` としたファイルです。
- ファイル名を指定したい場合は次の --list オプションを使用します。

### --lst

- 出力するリストファイル名を指定します。
- `--list` を指定した場合、`-l` は不要です。

### -s, --sym

- [シンボルファイル](output/output.md#シンボルファイル)を出力します。
- `-s`, `--sym` の扱いは `-l`, `--lst` と同様です。

### -m, --map

- [マップファイル](output/output.md#マップファイル)を出力します。
- `-m`, `--map` の扱いは `-l`, `--lst` と同様です。


### -a, --auto-proc

- 通常のラベルと `.` で始まるラベルとを自動的に  [PROC](directive/directive.md#proc) に変換してからアセンブルします。
- ailz80asm からソースファイルを移行する際に使用します。


```
addr1: 
.local1:
.local2:

addr2:
.local1:
.local2:
```
&nbsp;⇩
```
add1 proc
.local1
.local2
endp

addr2 proc
.local1
.local2
endp
```
