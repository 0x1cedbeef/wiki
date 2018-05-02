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

## 使い方など

作者の方がチュートリアルビデオを作っている

[video](https://youtu.be/KWG7prhH-ks){.youtube}

## gef-extras

