# WSL2のインストール
## Windows上でLinuxを動かすために、WSL(Windows Subsystem for Linux)を準備する。

Linuxディストリビューションとしては、Ubuntuを用いる。

画面左下のWindowsボタンを押して、検索窓にPowershellを入力し、表示される**管理者として実行する**をクリック。以下を入力してEnter。
```
wsl --install -d Ubuntu
```
画面左下のWindowsボタンを押して、検索窓にUbuntuを入力し、インストールされた**Ubuntu on Windows**を立ち上げる。

UsernameとPassword（Ubuntuにログインするためのもの）を聞かれたら、入力する。ただし、必ず**半角文字**にすること。日本語ダメ。

いちおう最初にパッケージをupdate（アップデートする必要があるパッケージを確認）、upgrade（アップデートを実行）する。順番に以下を入力してEnter。
```
sudo apt update -y
sudo apt upgrade -y
```
インストールおしまい

## Windowsターミナルのインストール
Windowsのスタートメニュー　→　Windows store　→　Windowsターミナルを見つけて入手する。

次回から、Windowsターミナルを使ってUbuntuを使うのがおすすめ。

## Ubuntuの初期設定いろいろ
### WindowsからWSL上のファイルにアクセスする
Windowsのエクスプローラーのアドレス欄に以下を入力してEnter.
```
\\wsl$
```
そうするといつものWindows画面からWSLのUbuntu上のファイルを見ることができます。編集も可。

### シンボリックリンク(Windowsでいうショートカットを作る)
例えば、Ubuntuの初期ディレクトリ(/home/「ユーザー名」)に、Windowsのデスクトップへのショートカットを作る。
```
cd
ln -s /mnt/c/User/「Windowsのユーザ名」/Desktop
```
これで、Ubuntu上からWindows上のファイルにアクセスするのも簡単。

### Ubuntu上のカレントディレクトリを一発でWindowsのエクスプローラーで開くエイリアスの作成
Ubuntu上で作業していて「このディレクトリをWindowsで開きたい」というときに使うショートカット。
初期ディレクトリに移動して、.bashrcを開く。
```
cd
vim .bashrc
```
そして、最終行に以下を追記する。
```
alias open=explorer.exe
```
vimを保存して閉じたら、
```
source .bashrc
```
で読み込んで設定完了。これ以降、
```
open .
```
で、現在のディレクトリをWindowsのエクスプローラーで開くことができる。

## その他(WindowsのPowershell上の操作)
※ちなみに、インストールされてるディストリビューションを確認するときは以下の通り。
```
wsl -l -v
```

※Ubuntuを消したり再インストールしたくなったら、以下のコマンドを入力すればきれいさっぱり。
```
wsl --unregister Ubuntu
```

※Ubutuをシャットダウンするときは以下の通り。
```
wsl --shutdown
```

