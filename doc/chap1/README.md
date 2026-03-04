# yas80 実行

## コマンド書式

```
> yas80 -h
Usage: yas80 [options] file [file...]
Options:
  -I, --I strings       directories to search for iclude
  -D, --D strings       define constants
  -o, --output string   output file name
  -f, --fill int        filler for DS and Segment Gap (default 255)
      --mzt             output with MZT format
      --t88             output with T88 format
  -l, --l               generate list file
      --list string     list filename
  -m, --m               generate map file
      --map string      map filename
  -s, --s               generate symbol file
      --sym string      symbol filename
      --auto-proc       generate PROC from normal label
  -R, --R800            assmble for R800
  -v, --version         print version
      --stdin           assemble stdin
      --astdebug int    debug level for Parse
      --yydebug int     debug level for go-yacc
      --evaldebug int   debug level for Evaluator
      --listdebug int   debug level for Lister
  -a, --arg             assemble Args[0]
```

## オプション

### -I

- `INCLUDE`, `INCBIN`, `CHARMAP` でファイルをロードする際、ファイルがソースファイルと同じ場所にない場合に、読み込むフォルダを指定します
- `-Idir`, `-I dir` のようにフォルダを指定します。
- `-Idir1 -Idir2` のように複数回指定できます。
- `-Idir1,dir2` のように `,` で区切って複数のフォルダを指定可能です。

### -D

- シンボルを定義します。
- `-Dname=value`, `-D name=value` のように名前と値を指定します。
- 値として指定できるのは数値のみです。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- `-Dname` のように名前のみの場合指定した場合は `-Dname=1` と `1` を指定したものと扱います。
- `-Dname1=1 -Dname2=2` のように複数回指定できます。
- `-Dname1=1,name2=2` のように `,` で区切って複数のシンボルを定義可能です。


### -o

- 出力ファイル名を指定します（※）
- 出力ファイル名の拡張子に応じた出力形式で出力します。
    - 拡張子 `.mzt` ･･･ MZT 形式（`--mzt` 指定）
    - 拡張子 `.t88` ･･･ T88 形式（`--t88` 指定）
    - これ以外の拡張子 ･･･ BIN 形式(デフォルト)
- 出力ファイル名と同時に `--mzt` もしくは `--t88` を指定した場合、拡張子に関わらず、その形式指定が優先されます。

※ アセンブル結果はデフォルトではソースファイル名と同じフォルダに、ソースファイルの拡張子を出力形式に応じたものに変更したファイルへ出力します。

### -f

- `DS/DSW/DSB`, `ALIGN`, `ORG` で埋める値を指定します。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。

### -l

- リストファイルを出力します。
- デフォルトではソースファイル名と同じフォルダに、ソースファイルの拡張子を `.lst` としたファイルです。

### --list

- 出力するリストファイル名を指定します。
- `--list` を指定した場合、`-l` は不要です。

### -m, --map

- マップファイルを出力します。
- `-m` でデフォルトのマップファイル（拡張子 `.map`）へ、`--map` で指定されてファイルへ出力します。

### -s, --sym

- シンボルファイルを出力します。
- `-s` でデフォルトのシンボイルファイル（拡張子 `.sym`）へ、`--sym` で指定されたファイルへ出力します。

### --auto-proc

- 通常のラベルと `.` で始まるラベルとを自動的に `PROC` に変換してからアセンブルします。

<div style="display:inline-block; width:45%;vertical-align:top">
<pre stye="padding:0"><code>addr1: 
.local1:
.local2:

addr2:
.local1:
.local2:
</code></pre>
</div>
<div style="display:inline-block;vertical-align:top">
<br><br>⇒
</div>
<div style="display:inline-block; width:45%"><pre stye="padding:!0;margin:!0;"><code>addr1 proc 
.local1:
.local2:
endp

addr2 proc
.local1:
.local2:
endp
</code></pre>
</div>



    
### ソースファイル

- ソースファイルは複数指定可能
- 複数指定した場合、それらを結合した1つのファイルとしてアセンブルする

## ソースファイル

### エンコーディング

- エンコーディングは utf-8（BOM の有無は問わない）
- utf-8 として不正な文字を検出した場合、ShiftJISとして扱い、utf-8 へのエンコディーング変換を試みる。
- 変換が失敗した場合、その旨表示するが、アセンブルは継続する

### 行

- ソースファイルは行単位でアセンブル処理を進める
- 物理行（行頭から改行記号までのテキスト）と論理行（アセンブラが1行として処理するもの）の対応
    - 1:1 
    - 1:n マルチステートメント
    - n:1 継続行


### マルチステートメント

- `\` でマルチステートメント記載可能
- エラー表示等は物理行単位で行う

例）
`ld a,(hl) \ inc hl` は

```
ld a,(hl)
inc hl
```
と同等。


### 継続行

- 物理行の行末が `\` の場合（`\` の直後が改行） の場合、次行と合わせて1行として扱う

例）
```
ld \
a \
, \
b \
```
は `ld a, b` と同等

### 行の種類

- 行は次のいずれか
    - 空白行
    - コメント行（コメントのみ含む）
    - Z80 / R800 命令行
    - 疑似命令（アセンブラディレクティブ）行

### コメント

- `;` 以降行末までコメントで、アセンブル処理の対象外となる
