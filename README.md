# SageAttention Windows ROCm Wheels

Unofficial prebuilt SageAttention wheels for Windows + AMD ROCm.

本仓库用于收集和分享 SageAttention 在 Windows ROCm 环境下编译的预构建 wheel，主要面向 AMD Radeon GPU 用户，尤其是 RDNA3 / RDNA4。

> [!WARNING]
> 本项目不是 SageAttention 官方项目。
>
> 预编译 wheel 与 Python、PyTorch、ROCm、GPU Architecture 等环境存在兼容性要求。
>
> 请务必选择与自身环境匹配的版本。

---

## Available Wheels

| GPU Arch | GPU | Python | PyTorch | ROCm | SageAttention | Status |
|---|---|---|---|---|---|---|
| gfx1201 | Radeon RX 9070 | 3.12 | 2.9.1+rocm7.2.1 | 7.2.1 | 2.2.0 / gfx12 native backend | ✅ Maintainer Tested |

### Status

| Status | Meaning |
|---|---|
| ✅ Maintainer Tested | 由仓库维护者在对应 GPU 上实机验证 |
| ✅ Community Tested | 由社区提交者报告已在对应硬件验证 |
| ⚠️ Build Only | 已成功构建，但未完成对应硬件运行验证 |
| ❌ Broken | 已知无法正常加载或运行 |

---

## Verified Environment

### gfx1201 / RDNA4

当前已验证环境：

| Item | Version |
|---|---|
| GPU | AMD Radeon RX 9070 |
| GPU Architecture | gfx1201 |
| OS | Windows x64 |
| Python | 3.12 |
| PyTorch | 2.9.1+rocm7.2.1 |
| ROCm | 7.2.1 |
| SageAttention | 2.2.0 / gfx12 native backend |
| Native Backend | Working |

验证结果：

```text
GFX12 native loaded: <module 'sageattention._qattn_gfx12_native' ...>

GFX12_NATIVE_ENABLED = True
```

---

## Installation

安装前建议先确认当前 Python、PyTorch 和 ROCm 版本。

### Standard Python

```bash
python --version
python -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

安装 wheel：

```bash
python -m pip install --force-reinstall --no-deps <wheel-file>.whl
```

### ComfyUI Portable

确认环境：

```bat
python_embeded\python.exe --version
python_embeded\python.exe -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

安装 wheel：

```bat
python_embeded\python.exe -m pip install --force-reinstall --no-deps <wheel-file>.whl
```

> [!IMPORTANT]
> 推荐使用 `--no-deps`，避免 pip 在安装 wheel 时自动替换当前 PyTorch / ROCm 相关依赖。

---

## Verify Native Backend

对于 gfx12 native build，建议至少完成以下两项验证。

### Verify native extension

普通 Python：

```bash
python -c "import torch; import sageattention._qattn_gfx12_native as m; print('GFX12 native loaded:', m)"
```

ComfyUI Portable：

```bat
python_embeded\python.exe -c "import torch; import sageattention._qattn_gfx12_native as m; print('GFX12 native loaded:', m)"
```

### Verify gfx12 backend status

普通 Python：

```bash
python -c "from sageattention.core import GFX12_NATIVE_ENABLED; print('GFX12_NATIVE_ENABLED =', GFX12_NATIVE_ENABLED)"
```

ComfyUI Portable：

```bat
python_embeded\python.exe -c "from sageattention.core import GFX12_NATIVE_ENABLED; print('GFX12_NATIVE_ENABLED =', GFX12_NATIVE_ENABLED)"
```

正常情况下应输出：

```text
GFX12_NATIVE_ENABLED = True
```

---

## Compatibility

SageAttention native wheel 与以下组件存在 ABI / binary compatibility 关系：

- Python version
- PyTorch version
- ROCm version
- GPU architecture
- Windows architecture

例如当前 gfx1201 wheel 是基于以下环境构建和验证的：

```text
Python: 3.12
PyTorch: 2.9.1+rocm7.2.1
ROCm: 7.2.1
GPU Architecture: gfx1201
OS: Windows x64
```

即使 wheel 文件名中包含：

```text
cp312-cp312-win_amd64
```

也不代表它能够兼容所有 Python 3.12 + Windows AMD ROCm 环境。

PyTorch、ROCm 或 GPU architecture 不匹配时，可能出现：

```text
ImportError: DLL load failed
```

或者：

```text
找不到指定的程序
```

或者：

```text
找不到指定的模块
```

因此建议优先使用与 Release 中标注环境一致或明确验证过的 wheel。

---

## Why Wheel Instead of Standalone `.pyd`

SageAttention native backend 最终会包含类似：

```text
_qattn_gfx12_native.cp312-win_amd64.pyd
```

但本仓库不推荐直接分发单独的 `.pyd` 作为正式安装方式。

