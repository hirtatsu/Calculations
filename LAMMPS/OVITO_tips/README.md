# OVITOの使い方メモ

最終更新: 2026-08 ／ 対象: OVITO Basic / Pro

MD��算の結果（LAMMPSのdumpファイルなど）を可視化・解析するソフトウェアです。GUI版とPython版があります。

- 公式サイト: [https://www.ovito.org/](https://www.ovito.org/)
- Basic版は無償で利用できます（一部の解析機能はPro版のみ）

以下は、よく使う操作の覚え書きです。

---

## アニメーションにタイムステップを表示する

1. **Viewport layers** を開く
2. **Add layer** から **Text label** を追加する
3. Text入力欄に `[Timestep]` と入力する

角括弧で囲むと、その時点の属性値に置き換えて表示されます。

---

## アニメーションを出力する

1. **Rendering** を開く
2. **Complete animation** を選択し、何フレームごとに出力するかを決める（全フレームを使う場合は **Every Nth frame: 1**）
3. **Output image size** を決める（**Presets** から選ぶと簡単です）
4. **Render output** の **Save to file** にチェックを入れ、**Choose** から保存先とファイル名を指定する
5. **Background** の色を決める（**Transparent** で背景を透明にできます）
6. **Render active viewport** をクリックする

---

## 断面を表示する

1. **Add modification** から **Slice** を選択する
2. **Normal** の x, y, z に、切断面の法線方向の指数を入力する
3. **Distance** に原点から面までの距離を入力する
4. **Reverse orientation** で切り取る側を反転できます

複数のSliceを重ねると、任意の領域だけを取り出して表示できます。

---

## MSD（平均二乗変位）を計算する

1. dumpファイルを開く
2. **初期座標を書き出す**。0ステップ目を表示した状態で **File → Export File** を選び、ファイル名を `0`、形式を **LAMMPS Dump File**、範囲を **Current frame only** として、**Select all** で書き出す
3. **Add modification → Displacement vectors** を選択する。**Reference configuration source** に **External file** を指定し、フォルダアイコンから手順2で書き出したファイルを読み込む
4. **File → Export File** を選び、ファイル名を `disp_mag.*`、形式を **LAMMPS Dump File**、範囲を **全フレーム** かつ **File sequence**、**Select all** として書き出す
5. 各フレームにおける全原子の座標と変位量が出力されるので、Python等で解析する

---

## 結晶粒を可視化する

1. データをインポートする
2. **Add modification → Modification → Wrap at periodic boundaries**
3. **Add modification → Structure identification → Polyhedral template matching**（**Lattice orientations** にチェックを入れること）
4. **Add modification → Analysis → Grain segmentation**

手順3で **Lattice orientations** を有効にしないと、手順4の粒界判定ができません。

---

## Python版（OVITO Python）

スクリプトから解析を自動化する場合に使います。condaでインストールします。

```
conda install --strict-channel-priority -c https://conda.ovito.org -c conda-forge ovito
```

バージョンを指定する場合は `ovito=3.14.1` のように記述します。利用できるバージョンは以下で確認できます。

```
conda search -c https://conda.ovito.org ovito
```

conda環境の準備については[Pythonの環境構築（Miniforge）](../../Setting_WSL/Python_installation2/README.md)を参照してください。

---

## 関連ページ

- [LAMMPSのインストール（基本版）](../LAMMPS_installation/README.md)
- [Atomskのインストールと使い方](../Atomsk_installation/README.md)
