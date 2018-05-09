<!-- TITLE: Stack 5 -->
<!-- SUBTITLE: A quick summary of Stack 5 -->

# 問題

## 問題のパス

`/opt/protostar/bin/stack5`

## 問題のソースコード

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

非常にシンプルな仕上がり

# ファイルのコピー(scp)

```sh
(local) $ scp Protostar:/opt/protostar/bin/stack5 .
(local) $ ls | grep stack5
stack5
```

このように`scp`してローカルで基礎解析をしたほうが楽

# GDB解析

例によって[`gef`](https://github.com/hugsy/gef/)を使う