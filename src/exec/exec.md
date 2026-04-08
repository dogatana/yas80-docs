# yas80 実行

## コマンド書式

```
> yas80 -h
Usage: yas80 [options] file [file...]
Options:
  -I, --I strings          directories to search for include/incbin/charmap
  -D, --D strings          define numeric constants
  -o, --output string      output file name
      --mzt                output with MZT format
      --t88                output with T88 format
  -e, --entry int          program start address (default -1)
  -n, --load-name string   load file name
  -a, --auto-proc          generate PROC from normal label
  -f, --fill int           filler for DS and Segment Gap (default 255)
  -R, --R800               assmble for R800
  -l, --l                  generate list file
      --lst string         lst filename
  -m, --m                  generate map file
      --map string         map filename
  -s, --s                  generate symbol file
      --sym string         symbol filename
  -v, --version            print version
```

### オプション(option)

- オプションは大文字・小文字を区別します。
- オプションは引数がないか、引数の 1 つをとるかのいずれかです。
    - 引数を取らないオプション `--mzt` など
    - 引数を 1 つとるオプション `-I` など
- 複数の引数を取るオプションはありませんが、複数の項目を取るものがあります（`strings`）
    - 複数回指定可能なオプション 
    - 引数は 1 つで、`","` で複数の項目のオプション

### ソースファイル(file)

- オプションの次にソースファイルを指定します。
- ソースファイルは複数指定可能です。

## オプション

### -I

