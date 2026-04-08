# 移植メモ

## メモについて

自作および公開されているアセンブラソースを yas80 用に修正する際に必要になった事柄のメモです。

修正後のソースファイルで同一のバイナリが生成されるまでは確認していますが、
修正はあくまで該当ソースファイルで使用している機能範囲に限定されており、
<span class="text-danger"><b>移植元のアセンブラの仕様を網羅するものではない</b></span>ため、
参考レベルの情報として"メモ"と呼ぶものです。

## z80as

### 変更点

- `@@, @n` のラベルを適当な名前に置き換えるか、`-a`オプションの利用を検討する。
- `DB/DW/DS` のラベルの `:` を削除する
- `:` 演算子を数式に置き換える
- ギャップが `0` のため、オプション `-f 0` を指定する

### patch.py

```python
import re
import sys
from collections import defaultdict


class Labels:
    def __init__(self):
        self.map = defaultdict(list)

    def append(self, name, line_number):
        self.map[name].append(line_number)

    def __len__(self):
        count = 0
        for v in self.map.values():
            count += len(v)
        return count

    def __in__(self, name):
        return name in self.map

    def replace(self, n, name):
        name = name.lower()
        if name == "@f":
            num = self.forward(n, "@@")
            return f"L___{num}"
        elif name == "@b":
            num = self.backward(n, "@@")
            return f"L___{num}"
        elif name.endswith("f"):
            num = self.forward(n, name[:-1])
            return f"L_{name[1]}_{num}"
        elif name.endswith("b"):
            num = self.backward(n, name[:-1])
            return f"L_{name[1]}_{num}"
        else:
            raise ValueError(f"Invalid label reference: {name}:{n}")

    def forward(self, n, name):
        lst = self.map.get(name, None)
        if lst is None:
            raise ValueError(f"Label {name}:{n} not found")

        for num in lst:
            if num >= n:
                return num
        raise ValueError(f"Label {name}:{n} not found")

    def backward(self, n, name):
        lst = self.map.get(name, None)
        if lst is None:
            raise ValueError(f"Label {name}:{n} not found")

        for num in reversed(lst):
            if num <= n:
                return num
        raise ValueError(f"Label {name}:{n} not found")


def main(infile, outfile):
    try:
        lines = read_lines(infile)
    except Exception as e:
        print(f"Error reading file: {e}")
        exit(1)
    labels = get_labels(lines)
    print(len(labels), "labels")
    new_lines = patch(lines, labels)
    with open(outfile, "w") as f:
        f.writelines(new_lines)

def read_lines(file) -> list[str]:
    try:
        with open(file, encoding="utf-8") as f:
            return f.readlines()
    except UnicodeDecodeError:
        with open(file, encoding="cp932") as f:
            return f.readlines()
            

def get_labels(lines) -> Labels:
    map = Labels()

    for n, line in enumerate(lines, start=1):
        m = re.match(r"(@[@\d])\s*:", line)
        if m is not None:
            map.append(m.group(1), n)
    return map


def patch(lines, labels) -> list[str]:
    new_lines = []
    for n, line in enumerate(lines, start=1):
        # ラベル定義
        new_line = re.sub(
            r"(@[@\d])\s*:", lambda m: f"L{m.group(1)}_{n}:".replace("@", "_"), line
        )
        # ラベル参照
        new_line = re.sub(
            r"@\d?[fb]",
            lambda m: labels.replace(n, m.group(0)),
            new_line,
            flags=re.IGNORECASE,
        )
        # DB/DW/DS の前の : を削除
        new_line = re.sub(r":(\s*(?:db|dw|ds))", r" \1", new_line, flags=re.IGNORECASE)
        # コロン演算子の置き換え
        new_line = re.sub(
            r"(\d+)\s*:\s*(\d+)", r"(\1 * 256) + \2", new_line, flags=re.IGNORECASE
        )
        # BININCLUDE -> INCBIN は機能追加したので変更不要
        # new_line = re.sub(r"BINCLUDE", "INCBIN", new_line, flags=re.IGNORECASE)
        new_lines.append(new_line)
    return new_lines


if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: patch.py input output")
        exit(1)
    main(*sys.argv[1:])
```

