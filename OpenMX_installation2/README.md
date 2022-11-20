## OpenMXのコンパイル(阪大スパコンSquid用)
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
