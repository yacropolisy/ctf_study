# XSS 入門

XSSとはCross site scripting の略で、Webの脆弱性をついた攻撃方法です。

原理と攻撃手法について、見ていきましょう。

## HTML のおさらい

まずはHTMLについてです。引用：http://www.ink.or.jp/~bigblock/html/

### タグとは？？
HTML文章（HTMLソース）は、タグと言うものを使って書いていきます。 < と > で囲まれたものをタグと言います。<タグ名>←のようにタグを書きます。
タグは基本的に開始タグ（始まりのタグ）と終了タグ（終わりのタグ）に分かれています。開始のタグは<タグ名>のように書き、終了タグは</タグ名>開始タグと終了タグでひとつのセットになります。
 <> は必ず半角文字で書いてください。開始タグと終了タグの間に内容を書いていきます。

```html
<タグ名>この囲まれた部分が内容になります。</タグ名>
```



### 基本的なタグの説明

基本的な４つのタグを説明します。この4つのタグは必ずHTML文章に必要なものなので確実に覚えてください。
`<html>～～～</html>`
このタグは、「これはHTML文章ですよ」と宣言しているタグです。
`<head>～～～</head>`
このタグの中に基本的なページの情報を書いていきます。（後で詳しく説明）
`<title>～～～</title>`
このタグは、ページのタイトルを指定します。（後で詳しく説明）
`<body>～～～</body>`
実際にブラウザの中に表示させる内容をこの中に書きます。
基本的にタグは、半角文字で書き大文字・小文字のどちらで書いてもかまいません。つまり、<HTML>と<html>は同じタグになります。どういう風にエディタに書けばいいのか下に書いておきます。

```html
<html>

 <head>

  <title>タイトルを指定する</title>

 </head>

 <body>

  ここに内容を書く

 </body>

</html>

```



### タイトルを指定する

`<title>タグの間に表示されるタイトルを書きます</title>`
このタグは、ページのタイトルを指定します。<head>から</head>の間に書きます。終了タグに / を書くのを忘れないでください。指定した内容はブラウザのタイトルバーに表示されます。
タイトルを指定しない場合はそのファイルのURLがそのまま表示されてしまいます。


### 属性と属性値

<body text="文字色">
これからたくさん出てくる `属性`  と `属性値`  ついて説明します。まずタグのことを `要素`  とも言います。これからはタグのことを`要素` と書いていきます。
`要素` の中に半角スペースで区切って書かれているものを `属性` と言います。上の`要素` では、`text` は`body` `要素` の`属性` であることが分かります。 " で囲まれて書かれているものを `属性値`  と言います。
この`要素` の場合、`属性値` には色の名前や16進数カラーコードを指定することができます。ページの文字色を赤色にしたいときには、`属性値` をREDにします。



### フォームの表示

ここではフォームの作り方を説明します。フォームはホームページを作るときはあまり使用しませんが、cgi（掲示板やチャット）を作るときによく使われます。

action属性 には、実行するスクリプトなどを書きます。

```html
<form action="処理するURL" name="フォームの名前"  method="送信形式"></form>
```

送信形式は、 GET か POST のどちらかを指定します。GETはURLの後ろに ? を付けてデータを送信する形式で、POSTは一度に多くのデータを送信するときに使います。これらはcgiを作るときによく使います。



## Javascript について

Javascriptは、ブラウザ上で動作するプログラミング言語だと思っておけば良いです。

一般的にはwebページでの、動的な制御を扱うために使われます。[例](https://www.recruit-jinji.jp/)



### javascriptの記入場所

http://www.pori2.net/js/kihon/2.html に書いてある通り、

- ヘッダやbody内に組み込む手法

```html
<script>
  ここにJavaScriptのソースを記入
</script>
```

- タグ内に記入する手法

```html
<a href="JavaScript:ソース記入">クリックしてね</a><br>

<input type="button" value="Click!" onclick="ソース記入">
```

があります。



例えば、以下のようなHTMLファイルを作成して、ブラウザで見て見ましょう。



```html
<html>

 <head>

  <title>javascriptの練習</title>

 </head>

 <body>
	<script>
		alert("hello world!")
	</script>
  

 </body>

</html>
```

`hello world` というダイアログが表示されればOKです。

ここで使った`alert` という関数は、javascriptでアラートダイアログを出す関数です。



## XSS

いよいよ本題に入ります。

HTMLに、ユーザーが入力した文字列をそのまま表示する箇所があったとします。

以下の例では、ユーザにユーザ名を入力させ、その内容をそのまま表示している例です。

```html
<html>
 <head>

  <title>javascriptの練習</title>

 </head>

 <body>
	こんにちは、[ユーザの入力した内容] さん！
  
 </body>

</html>
```



ここで、ユーザーが`<script>alert("XSS successed!")</script>` のように入力すると、HTMLは以下のようになり、javascriptが作動します。

```html
<html>
 <head>

  <title>javascriptの練習</title>

 </head>

 <body>
	こんにちは、<script>alert("XSS successed!")</script> さん！
  
 </body>

</html>
```



### 練習

[サンプルサイト](http://bogus.jp/xsssample/)があるので、やって見ましょう。



#### [レベル1](http://bogus.jp/xsssample/xsssample_01.php?word=hoge)

フォームに打ち込んだ内容が、そのまま表示されるようです。

以下のように打ち込みましょう。

```html
<script>alert("xss")</script>
```



#### [レベル2](http://bogus.jp/xsssample/xsssample_02_M9eec.php)

今度はログインフォームのようです。

ログインフォームの構造は、以下のようになっていることが多いです。

```html
<form method="get" acion="">login<br>
username<input name="user" type="text" value="ユーザーの入力"><br>
password<input name="pass" type="password" value="ユーザーの入力"><br>
<input type="submit" value="login">
</form>
```



ここでXSSをする場合は、`"` を閉じてあげる必要があるので、以下のような形になります。

```html
"><script>alert("xss")</script><"
```



