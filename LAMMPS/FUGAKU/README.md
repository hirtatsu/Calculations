# 富岳を用いたMDシミュレーション

## ログイン（ローカルアカウントでログインする場合）
```
ssh ユーザ名@login.fugaku.r-ccs.riken.jp
```
## ログイン（HPCIアカウントでログインする場合）
### Dockerコンテナからgsisshでログインする
- HPCIアカウントでログインする場合、Dockerコンテナ上からgsisshでログインする。代理証明書が必要。詳細はHPCIのマニュアル参照([https://www.hpci-office.jp/for_users/hpci_info_manuals](https://www.hpci-office.jp/for_users/hpci_info_manuals))
- Dockerの起動確認。Dockerが正常に自動起動していると、以下のコマンドでなんか表示される
```
docker ps
```
- （コンテナが起動していなければ）Dockerコンテナの起動（Ubuntu上の~/hpciworkと、Dockerコンテナ上の/home/hpciuser/workとをバインドマウントする。これにより、コンテナと Ubuntu の間でファイルを共有できる）
```
docker run -d --rm --name gsi-openssh -v ~/hpciwork:/home/hpciuser/work hpci/gsi-openssh:20231128
```
- Dockerコンテナ上のBASHに入る
```
docker exec -i -t gsi-openssh /bin/bash
```
- （コンテナを起動しなおした場合、有効期限が切れ場合は）代理証明書を発行([HPCI証明書発行システム](https://portal.hpci.nii.ac.jp/))して、以下のコマンドを実行する。詳細は[マニュアル](https://www.hpci.nii.ac.jp/gt6/docker/HPCI-Login-noDesktop-win10.html)。
```
myproxy-logon -s portal.hpci.nii.ac.jp -l [HPCI-ID]
```
- 代理証明書の情報確認
```
grid-proxy-info
```

- 富岳にログインする
```
gsissh -p 2222 login.fugaku.r-ccs.riken.jp
```

### 使用後は
- exitでログインサーバ からコンテナの bash に戻る。
- 再度exitでコンテナのbashから抜ける。このときDockerコンテナは停止しない。
- Dockerコンテナを停止したい場合は以下。ただし、コンテナ上に作成、保存したファイルも削除される。
```
docker stop gsi-openssh
```

## HPCI共用ストレージの利用
- クラウドストレージゲートウェイノードへログインする
```
gsissh -p2222 u00000@csgw.fugaku.r-ccs.riken.jp
```
- （必要あれば）マウントする
```
mount.hpci
```
- ディレクトリは以下になるはず
```
/gfarm/hp000000/u000000　（富岳グループID（課題ID）とユーザID）
```

## Python venv環境構築
```
python3 -m venv /home/XXXXXX/test-env/
source /home/XXXXXX/test-env/bin/activate
```


## Open Source Software (OSS)を使う。
- Spackを使って環境を読み込みます。
```
. /vol0004/apps/oss/spack/share/spack/setup-env.sh
```
- ビルド済みパッケージの確認
```
spack find -x
```
- 例えばpipを利用可能にしたい場合
```
spack load py-pip@23.0
```


## LAMMPSの利用
- LAMMPSは[RISTが利用支援の一環として整備したアプリケーションソフトウェア](https://www.hpci-office.jp/for_users/appli_software)のひとつなので、すでに富岳の計算ノードにインストールされている。
- [富岳にインストールされているLAMMPSの詳細](https://www.hpci-office.jp/for_users/appli_software/appli_lammps/lammps_r-ccs_riken-2)

### バッチジョブのスクリプト記述例
サンプルの入力ファイル (bench/in.lj)を用い、グループIDがhp000000の場合。

- 4MPIプロセス×12スレッド並列×1ノードの実行例。
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

- 4MPIプロセス×12スレッド並列×2ノードの実行例。
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

### ジョブの削除
```
pjdel 12345678
```

### 使用状況の確認
- すべて
```
accountj
```
- 過去に実行したジョブごとのノード時間積の使用履歴
```
pjstata
```
- ディスクの使用状況
```
accountd
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

富岳: 4MPIプロセス×12スレッド並列xノード数。
- 1ノードの場合(48CPUコア): 0:00:38
- 2ノードの場合(96CPUコア): 0:00:21
- 4ノードの場合(192CPUコア): 0:00:11
