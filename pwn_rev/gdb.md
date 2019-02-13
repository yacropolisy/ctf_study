# デバッグ

参考：katagaitai勉強会資料2 スライド124~

https://speakerdeck.com/bata_24/katagaitai-ctf-number-2?slide=124





端末を3つ使う。



## 端末1

```
# main.sh
gdbserver localhost:1025 ./file
```



```bash
chmod +x main.sh
socat TCP-LISTEN:1234,reuseaddr,fork EXEC:"./main.sh"
```



## 端末2

攻撃

```
nc localhost 1234
```



## 端末3

```
# cmd
file ./file
target remote localhost:1025
b *0x0804841C	# ret のアドレスにすると良い
# set follow-fork-mode parent 
c
```

```
gdb ./file -q -x cmd
```



エラーが出る場合は、set follow-fork-mode parent を設定する。