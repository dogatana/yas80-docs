## エラー・警告・情報

yas80 は次の分類に応じてメッセージを表示・出力します。
<table>
<thead>
<tr><th>分 類</th><th>表 記</th><th>内 容</th></tr>
<tbody>
<tr>
    <td class="nowrap">エラー</td>
    <td>ERR</td>
    <td>yas80 が出力するエラーと<code>ERROR</code>疑似命令による出力があります。<br>
    エラーが発生する工程によって、アセンブルを継続するかどうか、出力ファイルを生成するかどうかが決まります。</td>
</tr>
<tr>
    <td>警告</td>
    <td>WARN</td>
    <td>型変換に関するものなど。<br>
    <code>WARN</code>疑似命令による出力も可能です。
    警告があってもアセンブルは継続し、指定された出力ファイルも生成します。</td>
</tr>
<tr>
    <td>情報</td>
    <td>INFO</td>
    <td><code>INFO</code>疑似命令を使用して出力したもの。<br>アセンブル、出力ファイルへのは影響ありません。</td>
</tr>
</tbody>
</table>

## 重複出力の集約

yas80 は[マルチパスアセンブラ](/README.md#マルチパスアセンブラ)のため、
エラー・警告・情報がパス毎に出力され同じ内容が重複します。
そのため、アセンブル終了時、およびリストファイル出力時は"同じ内容の情報"は 1 つに集約して出力するようにしています。

<div class="iblock" style="width:27em"><pre><code>const val = 123
info $fmt("value: %d", val)
ld a, val</code></pre>
</div>
<div class="iblock top center" style="width:1.5em"><br><i class="fa fa-arrow-right"></i></div>
<div class="iblock top" style="width:23em"><pre><code>msg.asm:2 [INFO] val: 123


</div>

各パス毎の値の変化を`INFO`等で出力したい場合は、[システム変数](/syntax/syntax.md#システム変数)`$PASS`を出力すると集約を回避することが可能です。

<div class="iblock" style="width:27em"><pre><code>const val= 123
info $fmt("pass: %d, val: %d", $pass, val)
ld a, val</code></pre>
</div>
<div class="iblock top center" style="width:1.5em"><br><i class="fa fa-arrow-right"></i></div>
<div class="iblock top" style="width:23em"><pre><code>msg.asm:2 [INFO] pass: 1, val: 123
msg.asm:2 [INFO] pass: 2, val: 123

</div>




## 異常発生時の挙動

###  字句解析・構文解析

- syntax error が発生した場合、アセンブルを中止します。
- 中止した場合出力ファイルは生成しません。
- リストファイル、シンボルファイル、マップファイルは出力しません。

###  プリプロセス

- この工程ではエラー、警告は発生しません。


### 評価：シンボル解決

- エラーが発生した場合、評価を中止します。
- 警告、情報の場合、評価を継続します。
- 評価を中止した場合出力ファイルは生成しません。
- 指定されていればリストファイルを出力します。
- シンボルファイル、マップファイルは出力しません。

### 評価：生成コード確定

- 規定回数評価してもコードが安定しない場合、警告を表示します。
- 出力ファイルを生成します。
- 指定されていればリストファイル、シンボルファイル、マップファイルを出力します。

### 出力ファイル生成

- 出力ファイルを生成します。
- OS 側でのエラーなど、出力ファイル生成時にエラーが発生すればその内容を表示します。
- 指定されていればリストファイル、シンボルファイル、マップファイルを出力します。


