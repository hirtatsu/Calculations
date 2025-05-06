# DAMASKのインストール
## Conda環境の構築
Conda環境内にDAMASKがインストールされる。なのでまずはConda環境を整える。ここでは、WSL2上のUbuntu上で環境構築することにする。

- VenvのDeactivate
```
deactivate
```
-Minicondaのインストール
```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```
続いて
```
source ~/miniconda3/bin/activate
```
(venv環境を常用の場合は非推奨) もし毎回自動的にconda環境で起動＆PATHを通すには？
```
conda init --all
```
公式サイトによる手順は[こちら](https://www.anaconda.com/docs/getting-started/miniconda/install#linux)。

## VenvでPython環境を構築している場合は、Venv環境とConda環境を適宜切り替える必要がある。
### ① Conda → venv に切り替える
```
conda deactivate
source ~/test-env(自分のディレクトリ名)/bin/activate
```
### ② venv → Conda に切り替える
```
deactivate
source ~/miniconda3/bin/activate
```


画面左下のWindowsボタンを押して、検索窓にPowershellを入力し、表示される**管理者として実行する**をクリック。以下を入力してEnter。
```
wsl --install -d Ubuntu
```
画面左下のWindowsボタンを押して、検索窓にUbuntuを入力し、インストールされた**Ubuntu on Windows**を立ち上げる。

UsernameとPassword（Ubuntuにログインするためのもの）を聞かれたら、入力する。ただし、必ず**半角文字**にすること。日本語ダメ。

いちおう最初にパッケージをupdate（アップデートする必要があるパッケージを確認）、upgrade（アップデートを実行）する。順番に以下を入力してEnter。
```
sudo apt update
