# GPGPU環境の構築（CUDA on WSL）

最終更新: 2026-08 ／ 対象: CUDA Toolkit 13系、WSL2上のUbuntu 22.04 / 24.04

NVIDIA GPUを計算に使うための環境を、WSL2上のUbuntuに構築します。LAMMPSのGPU版をビルドする際の前提となる環境です。

構成は3段階です。

1. NVIDIAドライバのインストール（**Windows側**）
2. CUDA Toolkitのインストール（**WSL上のUbuntu側**）
3. cuDNNのインストール（機械学習ライブラリを使う場合のみ）

対応バージョンの組み合わせは[Support Matrix](https://docs.nvidia.com/deeplearning/cudnn/latest/reference/support-matrix.html)で確認できます。

---

## 0. 事前準備

WSLを最新にします。PowerShellを開いて以下を実行します。

```
wsl --update
```

WSL上のUbuntuでパッケージを更新します。

```
sudo apt update
sudo apt upgrade -y
```

---

## 1. NVIDIAドライバをWindowsにインストールする

[NVIDIAドライバのダウンロードページ](https://www.nvidia.com/download/index.aspx)から、お使いのGPUに対応したドライバを入手してWindowsにインストールします。

> **重要**: WSL上のUbuntuには**ドライバをインストールしません**。Windows側のドライバがWSLから利用される仕組みになっています。Ubuntu側にドライバを入れると、かえって動作しなくなります。

インストール後、WSL上で以下を実行してGPUが認識されているか確認します。

```
nvidia-smi
```

GPU名とドライバのバージョン、CUDAの対応バージョンが表示されれば成功です。

---

## 2. CUDA ToolkitをWSL上のUbuntuにインストールする

公式手順は[CUDA on WSL User Guide](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)にあります。以下はその要点です。

> ドライバのバージョンとCUDAのバージョンには対応関係があります。[対応表](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)を確認してください。

### リポジトリの登録

CUDAのリポジトリ鍵を `cuda-keyring` パッケージで登録します。

```
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
```

> **補足**: 以前の手順書では `sudo apt-key del 7fa2af80` で古い鍵を削除し、ローカルインストーラの `.deb` を版ごとにダウンロードする方法が案内されていました。`apt-key` はUbuntu 22.04で非推奨、24.04系では削除されているため、現在は上記の `cuda-keyring` を使う方法が正式です。

### Toolkitのインストール

インストール可能なバージョンを確認します。

```
apt list -a cuda-toolkit-*
```

目的のバージョンを指定してインストールします（例: 13.0系）。

```
sudo apt install cuda-toolkit-13-0
```

> **注意**: `sudo apt install cuda` としないでください。`cuda` メタパッケージはNVIDIAドライバまで引き込みます。WSLではWindows側のドライバを使うため、Linux側にドライバを入れると競合します。**Toolkitのみを指す `cuda-toolkit-XX-X` を指定してください。**

### PATHを通す

`~/.bashrc` の末尾に追記します。

```
cd
vim .bashrc
```

以下を追加します。`/usr/local/cuda` はインストールした版へのシンボリックリンクなので、バージョン番号を直接書くよりも版の更新に強くなります。

```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

反映させます。

```
source ~/.bashrc
```

### 確認

```
nvidia-smi      # GPUとドライバ
nvcc --version  # CUDAコンパイラ
```

### CUDA 13世代での変更点

CUDA 13系では以下の変更があります。古いGPUや古い環境を使う場合は注意してください。

- **13.0**: Maxwell / Pascal / Volta 世代のGPUに対するオフラインコンパイルのサポートが終了しました。またUbuntu 20.04のサポートが終了しています。
- **13.1以降**: LinuxパッケージへのWindowsディスプレイドライバの同梱が廃止されました。

これらの世代のGPUを使う場合は、[バージョンアーカイブ](https://developer.nvidia.com/cuda-toolkit-archive)から12系を導入する選択肢もあります。

---

## 3. cuDNNのインストール（機械学習ライブラリを使う場合）

LAMMPSのGPU計算だけであればcuDNNは不要です。PyTorchやTensorFlowなどを使う場合にインストールします。

公式手順は[cuDNN Installation Guide](https://docs.nvidia.com/deeplearning/cudnn/latest/installation/linux.html)、ダウンロードは[cuDNN Downloads](https://developer.nvidia.com/cudnn-downloads)にあります。

前項でCUDAのリポジトリを登録済みであれば、aptから導入できます。インストール可能なパッケージ名を確認します。

```
apt list -a 'cudnn*'
```

CUDAのバージョンに対応するものを選んでインストールします。

```
sudo apt install cudnn
```

> **補足**: 現行のcuDNNは9系です。以前の手順書では8系（`libcudnn8`）をローカルインストーラで導入する方法を案内していましたが、9系ではパッケージ名と導入方法が変わっています。正確なパッケージ名は上記の `apt list` の出力、またはダウンロードページの案内に従ってください。

確認します。

```
dpkg -l | grep cudnn
```

---

## 関連ページ

- [LAMMPS環境構築ガイド（統合版）](../../LAMMPS/LAMMPS_installation2/README.md) — GPU版・Kokkos版のビルド手順
- [WSL2のインストール](../WSL2_installation/README.md)
