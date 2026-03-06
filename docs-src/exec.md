# yas80 実行

## コマンド書式

```
> yas80 -h
Usage: yas80 [options] file [file...]
Options:
  -I, --I strings          directories to search for iclude
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
      --list string        list filename
  -m, --m                  generate map file
      --map string         map filename
  -s, --s                  generate symbol file
      --sym string         symbol filename
  -v, --version            print version
```

## オプション

### -I

- `INCLUDE`, `INCBIN`, `CHARMAP` でファイルをロードする際、ファイルがソースファイルと同じ場所にない場合に、読み込むフォルダを指定します
- `-Idir`, `-I dir` のようにフォルダを指定します。
- `-Idir1 -Idir2` のように複数回指定できます。
- `-Idir1,dir2` のように `,` で区切って複数のフォルダを指定可能です。

```
-I dir1 -Idir2,dir3
```
### -D

- シンボルを定義します。
- `-Dname=value`, `-D name=value` のように名前と値を指定します。
- 値として指定できるのは数値のみです。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- `-Dname` のように名前のみの場合指定した場合は `-Dname=1` と `1` を指定したものと扱います。
- `-Dname1=1 -Dname2=2` のように複数回指定できます。
- `-Dname1=1,name2=2` のように `,` で区切って複数のシンボルを定義可能です。

```
-D name=1,name2 -Dname3=3
```

### -o --output

- 出力ファイル名を指定します（※）
- 出力ファイル名の拡張子に応じた出力形式で出力します。
    - 拡張子 `.mzt` ･･･ MZT 形式（`--mzt` 指定）
    - 拡張子 `.t88` ･･･ T88 形式（`--t88` 指定）
    - これ以外の拡張子 ･･･ BIN 形式(デフォルト)
- 出力ファイル名と同時に `--mzt` もしくは `--t88` を指定した場合、拡張子に関わらず、その形式指定が優先されます。

※ アセンブル結果はデフォルトではソースファイル名と同じフォルダに、ソースファイルの拡張子を出力形式に応じたものに変更したファイルへ出力します。

### --mzt

- MZT 形式で出力します

### --t88

- T88 形式で出力します


### -f --fill

- `DS/DSW/DSB`, `ALIGN`, `ORG` で埋める値を指定します。
- 10進数の他、0x, 0o のプレフィックスで、16進数、8進数も指定可能です。
- このオプションを指定しない場合、255 を使用します。

### -l

- リストファイルを出力します。
- デフォルトではソースファイル名と同じフォルダに、ソースファイルの拡張子を `.lst` としたファイルです。

### --list

- 出力するリストファイル名を指定します。
- `--list` を指定した場合、`-l` は不要です。

### -m, --map

- マップファイルを出力します。
- `-m` でデフォルトのマップファイル（拡張子 `.map`）へ出力します。
- `--map file` で `file` へ出力します。

### -s, --sym

- シンボルファイルを出力します。
- `-s` でデフォルトのシンボイルファイル（拡張子 `.sym`）へ出力します。
- `--sym file` で `file` へ出力します。

### -a, --auto-proc

- 通常のラベルと `.` で始まるラベルとを自動的に `PROC` に変換してからアセンブルします。
- ailz80asm からの移行のための機能です。

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
