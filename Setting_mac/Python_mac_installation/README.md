## venvを用いた仮想環境の作成とパッケージのインストール
### venvを用いた仮想環境の作成
- 仮想環境「test-env」を作成する
```
python3 -m venv /Users/ユーザ名/test-env/
```

- 仮想環境「test-env」をActivateする
```
source /Users/ユーザ名/test-env/bin/activate
```
- Mac起動時に自動的に仮想環境が立ち上がるようにするためには、.zshrcの最後に以下を記載する
```
source /Users/ユーザ名/test-env/bin/activate
```

- (必要あれば) pipをupgradeする
```
pip install --upgrade pip
```
### venvを用いたパッケージのインストール
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

## VSCodeでPython仮想環境を使えるようにする
### インストール
- VSCodeのインストール
[https://code.visualstudio.com/download](https://code.visualstudio.com/download)

- 拡張機能のインストール (Python)

- PATHを通す: VSCodeを開いて、shift+command+pを押して、入力欄にshell command: install codeを入力して選択する

- 試しに任意の場所に、test.pyを作成する
```
echo "print('hello world')" > test.py
```

- TerminalからVSCodeを立ち上げる
```
code test.py
```
自動的にVScodeでtest.pyが開かれる

- 使うPython(インタープリター)を選ぶ。画面右下、「Pythonインタープリタを選択？」あるいは「3.X.XX」をクリック。「＋インタープリターパスを入力」にvenvで構築した仮想環境のパスを指定する。上述の通りであれば以下。
```
/Users/ユーザ名/test-venv/bin/
```
- 完了

