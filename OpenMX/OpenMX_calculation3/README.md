## 計算結果の各種解析
### エネルギー計算結果のパパっと可視化(gnuplot)
- log.txtファイルから、MD計算時のUtotデータを抽出して、テキストファイルとして出力する
```
# ①ログファイル開いて、②「Utot  」を含む行だけ抽出して、③該当する行の3列目(Utotの数値)のみ抽出して、④テキストファイルとして出力する
cat log.txt | grep "Utot  " | awk '{print $3}' > Utot_MD.txt
```

- そのあとに、その場でGnuplotで可視化
```
gnuplot
gnuplot> p 'Utot_MD.txt' ps 2 pt 2, 'Utot_MD.txt' w l
gnuplot> plot 'Utot_MD.txt' pointsize 2 pointtype 2, 'Utot_MD.txt' with line # 上と同じ意味。省略形。
gnuplot> exit
```

### AIScopeを用いた原子位置の可視化
- AIScopeのダウンロード
