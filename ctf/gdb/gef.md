<!-- TITLE: Gef -->
<!-- SUBTITLE: A quick summary of Gef -->

# GEF, GDB Enhanced Features

## インストール

他のGDBプラグインと同じように、Githubからクローンして、`~/.gdbinit`に1行追加するだけで良い

```sh
$ cd ~
$ git clone https://github.com/hugsy/gef.git
$ vim ~/.gdbinit
```

```vim
# 1行追加
source ~/gef/gef.py
```

**N.B** 他にpedaなどのプラグインがsourceされている場合、最後にsourceされたプラグインが有効になってしまう

このほかに[gef-extras](https://github.com/hugsy/gef-extras)といった、拡張機能も用意されている

## gef missing?

[missing](/ctf/gdb/gef/missing)

## 使い方など

作者の方がチュートリアルビデオを作っている

[video](https://youtu.be/KWG7prhH-ks){.youtube}

## コマンド

### telescope 

gdb-pedaのように、gefでも*telescope*コマンドが使える（正しくは*dereference*コマンドにaliasが貼られている）
pedaのように、`telescope 25`としてもスタックフレームは表示されない

代わりに、`telescope $esp l25`とすれば表示される

### context

*breakpoint*にひっかかると自動でこの画面が表示されるが、先述の*telescope*などでターミナル画面の上の方に行ってしまって、もう一度見たい！というときは、迷わずこのコマンドを打とう


```
gef➤  context
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────[ registers ]────
$eax   : 0xffffd248  →  "AAAA"
⋮
```


## gef-extras

### *x*コマンド上書き対策

公式で修正されたので削除



