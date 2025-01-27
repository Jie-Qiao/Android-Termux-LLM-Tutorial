### 在 Termux 上本地运行 LLM 的教程

本教程将指导你如何在 Termux 环境中安装并运行本地语言模型（LLM）。我们将使用 `llama.cpp` 项目，并通过 Vulkan 驱动来加速 GPU 计算。

---

### 1. 准备工作

首先，确保 Termux 可以访问外部存储空间：

```bash
termux-setup-storage
```

此命令将允许 Termux 访问设备的存储空间，以便下载和保存必要的文件。

---

### 2. 安装必要的软件包

更新包管理器并安装所需的依赖：

```bash
pkg update && pkg install tur-repo x11-repo vulkan-tools shaderc
```

这些包包括 Vulkan 工具链和其他必要的库。

---

### 3. 安装 Vulkan 驱动

为了使用 GPU 加速，我们需要安装 Vulkan 驱动。你可以从以下链接下载驱动：

[下载 Vulkan 驱动](https://github.com/Jie-Qiao/Android-Termux-LLM-Tutorial/raw/refs/heads/main/mesa-vulkan-icd-wrapper-dbg_24.2.5-5_aarch64.deb)

下载完成后，找到驱动文件并使用以下命令安装：

```bash
dpkg -i ./mesa-vulkan-icd-wrapper-dbg_24.2.5-7_aarch64.deb
```

安装完成后，验证 Vulkan 是否正常工作：

```bash
pkg install vkmark
vulkaninfo
```

如果一切正常，`vulkaninfo` 将显示 GPU 的相关信息。

---

### 4. 编译 `llama.cpp`

接下来，我们将编译 `llama.cpp` 项目。首先，安装必要的编译工具：

```bash
pkg install git cmake
```

然后，克隆 `llama.cpp` 和 Vulkan 头文件的仓库：

```bash
git clone https://github.com/KhronosGroup/Vulkan-Headers.git
git clone https://github.com/ggerganov/llama.cpp.git
```

进入 `llama.cpp` 目录并开始编译：

```bash
cd ~/llama.cpp
cmake -B build -DGGML_VULKAN=ON -DVulkan_LIBRARY=/system/lib64/libvulkan.so -DVulkan_INCLUDE_DIR=~/Vulkan-Headers/include
cmake --build build --config Release
```

**注意**：`DVulkan_LIBRARY` 参数指向系统的 `libvulkan.so` 文件，请确保你的设备支持 Vulkan。

---

### 5. 下载并运行模型

你可以从 Hugging Face 下载 GGUF 格式的模型。例如：

[DeepSeek-R1-Distill-Qwen-1.5B-GGUF](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-1.5B-GGUF)

下载模型文件（如 [DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-1.5B-GGUF/blob/main/DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf)）并将其放入 `llama.cpp` 的 `models` 目录中。

找到编译后bin文件中的llama-cli，运行模型：

```bash
./llama-cli -m /path/to/models/DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf -ngl 33
```

**参数说明**：
- `-ngl 33`：表示将模型的 33 层加载到 GPU。如果你的显存较小，可以适当减少这个数值。
- 如果你想远程调用模型，可以使用 `llama-server` 并设置 `--host 0.0.0.0`。

---
### 6. 参考资料

[llama.cpp/docs/build.md at master · ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md)

[Qualcomm drivers it's here! : r/termux](https://www.reddit.com/r/termux/comments/1gmnf7s/qualcomm_drivers_its_here/)

