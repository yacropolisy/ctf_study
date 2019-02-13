# Javascriptの難読化

難読化の手法に対して、大きく２種類の解読方法がある。



## 難読化1

難読化の手法には、`eval()` 関数を用いているものがある。

http://www.broadband-xp.com/obfuscation/crypt.html

```javascript
eval("alert('hello world')");
＝＞難読化
```

`eval()` 関数は、文字列を引数にとりその内容を実行するだけの関数である。

つまり、難読化コードはまず `alert('hello world')` のような文字列に変換され、それが`eval()` に渡されることで実行される。



よって、`eval()` 関数を引数の文字列を実行する関数ではなく、引数の文字列を表示する関数に書き換えてあげれば良い。

```javascript
eval = function(e) { console.log(e); };
```



## 難読化2

関数を作って、それを実行させる。

```javascript
hoge = function(){alert('hello world');};
hoge();
```

この場合、最後に`();` が残っているため見分けられる。

関数には、`toString()` と言うかメソッドがあり、これを用いることで、元の関数のソースコードがコンソールログに表示される。