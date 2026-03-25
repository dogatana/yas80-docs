## 生成コードの配置

生成コードの配置には [`ORG`](/directive/directive.md#ORG)疑似命令を使用します。

yas80 では`ORG`から次の`ORG`までを "セグメント" としてまとめ、
出力ファイル中での配置を決定します。

- `ORG`の第 1 引数で指定されたアドレスがセグメント内の最初の命令のベースアドレスになります。
- `ORG`の第 2 引数を指定しない場合、もしくは`ABS`を指定した場合は "絶対セグメント" となり、
`ORG`で指定されたアドレスがセグメントのアドレスになります。
- `ORG`の第 2 引数に`REL`を指定した場合は "相対セグメント" となり、先行する絶対セグメントの一部として配置されます。
- 相対セグメントに先行する絶対セグメントがない場合、`ORG 0`の絶対セグメントが指定されたものとして処理します。
- 絶対セグメントのアドレス範囲は重複できません。エラーになります。
- 相対セグメントは配置上はデータとして扱うため、他のセグメントとアドレス範囲は重複してもエラーになりません

<table style="width:47em">
<thead>
<tr><th>ORG 指定</th><th>セグメント種別</th><th>配置先</th></tr>
</thead>
<tbody>
<tr><td><code>ORG addr     </code></td><td class="center" rowspan="2">絶対</td><td rowspan="2"><code>addr</code></td></tr>
<tr><td><code>ORG addr, ABS</code></td></tr>
<tr><td><code>ORG addr, REL</code></td><td class="center"            >相対</td><td>先行する絶対・相対セグメントの最終アドレス + 1</td></tr>
</tbody>
</table>

### ORG なし

`ORG`を指定していない場合、
`addr`として`0`が指定された（`ORG 0`）として扱います。

### ORG addr

`addr`を命令のベースアドレスとしてアセンブルし、
出力ファイルの配置（ロード）アドレスとして出力します（MZT, T88 の場合）。

### 複数 ORG（RELなし）

複数の `ORG` を指定した場合、
- それぞれ指定されたアドレスをベースアドレスとしてアセンブル
- セグメントのアドレスを`ORG`で指定されたアドレスとする
- セグメントをアドレス順でソート
- セグメントのアドレスを元に出力ファイル内での位置を決定し、

出力ファイルを生成します。

出力ファイルのロードアドレスはソート後の先頭セグメントのアドレスです。

セグメントのアドレ範囲が重複する場合はエラーととします。<br>
また、セグメント間にギャップがある場合、[`--fill`](/exec/option.md#f---fill)オプションで指定した値で埋めます。


```as
org $20
jp  $    ; アドレス $20 から配置なので jp $0020

org $00
jp  $    ; アドレス $00 から配置なので jp $0000

org $10
jp  $    ; アドレス $10 から配置なので jp $0010
```
&nbsp;<i class="fa fa-arrow-down">
```
0000: c3 00 00 ff ff ff ff ff  ff ff ff ff ff ff ff ff  ; jp $0000 0003-000f はギャップ
0010: c3 10 00 ff ff ff ff ff  ff ff ff ff ff ff ff ff  ; jp $0010 0013-001f はギャップ
0020: c3 20 00                                          ; jp $0020
```

### 複数 ORG（RELあり）

相対セグメントを使用する場合の例を示します。

```
org $20
jp  $          ; jp $0020

org $100, REL  ; 相対セグメント
top1:
jp  top1       ; jp $0100

org $100, REL  ; 相対セグメントは他のセグメントとアドレス重複可能
top2:
jp  top2       ; jp $0100

org $10        ; ロードアドレス $10 から配置
jp  $          ; jp $0010
```
&nbsp;<i class="fa fa-arrow-down">
```
; 絶対セグメント $0010 + gap
0010: c3 10 00 ff ff ff ff ff  ff ff ff ff ff ff ff ff ; jp $0010 0003-000f はギャップ
; 絶対セグメント $0020 + 相対セグメント $0100 x 2
0020: c3 20 00 c3 00 01 c3 00  01                      ; jp $0020
                                                       ; jp $0100
                                                       ; jp $0100

```

この場合、シンボルファイルは同じアドレスに対し複数のシンボルが出力されます。
```
0100 TOP1
0100 TOP2
```
<br>

次の例は`2000 - 20ff`の 256 バイトの領域に、
`sub1`もしくは`sub2`の どちらかをロードし、
`main`からは`call 2000`とすることで異なる処理を実行することを想定したものです。

```as
org $1200
main:
  ld a, 0
  call select_bank ; 2000 からの領域に sub1 or sub 2 をロード
  call $2000       ; sub1, sub2 でも同じ
  halt

select_bank proc
  ret
endp

; 
org $2000

; bank 0
org $2000, rel 
sub1 proc
  ; do something
  ret
endp
align 256

; bank 1
org $2000, rel
sub2 proc
  ; do something
  ret
endp
align 256
```


