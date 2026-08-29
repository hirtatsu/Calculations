# Pythonの環境構築（Miniforge）

最終更新: 2026-08 ／ 対象: Ubuntu（WSL2 / ネイティブ）、macOS（Apple Silicon / Intel）

計算結果の解析や可視化に使うPython環境を、Miniforgeを用いて構築します。本リポジトリではPython環境をMiniforgeに一本化しています。

---

## Miniforgeとは、なぜMiniforgeなのか

Miniforgeは、パッケージ管理ツールCondaの最小構成版です。Condaは仮想環境を複数作成でき、依存関係を解決しながらライブラリを導入できるため、数値計算や機械学習の分野で広く使われています。Miniforgeは必要最低限のConda環境だけを提供し、必要なパッケージは各自が追加する形になります。

同種のツールにAnacondaやMinicondaがありますが、本リポジトリでMiniforgeを推奨するのは、**参照するパッケージ配布元（チャネル）が異なる**ためです。

- Anaconda / Miniconda は既定で **defaultsチャネル**（`repo.anaconda.com`）を参照します。このチャネルはAnaconda社の利用規約の対象で、一定規模以上の組織における利用が有償ライセンスの対象とされています。大学がこれに該当するかは解釈が分かれ、利用を見合わせている大学もあります。
- Miniforge は **conda-forgeチャネル**のみを既定で参照します。conda-forgeはコミュニティが運営する配布元で、この制約がありません。

科学計算で必要になるパッケージはconda-forgeにほぼ揃っているため、実用上の不便もありません。所属機関の方針を確認する手間を省く意味でも、Miniforgeから始めることをおすすめします。

公式ページは[こちら](https://github.com/conda-forge/miniforge)。

---

## インストール

### Ubuntu（WSL2 / ネイティブ）の場合

インストーラをダウンロードします。`$(uname)` と `$(uname -m)` の部分は自動的にOSとCPUアーキテクチャに置き換わるため、このまま実行して構いません。

```
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
```

実行します。

```
bash Miniforge3-$(uname)-$(uname -m).sh
```

- 適宜 `enter` を押しながら進める
- ライセンス条項に同意するため `yes` を入力する
- インストール先は特に希望がなければ `enter`（既定は `~/miniforge3`）
- 最後に初期化の要否（起動時に自動的に `conda` コマンドを使えるようにするか）を聞かれるので `yes` を入力する

### macOSの場合

Ubuntuと同じコマンドがそのまま使えます。Apple Silicon（arm64）とIntel（x86_64）のどちらも、`$(uname -m)` によって適切なインストーラが選ばれます。

```
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

macOSの既定シェルはzshのため、初期化の設定は `~/.zshrc` に書き込まれます。

Homebrewを導入済みであれば、以下でもインストールできます。

```
brew install --cask miniforge
```

---

## 動作確認と初期設定

- 一旦ターミナルを閉じて、開き直します
- プロンプトの行頭に `(base)` が表示されていればインストール成功です

バージョンを確認します。

```
conda --version
```

condaを最新にします。

```
conda update -n base conda
```

毎回 `(base)` 環境が有効になるのが煩わしい場合は、自動起動を無効にできます。

```
conda config --set auto_activate_base false
```

次回から自動では `(base)` が表示されなくなります。使用時に以下を実行してください。

```
conda activate    # Conda環境に入る（(base)が表示される）
conda deactivate  # Conda環境から出る
```

---

## 仮想環境の作成とPythonの導入

プロジェクトごとに仮想環境を分けておくと、パッケージのバージョン衝突を避けられます。

利用できるPythonのバージョンを確認します。

```
conda search python | tail -n 20
```

仮想環境を作成します。環境名（ここでは `test-env`）とPythonのバージョンを指定します。

```
conda create -n test-env python=3.13
```

> バージョンは `3.13` のようにマイナー版まで指定すれば、その系列の最新が入ります。特定のパッチ版に固定したい理由がなければ、細かく指定しない方が扱いやすいです。2026年8月時点の最新安定版は3.14系ですが、科学計算のパッケージが追従しているかは事前に確認してください。

作成した環境を有効化します。

```
conda activate test-env
```

行頭に `(test-env)` が表示されれば成功です。

パッケージを追加します。conda-forgeチャネルから導入されます。

```
conda install numpy pandas matplotlib
```

環境から出るときは以下です。

```
conda deactivate
```

### よく使う操作

| 操作 | コマンド |
|---|---|
| 環境の一覧を表示 | `conda env list` |
| 環境内のパッケージ一覧 | `conda list` |
| 環境の削除 | `conda env remove -n 環境名` |
| 環境の書き出し | `conda env export > environment.yml` |
| 書き出した環境の復元 | `conda env create -f environment.yml` |

環境の書き出しと復元は、別の計算機で同じ環境を再現したいときや、計算に使った環境を記録として残したいときに使います。

---

## 関連ページ

- [Homebrewのインストール（Mac）](../../Setting_mac/Homebrew_installation/README.md)
- [OVITOの使い方メモ](../../LAMMPS/OVITO_tips/README.md) — OVITOのPython版はcondaで導入します
