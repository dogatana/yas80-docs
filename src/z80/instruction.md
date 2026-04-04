# Z80/R800 命令・TStates


## Z80 命令 

- Z80 公開命令
    - [Z80 Family CPU User Manual](https://www.zilog.com/docs/z80/z80cpu_um.pdf) に記載の命令
    - `IN Flag, (C)` は `IN F, (C)` と表記します。
    - `C` は C レジスタですが、キャリーフラグが指定可な場所ではキャリーフラグとして評価します。
    - キャリーフラグとして `CY` も指定可能です。

- Z80 非公開命令
    - `LD/ADD/ADC/SUB/SBC/AND/AND/OR/XOR/CP/INC/DEC`命令で`IXH/IXL/IYH/IYL`を使用するもの

## Z80 TStates

- Z80 公開命令  [Z80 Family CPU User Manual](https://www.zilog.com/docs/z80/z80cpu_um.pdf) による
- Z80 非公開命令  [The Undocumented Z80 Documented](http://www.z80.info/zip/z80-documented.pdf) による 

## R800 命令

- [R800ユーザーズマニュアル 暫定版](https://d4.princess.ne.jp/msx/datas/R800UM/) に記載の命令
- 表記は zilog 表記です。
- 乗算命令 `mulub`, `muluw` はいずれも `mul` を使用します。


<table style="width:28em">
<thead>
<tr><th style="width:13em">R800 命令表の表記</th><th>yas80 での表記</th></tr>
</thead>
<tbodyu>
<tr><td><code>mulub a, r  </code></td><td><code>mul a, r  </code></td></tr> 
<tr><td><code>muluw hl, ss</code></td><td><code>mul hl, ss</code></td></tr> 
</tbody>
</table>


## R800 TStates

- [R800ユーザーズマニュアル 暫定版](https://d4.princess.ne.jp/msx/datas/R800UM/) による

## 表記揺らぎ対応

- 本来の書式に加え、次の書式を許容します。
- `IX`とある書式は`IY`にも適用します。


<table style="width:30em">
<thead>
<tr><th>本来の書式</th><th>揺らぎとして許容する書式</th></tr>
</thead>
<tbody>
<tr><td><code> JP (IX)         </code></td><td><code> JP (IX + 0)     </code></td></tr>
<tr><td><code> LD (IX + 0), op </code></td><td><code> LD (IX), op </code></td></tr>
<tr><td><code> LD op, (IX + 0) </code></td><td><code> LD op, (IX) </code></td></tr>
<tr><td><code> BIT n, (IX + 0) </code></td><td><code> BIT n, (IX)     </code></td></tr>
<tr><td><code> SET n, (IX + 0) </code></td><td><code> SET n, (IX)     </code></td></tr>
<tr><td><code> RES n, (IX + 0) </code></td><td><code> RES n, (IX)     </code></td></tr>
<tr><td><code> EX HL, DE       </code></td><td><code> EX DE, HL       </code></td></tr>    
<tr><td><code> EX AF, AF'      </code></td><td><code> EX AF', AF      </code></td></tr>
<tr><td><code> EX (SP), HL     </code></td><td><code> EX HL, (SP)     </code></td></tr>
<tr><td><code> EX (SP), IX     </code></td><td><code> EX IX, (SP)     </code></td></tr>
<tr><td><code> SUB op          </code></td><td><code> SUB A, op       </code></td></tr>
<tr><td><code> AND op          </code></td><td><code> AND A, op       </code></td></tr>
<tr><td><code> OR op           </code></td><td><code> OR A, op        </code></td></tr>
<tr><td><code> XOR op          </code></td><td><code> XOR A, op       </code></td></tr>
<tr><td><code> CP op           </code></td><td><code> CP A, op        </code></td></tr>
</tbody>
</table>

## 文字列、配列を数値として扱う{#ld-op2}

- 一部の命令でオペランドの数値として文字列、配列を数値として指定可能です。
    - 8bit `LD` 
    - 8bit 演算（`ADD/ADC/SUB/SBC,AND/OR/XR/CP`）
- 文字列、配列の数値化については"[中置演算子と型混合演算](/syntax/syntax.md#中置演算子と型混合演算)"を参照ください。
    - 長さ 1 の文字列
    - 要素数 1/2 の配列

```
    1  0000 3e 61                    [ 7]       ld a, 'a'
    2  0002 21 a0 82                 [10]       ld hl, 'あ'
    3  0005 3e 01                    [ 7]       ld a, [1]
    4  0007 21 02 01                 [10]       ld hl, [1, 2]
```