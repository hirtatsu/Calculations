# WSL上へのPythonの環境構築
詳細は[こちら](https://learn.microsoft.com/ja-jp/windows/python/web-frameworks)

## pyenvのインストール
Pythonのバージョン管理の利便性のため、pyenv上にpythonをインストールすることにします。
- pyenvの公式は[こちら](https://github.com/pyenv/pyenv)

- まず必要なライブラリを一括インストールしておく
```
# パッケージリストの更新
sudo apt update

# 依存ライブラリの一括インストール（長いですが全部必要です）
sudo apt install build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
libffi-dev liblzma-dev
```

- 以下を実行する
```
curl -fsSL https://pyenv.run | bash
```
- 次に、ホームディレクトリにある.bashrcの末尾に追記するためのコマンドをまとめて実行する。
```
echo 'pyenv用のスクリプト' >> ~/.bashrc
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc
```
- その次に、ホームディレクトリに.profile, .bash_profile, あるいは.bash_loginがあれば、これらすべてに同様に追記するため、以下を実行する。
  - ~/.profileの場合
    ```
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
    echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
    echo 'eval "$(pyenv init - bash)"' >> ~/.profile    
    ```
  - ~/.bash_profileの場合
    ```
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
    echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
    echo 'eval "$(pyenv init - bash)"' >> ~/.bash_profile
    ```
  - ~/.bash_loginの場合
    ```
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_login
    echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_login
    echo 'eval "$(pyenv init - bash)"' >> ~/.bash_login    
    ```
- `PATH`に反映するために以下を実行する。
```
exec "$SHELL"
```
- pyenvが使えるか確認。バージョンが表示されればOK。
```
pyenv --version
```

## pyenvでPython環境を構築する
### Pythonをインストールする
- まず、利用できるpythonのバージョンを確認する
```
pyenv install -l
```
- 所望のバージョンを決めて(とりあえず3.XX.Yの中から新しそうなもの？)、以下を実行する。ここでは例として、3.13.11の場合を示す。
```
pyenv install 3.13.11
```
- インストールできたか確認する。`system`とは別に、先ほど指定したバージョン(ここでは`3.13.11`)が表示されていればOK。
```
pyenv versions
```
### Pythonのバージョンを指定する
- PC全体のデフォルトとなるバージョンを指定する（最初だけ。ここでは3.13.11を例に。）
```
pyenv global 3.13.11
```
- プロジェクトごとにバージョンを指定する。当該プロジェクトのディレクトリに移動して、以下を実行する（このプロジェクトで使うバージョンが固定される）。
```
# 1. プロジェクトのディレクトリを作成し移動。
mkdir ~/my_test_project
cd ~/my_test_project
# 2. このプロジェクトで使うPythonのバージョンを指定（固定）する。
pyenv local 3.13.11
```

## venvによるパッケージ管理環境の構築
### venvのインストール
```
sudo apt install python3-venv
```

### 仮想環境の作成とパッケージのインストール
- 仮想環境「test-env」を作成する。とりあえずここではPC全体のデフォルトとして構築する。
※プロジェクトごとに管理する場合は、上述のプロジェクトのディレクトリ内（例では~/my_test_project）で実行すること。
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
