在 Arch Linux 上安装 **Stable Diffusion WebUI（AUTOMATIC1111 的 sd-webui）** 可以通过以下几步完成。这个教程假设你有基本的 Arch 使用经验

------

### 第一步：先决条件

确保你的系统中已经安装了以下依赖项：

```
sudo pacman -S git ffmpeg wget unzip
```

如果你使用 NVIDIA 显卡，还需要安装相关驱动和 CUDA：

```
sudo pacman -S nvidia nvidia-utils cuda cudnn
```

AMD 用户可跳过 CUDA，只保证有 mesa 和 rocm 相关包

> [!WARNING]
>
> #### CUDA 和 cuDNN 的区别：

| 名称      | 全称                                | 作用                                                         | 举例                               |
| --------- | ----------------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| **CUDA**  | Compute Unified Device Architecture | NVIDIA 推出的 **通用 GPU 加速平台**，让程序能用显卡运算      | PyTorch 用 CUDA 来加速             |
| **cuDNN** | CUDA Deep Neural Network library    | 基于 CUDA 的 **深度学习加速库**，专门优化神经网络的计算（更快） | Stable Diffusion 用 cuDNN 提升速度 |

 通俗类比：

- **CUDA = 显卡的驱动+操作系统接口**，就像是电脑的“显卡系统”
- **cuDNN = 深度学习专用的外挂插件**，加速神经网络中的矩阵运算（比如卷积、反向传播）

你可以理解为：

```
Stable Diffusion → 用 PyTorch → PyTorch 用 CUDA → CUDA 用 cuDNN → 显卡疯狂加速跑图
```

- nvidia → 显卡驱动
- nvidia-utils → 基础工具（比如 nvidia-smi）
- cuda → CUDA 工具包（带 nvcc 编译器）
- cudnn → cuDNN 库（专门给 AI 加速用）

### 第二步：Python 选择

确保使用的是 Python 3.11。WebUI 不兼容 3.12

1. 如果不是需要手动安装：

   ```
   mkdir -p ~/src && cd ~/src
   wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz
   tar -xzf Python-3.11.0.tgz
   cd Python-3.11.0
   ```

2. 配置安装路径（不会覆盖系统的 python）

   ```
   ./configure --prefix=$HOME/src/python311 --enable-optimizations
   make -j$(nproc)
   make install
   ```

   > [!NOTE]
   >
   > 安装路径在~/src/python311/bin/python3.11 --version
   >
   > **输出应为 Python 3.10.14**
   >
   > make -j <你 CPU 的核心数量>
   >
   > - make是用于根据 Makefile 编译源码的命令（大多数 C/C++ 项目用它来构建）
   >
   > - -j表示「并行编译」，加快速度。
   >
   > - $(nproc)是一个 Shell 命令，返回当前 CPU 的核心数。比如：8
   >
   > - 可以将新安装的Python加入PATH变量：
   >
   >   ```
   >   vim ~/.bashrc
   >   export PATH=$HOME/src/python311/bin:$PATH
   >   ```

### 第三步：克隆 sd-webui 项目

```
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
```

### 第四步：下载模型文件（必须）

Stable Diffusion 的核心模型需要手动下载，比如：

1. 下载 v1.5 模型

```
cd models/Stable-diffusion
wget https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.safetensors
```

也可以换其他 `.ckpt` 或 `.safetensors` 模型。

### 第五步：使用这个3.10.14版本创建虚拟环境

进入 stable-diffusion-webui 目录后：

```
python3.11 -m venv venv
source venv/bin/activate
```

### 第六步：补全依赖

1. 先安装 CUDA 12.1 版本的 torch/torchvision

   ```
   pip3.11 install torch==2.1.2+cu121 torchvision==0.16.2+cu121 --extra-index-url https://download.pytorch.org/whl/cu121
   ```

2. 修改requirements.txt

   ```
   GitPython>=3.1.30
   Pillow>=9.4.0,<11.0.0
   accelerate>=0.18.0
   blendmodes>=2022.9.0
   clean-fid>=0.1.35
   diskcache>=5.6.1
   einops>=0.6.0
   facexlib>=0.2.5
   fastapi>=0.90.1
   gradio==3.41.2
   inflection>=0.5.1
   jsonmerge>=1.8.0
   kornia>=0.6.8
   lark>=1.1.5
   numpy>=1.23.0
   omegaconf>=2.2.3
   open-clip-torch>=2.0.2
   piexif>=1.1.3
   protobuf==3.20.0
   psutil>=5.9.4
   pytorch_lightning>=2.0.0
   requests>=2.28.2
   resize-right>=0.0.2
   safetensors>=0.2.7
   scikit-image>=0.19
   tomesd>=0.1.2
   torchdiffeq>=0.2.3
   torchsde>=0.2.5
   transformers==4.30.2
   pillow-avif-plugin==1.4.3
   xformers==0.0.23
   ```

3. 安装依赖

   ```
   pip3.11 install -r requirements.txt
   ```

### 第七步：启动webui

第一次运行主脚本需要下载很多依赖文件，这里需要科学上网才行

```
proxychains ./webui.sh --xformers
```

> [!CAUTION]
>
> **proxychains** 是我的终端代理程序，这个需要另外设置
>
> **--xformers** 是启动 Stable Diffusion WebUI 的加速参数，让 WebUI 使用 xformers 这个库来提升生成速度、减少显存占用（尤其是在高分辨率/多图场景下）

如果使用 NVIDIA 显卡，默认会启用 CUDA。AMD 用户可加上参数：

```
./webui.sh --skip-torch-cuda-test --precision full --no-half
```

所有依赖都下载完成之后，需要再次运行./webui.sh，但这里就不需要proxychains代理程序了

### 第七步：访问 WebUI

默认监听地址是：

```
http://127.0.0.1:7860
```

你可以通过浏览器访问它

### 第八步：退出

- 按键盘：**CTRL+C**
- 输入：**deactivate**，即可推出venv命令
- 最后：**cd**退出整个目录
