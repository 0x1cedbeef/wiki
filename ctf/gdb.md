<!-- TITLE: Gdb -->
<!-- SUBTITLE: A quick summary of Gdb -->

# GDB
gdbとはGNU Debuggerのことで、C言語やC++で書かれたプログラムの実行ファイルを、対話型で逆アセンブル(disassemble)して表示してくれる
CTFではpwnやrevなどで使われることが多いが、そのまま生で使用するとUI的につらいため、拡張プラグインを用いることが一般的である
有名なものとしては[gdb-peda](https://github.com/longld/peda)や[pwndbg](https://github.com/pwndbg/pwndbg)などがある
ここでは、現在私が使っているGDB Enhanced Featuresこと、[gef](https://github.com/hugsy/gef)について自分なりに使い方をまとめる

## はじめに

`~/.gdbinit`の基本設定など

[gdbinit](/ctf/gdb/gdbinit)

## gef

[gef](/ctf/gdb/gef)

