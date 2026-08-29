# 材料科学計算の環境構築と実行手順

最終更新: 2026-08

材料科学計算を行うための計算環境の構築手順と、各ソフトウェアの利用手順をまとめた記録です。分子動力学法（LAMMPS）と第一原理計算（OpenMX）を中心に、そこに至るまでのLinux環境の整備、並列計算環境、スーパーコンピュータの利用までを扱っています。

想定している読者は、研究室に配属されて計算を始める学生と、同種の計算に取り組む研究者の方です。

各ページは、特定の環境で実際に作業した記録をもとにしています。手順をそのまま実行できるとは限らず、お使いの環境に応じた読み替えが必要です。バージョン番号やダウンロードURLは可能な範囲で固定を避けていますが、記載時点の情報である点にご留意ください。各ページの冒頭に最終更新時期と対象バージョンを記載しています。

作成者: Hiroaki Tatsumi（[hiroakitatsumi.com](https://hiroakitatsumi.com)）
所属: 大阪大学 接合科学研究所 微細接合学分野（The University of Osaka, Joining and Welding Research Institute）

---

## 1. 計算環境の準備

### Windows（WSL2）環境の準備
- [WSL2のインストール](./Setting_WSL/WSL2_installation/README.md)

### Mac環境の準備
- [Homebrewのインストール](./Setting_mac/Homebrew_installation/README.md)
- [X Window Systemのセットアップ](./Setting_mac/Xwindowsystem_mac_installation/README.md) — サーバ上のGUIアプリをMacで表示するにはXQuartzが必要です。
- [Pythonの環境構築](./Setting_mac/Python_mac_installation/README.md)

### Linux環境の整備
- [研究室NASのマウント](./Setting_WSL/NASMount/README.md)
- [MPI環境の構築（Intel oneAPI）](./Setting_WSL/MPI/README.md)
- [GPGPU環境の構築（CUDA on WSL）](./Setting_WSL/GPGPU/README.md)
- [Pythonの環境構築（miniforge版）](./Setting_WSL/Python_installation2/README.md) — 推奨
- ~~[Pythonの環境構築（pyenv, venv版）](./Setting_WSL/Python_installation/README.md)~~ — 非推奨。miniforge版に一本化しました。

### 研究室の計算サーバ
- [計算サーバへの接続方法](./Server_setting/Howtoaccess_server/README.md)
- [sshの初期設定（サーバ側）](./Server_setting/SSH_installation/README.md)

---

## 2. 分子動力学（MD）シミュレーション: LAMMPS

- [LAMMPSのインストール（基本版）](./LAMMPS/LAMMPS_installation/README.md) — 初めてビルドする場合はこちらから
- [LAMMPS環境構築ガイド（CPUサーバー / NVIDIA GPU / Kokkos / AMD APU 統合版）](./LAMMPS/LAMMPS_installation2/README.md)
- [Atomskのインストールと使い方](./LAMMPS/Atomsk_installation/README.md) — 計算モデルの作成
- [OVITOの使い方メモ](./LAMMPS/OVITO_tips/README.md) — 計算結果の可視化
- [富岳を用いたLAMMPS計算](./LAMMPS/FUGAKU/README.md)

---

## 3. 密度汎関数理論（DFT）に基づく第一原理計算: OpenMX

Ver 3.9.9 と Ver 4.0 の両方を対象としています。

- [OpenMXのインストール](./OpenMX/OpenMX_installation/README.md)
- [DFT計算1（インプットファイルの作成）](./OpenMX/OpenMX_calculation/README.md)
- [DFT計算2（計算の実行）](./OpenMX/OpenMX_calculation2/README.md)
- [DFT計算3（計算結果の解析）](./OpenMX/OpenMX_calculation3/README.md)
- [阪大スパコンSQUIDでのOpenMXコンパイル](./OpenMX/OpenMX_installation2/README.md) — SQUIDは2027年6月末でサービス終了予定

---

## 4. その他の計算・解析ツール

### Matlantis（汎用原子レベルシミュレータ）
- [Matlantis環境構築](./Matlantis/README.md)

### 2次元画像から3次元構造を予測する（GAN: 敵対的生成ネットワーク）
- [SliceGAN用の画像前処理](./SliceGAN/README.md)（本体は [stke9/SliceGAN](https://github.com/stke9/SliceGAN)）

### まず材料科学計算に触れてみたい方へ
- [MateriApps LIVE!のインストール](./Others/MateriAppsLive_installation/README.md) — 環境構築なしで各種計算を試せるLinuxシステム

---

## 5. 参考

- [Linux作業Tips](./Others/Tips/README.md) — ユーザ管理、ストレージの増設、ファイル操作などの備忘録
- [Gitの設定](./Others/Git/README.md)

---

## ライセンス

本リポジトリの文書は [Creative Commons 表示 4.0 国際 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/deed.ja) の下で提供されています。出典を明示していただければ、複製・改変・再配布が可能です。

出典表記の例:
> Hiroaki Tatsumi, "材料科学計算の環境構築と実行手順", https://github.com/hirtatsu/Calculations

なお、各ページで紹介している外部ソフトウェアには、それぞれ個別のライセンスが適用されます。
