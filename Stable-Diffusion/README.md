# Stable Diffusion
画像生成AIを使ってみましょう

## Stable Diffusionのインストール
### 環境
- Windows11のWSL2上のUbuntu 22.04.2 LTS
- GPUはNVIDIA RTX3060 12GB

### NVIDIA GPUのドライバをWindows上にインストール
- [https://www.nvidia.co.jp/Download/index.aspx?lang=jp](https://www.nvidia.co.jp/Download/index.aspx?lang=jp)
- Game ReadyとStudio、どっちでもOK。Studioの方が安定してる？？
### NVIDIA CUDA ToolkitをUbuntu上にインストール
- WSL上のUbuntu上でnvidia-cuda-toolkitをインストール
'''
sudo apt update
sudo apt upgrade
sudo apt install nvidia-cuda-toolkit
'''