- [`INCLUDE`](/directive/directive.md#include),[`INCBIN`](/directive/directive.md#incbin),[`CHARMAP`](/directive/directive.md#charmap) がファイルをロードするフォルダを指定します。
- `-Idir`, `-I dir` のようにフォルダを指定します。
- `-Idir1 -Idir2` のように複数回指定できます
- さらに`-Idir1,dir2` のように `","`で区切って複数のフォルダを指定可能です。
- 検索順は次のとおりです
    1. ソースファイルと同じフォルダ
    2. `-I`で最初に指定したフォルダ
    3. `-I`で次に指定フォルダ
    4. 以降`-I`の指定数まで繰り返し

```
-I dir1 -Idir2,dir3
```
### -D

- シンボルを定義します。ソースファイル内で [`CONST/EQU`](/directive/directive.md#constequ)  を使用したのと同じ効果です。
- `-Dname=value`もしくは`-D name=value`のように名前と値を指定します。
- 値として指定できるのは数値のみです。
- 10進数の他、`0x, 0o`のプレフィックスで、16進数、8進数も指定可能です。
- `-Dname` のように名前のみ指定した場合は `-Dname=1` と指定したものとして扱います。
- `-Dname1=1 -Dname2=2` のように複数回指定できます。
- `-Dname1=1,name2=2` のように `","` で区切って複数のシンボルを定義可能です。

```
-D name=1,name2 -Dname3=3
```

### -o --output

- 出力ファイル名を指定します（※）
- 出力ファイル名の拡張子に応じた出力形式で出力します。
    - 拡張子 `.mzt` ･･･ [MZT 形式](/output/output.md#mzt-形式)
    - 拡張子 `.t88` ･･･ [T88 形式](/output/output.md#t88-形式)
    - これ以外の拡張子
        - `--mzt` 指定 ･･･ MZT 形式
        - `--t88` 指定 ･･･ T88 形式
        - 指定なし･･･ [BIN 形式](/output/output.md#bin-形式)

### --mzt

- MZT 形式で出力します。
- `-o --output` で指定した拡張子が t88 の場合、エラーとします。

### --t88

- T88 形式で出力します
- `-o --output` で指定した拡張子が mzt の場合、エラーとします。

### オプション組み合わせ

<table style="width:35em">
<thead>
<tr>
  <th><code>-o</code>拡張子</th><th style="width:7em"><code>--mzt</code></th><th style="width:7em"><code>--t88</code></th><th>出力形式</th>
</tr>
</thead>
<tbody>
<tr><td rowspan="3"><code>.mzt</code></td><td></td><td></td><td>MZT   </td></tr>
<tr>  <td class="center">✓ </td><td class="center">  </td><td>MZT   </td></tr>
<tr>  <td class="center">n/a</td><td class="center">✓</td><td>エラー</td></tr>

<tr><td rowspan="3"><code>.t88</code></td><td></td><td></td><td>T88   </td></tr>
<tr>  <td class="center">✓</td><td class="center">n/a</td><td>エラー</td></tr>
<tr>  <td class="center">  </td><td class="center">✓ </td><td>T88   </td></tr>

<tr><td rowspan="3"><code>.mzt .t88</code>以外</td><td></td><td></td><td>BIN</td></tr>
<tr>  <td class="center">✓</td><td class="center">  </td><td>MZT</td></tr>
<tr>  <td class="center">  </td><td class="center">✓</td><td>T88</td></tr>
</tbody>
</table>



### -e --entry

- 出力ファイルの実行開始アドレスを指定します。
- ソースコードでは [`END`](/directive/directive.md#END),[`ENTRY`](/directive/directive.md#entry) で指定しますが、このオプションの指定を優先します。
- 出力ファイルの形式が [MZT 形式](/output/output.md#mzt-形式) の場合のみ有効です。

### -n --load-name 

- 出力ファイルの形式が [MZT 形式](/output/output.md#mzt-形式) 、[T88 形式](/output/output.md#t88-形式)の場合で有効です。
- 出力ファイル内に記載されるファイル名を指定します。
- デフォルトではソースファイル名（複数の場合は最初のファイル）を元にファイル名を決定しますが、このオプションの指定を優先します。


### -f --fill

- [生成コードの配置](/README.md#生成コードの配置),[`ALIGN`](/directive/directive.md#align),[`DS`](/directive/directive.md#ds) で領域を埋める際の値を指定します。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- このオプションを指定しない場合、255 を使用します。
- [`ALIGN`](/directive/directive.md#align),[`DS/DSB`](/directive/directive.md#ds),[`DSW`](/directive/directive.md#dsw) で個別に値が指定されている場合は、そちらの値が使用されます。

### -R --R800 

- Z80 でなく R800 をターゲットとしてアセンブルします。
    - [`MUL`](/z80/instruction.md#r800-命令) 命令がアセンブル可能
    - リストファイルの[実行クロック（TStates）](/z80/instruction.md#r800-tstates)表記が R800 のものに変わる

### -l

- [リストファイル](/output/output.md#リストファイル)を出力します。
- ソースファイル名（複数指定した場合は最初のファイル名）と同じフォルダに、ソースファイルの拡張子を `.lst` としたファイルです。
- ファイル名を指定したい場合は次の`--list`オプションを使用します。

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
- 通常のラベルと`"."`で始まるラベルとを自動的に[`PROC`](/directive/directive.md#proc) に変換してからアセンブルします。
- 処理内容については次項のとおりです。

## auto-proc 処理

__【注意】__ 本処理は実験的に実装されているものであり今後削除・修正する可能性があります。


### PROC 範囲開始と判定する文

- ラベルのみの文
- ラベル付きの文
    - Z80/R800 命令文
    - `DB/DW/DD`文
    - `DS`文
    - `MACRO`呼出し文
    - `REPT`文

### PROC 範囲終了と判定する文

- `"."` で始まらないラベルのみの文
- `"."` で始まらないラベル付きの文
    - Z80/R800 命令文
    - `DB/DW/DD`文
    - `DS`文
    - `MACRO`呼出し文
    - `REPT`文
- `PROC`文
- `MACRO`文
- `FUNC/FUNCTION`文
- `ENUM`文

### PROC 変換条件

開始、終了範囲に `.` で始まるラベルを含まない場合は PROC 文への変換は行いません。

### 適用例

<div class="iblock" style="width:15em"><pre><code>addr1: nop
.local1: nop
.local2: nop

addr2:
.local1: nop

addr3: nop </code></pre></div>

<div class="iblock top center"><br><i class="fa fa-arrow-right" aria-hidden="true"></i></div>

<div class="iblock" style="width:15em"><pre><code>add1 proc \ nop
.local1: nop
.local2: nop \ endp

addr2 proc
.local1: nop \ endp

addr3: nop</code></pre></div>

