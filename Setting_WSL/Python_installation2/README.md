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

