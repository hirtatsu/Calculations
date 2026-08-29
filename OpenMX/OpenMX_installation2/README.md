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
