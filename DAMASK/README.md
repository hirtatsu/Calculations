# DAMASKのインストール
### 公式に基づいて、パッケージ管理からインストールする
詳しい方法は、[https://damask-multiphysics.org/installation/index.html](https://damask-multiphysics.org/installation/index.html)
から、Method→[Package Manager](https://damask-multiphysics.org/installation/package_manager.html#package-manager)を選択。

```
sudo add-apt-repository ppa:damask-multiphysics/ppa
sudo apt update
sudo apt install -y damask
```
これでインストール完了！以下のコマンドを実行してなんか表示されればOK。

- DAMASK_grid, the included solver for regular grids
```
DAMASK_grid --help
```
- DAMASK_mesh, the included solver for unstructured meshes
```
DAMASK_mesh --help
```
- Pythonパッケージである<b>damask</b>もインストールされているはず。プレ＆ポストプロセス用。 Python3で使えるモジュール。ただし、/usr/lib/python3/dist-packages/にインストールされるので、Venv環境からは使えないっぽい。もしVenv環境下であるなら、先にdeactivateすること。
```
Python3
>>> import damask
```
