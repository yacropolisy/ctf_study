# IDA

[IDA Pro 使い方メモ](https://rancha.hatenablog.com/entry/2018/02/03/115355)

- 設定変更　
  optionタブ -> General -> Disassembly -> Display disassembly line parts で、Auto comments 以外チェックつける
- "N" 変数の名前かえる
- ":" コメントつける ";" だと同じコード全てにコメントがつく
- "R" 16進 <=> ASCIIの変換
- "D" , "C" アドレスの中途半端なところにjump してる場合、そこを逆アセンブルできる。Dで展開してCでアセンブリ言語に。



[begginers write up](http://rintaro.hateblo.jp/entry/2014/07/24/222327)

- `search` -> `Text` で文字列検索し、`.text` セクションで文字列ヒットすればその辺を探索すればよい。



katagaitai 1

- Structure window で構造体を作成できる
  - I で作成、構造体の名前をつける
  - D でメンバの型を指定 db: 1byte, dw: 2byte, dd: 4byte
  - N でメンバの名前を変更

# radare2

- 基本　`aaa` -> `afl` で関数一覧を確認
- main があれば `pdf @ main` なければそれっぽい関数を `pdf`
- `s` で移動 `s main` `pdf` でもよい。
- 移動したところで`VV` を押すとビジュアルモード
- 困ったらコマンドに`?` つけてヘルプを見る。
- メモリの中身を見る `pv @ 0x012345678`







# Ollydbg

- run `F9` 
- 再読み込み `Ctr + F2`
- ステップイン `F7`  ステップアウト `F8` 
- ブレークポイント `F2` 
- 関数呼び出し一覧 `Ctr + N` 、+ `a` でaから始まる関数を検索 -> 関数選択してEnter
- アドレスジャンプ `Ctr + G` 
- 