Wheel 可以同时包含：

```text
sageattention/
    __init__.py
    core.py
    ...
    _qattn_gfx12_native.cp312-win_amd64.pyd

sageattention-*.dist-info/
    METADATA
    RECORD
    WHEEL
```

相比手动复制 `.pyd`，wheel 具有以下优势：

- 可以使用 pip 正常安装
- 可以使用 pip 正常卸载
- 包含完整 Python package
- 包含 package metadata
- 更容易保证 Python 代码与 native extension 来自同一构建
- 更适合版本管理和公开分发

推荐流程：

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

## ComfyUI

在 ComfyUI 中使用 SageAttention 时，可根据工作流需求添加启动参数：

```text
--use-sage-attention
```

部分 gfx12 native 工作流或节点可能使用以下环境变量：

### CMD

```bat
set SAGEATTN_QK_DTYPE=INT8
set SAGEATTN_M=128
set SAGEATTN_N=16
set TORCH_BLAS_PREFER_HIPBLASLT=1
set ROCBLAS_USE_HIPBLASLT=1
```

### PowerShell

```powershell
$env:SAGEATTN_QK_DTYPE = "INT8"
$env:SAGEATTN_M = "128"
$env:SAGEATTN_N = "16"
$env:TORCH_BLAS_PREFER_HIPBLASLT = "1"
$env:ROCBLAS_USE_HIPBLASLT = "1"
```

> [!NOTE]
> 具体参数应以所使用的 SageAttention backend、ComfyUI 节点和工作流要求为准。

---

## Wheel Naming

为方便识别不同兼容环境，Release Asset 名称建议包含关键版本信息。

例如：

```text
sageattention-2.2.0+gfx1201.rocm721.torch291-cp312-cp312-win_amd64.whl
```

详细命名和提交要求请参考：

[CONTRIBUTING.md](CONTRIBUTING.md)

---

## Build Instructions

如果需要自行编译，请参考：

[BUILDING.md](BUILDING.md)

其中包含：

- ROCm SDK development components
- `rocm_sdk init`
- `hipcc`
- `clang-cl`
- Visual Studio 2022 x64 Native Tools
- `PYTORCH_ROCM_ARCH`
- gfx1201 target
- ComfyUI embedded Python development headers
- `Python.h`
- `python312.lib`
- Wheel 构建流程
- 常见错误排查
- ABI compatibility 说明

---

## Community Builds

欢迎提交其他 Windows ROCm 环境下编译的 SageAttention wheels。

包括但不限于不同：

- GPU architecture
- GPU model
- Python version
- PyTorch version
- ROCm version
- SageAttention version

例如：

```text
gfx1100
gfx1101
gfx1102
gfx1200
gfx1201
```

以及：

```text
Python 3.11
Python 3.12
Python 3.13
Different PyTorch ROCm versions
Different ROCm versions
```

详细提交要求、验证规范和 binary contribution 规则请参考：

[CONTRIBUTING.md](CONTRIBUTING.md)

---

## Releases

预编译 wheel 建议通过 GitHub Releases 发布，而不是直接提交到 Git repository history。

推荐结构：

```text
Repository
├── README.md
├── BUILDING.md
├── CONTRIBUTING.md
└── LICENSE

GitHub Releases
├── Wheel files
├── Build information
├── Compatibility information
└── SHA256
```

这样可以避免大型 binary 文件长期占用 Git 历史，同时方便按不同 Python / PyTorch / ROCm / GPU architecture 分类发布。

---

## Security Notice

Wheel 文件包含可执行 native code。

社区提交的 wheel 不代表已经经过安全审计。

安装第三方 wheel 前建议：

- 确认 Release 来源
- 确认构建环境
- 确认对应 source commit
- 核对发布页面提供的 SHA256
- 避免安装来源不明的 binary

仓库维护者不保证第三方 community build 的安全性、稳定性或兼容性。

---

## Upstream

本仓库提供以下项目的非官方 Windows ROCm binary builds：

- SageAttention
- THU-ML
- AMD ROCm / TheRock

本仓库不替代任何上游项目，也不提供上游项目的官方支持。

---

## Credits

感谢：

- THU-ML / SageAttention contributors
- SageAttention gfx12 native backend contributors
- AMD ROCm / TheRock contributors
- Community build contributors

---

## License

SageAttention is licensed under the Apache License 2.0.

This repository distributes unofficial Windows ROCm builds derived from SageAttention under the same Apache License 2.0.

See:

[LICENSE](LICENSE)

本项目与 THU-ML、SageAttention 官方项目以及 AMD 不存在官方隶属或官方支持关系。

---

## Disclaimer

This project is unofficial.

All trademarks, project names, GPU names, and software names belong to their respective owners.

Use community-provided binary wheels at your own risk.
