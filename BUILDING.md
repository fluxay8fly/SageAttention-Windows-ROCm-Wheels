# Building SageAttention on Windows ROCm

本文档介绍如何在 Windows + AMD ROCm 环境下从源码编译 SageAttention wheel。

本文档主要基于以下环境完成验证：

| Item | Version |
|---|---|
| GPU | AMD Radeon RX 9070 |
| GPU Architecture | gfx1201 |
| OS | Windows x64 |
| Python | 3.12.10 |
| PyTorch | 2.9.1+rocm7.2.1 |
| ROCm | 7.2.1 |
| SageAttention | 2.2.0 / gfx12 native backend |
| Compiler | AMD clang 22 + Visual Studio 2022 |
| Build Result | Working |

> [!IMPORTANT]
> SageAttention native extension 与 Python、PyTorch、ROCm 和 GPU Architecture 存在二进制兼容性关系。
>
> 建议始终使用最终运行 SageAttention 的同一套 Python / PyTorch / ROCm 环境进行编译。

---

## 1. Requirements

需要准备：

- Windows 10 / Windows 11 x64
- Visual Studio 2022 C++ Build Tools
- AMD ROCm Windows Python packages
- ROCm SDK development components
- PyTorch ROCm
- Git
- Python development headers
- SageAttention source code

对于 gfx12 / RDNA4 native backend，需要使用包含 gfx12 支持的 SageAttention 源码。

例如：

```text
SageAttention PR #368
gfx12 native backend
```

---

## 2. Verify Python and PyTorch

首先确认最终运行 SageAttention 的 Python 环境。

ComfyUI Portable 示例：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe --version
```

确认 PyTorch：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

已验证环境输出类似：

```text
2.9.1+rocm7.2.1
7.2.x
```

如果：

```text
torch.version.hip
```

返回 `None`，说明当前并不是 ROCm PyTorch 环境。

---

## 3. Install ROCm Development Components

运行时 ROCm 包通常不足以编译 HIP extension。

需要安装 ROCm SDK development components，例如：

```text
rocm-sdk-devel
```

安装完成后运行：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -m rocm_sdk init
```

成功后应展开 development SDK。

查询 SDK root：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -m rocm_sdk path --root
```

示例：

```text
D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\_rocm_sdk_devel
```

查询工具目录：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -m rocm_sdk path --bin
```

示例：

```text
D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\_rocm_sdk_devel\bin
```

---

## 4. Verify HIP Compiler

确认 `hipcc`：

```bat
hipcc --version
```

正确环境应输出 AMD clang / HIP 版本，例如：

```text
HIP version: 7.2.x
AMD clang version 22.x
Target: x86_64-pc-windows-msvc
```

如果找不到 `hipcc`，需要将 ROCm SDK bin 加入 `PATH`。

---

## 5. Use Visual Studio x64 Native Tools

建议使用：

```text
x64 Native Tools Command Prompt for VS 2022
```

不要使用未初始化 Visual Studio 编译环境的普通 CMD。

确认：

```bat
where cl
where link
```

两条命令都应找到 Visual Studio 的编译工具。

例如：

```text
...\VC\Tools\MSVC\...\bin\Hostx64\x64\cl.exe
...\VC\Tools\MSVC\...\bin\Hostx64\x64\link.exe
```

---

## 6. ComfyUI Embedded Python Development Files

### Why this is required

ComfyUI Portable 的：

```text
python_embeded
```

通常是精简版 Python。

它可以正常运行 Python，但默认可能没有编译 C/C++ extension 所需要的：

```text
Include\Python.h
libs\python312.lib
```

如果缺失，SageAttention 编译可能出现：

```text
fatal error: 'Python.h' file not found
```

检查：

```powershell
Test-Path "D:\ComfyUI_windows_portable\python_embeded\Include\Python.h"
Test-Path "D:\ComfyUI_windows_portable\python_embeded\libs\python312.lib"
```

如果返回：

```text
False
False
```

需要补充 Python development files。

### Install matching CPython

建议安装与 embedded Python 完全相同版本的官方 CPython。

例如：

```text
ComfyUI embedded Python: 3.12.10
Official CPython:        3.12.10 x64
```

假设完整 Python 安装到：

```text
D:\python312
```

确认：

```powershell
Test-Path "D:\python312\Include\Python.h"
Test-Path "D:\python312\libs\python312.lib"
```

应该均为：

```text
True
```

### Copy development files

