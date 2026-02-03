## 研究室の計算サーバ(Linux)に手元のPCから接続する方法と使い方

### 準備1
- 計算サーバへのユーザ作成を教員に依頼する
- 計算サーバのホスト名を教えてもらう

### 準備2 - データ転送ソフトのインストール
### Windowsの場合: WinSCP
- サーバとのファイル転送ソフトウェアであるWinSCPを手元のPCにインストールする。[こちら](https://forest.watch.impress.co.jp/library/software/winscp/)
- 初期設定する。まずWinSCPを開く。
- 「ホスト名」に計算サーバのホスト名、「ユーザー名」に計算サーバ上の自分のユーザ名、「パスワード」に計算サーバ上の自分のパスワードを入力し、「ログイン」をクリック。
- 「左側」にローカル(手元のPC)、「右側」にリモート(計算サーバ)のファイルが表示される。転送したいファイルやディレクトリをドラッグ＆ドロップして転送できる。

### Macの場合: XXXX
XXXX

### Linuxの場合: sftp
- 接続する
```
sftp ユーザ名@ホスト名
```
- パスワードを聞かれたら入力
- つながったら、cdコマンドでリモートの当該ディレクトリに移動
- 続いて、lcdコマンドでローカルの当該ディレクトリに移動
- ローカルからリモートにファイルを送るときは
```
put ローカルのファイル名 リモートのディレクトリ名
put -r ローカルのディレクトリ名 リモートのディレクトリ名
```
- ローカルにリモートからファイルをもらうときは
```
get ローカルのファイル名 リモートのディレクトリ名
get -r ローカルのディレクトリ名 リモートのディレクトリ名
```


## 手元のPCからのSSH接続
### SSH接続の基礎知識は[こちら](https://qiita.com/tag1216/items/5d06bad7468f731f590e#fn2)
### 名前解決
~/.ssh/configを書く
```
vim ~/.ssh/config
```
内容は以下の通り。
```
# ==========================================
# 共通設定
# ==========================================
Host *
  IdentityFile ~/.ssh/id_ed25519
  ServerAliveInterval 60
  AddKeysToAgent yes
  UseKeychain yes
  # ↓これを書いておくと、接続時にknown_hostsエラーが出た時に
  #   勝手に/dev/nullに飛ばして無視する（開発環境なら便利だが自己責任で）
  # StrictHostKeyChecking no
  # UserKnownHostsFile /dev/null

# ==========================================
# Linuxサーバー & NAS
# ==========================================
# サーバーごとに設定
Host XXXX # 接続するときに使う名前
  HostName xxx.xxx.xxx.xxx
  User xxxxxxx
```
### 接続
- ssh接続する
```
ssh XXXX # 接続するときに使う名前として設定したもの
```

- ssh接続を切断する
```
exit
```

### ssh接続切断後もバックグラウンドで実行続ける方法
- 計算実行コマンドの最初にnohupをつけて、最後に&をつける。例えばLAMMPSでin.meltを計算する場合。
```
nohup mpirun -np 12 lmp < in.lmp &
```
- ちなみに、実行中に表示されるコマンドラインは自動的にnohup.outに出力される。途中経過を見たい場合は以下。
```
tail -F nohup.out
```
# SSH接続
### 鍵作成
```
ssh-keygen -t ed25519 -f .ssh/id_ed25519_hoge
```

### Configの設定
`~/.ssh/config`ファイルに設定を書く。
```
Host hoge
  User sampleuser
  Port 22
  HostName xxx.xxx.xxx.xxx
  IdentityFile ~/.ssh/id_rsa
```



#
