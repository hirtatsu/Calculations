# 富岳を用いたMDシミュレーション
### ログイン
```
ssh ユーザ名@login.fugaku.r-ccs.riken.jp
```

### LAMMPSの利用
- LAMMPSは[RISTが利用支援の一環として整備したアプリケーションソフトウェア](https://www.hpci-office.jp/for_users/appli_software)のひとつなので、すでに富岳の計算ノードにインストールされている。
- [富岳にインストールされているLAMMPSの詳細](https://www.hpci-office.jp/for_users/appli_software/appli_lammps/lammps_r-ccs_riken-2)

### バッチジョブのスクリプト記述例
サンプルの入力ファイル (bench/in.lj)を用い、グループIDがhp000000の場合。

- 4MPIプロセス×12スレッド並列（1ノード）の実行例。
```
#!/bin/sh
#PJM -g hp000000*
#PJM -L "node=1"
#PJM -L "rscgrp=small"
#PJM -L "elapse=0:10:00"
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

- 4MPIプロセス×12スレッド並列（2ノード）の実行例。
```
#!/bin/sh
#PJM -g hp000000*
#PJM -L "node=2"
#PJM -L "rscgrp=small"
#PJM -L "elapse=0:10:00"
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


### バッチジョブの投入
- 以下のコマンドを実行（ジョブスクリプトファイル名がlammps.shの場合）。ただしdataディレクトリ内で行うこと。
```
pjsub ./lammps.sh
```
### ジョブ状態の表示
- 以下のコマンド。
```
pjstat
```

### 計算時間の比較
計算内容: examples/melt/in.meltをもとに、「region box block 0 50 0 50 0 50」「run 2500」に変更。50,0000原子。

比較環境1: CPUにIntel Core i5 12400F、GPUにNVIDIA GeForce RTX 3060。

- 並列なしの場合: 0:06:59 (Ref.)
- 8並列の場合: 0:01:39 (-76%)
- 12並列の場合: 0:01:18 (-81%)
- GPUの場合: 0:00:29 (-93%)

比較環境2: CPUにIntel Core i7 13700KF、GPUにNVIDIA RTX A4000。

- 並列なしの場合: 0:05:40 (Ref.)
- 12並列の場合: 0:00:52 (-84%)
- 20並列の場合: 0:00:42 (-88%)
- GPUの場合: 0:00:24 (-93%)

比較環境3: CPUにIntel Core i7 13700KF、GPUにNVIDIA RTX 4090。

- GPUの場合: 0:00:19 (-94%)

富岳: 4MPIプロセス×12スレッド並列（1ノード）。
- 1ノードの場合: 0:00:38
- 2ノードの場合: 0:00:21
- 4ノードの場合: 
