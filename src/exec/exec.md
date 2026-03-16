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