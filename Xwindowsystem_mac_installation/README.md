## サーバー上のGUIアプリを使えるようにする
研究室の計算サーバ上で動くGUIアプリを、手元のMacで表示できるように設定します。

### Mac側のインストール (XQuartz)
- [https://www.xquartz.org/](https://www.xquartz.org/)からXQuartzをダウンロードしてインストールする。
```
brew install --cask xquartz
```
- Macを一旦ログアウトして再度ログインする
- ディスプレイの環境変数が設定されていることを確認する
```
echo $DISPLAY
```
- 何か表示されれば成功

### SSHで研究室のサーバーでの動作確認
- 研究室サーバーにアクセスし、以下のコマンドでマウスに連動する目玉が出てきたら成功
```
xeyes
```
<img width="133" alt="xlaunch4" src="https://user-images.githubusercontent.com/64639043/204118509-92d7c6c8-0a77-45ad-9989-8eff1024dccf.png">

