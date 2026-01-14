## Intel oneAPIを準備する(計算サーバにはあらかじめ導入済みなのでスキップ)
## 最初に必要なパッケージを準備する
```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential
```
### Intel oneAPI Toolkitsで入手する
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

---


## Intel oneapiで入手したコンパイラのPATHを通す
```
cd
vim .bashrc
```
で開いて、最後に行を追加して以下を入力して保存する。
```
source /opt/intel/oneapi/setvars.sh
```
そして、以下のコマンドでPATHを反映させる。
```
source .bashrc
```
ちゃんとインストールできたか確認する。バージョンとか表示されればOK。
```
icx -v
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
さらに、最新パッチ（伊藤先生ご提供）をダウンロード。
```
wget https://github.com/atsushi-m-ito/openmx-patch-oneapi/archive/refs/tags/v1.tar.gz
```

次に、解凍した後にその中に入る。以下のディレクトリが見つかるはず。
```
tar xvfz openmx3.9.tar.gz
cd openmx3.9
ls
```
- DFT_DATA19　←　基底関数＆擬ポテンシャル
- source　←　プログラムのソースファイル
- work　←　サンプルファイル

## ソースにパッチを適用する
まずは、ダウンロードしたパッチファイルをpatchディレクトリに格納しておく。
```
mkdir patch
mv ../patch3.9.9.tar.gz ./patch/
mv ../v1.tar.gz ./patch/
```
### もともとのsourceディレクトリ(Ver.3.9用)にパッチを適用した、source3.9.9を作成する。
```
cp -rp source source3.9.9
cd source3.9.9
tar xvfz ../patch/patch3.9.9.tar.gz
mv kpoint.in ../work/
```
### さらに、最新のパッチを当てたsource3.9.9-v1を作成する。詳細は[こちら](https://qiita.com/pochman/items/1a7b80107850e027ad31))。
```
cd ../
cp -rp source3.9.9 source3.9.9-v1
cd source3.9.9-v1
tar xvfz ../patch/v1.tar.gz --strip-components 1
```

    - （補足）makefileを編集する（2025年7月時点で不要。代わりに以下のパッチを充てる）
    ```
    cd ~/DFT/openmx3.9/source3.9.9
    vim makefile
    ```
    - openmx3.9.9/source/makefileを次のように編集する。
    ```
    MKLROOT = /opt/intel/oneapi/mkl/2025.1/
    CC  = mpiicx -O3 -xHOST -fiopenmp -fcommon -Wno-error=implicit-function-declaration -I${MKLROOT}/include/fftw
    FC  = mpiifx -O3 -xHOST -fiopenmp
    LIB= -L${MKLROOT}/lib/intel64 -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core -lifcore -lmkl_blacs_intelmpi_lp64 -liomp5 -lpthread -lm -ldl
    ```

### Makeする
```
make -j 4 # 数字は使用するCPUコア数
```
エラーが出たら何回も繰り返す。
最後に、以下で終わり。
```
make install
```

---

## Intel oneAPIを使わない場合
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
---


## OpenMXの実行テスト
```
cd ~/DFT/openmx3.9/work/
```
に移動して、以下を実行する
```
mpirun -np 8 ./openmx -runtest -nt 2 > log.txt & # 最後に&をつけることでバックグラウンド実行されます
# -np X (並列数), -nt Y (並列数)
```

※もしIntelMPIのmpirunでバックグラウンドで実行したときに勝手に計算が止まるようなら、以下のようにするといいかも。
```
mpirun -np 8 ./openmx -runtest -nt 2 </dev/null>& log.txt &
```

実行状況は以下で確認、停止、再開できます。
```
jobs # 状況確認
bg XXXX # jobsで確認した停止中のジョブ番号XXXXを再開する
kill XXXX # jobsで確認した実行中のジョブ番号XXXXを停止する
```


## OpenMXの実行
コンパイルして作成したファイル(openmx)と、インプットファイル(xx.dat)を同一ディレクトリに格納したうえで、
```
mpirun -np 8 ./openmx xx.dat -nt 2 </dev/null>& log.txt &
# CPU実行コア数 8とか2とかは環境に応じて
```
を実行する。

実行中のlog.txtファイルを見たいときは
```
tail -F log.txt
```


