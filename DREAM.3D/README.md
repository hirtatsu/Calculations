# DREAM.3D Version 6.5.171をインストール（DREAM3D NXは有料版）
### プレビルト版をダウンロード
[https://dream3d.bluequartz.net/Download/](https://dream3d.bluequartz.net/Download/)

| Operating System	| Notes |
| ---- | ---- |
| MacOS | DREAM3D-6.5.171-OSX.dmg	MacOS 10.15 and greater required, including macOS 11/12/13. Download is a Disk Image |
| MacOS | DREAM3D-6.5.171-OSX.zip	MacOS 10.15 and greater required, including macOS 11/12/13. Download is a Zip file |
| MacOS | DREAM3D-6.5.171-OSX-arm64.dmg	Apple M1 Arm build. Download is a DMG File |
| Windows | DREAM3D-6.5.171-Win64.zip	Windows version 8 or 10 |
| Linux | DREAM3D-6.5.171-Linux-x86_64.tar.gz	Ubuntu 18.04 or Equivelant. Self contained tar archive. |

(2025年5月6日時点)(Windows版でいいかな)

### 使用
- Windows版の場合、Zipファイルを適当なディレクトリで解凍したあと、DREAM3D.exeを実行したら使える。
- 基本的に、個別の処理(Filterという)を順番に並べた一連の処理(Pipelineという)を実行することで、モデル作成やデータ解析を行う。
- BookmarksのPrebuilt pipelinesを参考にするのがわかりやすそう(例えば、Prebuilt pipelines → Workshop → EBSD Reconstruction)。

### [公式マニュアル](https://dream3d.bluequartz.net/Help/)
### [公式Youtubeチャンネル](https://www.youtube.com/channel/UCjeF8pFMzET5ZN3vsBHATpg)

### EBSDからDAMASKの元データ作成の流れ（概要）
- Convert Hexagonal Grid Data to Square Grid Data (TSL - .ang)
- Import Orientation File(s) to H5EBSD
- Import H5EBSD File
- モデル調整(不要なデータの削除、セグメンテーション、微小欠陥部の除去等)
- 必要に応じて可視化(Generate IPF Colors、ITK::Image Writer)
- Export DAMASK Files(以下のファイルが生成される)
  - material.config
  - xxxxx.geom
- Write DREAM.3D Data File
