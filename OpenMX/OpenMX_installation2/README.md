# 阪大スパコンSQUIDでのOpenMXコンパイルとジョブ投入

最終更新: 2026-08 ／ 対象: Ver 3.9.9（Ver 4.0は未検証）

大阪大学D3センターのスーパーコンピュータ SQUID 上で OpenMX をコンパイルし、バッチジョブとして実行する手順です。

> ## ⚠️ SQUIDはサービス終了が予告されています（2026-08時点）
>
> SQUIDは **2027年6月30日** をもってサービス終了が予定されています。後継機の導入予定は未定で、終了後はGPUノード・ベクトルノードと同等の計算資源が提供されない見込みです。
>
> 汎用CPUノードについては、**OCTOPUS**（140ノード）が引き続き利用可能な予定です。OpenMXは汎用CPUノードで実行するため、OCTOPUSへの移行が現実的な選択肢になります。
>
> 新たに大規模計算を計画する場合は、他機関の計算資源やクラウドの利用も含めて早めに検討してください。最新の情報は[D3センターのお知らせ](https://www.hpc.cmc.osaka-u.ac.jp/)で確認できます。

---

## 1. ソースの取得とパッチの適用

この手順はローカル環境と共通です。[OpenMXのインストール](../OpenMX_installation/README.md)の「Ver 3.9.9 の導入」を参照し、SQUIDのホームディレクトリ上で以下まで進めてください。

- ソース（`openmx3.9.tar.gz`）とパッチ（`patch3.9.9.tar.gz`）の取得
- 展開
- `source3.9.9` の作成とパッチの適用

> **注意**: SQUIDでは、oneAPI対応の追加パッチ（伊藤先生ご提供）は使用しません。SQUIDが提供するコンパイラ環境に合わせた設定を、次項のmakefile編集で行います。

---

## 2. コンパイル

### 環境の読み込み

SQUIDでは Environment Modules によって環境設定を行います。汎用CPUノード向けの推奨環境を読み込みます。

```
module load BaseCPU
```

> バージョンを指定せず `BaseCPU` とだけ書くのが公式の案内です。`BaseCPU/2023` のように固定すると、提供が終了した際に動かなくなります。

### makefileの編集

`openmx3.9/source3.9.9/makefile` を以下のように編集します。

```
CC  = mpiicc -O3 -xCORE-AVX512 -ip -no-prec-div -qopenmp -I${MKLROOT}/include/fftw
FC  = mpiifort -O3 -xCORE-AVX512 -ip -no-prec-div -qopenmp
LIB = -L${MKLROOT}/lib/intel64 -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core -lmkl_blacs_intelmpi_lp64 -lpthread -lifcore
```

> **補足**: ローカル環境（oneAPI 2024以降）では、コンパイラが `icx` / `ifx` に置き換わり、MPIラッパも `mpiicx` / `mpiifx` を使います。一方SQUIDの `BaseCPU` 環境では、公式マニュアルが引き続き `mpiicc` / `mpiifort` を案内しています。**ページによってコマンドが違うのは環境の違いによるもので、誤りではありません。**
>
> `-xCORE-AVX512` は SQUID の汎用CPUノード（Intel Xeon Platinum 8368、Ice Lake）向けの最適化指定です。

### ビルド

```
make -j
make install
```

エラーが出る場合は、`module load BaseCPU` が済んでいるか確認してください。

> **要確認**: Ver 4.0 を SQUID でビルドする手順は未検証です。Ver 4.0 の `README.txt` に記載されたコンパイラ指定を、上記の SQUID 向けオプションに読み替えて適用することになります。

---

## 3. ジョブの投入

SQUIDはログインノードで計算を実行しません。ジョブスクリプトを作成してバッチリクエストとして投入します。

### ジョブスクリプトの例

以下は **2ノード・1時間** を使用する場合の例です（ファイル名を `openmx.sh` とします）。

```
#!/bin/bash
#------- qsub option -----------
#PBS -q SQUID                  # 投入するキュー名
#PBS --group=グループ名          # 所属するグループ名（課題ごとに割り当てられます）
#PBS -m b                      # 実行開始時にメールを送信
#PBS -l cpunum_job=76          # 1ノードあたりのCPUコア数
#PBS -b 2                      # 使用するノード数
#PBS -T intmpi                 # Intel MPIを使用
#PBS -l elapstim_req=01:00:00  # 最大実行時間（1時間）
#PBS -v OMP_NUM_THREADS=2      # 1プロセスあたりのOpenMPスレッド数
#------- Program execution -----------
module load BaseCPU            # ベース環境をロード
cd $PBS_O_WORKDIR              # qsub実行時のディレクトリへ移動

mpirun ${NQSV_MPIOPTS} -np 76 ./openmx in.dat -nt 2 > log.txt
```

### 並列数の考え方

SQUIDの汎用CPUノードは **1ノードあたり76コア**（Intel Xeon Platinum 8368、38コア×2基）です。上の例では以下のように対応しています。

| 項目 | 値 | 意味 |
|---|---|---|
| `cpunum_job=76` | 76 | 1ノードあたり76コアを確保 |
| `-b 2` | 2 | 2ノード使用 → 合計152コア |
| `-np 76` | 76 | **総**MPIプロセス数（`-np` はノードあたりではなく総数です） |
| `-nt 2` | 2 | 1プロセスあたり2スレッド |
| `OMP_NUM_THREADS=2` | 2 | 上の `-nt` と一致させます |

76プロセス × 2スレッド = 152スレッドとなり、確保した152コアと一致します。

> **注意**: `OMP_NUM_THREADS` は `-nt` と同じ値にしてください。異なる値を指定すると、意図しないスレッド数で動作したり、コア数を超えて性能が低下したりします。
>
> なお `#PBS -v` で指定した環境変数は全ノードに反映されます。`export` や `setenv` で書くとマスターノードにしか設定されないため、この書き方を使ってください。

### 投入と確認

計算するディレクトリに、コンパイルした `openmx`、入力ファイル、ジョブスクリプトを置いて投入します。

```
qsub openmx.sh
```

| 操作 | コマンド |
|---|---|
| ジョブの状態を確認 | `qstat` |
| ジョブを削除 | `qdel リクエストID` |
| 使用状況の確認 | 利用者ポータルから確認できます |

---

## 参考

- [SQUIDの利用方法（D3センター）](https://www.hpc.cmc.osaka-u.ac.jp/system/manual/squid-use/)
- [OpenMXのインストール（ローカル環境）](../OpenMX_installation/README.md)
- [DFT計算1（インプットファイルの作成）](../OpenMX_calculation/README.md)
