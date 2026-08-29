# Atomskのインストールと使い方

最終更新: 2026-08 ／ 対象: Atomsk beta-0.13系、Ubuntu（WSL2 / ネイティブ）

MD��算に用いる原子構造モデルを作成・変換するためのコマンドラインツールです。結晶構造の生成、スラブの作成、モデルの結合、ファイル形式の変換などを一連のコマンドで行えます。

- 公式サイト: [https://atomsk.univ-lille.fr/](https://atomsk.univ-lille.fr/)
- マニュアル: [https://atomsk.univ-lille.fr/doc/en/index.html](https://atomsk.univ-lille.fr/doc/en/index.html)

---

## インストール

### Ubuntuの場合（Debianパッケージ）

[ダウンロードページ](https://atomsk.univ-lille.fr/dl.php)から「Debian/Ubuntu (64 bits)」の `.deb` ファイルを入手します。バージョンは更新されるため、ページで最新のファイル名を確認してください。

```
cd
wget （ダウンロードページで確認したURL）
sudo dpkg -i atomsk_bX.XX.X_amd64.deb        # 入手したファイル名に読み替え
```

### Debianパッケージが使えない環境の場合

`dpkg` が使えない環境（Matlantisのクラウド環境など）では、静的リンクされたバイナリ版を展開して使います。手順は[Matlantis環境構築](../../Matlantis/README.md)の「Atomskのインストール」を参照してください。

同じソフトウェアですが、環境によって入手経路が異なります。

### 動作確認

格子定数4.02 Å の fcc Al の構造ファイルを作成します。各原子の座標が記載されたXSFファイルが生成されれば成功です。

```
atomsk --create fcc 4.02 Al aluminium.xsf
```

---

## モデリングの基本操作

### ユニットセルを作成する

結晶方位を指定しない場合（XYZ方向がそれぞれ [100] [010] [001] になります）:

```
atomsk --create fcc 3.6147 Cu ファイル.xsf
```

結晶方位を指定する場合（例: XYZ方向をそれぞれ [121] [-101] [1-11] とする）:

```
atomsk --create fcc 3.6147 Cu orient [121] [-101] [1-11] ファイル.xsf
```

方位を指定する場合、3つのベクトルは互いに直交している必要があります。

### セルを繰り返してスラブを作る

```
atomsk 元のファイル.xsf -duplicate X方向の繰り返し数 Y方向の繰り返し数 Z方向の繰り返し数 出力ファイル.xsf
```

### 指定した領域の原子を削除する

```
atomsk 元のファイル.xsf -select in box Xmin Ymin Zmin Xmax Ymax Zmax -remove-atom select 出力ファイル.xsf
```

上下限には以下が使えます。

| 指定方法 | 例 | 意味 |
|---|---|---|
| 数値 | `10.0` | 端からの距離（Å） |
| 無限大 | `INF` / `-INF` | 制限なし |
| ボックス比 | `0.1*box` | セル寸法に対する比率 |

### 原子の位置をシフトする

```
atomsk 元のファイル.xsf -shift X方向のシフト量 Y方向のシフト量 Z方向のシフト量 出力ファイル.xsf
```

### 2つのモデルを結合する

```
atomsk --merge Z 2 1つ目のファイル.xsf 2つ目のファイル.xsf 出力ファイル.xsf
```

`--merge` の後に、結合する方向（X / Y / Z）と結合するファイル数を指定します。異種材料の界面モデルを作る際に使います。

### ボックスを原子配置に合わせて再定義する

```
atomsk 元のファイル.xsf -rebox 出力ファイル.xsf
```

### 設定ファイルに基づいて変換する

複数の操作をまとめて適用する場合は、設定ファイルを用意します。詳細は[公式ドキュメント](https://atomsk.univ-lille.fr/doc/en/option_properties.html)を参照してください。

設定ファイル（例: `xxx.txt`）の内容:

```
# 全ての原子の位置をZ方向に10オングストローム動かす
displacement function
ux = 0
uy = 0
uz = 10

# シフトした後にスーパーセルを指定した寸法に変更する
conventional
# XYZサイズ
100.0 100.0 100.0
# 各ベクトルの成す角
90.0 90.0 90.0
```

実行します。

```
atomsk 元のファイル -properties xxx.txt 出力ファイル
```

---

## LAMMPS用のファイルに変換する

作成したモデルをLAMMPSで読み込むには、LAMMPS data形式（`.lmp`）に変換します。Atomskは拡張子から出力形式を判別するため、出力ファイル名を `.lmp` にするだけです。

```
atomsk モデル.xsf モデル.lmp
```

生成されたファイルは、LAMMPSの入力ファイル内で `read_data` により読み込みます。

```
read_data モデル.lmp
```

原子スタイル（`atom_style`）を指定して出力することもできます。

```
atomsk モデル.xsf lammps モデル.lmp
```

作成から変換までを一度に行うこともできます。

```
atomsk --create fcc 3.6147 Cu -duplicate 10 10 10 Cu_slab.lmp
```

対応している入出力形式の一覧は[公式ドキュメント](https://atomsk.univ-lille.fr/doc/en/formats.html)を参照してください。

---

## 関連ページ

- [LAMMPSのインストール（基本版）](../LAMMPS_installation/README.md)
- [OVITOの使い方メモ](../OVITO_tips/README.md) — 作成したモデルの確認にも使えます
- [Matlantis環境構築](../../Matlantis/README.md) — Matlantis上でのAtomsk導入
