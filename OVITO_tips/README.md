## OVITOの使い方メモ
### アニメーションへのTimestepの追加
- Viewport layersを開く
- Add layerからText labelを開く
- Text入力欄に、[Timestep]　を入力する
- 以上

### Animationの作成
- Renderingを開く
- Complete animationを選択し、何フレーム間隔で動画出力するか決める。全フレームを使う場合はEvery Nth frame: 1とする。
- Output image sizeを決める。Presetsから選ぶとよい。
- Render outputのSave to fileにチェックを入れて、保存先とファイル名をChooseから入力する。
- Backgroundの色を決める。透明Transparentも可能。
- I render active viewpointをクリック。
- 終わり。

### 断面を見る
- Add modificationからSliceを選択する
- どの面方位に沿って断面を切るか、Normal x,y,zに指数を入力する
- 原点と面の距離を入力する
- Reverse orientationもできる
- 複数のSliceを駆使して任意の箇所のみを可視化することも

### MSD (Mean Square Displacement)を計算する
- dumpファイルを開く
- 初期座標をエクスポートする。0stepを表示しながら、File -> ExportFile。ファイル名:「0」、LAMMPS Dump File形式、Current flame Onlyとして、SelectAllでエクスポートする
- Add modification -> Displacement Vectorsを選ぶ。Refrece configuration sourceにExternal fileを選択し、フォルダアイコンからエクスポートした0stepのデータを読み込む
- File -> Export File。ファイル名:「disp_mag.*」、LAMMPS Dump File形式、Range: 全フレーム＆File sequence、Select allでOK。各フレームにおける全原子の座標とDisplacementが出力される
- Pythonスクリプトでデータ解析する
