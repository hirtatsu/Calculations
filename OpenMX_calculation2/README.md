## 計算の実行
### ディレクトリの作成とインプットファイルの準備
- openmx3.9/に移動して、計算用の親ディレクトリ(ここではworkdir)を作成する。
```
mkdir workdir
cd workdir
```
- さっそく、test01を計算することにする。test01用の子ディレクトリ作って移動する。
```
mkdir test01
cd test01
```
- ここに、①in.dat(インプットファイル), ②openmx(コンパイルした実行ファイル)を格納する。

### 計算を実行する
- 以下のコマンドを実行する。
```
mpirun -np 8 ./openmx in.dat -nt 2 </dev/null>& log.txt &
# 8, 2: 並列数。環境に応じて適宜変更。
```
- ジョブの状況確認は以下。
```
jobs # 状況確認
bg XXXX # jobsで確認した停止中のジョブ番号XXXXを再開する
kill XXXX # jobsで確認した実行中のジョブ番号XXXXを停止する
```
- 計算中のログファイルの状況確認は以下。元に戻るときはctrl+c。
```
tail -F log.txt
```

- 

