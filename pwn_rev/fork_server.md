# fork-server型

socket→bind→listen→accept→forkの順番で行われるserverのこと



xinetd型と言われるserverとは違い、標準入出力とは繋がっていない。
socketディスクリプタを使用して入出力が行われるため、system("/bin/sh")だけを起動しても全く意味がない。
標準入出力にも自分が送っている内容を影響させるにはdup2などを使って自分が使っているsocketディスクリプタを繋げてあげる必要がある。



## send

send(sockfd, buf, len);という形。

第一引数にソケット、第二引数に送る文字列のアドレス、第三引数に送る長さ。



## recv

recv(sockfd, buf, len);という形。

引数の意味はsendと同様である。