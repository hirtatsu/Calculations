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

## Pytorchのインストール
[Pytorchホームページ](https://pytorch.org/)のInstall pytorchで、環境に応じて選択ののち、コマンドをコピーして実行。
例。
```
pip3 install torch torchvision torchaudio
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

