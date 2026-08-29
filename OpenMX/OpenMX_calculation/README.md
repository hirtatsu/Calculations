# DFT計算1: インプットファイルの作成

最終更新: 2026-08 ／ 対象: Ver 4.0 および Ver 3.9.9

OpenMXの入力ファイル（`.dat`）の書き方と、主要なキーワードの意味を説明します。

公式マニュアルは以下にあります。キーワードの詳細は必ずこちらを確認してください。

- [Ver 4.0 ユーザーマニュアル（日本語）](https://www.openmx-square.org/openmx_man4.0jp/)
- [Ver 3.9 ユーザーマニュアル](https://www.openmx-square.org/openmx_man3.9/)

---

## 入力ファイルを用意する

最も簡単なのは、既存の結晶構造データを変換する方法です。

1. [Materials Project](https://materialsproject.org/) や [NIMSの結晶構造データベース](https://crystdb.nims.go.jp/) から、目的の結晶構造のCIFファイルを入手する
2. [OpenMX Viewer](https://www.openmx-square.org/viewer/) にドラッグ＆ドロップする

OpenMX Viewerはブラウザ上で動作し、構造の可視化と入力ファイルの生成ができます。

> **注意**: OpenMX Viewer は2026年5月26日に更新されています。以前に使ったことがある場合、ブラウザのキャッシュに残った古いJavaScriptが読み込まれることがあります。**キャッシュを削除するか、ハードリロード（`Ctrl+Shift+R` / `Cmd+Shift+R`）してから利用してください。**

以下では、生成された入力ファイルを自分で調整するために、主要なキーワードを順に説明します。

---

## ファイル名などの設定

```
#
#      File Name      
#

System.CurrrentDirectory         ./    # default=./
System.Name                      result
level.of.stdout                   1    # default=1 (1-3)
level.of.fileout                  1    # default=1 (1-3)
DATA.PATH                        ../DFT_DATA19 
```

- `System.Name` は出力ファイルの接頭辞になります（上の例では `result.out`、`result.md` など）
- `DATA.PATH` には基底関数と擬ポテンシャルが格納されたディレクトリを指定します。**相対パスの階層に注意してください。** 計算を実行するディレクトリから見た位置になります

> **要確認**: Ver 3.9 では `DFT_DATA19` です。Ver 4.0 での名称は、展開したソースの内容を確認してください。

---

## 原子種の定義

```
#
# Definition of Atomic Species
#

Species.Number       1
<Definition.of.Atomic.Species
   Cu   Cu6.0S-s2p2d2   Cu_PBE19S
Definition.of.Atomic.Species>
```

`Species.Number` には計算で扱う原子種の数を指定します。`Definition.of.Atomic.Species` には、原子種ごとに3つの項目を記述します。

### (1) 原子記号

計算に使う名前です。同じ元素でも異なる基底や擬ポテンシャルを割り当てたい場合は、`Cu1`、`Cu2` のように区別できます。

### (2) 基底関数（Pseudo-atomic orbitals）

`Cu6.0S-s2p2d2` は「カットオフ半径6.0 Bohr、ソフト版（S）、s軌道2個・p軌道2個・d軌道2個」を意味します。

選び方は[OpenMX公式のデータベース](https://www.openmx-square.org/vps_pao2019/)を参照します。各元素のページに「Calculation of the total energy as a function of lattice constant in the diamond structure」などのグラフが掲載されており、基底関数の数を増やしたときに全エネルギー曲線がどの程度収束するかを確認できます。

**基底関数を増やすほど精度は上がりますが、計算コストも増加します。** グラフ上で十分に収束している範囲の中から、軌道数の少ないものを選ぶのが実用的です。まず小さめの基底で計算を回し、結果が基底の選択に依存しないことを確認してから本番の条件を決めると確実です。

### (3) 擬ポテンシャル（Pseudopotential）

同じくデータベースから選びます。`Cu_PBE19S` の各部分は以下を意味します。

| 部分 | 意味 |
|---|---|
| `Cu` | 元素 |
| `PBE` | 擬ポテンシャルの生成に用いた交換相関汎関数。後述の `scf.XcType` と揃えます |
| `19` | データベースのバージョン（2019年版） |
| `S` | ソフト版（価電子数が少なく計算が軽い。精度は要確認） |

各元素の価電子数もデータベースに記載されています。次項の座標指定で必要になります。

> **相対論効果の扱いについて**: 2019年版データベースの擬ポテンシャルは、j依存の完全相対論的な形式で生成されています。スピン軌道相互作用を実際に考慮するかどうかは、`scf.SpinOrbit.Coupling`（既定は `off`）で指定します。`off` の場合はj依存の擬ポテンシャルがj縮退度で平均され、スカラー相対論的な扱いになります。重元素を含む系でスピン分裂を議論する場合は `on` にしてください。

---

## 原子の座標

```
#
# Atoms
#

Atoms.Number         32
Atoms.SpeciesAndCoordinates.Unit   FRAC # Ang|AU
<Atoms.SpeciesAndCoordinates
	1	Cu	0.00	0.00	0.00	6.0		5.0
	2	Cu	0.25	0.25	0.00	6.0		5.0
	（以下、原子数分の行が続く）
Atoms.SpeciesAndCoordinates>
Atoms.UnitVectors.Unit             Ang #  Ang|AU
<Atoms.UnitVectors
  7.229500000000  0.000000000000  0.000000000000
  0.000000000000  7.229500000000  0.000000000000
  0.000000000000  0.000000000000  7.229500000000
Atoms.UnitVectors>
```

- `Atoms.Number`: 原子の総数
- `Atoms.SpeciesAndCoordinates.Unit`: 座標の単位。`Ang`（絶対座標、Å）、`AU`（原子単位）、`FRAC`（分率座標。各セルベクトルの長さを1としたときの比率）から選びます
- `Atoms.SpeciesAndCoordinates`: 各行に「通し番号、原子種、座標(x, y, z)、up spinの初期占有数、down spinの初期占有数」を記述します。**最後の2つの合計が、その原子種の価電子数になるようにしてください**（例では 6.0 + 5.0 = 11.0）
- `Atoms.UnitVectors`: 計算セルを構成する単位ベクトル。これでセルの寸法が決まります

<details>
<summary>参考: fcc Cu（格子定数3.61475 Å）の2×2×2セル、32原子の座標例</summary>

```
Atoms.Number         32
Atoms.SpeciesAndCoordinates.Unit   FRAC
<Atoms.SpeciesAndCoordinates
	1	Cu	0.00	0.00	0.00	6.0		5.0
	2	Cu	0.25	0.25	0.00	6.0		5.0
	3	Cu	0.25	0.00	0.25	6.0		5.0
	4	Cu	0.00	0.25	0.25	6.0		5.0
	5	Cu	0.50	0.00	0.00	6.0		5.0
	6	Cu	0.75	0.25	0.00	6.0		5.0
	7	Cu	0.75	0.00	0.25	6.0		5.0
	8	Cu	0.50	0.25	0.25	6.0		5.0
	9	Cu	0.00	0.50	0.00	6.0		5.0
	10	Cu	0.25	0.75	0.00	6.0		5.0
	11	Cu	0.25	0.50	0.25	6.0		5.0
	12	Cu	0.00	0.75	0.25	6.0		5.0
	13	Cu	0.00	0.00	0.50	6.0		5.0
	14	Cu	0.25	0.25	0.50	6.0		5.0
	15	Cu	0.25	0.00	0.75	6.0		5.0
	16	Cu	0.00	0.25	0.75	6.0		5.0
	17	Cu	0.50	0.50	0.00	6.0		5.0
	18	Cu	0.75	0.75	0.00	6.0		5.0
	19	Cu	0.75	0.50	0.25	6.0		5.0
	20	Cu	0.50	0.75	0.25	6.0		5.0
	21	Cu	0.50	0.00	0.50	6.0		5.0
	22	Cu	0.75	0.25	0.50	6.0		5.0
	23	Cu	0.75	0.00	0.75	6.0		5.0
	24	Cu	0.50	0.25	0.75	6.0		5.0
	25	Cu	0.00	0.50	0.50	6.0		5.0
	26	Cu	0.25	0.75	0.50	6.0		5.0
	27	Cu	0.25	0.50	0.75	6.0		5.0
	28	Cu	0.00	0.75	0.75	6.0		5.0
	29	Cu	0.50	0.50	0.50	6.0		5.0
	30	Cu	0.75	0.75	0.50	6.0		5.0
	31	Cu	0.75	0.50	0.75	6.0		5.0
	32	Cu	0.50	0.75	0.75	6.0		5.0
Atoms.SpeciesAndCoordinates>
Atoms.UnitVectors.Unit             Ang
<Atoms.UnitVectors
  7.229500000000  0.000000000000  0.000000000000
  0.000000000000  7.229500000000  0.000000000000
  0.000000000000  0.000000000000  7.229500000000
Atoms.UnitVectors>
```

</details>

---

## 計算条件（SCF）

```
#
# SCF or Electronic System
#

scf.XcType                 GGA-PBE     # LDA|LSDA-CA|LSDA-PW|GGA-PBE
scf.SpinPolarization       off         # On|Off|NC
scf.ElectronicTemperature  300.0       # default=300 (K)
scf.energycutoff           160.0       # default=150 (Ry)
scf.maxIter                100         # default=40
scf.EigenvalueSolver       band        # Recursion|Cluster|Band|DC
scf.Kgrid                  4 4 4       # means 4x4x4
scf.Mixing.Type           rmm-diisk    # Simple|Rmm-Diis|Gr-Pulay|Kerker|Rmm-Diisk
scf.Init.Mixing.Weight     0.010       # default=0.30 
scf.Min.Mixing.Weight      0.001       # default=0.001 
scf.Max.Mixing.Weight      0.200       # default=0.40 
scf.Mixing.History         15          # default=5
scf.Mixing.StartPulay       5          # default=6
scf.criterion             1.0e-7       # default=1.0e-6 (Hartree) 
```

主要なキーワードの目安は以下のとおりです。

| キーワード | 目安 |
|---|---|
| `scf.XcType` | まずは `GGA-PBE`。擬ポテンシャルの名前（`_PBE19`）と揃えます |
| `scf.SpinPolarization` | 磁性材料を扱う場合のみ `on`。非共線磁性は `NC` |
| `scf.energycutoff` | 既定は150 Ry。150〜200から始め、**精度が出なければ**300〜400まで上げます |
| `scf.maxIter` | 収束に必要な回数は系によります。多めに設定しておきます |
| `scf.EigenvalueSolver` | 周期境界条件では `band`。孤立系では `cluster`、大規模系では `DC` |
| `scf.Kgrid` | 2 2 2 または 3 3 3 から始め、エネルギーが収束するまで増やします。セルが大きいほど少なくて済みます |
| `scf.criterion` | **値が小さいほど収束条件が厳しくなります。** 既定の `1.0e-6` より厳しい `1.0e-7` 以下を推奨します |

> **収束しない場合**: `scf.Mixing.Type` を `rmm-diisk` にしたうえで、`scf.Init.Mixing.Weight` を小さく、`scf.Mixing.History` を大きくすると安定することがあります。上の例はその設定になっています。

### エネルギーカットオフとK点は必ず収束確認を

`scf.energycutoff` と `scf.Kgrid` は、値を変えながら全エネルギーの変化を確認し、十分に収束する条件を選んでください。既定値のまま本番計算を行うと、系によっては精度が不足します。

### Ver 4.0 で追加されたキーワード

Ver 4.0 では以下が利用できます。詳細は Ver 4.0 マニュアルを参照してください。

| キーワード | 内容 |
|---|---|
| `scf.eigen.lib` | 固有値ソルバとして `elpa1` または `elpa2` を選択（既定は `elpa1`）。計算速度は両者で同程度とされています |
| `ESM.direction` | 有効遮蔽媒質法（ESM）を適用する方向（`x`/`y`/`z`、既定は `x`） |

---

## 構造最適化・MDの条件

```
#
# MD or Geometry Optimization
#

MD.Type                     Nomd        # Nomd|Opt|DIIS|EF|RF|NVE|NVT_VS|NVT_NH
MD.Opt.DIIS.History          7         # default=7
MD.Opt.StartDIIS             5         # default=5
MD.maxIter                  100        # default=1
MD.TimeStep                1.0         # default=0.5 (fs)
MD.Opt.criterion          1.0e-4       # default=1.0e-4 (Hartree/bohr)
```

- `MD.Type`: 構造最適化を行わない場合は `Nomd`。最適化する場合は `Opt` のほか `DIIS`、`EF`、`RF` などの手法があります。分子動力学を行う場合は `NVE`、`NVT_VS`、`NVT_NH` などを指定します
- `MD.Opt.criterion`: 構造最適化の収束判定（原子に働く力の最大値）。`1.0e-3` で様子を見て、精度が必要なら `3.0e-4` 程度まで下げます

手法ごとの特徴と使い分けはマニュアルを参照してください。

---

## 次に読むページ

- [DFT計算2（計算の実行）](../OpenMX_calculation2/README.md)
- [DFT計算3（計算結果の解析）](../OpenMX_calculation3/README.md)
