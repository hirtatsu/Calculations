# DFT計算2: 計算の実行

最終更新: 2026-08 ／ 対象: Ver 4.0 および Ver 3.9.9

作成した入力ファイルを使って、実際にOpenMXの計算を実行し、結果を確認するまでの手順です。

---

## 1. ディレクトリの準備

OpenMXを展開したディレクトリの下に、計算用の親ディレクトリを作成します（ここでは `workdir` とします）。

```
cd ~/DFT/openmx4.0/      # Ver 3.9.9 の場合は openmx3.9/
mkdir workdir
cd workdir
```

計算ごとに子ディレクトリを作ります。ここでは `test01` とします。

```
mkdir test01
cd test01
```

このディレクトリに以下の2つを置きます。

- `in.dat` — 入力ファイル（[DFT計算1](../OpenMX_calculation/README.md)を参照）
- `openmx` — コンパイルした実行ファイル

> 入力ファイル内の `DATA.PATH` は、このディレクトリから見た相対パスになります。階層が変わると計算が始まらないので注意してください。

---

## 2. 計算の実行

```
mpirun -np 8 ./openmx in.dat -nt 2 < /dev/null > log.txt 2>&1 &
```

- `-np` はMPIプロセス数、`-nt` は1プロセスあたりのOpenMPスレッド数です。**両者の積が物理コア数を超えないようにしてください**（コア数は `nproc` で確認できます）
- `< /dev/null` を付けておくと、バックグラウンド実行時に標準入力待ちで停止するのを防げます
- 末尾の `&` によりバックグラウンドで実行されます

### 進捗の確認

```
tail -F log.txt
```

表示を終了するには `Ctrl+C` を押します。計算はそのまま継続します。

### ジョブの操作

```
jobs        # 実行中・停止中のジョブを確認
bg %1       # 停止中のジョブ番号1を再開する
kill %1     # 実行中のジョブ番号1を停止する
```

### SSH接続を切断しても計算を続ける

計算サーバで長時間の計算を回す場合は、`tmux` を使うと接続し直して状況を確認できます。詳細は[計算サーバへの接続方法](../../Server_setting/Howtoaccess_server/README.md)を参照してください。

`nohup` を使う方法もあります。

```
nohup mpirun -np 8 ./openmx in.dat -nt 2 < /dev/null > log.txt 2>&1 &
```

---

## 3. 計算結果の確認

### ログファイルを読む

```
less log.txt        # 先頭から順に読む（Enterで進む、qで終了）
cat log.txt         # 全体を一度に表示
```

### 特定の情報だけを取り出す

```
grep 'XXXXX' log.txt                      # 指定した文字列を含む行を表示
grep 'Total Computational Time' log.txt   # 計算にかかった時間
grep 'Utot  ' log.txt                     # 系全体の全エネルギー（単位: Hartree）
```

`Utot` の後ろの空白2つは、他のキーワード（`Utot.` を含む行など）と区別するためのものです。

構造最適化やMD計算では `Utot` が各ステップごとに出力されます。その推移をグラフにする方法は[DFT計算3](../OpenMX_calculation3/README.md)を参照してください。

### 収束したか確認する

SCF計算が収束せずに `scf.maxIter` に達した場合、その旨がログに出力されます。結果を使う前に必ず確認してください。

---

## 4. 出力ファイル

計算が終わると、`System.Name` で指定した接頭辞（例では `result`）を持つファイル群が生成されます。当面重要なものは以下です。

| ファイル | 内容 |
|---|---|
| `result.out` | 計算結果の要約（全エネルギー、原子に働く力、電荷など） |
| `result.md` | 各MDステップにおける原子配置 |
| `result.md2` | 最終MDステップにおける原子配置 |
| `result.cif` | 初期構造のCIFファイル |
| `result.tden.cube` | 全電子密度（Gaussian cube形式） |
| `result.dden.cube` | 原子密度から計算した差電子密度 |
| `result.v0.cube` | Kohn-Shamポテンシャル |
| `result.vhart.cube` | Hartreeポテンシャル |
| `result_rst/` | リスタート用ファイルを格納するディレクトリ |

cube形式のファイルは VESTA や OVITO などで可視化できます。

原子配置の可視化（AIScope用ファイルへの変換を含む）と、エネルギー推移のグラフ化については[DFT計算3](../OpenMX_calculation3/README.md)で扱います。

---

## 次に読むページ

- [DFT計算3（計算結果の解析）](../OpenMX_calculation3/README.md)
