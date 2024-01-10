# 富岳を用いたMDシミュレーション
### ログイン
```
ssh ユーザ名@login.fugaku.r-ccs.riken.jp
```

### LAMMPSの利用
- LAMMPSは[RISTが利用支援の一環として整備したアプリケーションソフトウェア](https://www.hpci-office.jp/for_users/appli_software)のひとつなので、すでに富岳の計算ノードにインストールされている。
- [富岳にインストールされているLAMMPSの詳細](https://www.hpci-office.jp/for_users/appli_software/appli_lammps/lammps_r-ccs_riken-2)

### バッチジョブのスクリプト記述例
- サンプルの入力ファイル (bench/in.lj)を用いた、4MPIプロセス×12スレッド並列（1ノード）の実行例。
```
#!/bin/sh
#PJM -g hp000000*
#PJM -L "node=1"
#PJM -L "rscgrp=small"
#PJM -L "elapse=0:30:00"
#PJM --mpi "max-proc-per-node=4"
#PJM -x PJM_LLIO_GFSCACHE=/vol0004
#PJM -S 
 
export OMP_NUM_THREADS=12
 
. /vol0004/apps/oss/spack/share/spack/setup-env.sh

spack load lammps@20220623.2
export LD_LIBRARY_PATH=/lib64:${LD_LIBRARY_PATH}

llio_transfer `which lmp`
 
mpiexec -np $PJM_MPI_PROC lmp -in in.lj -sf omp -pk omp $OMP_NUM_THREADS -log log.lj
```
*hp000000には使用するグループIDを記入する。

### バッチジョブの投入
- 以下のコマンドを実行（ジョブスクリプトファイル名がlammps.shの場合）。ただしdataディレクトリ内で行うこと。
```
pjsub lammps.sh
```
### ジョブ状態の表示
- 以下のコマンド。
```
pjstat
```
- 