## AILZ80ASM

### 変更点

- `#if/#elif/#else/#endif`の`#`を削除
- `.local`単独行に`:`を追加
- `.local`が多い場合は `-a` オプションの利用を検討する
- `exists`は組み込み関数`$defined`が使用できないか検討
- `.@H,@L` は組み込み関数`$H(), $L()`に置き換える
- charmap 定義の`@`を`_`に変更
- charmap 適用の`@`を`_` に変更
- charmap 適用方法修正 ':' -> 関数呼出し形式
- charmap は前方参照できないので、必要に応じて定義位置を変更
- `db/dw/ds` のラベルに`:`がある場合削除
- `include "file", b`を`incbin`に変更
- `function name() => expression`の`=>`を削除
- macro 定義内の`.name`を`@name`に変更
- `REPT ENDM`は`REPT ENDR`に変更
- `REPT m LAST -n`は`EXITM IF`利用へ書き換える

### patch.py

- 上の変更点の一部に対応したもの

```python
import re
import sys

def main(infile, outfile):
    try:
        lines = read_lines(infile)
    except Exception as e:
        print(f"Error reading file: {e}")
        exit(1)

    new_lines = patch(lines)
    with open(outfile, "w", encoding="utf-8") as f:
        f.writelines(new_lines)

def read_lines(file) -> list[str]:
    try:
        with open(file, encoding="utf-8") as f:
            return f.readlines()
    except UnicodeDecodeError:
        with open(file, encoding="cp932") as f:
            return f.readlines()
            

def patch(lines:list[str]) -> list[str]:
    new_lines = []
    for line in lines:
        line = delete_pound(line)
        line = exists_to_defined(line)
        line = add_colon(line)
        line = add_colon2(line)
        line = remove_arrow(line)
        line = charmap(line)
        line = incbin(line)
        line = remove_color(line)
        new_lines.append(line)
    return new_lines

# if/elif/else/endifの # prefix を削除
def delete_pound(line:str) ->str:
    return re.sub(r"#(if|elif|else|endif)", r"\1", line, flags=re.IGNORECASE)

# exists name を $defined(name) に置換
def exists_to_defined(line:str) -> str:
    return re.sub(r"\sexists\s+(\w+)\s*$", r" $defined(\1)" "\n", line, flags=re.IGNORECASE)

# .local を .local: に置換
def add_colon(line:str)->str:
    m = re.match(r"\s*(\.\w+)\s*(?:;|$)", line)
    if m is None:
        return line
    return f"{m.group(1)}:\n"

# .local instruction => .local: instruction
def add_colon2(line:str)->str:
    m = re.match(r"\s*(\.\w+)\s+(\w+)\s(.*)", line, flags=re.IGNORECASE)
    if m is None:
        return line
    if m.group(2).upper() in ["DB", "DW", "DS"]:
        return line
    return f"{m.group(1)}:\t{m.group(2)} {m.group(3)}\n"

# charmap
def charmap(line:str) -> str:
    line = re.sub(r"charmap(\s+)@", r"charmap\1_", line, flags=re.IGNORECASE)
    return re.sub(r'@(\w+):("[^"]*")', r"_\1(\2)", line)

# incbin
def incbin(line:str) -> str:
    return re.sub(r'include\s+("[^"]+")\s*,\s*B\s*$', "INCBIN \\1\n", line, flags=re.IGNORECASE)

# function name() => expression の => を削除
def remove_arrow(line:str) -> str:
    return re.sub(r"\s=>\s", " ", line)

# db/dw のラベルの: を削除
def remove_color(line:str) -> str:
    return re.sub(r":(\s*)(db|dw|ds)(\s+)", r" \1\2\3", line, flags=re.IGNORECASE)

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: patch.py input output")
        exit(1)
    main(*sys.argv[1:])
```