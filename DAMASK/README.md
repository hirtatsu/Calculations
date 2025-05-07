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

- DAMASK_grid (ソルバー for regular grids)
```
DAMASK_grid --help
```
- DAMASK_mesh (ソルバー for unstructured meshes)
```
DAMASK_mesh --help
```
Pythonパッケージである<b>damask</b>もインストールされているはず。プリ＆ポストプロセス用。
- Python3で使えるモジュール。/usr/lib/python3/dist-packages/にインストールされる。
- Venv環境からは使えないっぽい。もしVenv環境下であるなら、先にdeactivateすること。
```
deactivate
```
- インストール確認。importしてエラー出なければOK。
```
python3
>>> import damask
```

# DAMASK Pre-processing
### Pythonのdamaskモジュールを使ってインプットファイル作成
- python3からdamaskモジュールを呼び出す
```
Python3
>>> import damask
```
- DREAM3Dデータを読み込む(読み込めるはず)
```
>>> config_material = damask.ConfigMaterial.load_DREAM3D('test01.dream3d')
```
- materialのphase名を変更
```
# すべての material の constituent に含まれる phase を 'Beta-Sn' に変える
for material in config_material['material']:
    for constituent in material['constituents']:
        if constituent['phase'] == 1:
            constituent['phase'] = 'Beta-Sn'

# そして phase 定義のキーも 'Beta-Sn' に変える
config_material['phase']['Beta-Sn'] = config_material['phase'].pop('1')
```

- 弾性定数を定義(BCT構造のStiffness Matrixは[こちら](https://damask-multiphysics.org/documentation/crystal_structures/body-centered-tetragonal.html#body-centered-tetragonal-ti))。
```
config_material['phase']['Beta-Sn'] = {
  'mechanical': {
    'elastic': {
      'type': 'Hooke',
      'c': [[72.3, 59.4, 35.8,  0,    0,    0],
            [0,    72.3, 35.8,  0,    0,    0],
            [0,    0,    88.4,  0,    0,    0],
            [0,    0,    0,     22.0, 0,    0],
            [0,    0,    0,     0,    22.0, 0],
            [0,    0,    0,     0,    0,    24.0]]}}}
```
※β-Sn。[参考文献](https://doi.org/10.2320/matertrans.MH201808)

- ファイルの妥当性チェック
```
config_material.is_complete # Trueならファイル構成はOK。
```
※Phase名が1だと通らない。なぜ。。。とりあえずテキストエディタで"A"に置き換えたらいけた。

- YAMLファイルを保存する。
```
config_material.save('test01.yaml')
```
- YAMLファイルを読み込む。
```
config_material = damask.ConfigMaterial.load('test01.yaml')
```

