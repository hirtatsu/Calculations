## 最初に必要なパッケージを準備する
```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential
```

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

## OpenMXのコンパイル(方法1: ストレージ容量に余裕のある場合, recommended)
### インストールで必要なパッケージをIntel oneAPI Toolkitsで入手する
```
cd
```

Windows上で[Intel oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.d1jvm6)にアクセスして、Intel oneAPI Base ToolkitとIntel oneAPI HPC Toolkitを以下の通りインストールする。

まず、[Intel oneAPI Base Toolkitのダウンロード](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html)をクリック。

- Operating system: Linux
- Select distribution: Online

を選択して、表示される'Command Line Download'に記載のコードをUbuntu上で実行する。
すると、インストール画面が別ウインドウで立ち上がるので、画面の指示に従ってインストールする。
(カスタムインストールを選択する場合は、Math Carnel Libraryを必ずインストールすること)

続いて、[Intel oneAPI HPC Toolkitのダウンロード](https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit-download.html)をクリック。上と同様にインストールする。
カスタムインストールにて、DPC++/C++ Compiler, MPI Library, Fortran Compilerをインストールする。

### Intel oneapiで入手したコンパイラのPATHを通す
```
cd
vim .bashrc
```
で開いて、最後に行を追加して以下を入力して保存する。

source /opt/intel/oneapi/setvars.sh

そして、以下のコマンドでPATHを反映させる。
```
source .bashrc
```
ちゃんとインストールできたか確認する。バージョンとか表示されればOK。
```
icc -v
```


### makefileを編集する
```
cd DFT/openmx3.9/source3.9.9
vim makefile
```

openmx3.9.9/source/makefileを次のように編集する。
```
MKLROOT = /opt/intel/oneapi/mkl/latest
CC = mpiicc -O3 -ip -no-prec-div -qopenmp -I$(MKLROOT)/include/fftw
FC = mpiifort -O3 -ip -no-prec-div -qopenmp
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

## OpenMXのコンパイル(方法2: ストレージ容量に余裕のない場合)
### インストールで必要なパッケージをUbuntuのaptで入手したあと、makefileを作成する
[こちら](https://qiita.com/pochman/items/f8aecd3ffc7beba1b6d1)を参照。

### makeする
```
make -j
```
エラーが出たら何回も繰り返す。
最後に、以下で終わり。
```
make install
```

## OpenMXのコンパイル(方法3: 阪大スパコンSquid用)
### スパコン側で標準で用意してくれている環境を呼び出す
```
module load BaseCPU/2021
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

## OpenMXの実行

