## Atomskのインストール
MDで計算するモデルを簡単に作成するためのソフトウェア。

公式は[こちら](https://atomsk.univ-lille.fr/tutorial_install.php)

### Ubuntuへのインストール
- 必要なパッケージのインストール
```
sudo apt install make gfortran liblapack-dev
```

- .debファイルのダウンロード [https://atomsk.univ-lille.fr/dl.php](https://atomsk.univ-lille.fr/dl.php)
```
wget https://atomsk.univ-lille.fr/code/atomsk_b0.11.2_amd64.deb
```
- 解凍・インストール
```
sudo dpkg -i atomsk_b0.11.2_amd64.deb
```
- 動作確認。格子定数4.02ÅでFCC構造のAlの構造ファイル.xsfを作成する。各原子の座標が記載されたXSFファイルが作成される
```
atomsk --create fcc 4.02 Al aluminium.xsf
```

### モデリング
- ユニットセルを作成する (結晶方位を指定しない場合 (XYZ方向が[100]方向になる))
```
atomsk --create fcc 3.6147 Cu ファイル名.xsf
```
- ユニットセルを作成する (結晶方位を指定する場合 (例: XYZ方向にそれぞれ[121], [-101], [1-11]の場合))
```
atomsk --create fcc 3.6147 Cu orient [121] [-101] [1-11] ファイル名.xsf
```
- ユニットセルを繰り返してスラブを作る
```
atomsk 元のファイル名.xsf -duplicate X方向の繰り返し数 Y方向の繰り返し数 Z方向の繰り返し数 出来上がったファイル名.xsf
```

- 指定の領域から原子を削除する
```
atomsk 元のファイル名.xsf -select in box X方向の下限 Y方向の下限 Z方向の下限 X方向の上限 Y方向の上限 Z方向の上限-remove-atom select 出来上がったファイル名.xsf
```
X,Y,Z方向の上下限には、① INFか-INF、② 端からの距離[Å]、または、③ Boxに対する比率(例: 0.1*box)が使える。



