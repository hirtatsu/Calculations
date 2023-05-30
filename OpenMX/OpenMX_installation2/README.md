## OpenMXのインストール準備
### OpenMXをダウンロード、解凍して当該ディレクトリへ移動
任意の場所にDFT用のディレクトリを作成する。例えば「DFT」という名称のディレクトリを作る。その後、作成したディレクトリに移動する。

```
mkdir DFT
cd DFT
```

[OpenMXのWebサイト](http://www.openmx-square.org/)のDownloadの「openmx3.9」で右クリックしてリンクのアドレスをコピー。

そのあとWSLのコマンドラインに戻って、wgetと入力した後右クリックして貼り付けしてEnter。ダウンロードされる(2022/9/22時点では以下)。

```
wget http://t-ozaki.issp.u-tokyo.ac.jp/openmx3.9.tar.gz
```

同様に、「+patch」で右クリックしてリンクのアドレスをコピー。
wget入力して貼り付けしてEnter。ダウンロードされる(2022/9/22時点では以下)。
```
wget http://www.openmx-square.org/bugfixed/21Oct17/patch3.9.9.tar.gz
```
次に、解凍。
```
tar xvfz openmx3.9.tar.gz
```
ディレクトリに入る。
```
cd openmx3.9
```
中には以下のディレクトリが見つかるはず。
```
ls
```
- DFT_DATA19　←　基底関数＆擬ポテンシャル
- source　←　プログラムのソースファイル
- work　←　サンプルファイル

## ソースにパッチを適用する
もともとのsourceディレクトリ(Ver.3.9用)にパッチを適用した、source3.9.9を作成する。
```
mkdir patch
mv ../patch3.9.9.tar.gz ./patch/

cp -rp source source3.9.9
cd source3.9.9
tar xvfz ../patch/patch3.9.9.tar.gz
mv kpoint.in ../work/
```

## OpenMXのコンパイル(阪大スパコンSquid用)
### スパコン側で標準で用意してくれている環境を呼び出す
```
module load BaseCPU/2023
```
### makefileを編集する
openmx3.9/source3.9.9/makefileを次のように編集する。
```
CC = mpiicc -O3 -xCORE-AVX512 -ip -no-prec-div -qopenmp -I${MKLROOT}/include/fftw
FC = mpiifort -O3 -xCORE-AVX512 -ip -no-prec-div -qopenmp
LIB= -L${MKLROOT}/lib/intel64 -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core -lmkl_blacs_intelmpi_lp64 -lpthread -lifcore
```

### makeする
```
make -j
```
エラーが出たら何回も繰り返す。
最後に、以下で終わり。
```
make install
```
## ジョブ投入方法
### ジョブスクリプト
- ジョブスクリプトファイル（ファイル名：openmx.sh）を作成する例。2ノード・1時間使用の場合。
```
#!/bin/bash
#------- qsub option -----------
#PBS -q SQUID            #バッチリクエストを投入するキュー名の指定
#PBS --group=グループ名   #所属するグループ名
#PBS -m b                #バッチリクエスト実行開始時にメールを送信
#PBS -l cpunum_job=76    #使用するCPUコア数の要求値
#PBS -T intmpi
#PBS -b 2		 #使用するノード数
#PBS -l elapstim_req=01:00:00   #ジョブの最大実行時間の要求値  1時間の例
#PBS -v OMP_NUM_THREADS=76
#------- Program execution -----------
module load BaseCPU/2022 #ベース環境をロードします
cd $PBS_O_WORKDIR        #qsub実行時のカレントディレクトリへ移動
mpirun ${NQSV_MPIOPTS} -np 76 ./openmx in.dat -nt 2 > log.txt     #プログラムの実行
```
- 計算したいディレクトリにコンパイルしたopenmxファイル、インプットファイル、ジョブスクリプトファイルを格納したうえで
```
qsub openmx.sh
```
