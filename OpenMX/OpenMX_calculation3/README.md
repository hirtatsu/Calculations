# DFT計算3: 計算結果の解析

最終更新: 2026-08 ／ 対象: Ver 4.0 および Ver 3.9.9

計算で得られたログファイルや出力ファイルから、必要な情報を取り出して可視化する手順です。

---

## 1. エネルギーの推移を可視化する（gnuplot）

構造最適化やMD計算では、各ステップの全エネルギー（`Utot`）がログに出力されます。その推移をグラフにして、収束の様子を確認します。

### データの抽出

ログファイルから `Utot` を含む行を取り出し、数値の列だけをテキストファイルに書き出します。

```
grep 'Utot  ' log.txt | awk '{print $3}' > Utot_MD.txt
```

処理の内容は以下のとおりです。

1. `grep 'Utot  '` — `Utot` を含む行だけを抽出する（後ろの空白2つは他のキーワードと区別するため）
2. `awk '{print $3}'` — その行の3列目（`Utot` の数値）だけを取り出す
3. `> Utot_MD.txt` — テキストファイルに書き出す

書き出した内容は念のため確認しておきます。

```
head Utot_MD.txt
wc -l Utot_MD.txt      # 行数（＝ステップ数）
```

### gnuplotで表示する

```
gnuplot
```

```
gnuplot> plot 'Utot_MD.txt' with linespoints pointtype 7
gnuplot> exit
```

`plot` は `p`、`with linespoints` は `w lp` のように省略できます。

軸ラベルを付けて画像として保存する場合は以下のようにします。

```
gnuplot> set xlabel 'MD step'
gnuplot> set ylabel 'Total energy (Hartree)'
gnuplot> set terminal png size 800,600
gnuplot> set output 'Utot_MD.png'
gnuplot> plot 'Utot_MD.txt' with linespoints pointtype 7
gnuplot> set output
gnuplot> exit
```

> `set output` を引数なしで実行すると、書き込み中のファイルが閉じられます。これを忘れると画像が正しく保存されないことがあります。

---

## 2. 原子配置を可視化する（AIScope）

AIScopeは、核融合科学研究所の伊藤篤史先生が公開されている可視化ソフトウェアです。

- [AIScope 配布ページ](http://www-fps.nifs.ac.jp/ito/software/aiscope/index.html)

### 可視化用ファイルへの変換

OpenMXが出力する `result.md`（各MDステップの原子配置）は、そのままではAIScopeで読み込めません。セルベクトルの記述を変換して `.md3` ファイルを作成します。

```
sed 's/Cell_Vectors=/\nBOX/g' result.md > result.md3
```

`result.md` 中の `Cell_Vectors=` を、改行 + `BOX` に置き換えています。

### 表示する

AIScopeを起動し、作成した `result.md3` をドラッグ＆ドロップします。

---

## 3. 電子密度などを可視化する

計算で出力される cube 形式のファイル（`result.tden.cube`、`result.dden.cube` など）は、以下のソフトウェアで可視化できます。

| ソフトウェア | 用途 |
|---|---|
| [VESTA](https://jp-minerals.org/vesta/jp/) | 等値面表示、結晶構造との重ね合わせ |
| [OVITO](https://www.ovito.org/) | 原子配置の表示、断面の切り出し |

出力ファイルの一覧は[DFT計算2](../OpenMX_calculation2/README.md)を参照してください。

---

## 4. その他のログ抽出の例

```
grep 'Total Computational Time' log.txt    # 計算にかかった時間
grep 'Chemical potential' log.txt          # 化学ポテンシャル（フェルミ準位）
grep -c 'Utot  ' log.txt                   # ステップ数を数える
```

構造最適化の収束判定に使われる力の最大値など、必要な量に応じてキーワードを変えて抽出してください。ログ中のキーワードは `less log.txt` で全体を眺めると把握できます。

---

## 関連ページ

- [DFT計算2（計算の実行）](../OpenMX_calculation2/README.md)
- [OVITOの使い方メモ](../../LAMMPS/OVITO_tips/README.md)
