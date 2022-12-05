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

