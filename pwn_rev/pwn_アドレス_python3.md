# pwn 攻撃の書き方 python3


python3では、ユニコード文字列をうまく扱えないので、
常にバイト文字列で扱う。

`b'\x54\x9a'`と行った感じである。

これを攻撃に使うには、ソケット通信で`send`するか、
`echo`に`-e`をつけて、パイプで渡せばよい。

後者は以下のようなコマンドになる。

```
echo -e '\x54\x9a' | ./[問題ファイル]
```


## アドレス

アドレスはリトルエンディアンに直す必要があるので、


1. `struct`の`pack`を使う
以下のように`pI`関数を定義すれば便利。


```
import struct

def pI(addr):
    return struct.pack('<I', addr)
```

以下のように、アドレス（int）を引数で与えれば、
リトルエンディアンでバイト文字列を返してくれる

```
>>> pI(0x08048490)

b'\x90\x84\x04\x08'
```

2. pwntoolsの`p32`を使う

python3系のpwntoolsは[こちら](https://github.com/arthaud/python3-pwntools)から。
入れるのが大変。

```
from pwn import *
```

p32関数と同じ使い方ができる。

```
>>> p32(0x8888888)
b'\x88\x88\x88\x08'
```

対応するのが`u32`で、バイト列からint型への変換ができる。



## float型に変換

入力がfloat型で格納される場合がある。

その場合は文字列でアドレスを送っても無意味。

http://byte-off.com/golang/picoctf-enter-the-matrix-writeup/



```python
import struct

address = struct.unpack('<I', struct.pack('<f', addr_as_float))[0]
addr_as_float = struct.unpack('<f', struct.pack('<I', address))[0]
```



