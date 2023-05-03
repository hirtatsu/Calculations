# Stable Diffusion
画像生成AIを使ってみましょう

## Stable Diffusionのインストール
### 環境
- Windows11のWSL2上のUbuntu 22.04.2 LTS
- GPUはNVIDIA RTX3060 12GB
- あらかじめUbuntu上にPython環境を構築しておくこと

### NVIDIA GPUのドライバをWindows上にインストール
- [https://www.nvidia.co.jp/Download/index.aspx?lang=jp](https://www.nvidia.co.jp/Download/index.aspx?lang=jp)
- Game ReadyとStudio、どっちでもOK。Studioの方が安定してる？？
### NVIDIA CUDA ToolkitをUbuntu上にインストール
- WSL上のUbuntu上でnvidia-cuda-toolkitをインストール -> [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)
(Operating System: Linux, Architecture: x86_64, Distribution: WSL-Ubuntu, Version: 2.0, Installer Type: どれでも)

- インストールされているか確認する
```
nvidia-smi
```
- nvccも
```
nvcc --version
```

### Stable Diffusion WebUI の設定
- ユーザーの home ディレクトリで git clone コマンドを実行する
```
cd
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```
- webui_user.sh の編集（xformers を有効にするために、webui-user.sh 内の COMMANDLINE_ARGS の先頭の # を消して有効にして、xformers のオプションを入力する
```
export COMMANDLINE_ARGS="--xformers"
```

### 学習済みモデルのダウンロード
- 以下のコマンドを入力
```
wget https://huggingface.co/stabilityai/stable-diffusion-2-1/resolve/main/v2-1_768-ema-pruned.safetensors
```

### Stable Diffusion WebUIの実行（初回はインストールも）
```
cd ~/stable-diffusion-webui
./webui.sh
```
- 表示されたアドレスをコピーしてブラウザで開く




