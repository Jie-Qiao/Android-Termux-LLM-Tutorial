# Android-Termux-LLM-Tutorial
This tutorial will guide you through the process of installing and running a local language model (LLM) in the Termux environment. We will use the llama.cpp project and accelerate GPU computation through the Vulkan driver.

中文版本 [README_CN.md](README_CN.md).

---

### 1. Preparation

First, ensure that Termux has access to external storage:

```bash
termux-setup-storage
```

This command will allow Termux to access your device's storage, enabling the download and saving of necessary files.

---

### 2. Install Required Packages

Update the package manager and install the necessary dependencies:

```bash
pkg update && pkg install tur-repo x11-repo vulkan-tools shaderc
```

These packages include the Vulkan toolchain and other essential libraries.

---

### 3. Install Vulkan Driver

To utilize GPU acceleration, we need to install the Vulkan driver. You can download the driver from the following link:

[Download Vulkan Driver](https://github.com/Jie-Qiao/Android-Termux-LLM-Tutorial/raw/refs/heads/main/mesa-vulkan-icd-wrapper-dbg_24.2.5-5_aarch64.deb)

After downloading, locate the driver file and install it using the following command:

```bash
dpkg -i ./mesa-vulkan-icd-wrapper-dbg_24.2.5-7_aarch64.deb
```

Once installed, verify that Vulkan is functioning correctly:

```bash
pkg install vkmark
vulkaninfo
```

If everything is working, `vulkaninfo` will display information about your GPU.

---

### 4. Compile `llama.cpp`

Next, we will compile the `llama.cpp` project. First, install the necessary compilation tools:

```bash
pkg install git cmake
```

Then, clone the `llama.cpp` and Vulkan headers repositories:

```bash
git clone https://github.com/KhronosGroup/Vulkan-Headers.git
git clone https://github.com/ggerganov/llama.cpp.git
```

Navigate to the `llama.cpp` directory and start the compilation:

```bash
cd ~/llama.cpp
cmake -B build -DGGML_VULKAN=ON -DVulkan_LIBRARY=/system/lib64/libvulkan.so -DVulkan_INCLUDE_DIR=~/Vulkan-Headers/include
cmake --build build --config Release
```

**Note**: The `DVulkan_LIBRARY` parameter points to the system's `libvulkan.so` file. Ensure that your device supports Vulkan.

---

### 5. Download and Run the Model

You can download a GGUF format model from Hugging Face. For example:

[DeepSeek-R1-Distill-Qwen-1.5B-GGUF](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-1.5B-GGUF)

Download the model file (e.g., [DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-1.5B-GGUF/blob/main/DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf)) and place it in the `models` directory of `llama.cpp`.

Locate the `llama-cli` binary in the compiled files and run the model:

```bash
./llama-cli -m /path/to/models/DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf -ngl 33
```

**Parameter Explanation**:
- `-ngl 33`: Loads 33 layers of the model onto the GPU. If your VRAM is limited, you can reduce this number.
- If you want to call the model remotely, you can use `llama-server` and set `--host 0.0.0.0`.

---
### 6. References

[llama.cpp/docs/build.md at master · ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md)

[Qualcomm drivers it's here! : r/termux](https://www.reddit.com/r/termux/comments/1gmnf7s/qualcomm_drivers_its_here/)
