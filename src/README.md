# yas80 概要
<div class="right" style="position:relative; top:-1.5em">Last Update: {{ book.buildDate }}</div>

<div style="margin-top:1em">
yas80 - Yet Another Assembler for Z80 and R800 - は
Z80/R800 向けアブソリュートアセンブラです。
</div>


## マルチパスアセンブラ

多くのアセンブラでは、前方参照シンボル等の未解決シンボルが発生するケースの処理のため、
2 パスアセンブル方式が採用されていますが、yas80 はパス数可変のマルチパスアセンブラとしています。

yas80 のアセンブル処理は大きく分けて

![マルチパス](/images/マルチパス.svg)

の 4 工程に分かれており、評価工程は更に

- シンボル解決 
    - 未解決シンボルがなくなるまで最大 256 回繰り返し評価
- 生成コード確定
    - 生成コードが安定する（評価し生成コードが同一になる）まで、最大 16 回繰り返し評価

の 2 段階で構成しています。

現在のパス番号（1～）は[システム変数](/syntax/syntax.md#システム変数) `$PASS` で参照できます。

### プリプロセス

yas80 のプリプロセスは C言語のプリプロセスとは異なり構文解析の後、意味解析の前に実行します。
これは[`-a --auto-proc`](/exec/option.md#a---auto-proc)オプションを指定した場合の
[auto-proc 処理](/exec/option.md#auto-proc-処理)です。
