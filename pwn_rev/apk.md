# MACでapkファイルをjavaのソースコードへ逆コンパイルする方法

`dex2jar` と `jad` を使う。

`.apk` -> `.dex` -> `.jar` -> `.class` -> `.jad` という流れ。

`.jad` ファイルを開けば`java` のソースコードが見れる。

参考：

https://qiita.com/le_skamba/items/04c0a0cfe420be862122

https://hacknote.jp/archives/10259/

http://inarizuuuushi.hatenablog.com/entry/2017/05/09/231600



## 0. dex2jar, jad のインストール

### dex2jar 

brewを使う。

パスを通す必要あり。

```bash
brew install dex2jar

echo 'export PATH="$PATH:/usr/local/Cellar/dex2jar/2.0/bin"' >> ~/.bash_profile
source ~/.bash_profile
```

### jad

brew cask を使う。

```bash
brew tap caskroom/cask
brew install caskroom/cask/jad
```



## 1. apk -> dex 

```bash
unzip hoge.apk
```

## 2. dex -> jar 

```bash
d2j-dex2jar hoge.dex
```

## 3. jar -> class

```bash
jar xf hoge.jar 
```

## 4. class -> java

```bash
jad -r path/hoge.class 
```



# Google Chrome で apkファイルを実行する方法

参考：https://ottan.xyz/mac-google-chrome-android-596/

https://qiita.com/yuyuport/items/bb12acf67713a9d2e4fd



```
chromeos-apk hoge.apk
```

上のコマンドでフォルダを作成し、Chromeに拡張機能として取り込む。

`Archon There is no "message" element for key extName.` のエラーが出る場合、 `_locales/messages.json` の

```
"extName": {
    "description": "Extension name"
  }
```

この部分を以下のように変更する。

```
"extName": { 
"description": "Extension name", 
"message": "Extension name" 
}
```



これでいけるらしいが、ムムム。





## BlueStacks

これでいけた。

ダウンロードして、apkファイルをこのappで開けば実行できた。