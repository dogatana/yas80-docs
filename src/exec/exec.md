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
- オプションは引数の有無によって次の 2 通りあります。
    - 引数を取らないオプション `--mzt` など
    - 引数を 1 つとるオプション `-I` など
- 複数の引数を取るオプションはありませんが、次の方法で複数の項目を指定可能です。
    - 複数回指定可能なオプション 
    - 引数は 1 つで、`","` で複数の項目のオプション

### ソースファイル(file)

- オプションの次にソースファイルを指定します。
- ソースファイルは複数指定可能です。

