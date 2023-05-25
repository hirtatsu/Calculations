## WSL上でのGitの初期設定
### インストール
```
sudo apt update
sudo apt install git
```

### 準備
- .gitconfigを設定する
```
git config --global user.name 'Githubのアカウント名'
git config --global user.email 'Githubに登録しているメールアドレス'
git config --global core.editor 'vim'
```
- 一応確認する
```
git config --list
```

### リモートリポジトリへの認証(SSH)設定
- SSHキーの作成
```
cd
ssh-keygen -t rsa
```
何回かEnterを押す。

- Githubに公開鍵を登録
[https://github.com/settings/ssh](https://github.com/settings/ssh)

    「NEW SSH key」をクリック

- Githubの入力欄Titleを入力する(なんでもOK。Ubuntuとか)
- Githubの入力欄Keyを入力する
    下記コマンドで公開鍵の中身をクリップボードへコピーしたあと、Key欄に貼り付ける。
```
cat ~/.ssh/id_rsa.pub | clip.exe
```
- 最後にAdd SSH keyをクリックして完了。

### 接続の確認
- 以下のコマンド入力する
```
ssh -T git@github.com
```
- 途中でyesを入力してEnter。
- 最後にHi~~~~と表示されればOK。

### Gitで管理したいディレクトリを初期登録する
- 管理したいディレクトリへ移動
```
cd 管理したいディレクトリの場所
git init
```


