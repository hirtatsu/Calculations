# Pythonの環境構築（Mac）

最終更新: 2026-08 ／ 対象: macOS（Apple Silicon / Intel）

MacでPythonの計算環境を構築し、VSCodeから利用できるようにします。

本リポジトリでは、Python環境を **Miniforge（conda）** に一本化しています。導入手順はWindows（WSL）と共通のため、[Pythonの環境構築（Miniforge）](../../Setting_WSL/Python_installation2/README.md)を参照してください。macOS向けの導入方法もそちらに記載しています。

このページでは、環境を作った後の **VSCodeとの連携**を中心に扱います。

---

## 1. 仮想環境を用意する

Miniforgeの導入が済んでいれば、以下で仮想環境を作成できます。

```
conda create -n test-env python=3.13
conda activate test-env
```

行頭に `(test-env)` が表示されれば、その環境が有効になっています。

パッケージを追加します。

```
conda install numpy pandas matplotlib
```

環境から出るときは以下です。

```
conda deactivate
```

詳細は[Pythonの環境構築（Miniforge）](../../Setting_WSL/Python_installation2/README.md)を参照してください。

---

## 2. VSCodeから使えるようにする

### VSCodeのインストール

[公式サイト](https://code.visualstudio.com/download)からダウンロードしてインストールします。Homebrewを導入済みであれば以下でも入ります。

```
brew install --cask visual-studio-code
```

### 拡張機能の導入

VSCodeを起動し、拡張機能（Extensions）から **Python**（Microsoft製）をインストールします。

### ターミナルから起動できるようにする

VSCodeを開いた状態で `Shift + Command + P` を押し、入力欄に `shell command` と入力して **Shell Command: Install 'code' command in PATH** を選択します。

以降、ターミナルから `code ファイル名` でVSCodeを開けます。試してみます。

```
echo "print('hello world')" > test.py
code test.py
```

### 使用するPythonを選択する

1. VSCodeで `.py` ファイルを開きます
2. `Shift + Command + P` を押し、**Python: Select Interpreter** を選択します
3. 一覧から、手順1で作成したconda環境（`test-env`）を選びます

conda環境は自動的に検出されるため、通常はパスを手入力する必要はありません。一覧に表示されない場合のみ、**Enter interpreter path** から以下のように**実行ファイル本体**を指定します。

```
/Users/ユーザ名/miniforge3/envs/test-env/bin/python
```

> ディレクトリではなく `bin/python` までを指定してください。

画面右下または左下に選択した環境名が表示されれば設定完了です。VSCode内のターミナルを開くと、その環境が自動的に有効化されます。

---

## 参考: venvを使う場合

condaを使わず、軽量な仮想環境で済ませたい場合は標準の `venv` も利用できます。

```
python3 -m venv ~/test-env
source ~/test-env/bin/activate
```

```
pip install --upgrade pip
pip install numpy pandas matplotlib
pip list
deactivate
```

ただし、科学計算で用いるライブラリの中には、コンパイル済みのバイナリや外部ライブラリに依存するものがあります。それらの依存解決はcondaのほうが容易なため、本リポジトリではMiniforgeを推奨しています。

### 自動で仮想環境を有効化する設定について

> ~~`.zshrc` の末尾に `source ~/test-env/bin/activate` を記載して、ターミナル起動時に自動的に仮想環境を有効化する~~
>
> **この設定は推奨しません。** すべてのターミナルで単一の仮想環境が常時有効になるため、プロジェクトごとに環境を切り替えられなくなり、仮想環境を使う意味が失われます。また、想定と異なる環境でパッケージをインストールしてしまう事故につながります。
>
> 作業を始めるたびに `conda activate 環境名`（または `source ~/環境名/bin/activate`）を実行してください。ディレクトリごとに自動で切り替えたい場合は、`direnv` などのツールを利用する方法があります。

---

## 関連ページ

- [Pythonの環境構築（Miniforge）](../../Setting_WSL/Python_installation2/README.md) — 導入手順の本体
- [Homebrewのインストール](../Homebrew_installation/README.md)
