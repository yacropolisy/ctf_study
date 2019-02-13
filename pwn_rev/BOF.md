# BOF ~Buffer over flow ~



## stack pivot

katagaitai 勉強会 #2 p97~参照

以下のようにオーバーフローさせる。



```
read@plt
p3ret
fd
.bss addr
size
pop ebp; ret
.bss addr -4
leave; ret
```

まずreadに飛んで、.bssに続きのropを書き込む

→ p3retを介して、pop ebp; ret により、ebp が .bss -4にセットされる

→ leave = (mov esp, ebp; pop ebp) 、mov esp, ebp でesp が.bss -4にセットされ pop ebp によってesp が .bssになる



なお、大抵のバイナリではデフォルトでleave; retによって関数を抜けるので、 pop ebp;ret のガジェットは用いなくて良い場合が多い。

その場合は以下

```
.bss addr -4
read@plt (関数のリターンアドレスが格納されている場所)
p3ret ガジェット
fd
.bss addr
size
leave; ret ガジェット
```

