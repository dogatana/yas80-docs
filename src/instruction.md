# Z80/R800 命令・TStates

## Z80

### Z80 命令 

- Z80 公開命令
    - [Z80 Family CPU User Manual](https://www.zilog.com/docs/z80/z80cpu_um.pdf) に記載の命令
- Z80 非公開命令
    - `LD/ADD/ADC/SUB/SBC/AND/AND/OR/XOR/CP/INC/DEC` 命令で `IXH/IXL/IYH/IYL` を使用するもの

### Z80 TStates

- Z80 公開命令  [Z80 Family CPU User Manual](https://www.zilog.com/docs/z80/z80cpu_um.pdf) による
- Z80 非公開命令  [The Undocumented Z80 Documented](http://www.z80.info/zip/z80-documented.pdf) による 

## R800

### R800 命令

- [R800ユーザーズマニュアル 暫定版](https://d4.princess.ne.jp/msx/datas/R800UM/) に記載の命令
- 表記は zilog 表記
- Z80 にない乗算命令は `mulub`, `muluw` とも `mul` を使用します。


| R800 命令表の表記 | yas80 での表記 |
| --                | --             |
| mulub a, r        | mul a, r       |
| muluw hl, ss      | mul hl, ss     |


### R800 TStates

- [R800ユーザーズマニュアル 暫定版](https://d4.princess.ne.jp/msx/datas/R800UM/) による

### 表記揺らぎ対応

- 本来の書式に加え、次の書式を許容します。
- IX とあるケースは IY にも適用されます。

| 本来の書式      | 揺らぎとして許容する書式 |
| --              | -- |
| JP (IX)         | JP (IX + 0)     |
| LD (IX + 0), op | LD (IX + 0), op |
| LD op, (IX + 0) | LD op, (IX + 0) |
| BIT n, (IX + 0) | BIT n, (IX)     |
| SET n, (IX + 0) | SET n, (IX)     |
| RES n, (IX + 0) | RES n, (IX)     |
| EX HL, DE       | EX DE, HL       |    
| EX AF, AF'      | EX AF, AF'      |
| EX (SP), HL     | EX HL, (SP)     |
| EX (SP), IX     | EX IX, (SP)     |
| SUB op          | SUB A, op       |
| AND op          | AND A, op       |
| OR op           | OR A, op        |
| XOR op          | XOR A, op       |
| CP op           | CP A, op        |