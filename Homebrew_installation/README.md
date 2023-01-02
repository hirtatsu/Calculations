### Homebrewのインストール
Linuxでいうaptやyumに相当する(?)パッケージ管理ソフト(Homebrew)をインストールします。

- Macのアプリからターミナルを起動する
- [Homebrewページ](https://brew.sh/index_ja)にアクセスする
- 記載されているインストールコマンド（シェルスクリプト）をターミナル画面にペーストし実行（Enter）する。(2023年1月2日時点では以下の通り)
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- パスワードを聞かれたら適宜入力する
- 以下のコマンドを入力してエラーが出なければインストール完了
```
brew --version
```

### Homebrewでよく使うコマンド
- パッケージリストの更新
```
brew update
```
- パッケージの更新の実行
```
brew upgrade
```
- インストールされているパッケージの一覧表示
```
brew list
```
- パッケージのインストール
```
brew install パッケージ名
```
- パッケージのアンインストール
```
brew uninstall パッケージ名
```


