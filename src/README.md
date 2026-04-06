# yas80 概要
<div class="right" style="position:relative; top:-1.5em">Last Update: {{ book.buildDate }}</div>

<div style="margin-top:1em">
yas80 - Yet Another Assembler for Z80 and R800 - は
Z80/R800 向けアブソリュートアセンブラです。
</div>


## マルチパスアセンブラ

多くのアセンブラでは、前方参照シンボル等の未解決シンボルが発生するケースの処理のため、
2 パスアセンブル方式が採用されていますが、yas80 はパス数可変としてシンボルを解決する
マルチパスアセンブラとしています。

yas80 のアセンブル処理は大きく

![マルチパス](/images/マルチパス.svg)

の 4 工程に分かれており、評価工程は更に

- シンボル解決 
    - 未解決シンボルがなくなるまで最大 256 回繰り返し評価
    - [システム変数](/syntax/syntax.md#システム変数)`$STAGE2`が 0
- 生成コード確定
    - 生成コードが安定する（評価し生成コードが同一になる）まで、最大 16 回繰り返し評価
    - [システム変数](/syntax/syntax.md#システム変数)`$STAGE2`が 1
    - 2 パスアセンブラのパス 2 に相当します。

の 2 段階で構成しています。

現在のパス番号（1～）は[システム変数](/syntax/syntax.md#システム変数)`$PASS`で参照できます。


## マルチパスアセンブルと`$DEFINED`{#defined}

yas80 は[マルチパスアセンブラ](#マルチパスアセンブラ)のため、最低 2 回の評価を実行します。
このため、[組み込み関数](/syntax/syntax.md#組み込み関数)`$DEFINED`を使用して条件アセンブルする際には注意してください。


```
const v1 = $defined(x)
const v2 = $defined(v2)
const v3 = $defined(fwd)
const fwd = 1

info $fmt("pass %d v1 %d", $pass, v1)
info $fmt("pass %d v2 %d", $pass, v2)
info $fmt("pass %d v3 %d", $pass, v3)
```
アセンブル時の表示内容です。
```
temp\test.asm:6 [INFO] pass 1 v1 0
temp\test.asm:7 [INFO] pass 1 v2 0 
temp\test.asm:8 [INFO] pass 1 v3 0 
temp\test.asm:6 [INFO] pass 2 v1 0
temp\test.asm:7 [INFO] pass 2 v2 1 ; 0->1 へ変化
temp\test.asm:8 [INFO] pass 2 v3 1 ; 0->1 へ変換
```

- `v1`について`x` は未定義なので、`$defined(x)`は常に 0 です
- `v2`について pass 1 で`$defined(v2)`を評価する時点では未定義で 0 となり`v2`に設定しているため、pass 2 では定義済みとなり、`$defined(v2)`は 1 に変わります。
- `v3`について pass 1 で`$defined(fwd)`を評価する時点では`fwd`を前方参照しているため未定義で 0。 pass 2 時点では定義済みのため`$defined(v2)`は 1 に変わります。

## プリプロセス

yas80 のプリプロセスは C言語のプリプロセスとは異なり構文解析の後、意味解析の前に実行します。<br>
これは[`-a --auto-proc`](/exec/option.md#a---auto-proc)オプションを指定した場合に実行され、
構文解析工程の出力である抽象構文木を処理する[auto-proc 処理](/exec/option.md#auto-proc-処理)です。