复制 headers：

```powershell
Copy-Item "D:\python312\Include\*" `
"D:\ComfyUI_windows_portable\python_embeded\Include\" `
-Recurse -Force
```

创建 `libs`：

```powershell
New-Item "D:\ComfyUI_windows_portable\python_embeded\libs" `
-ItemType Directory `
-Force
```

复制 import libraries：

```powershell
Copy-Item "D:\python312\libs\*" `
"D:\ComfyUI_windows_portable\python_embeded\libs\" `
-Recurse -Force
```

再次确认：

```powershell
Test-Path "D:\ComfyUI_windows_portable\python_embeded\Include\Python.h"
Test-Path "D:\ComfyUI_windows_portable\python_embeded\libs\python312.lib"
```

应为：

```text
True
True
```

> [!WARNING]
> 不要使用不同 Python minor version 的开发文件。
>
> 例如 Python 3.13 的 headers / libraries 不应该用于 Python 3.12 embedded runtime。

---

## 7. Configure Build Environment

### CMD / Visual Studio Native Tools

对于 RX 9070 / RX 9070 XT：

```bat
set PYTORCH_ROCM_ARCH=gfx1201
set MAX_JOBS=1
```

设置 ROCm：

```bat
set "ROCM_HOME=D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\_rocm_sdk_devel"
```

设置 PATH：

```bat
set "PATH=%ROCM_HOME%\bin;%ROCM_HOME%\lib\llvm\bin;D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\_rocm_sdk_core\bin;D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\torch\lib;%PATH%"
```

确认：

```bat
where hipcc
where clang-cl
where cl
where link
```

### PowerShell

PowerShell 环境变量语法不同：

```powershell
$env:PYTORCH_ROCM_ARCH = "gfx1201"
$env:MAX_JOBS = "1"

$env:ROCM_HOME = "D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\_rocm_sdk_devel"
```

然后：

```powershell
$env:PATH = `
"$env:ROCM_HOME\bin;" +
"$env:ROCM_HOME\lib\llvm\bin;" +
"D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\_rocm_sdk_core\bin;" +
"D:\ComfyUI_windows_portable\python_embeded\Lib\site-packages\torch\lib;" +
$env:PATH
```

> [!IMPORTANT]
> CMD 与 PowerShell 不要混用环境变量语法。
>
> CMD：
>
> ```bat
> set PYTORCH_ROCM_ARCH=gfx1201
> ```
>
> PowerShell：
>
> ```powershell
> $env:PYTORCH_ROCM_ARCH = "gfx1201"
> ```

---

## 8. Verify Target Architecture

确认 PyTorch：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

gfx1201 环境变量：

```bat
echo %PYTORCH_ROCM_ARCH%
```

应输出：

```text
gfx1201
```

构建日志中也应该出现类似：

```text
Target AMD GPU architectures: ['gfx1201']
```

---

## 9. Build the Wheel

进入 SageAttention 源码：

```bat
cd /d D:\SageAttention-gfx12
```

建议先清理旧构建：

```bat
rmdir /s /q build 2>nul
rmdir /s /q sageattention.egg-info 2>nul
```

创建输出目录：

```bat
mkdir D:\sage_wheel 2>nul
```

构建：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -m pip wheel --no-build-isolation --no-deps -v . -w D:\sage_wheel
```

推荐保留：

```text
--no-build-isolation
--no-deps
```

原因：

- 使用当前已经配置好的 PyTorch ROCm 环境
- 避免 pip 创建另一套隔离编译环境
- 避免自动替换 PyTorch / ROCm dependencies

成功后：

```bat
dir D:\sage_wheel
```

应看到生成的 `.whl`。

---

## 10. Install the Wheel

安装：

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -m pip install --force-reinstall --no-deps D:\sage_wheel\<wheel-file>.whl
```

---

## 11. Verify SageAttention

