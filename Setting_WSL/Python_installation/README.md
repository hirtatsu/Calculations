## WSL上へのPythonの環境構築
詳細は[こちら](https://learn.microsoft.com/ja-jp/windows/python/web-frameworks)

### Python3, pip, venvのインストール
- Python3。バージョン番号が表示されればOK。
```
python3 --version
```
- Pythonのパッケージ管理ツールpipのインストール
```
sudo apt install python3-pip
```
- Python仮想環境作成のためのツールvenvをインストール
```
sudo apt install python3-venv
```

### 仮想環境の作成とパッケージのインストール
- 仮想環境「test-env」を作成する
```
python3 -m venv /home/ユーザ名/test-env/
```

- 仮想環境「test-env」をActivateする
```
source /home/ユーザ名/test-env/bin/activate
```
- WSL起動時に自動的に仮想環境が立ち上がるようにするためには、.bashrcの最後に以下を記載する
```
source /home/ユーザ名/test-env/bin/activate
```

- (必要あれば) pipをupgradeする
```
pip install --upgrade pip
```

- パッケージをインストールする(例: numpy, pandas, matplotlibをインストールする)
```
pip install numpy pandas matplotlib
```

- インストール済みのパッケージを表示する
```
pip list
```

- 仮想環境をDeactivateする
```
deactivate
```



## WindowsのVSCodeで、WSL上のVenvを使ったPython仮想環境を使えるようにする
### VSCodeのインストール
[https://azure.microsoft.com/ja-jp/products/visual-studio-code/](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)

### 拡張機能のインストール
- Python
- WSL

### WSLからWindows上のVSCodeを立ち上げる
- 試しにWSL上の任意の場所に、test.pyを作成する
```
echo "print('hello world')" > test.py
```

- VSCodeを立ち上げる
```
code test.py
```
自動的にWindows上でVScodeでtest.pyが開かれる

- WSL上で使うPythonを選ぶ。画面右下、「Pythonインタープリタを選択？」をクリック。「＋インタープリターパスを入力」にvenvで構築した仮想環境のパスを指定する。上述の通りであれば以下。
```
/home/ユーザ名/test-venv/bin/
```
- 完了

## （番外編）Pytorchのインストール
[Pytorchホームページ](https://pytorch.org/)のInstall pytorchで、環境に応じて選択ののち、コマンドをコピーして実行。
- 例。
```
pip3 install torch torchvision torchaudio
```

- 動作確認
```
python3
>> import torch
>> print(torch.__version__)
# 1.7.1
>> print(torch.cuda.is_available())
# True
>> print(torch.cuda.device_count())
# 1
>> print(torch.cuda.current_device())
# 0
>> print(torch.cuda.get_device_name())
# GeForce GTX 1080 Ti
>> print(torch.cuda.get_device_capability())
# (6, 1)
```

## （番外編）Conda環境の構築
ここでは、WSL2上のUbuntu上で環境構築することにする。

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
(venv環境を常用の場合は非推奨) もし毎回自動的にconda環境で起動＆PATHを通すには以下の通り。.bashrcに勝手にいろいろ記入される。
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
