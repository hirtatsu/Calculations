# DAMASKのインストール


画面左下のWindowsボタンを押して、検索窓にPowershellを入力し、表示される**管理者として実行する**をクリック。以下を入力してEnter。
```
wsl --install -d Ubuntu
```
画面左下のWindowsボタンを押して、検索窓にUbuntuを入力し、インストールされた**Ubuntu on Windows**を立ち上げる。

UsernameとPassword（Ubuntuにログインするためのもの）を聞かれたら、入力する。ただし、必ず**半角文字**にすること。日本語ダメ。

いちおう最初にパッケージをupdate（アップデートする必要があるパッケージを確認）、upgrade（アップデートを実行）する。順番に以下を入力してEnter。
```
sudo apt update
