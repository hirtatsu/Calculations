# Miniforgeを用いた環境構築
- Miniforgeは、Condaの軽量版とも言えるパッケージ管理ツール。
- Condaはデータサイエンスや機械学習の分野で広く利用されており、複数の仮想環境を作成し、依存関係を解決しながらライブラリを簡単に管理できる利点がある。
- Miniforgeは必要最低限のConda環境のみを提供し、ユーザーが自分に必要なパッケージだけを追加できる点が特徴。

# miniforgeのインストール
公式ページは[こちら](https://github.com/conda-forge/miniforge)

### ダウンロード
```
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
```
### インストール作業
- スクリプトの実行
```
bash Miniforge3-$(uname)-$(uname -m).sh
```
- 適宜`enter`押しながらインストール進める
- ライセンス条項を受け入れるために`yes`を入力する
- インストールするディレクトリに特に希望なければ`enter`
- Initializationの要否についての質問（起動時に自動的に`conda`コマンドを使えるようにしますか？的な）に対して、`yes`を入力する
- 完了！

### 動作確認と最新バージョンのインストール
- 一旦ターミナルを立ち下げて再度Ubuntuを起動
- 行のはじめに`(base)`が表示されていればOK
- 以下でバージョン確認できるはず
```
conda --version
```
- 最新のバージョンにアップデート
```
conda update conda
```
- 毎回勝手に起動するのが煩わしいので、以下で自動起動を`false`に
```
conda config --set auto_activate_base false
```
- 次回から、自動的には`(base)`が出なくなる。使用時に適宜以下を実行する
```
conda activate  # Conda環境に入る (baseが出る)
conda deactivate # Conda環境から出る
```

# Pythonのインストール
- 利用できるPythonのバージョンをチェック
```
conda search python | tail -n 20 # とりあえず最新を20件表示
```
- 仮想環境を構築（ここではPython バージョン 3.13.11を例に）
```
conda create -n test-env python=3.13.11
```
- 作成した環境をアクティベイト
```
conda activate test-env
```
行頭に`(test-env)`が表示されれば成功

- 環境を無効化するときは以下
```
conda deactivate
```