### Native extension

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -c "import torch; import sageattention._qattn_gfx12_native as m; print('GFX12 native loaded:', m)"
```

成功示例：

```text
GFX12 native loaded: <module 'sageattention._qattn_gfx12_native' ...>
```

### Native backend

```bat
D:\ComfyUI_windows_portable\python_embeded\python.exe -c "from sageattention.core import sageattn_qk_int8_pv_gfx12_native, GFX12_NATIVE_ENABLED; print('GFX12_NATIVE_ENABLED =', GFX12_NATIVE_ENABLED)"
```

目标：

```text
GFX12_NATIVE_ENABLED = True
```

只有 wheel 构建成功并不代表运行环境一定正确。

建议至少完成上述两项验证。

---

## 12. ComfyUI

根据工作流需要，可以使用：

```text
--use-sage-attention
```

部分 gfx12 native 工作流可能使用：

```bat
set SAGEATTN_QK_DTYPE=INT8
set SAGEATTN_M=128
set SAGEATTN_N=16
set TORCH_BLAS_PREFER_HIPBLASLT=1
set ROCBLAS_USE_HIPBLASLT=1
```

具体参数应以使用的 SageAttention backend / ComfyUI node 为准。

---

# Troubleshooting

## `Python.h` not found

错误：

```text
fatal error: 'Python.h' file not found
```

原因：

ComfyUI embedded Python 缺少 development headers。

检查：

```powershell
Test-Path "D:\ComfyUI_windows_portable\python_embeded\Include\Python.h"
```

解决方法：

使用相同 Python 版本的完整 CPython 提供 `Include` development files。

---

## `python312.lib` not found

检查：

```powershell
Test-Path "D:\ComfyUI_windows_portable\python_embeded\libs\python312.lib"
```

如果不存在，需要从相同版本的完整 Python 安装中复制 `libs`。

---

## `hipcc` not found

检查：

```bat
where hipcc
```

确认已经执行：

```bat
python -m rocm_sdk init
```

并将：

```text
_rocm_sdk_devel\bin
```

加入 `PATH`。

---

## `cl` or `link` not found

检查：

```bat
where cl
where link
```

如果找不到，请使用：

```text
x64 Native Tools Command Prompt for VS 2022
```

---

## `failed to run offload-arch: binary not found`

可能看到：

```text
[WARNING] failed to run offload-arch: binary not found.
```

如果已经显式设置：

```text
PYTORCH_ROCM_ARCH=gfx1201
```

并且构建日志显示：

```text
Target AMD GPU architectures: ['gfx1201']
```

该 warning 本身不一定会阻止构建。

重点应继续检查实际 compiler error。

---

## `Error checking compiler version for clang-cl`

PyTorch 可能输出：

```text
Error checking compiler version for clang-cl
```

如果后续仍能看到：

```text
clang-cl ...
hipcc ...
```

并实际开始编译 source files，则这条信息本身不一定是最终失败原因。

应该继续查找最早出现的：

```text
fatal error:
error:
FAILED:
```

---

## DLL load failed after successful build

例如：

```text
ImportError: DLL load failed
```

或者：

```text
找不到指定的程序
```

可能意味着 wheel 是针对另一套 PyTorch / ROCm ABI 构建的。

检查：

```bat
python -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

并与 wheel 发布说明中的版本进行比较。

---

# Why Not Distribute Only the `.pyd`?

SageAttention native backend 最终确实包含类似：

```text
_qattn_gfx12_native.cp312-win_amd64.pyd
```

但不推荐将单独 `.pyd` 作为主要发布方式。

直接复制 `.pyd` 存在以下问题：

- 不包含完整 Python package
- 不包含 package metadata
- pip 无法正确管理版本
- pip 无法正常卸载
- Python 代码与 native extension 可能来自不同 commit
- 很容易产生 PyTorch / ROCm ABI 不匹配
- `cp312-win_amd64` 并不代表它可以兼容所有 Python 3.12 Windows ROCm 环境

推荐发布：

```text
.whl
```

Wheel 可以同时包含：

```text
sageattention/
    core.py
    __init__.py
    _qattn_gfx12_native.cp312-win_amd64.pyd

sageattention-*.dist-info/
    METADATA
    RECORD
    WHEEL
```

因此推荐流程为：

```text
Source
  ↓
Build against target environment
  ↓
Wheel
  ↓
pip install
  ↓
Runtime verification
```

---

# Reproducibility

发布社区 wheel 时建议记录：

```text
SageAttention commit
Python version
PyTorch version
ROCm version
GPU architecture
Compiler version
Build command
SHA256
```

这样其他用户才能判断该 wheel 是否与自己的环境兼容。

---

# Notes

不同 ROCm / PyTorch 版本之间可能存在 C++ ABI 或 exported symbol 差异。

即使两个 wheel 都显示：

```text
cp312-win_amd64
```

也不能因此认为它们可以跨不同 PyTorch / ROCm 版本通用。

始终优先使用与目标运行环境匹配的构建。
