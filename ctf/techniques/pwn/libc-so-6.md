<!-- TITLE: libc.so.6 -->
<!-- SUBTITLE: A quick summary of Libc So 6 -->

# `patchelf`によるglibcの変更

[\[Linux\] 1つのホスト上の複数のglibcライブラリ gcc | CODE Q&amp;A [日本語]](https://code.i-harness.com/ja/q/ced4b)

```console
$ patchelf --set-rpath /path/to/new/libc.so.6 {elf_file_name}
```

もしくは

```console
$ patchelf --replace-needed libc.so.6 /path/to/new/libc.so.6 {elf_file_name}
```

のどちらか（試していないのでどちらが正しいかわからない）

