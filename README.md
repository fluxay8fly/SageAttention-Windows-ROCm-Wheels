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

状态说明：

| Status | Meaning |
|---|---|
| ✅ Maintainer Tested | 由仓库维护者在对应 GPU 上实机验证 |
| ✅ Community Tested | 由社区提交者报告已在对应硬件验证 |
| ⚠️ Build Only | 已成功构建，但未在对应硬件完成运行验证 |
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

## Installation

安装前建议先确认当前 Python、PyTorch 和 ROCm 版本。

普通 Python 环境：

```bash
python --version
python -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

ComfyUI Portable：

```bash
python_embeded\python.exe --version
python_embeded\python.exe -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

安装 wheel：

```bash
python -m pip install --force-reinstall --no-deps <wheel-file>.whl
```

ComfyUI Portable：

```bash
python_embeded\python.exe -m pip install --force-reinstall --no-deps <wheel-file>.whl
```

---

## Verify Native Backend

对于 gfx12 native build，可使用以下命令进行验证。

### Verify native extension

普通 Python：

```bash
python -c "import torch; import sageattention._qattn_gfx12_native as m; print('GFX12 native loaded:', m)"
```

ComfyUI Portable：

```bash
python_embeded\python.exe -c "import torch; import sageattention._qattn_gfx12_native as m; print('GFX12 native loaded:', m)"
```

### Verify gfx12 backend status

普通 Python：

```bash
python -c "from sageattention.core import GFX12_NATIVE_ENABLED; print('GFX12_NATIVE_ENABLED =', GFX12_NATIVE_ENABLED)"
```

ComfyUI Portable：

```bash
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

不建议在明显不同的 PyTorch / ROCm 环境中强行安装。

如果版本不匹配，可能出现以下错误：

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

这些错误通常与 Python / PyTorch / ROCm ABI 不兼容有关。

---

## ComfyUI

在 ComfyUI 中使用 SageAttention 时，可根据工作流需求添加启动参数：

```text
--use-sage-attention
```

对于 gfx12 native backend，可根据对应节点或工作流需要设置环境变量。

CMD：

```bat
set SAGEATTN_QK_DTYPE=INT8
set SAGEATTN_M=128
set SAGEATTN_N=16
set TORCH_BLAS_PREFER_HIPBLASLT=1
set ROCBLAS_USE_HIPBLASLT=1
```

PowerShell：

```powershell
$env:SAGEATTN_QK_DTYPE = "INT8"
$env:SAGEATTN_M = "128"
$env:SAGEATTN_N = "16"
$env:TORCH_BLAS_PREFER_HIPBLASLT = "1"
$env:ROCBLAS_USE_HIPBLASLT = "1"
```

---

## Wheel Naming

建议 wheel 文件名包含关键兼容性信息。

例如：

```text
sageattention-2.2.0+gfx1201.rocm721.torch291-cp312-cp312-win_amd64.whl
```

推荐包含：

```text
SageAttention version
GPU architecture
ROCm version
PyTorch version
Python version
Windows architecture
```

---

## Build Instructions

详细的 Windows + ROCm 编译说明请参考：

[BUILDING.md](BUILDING.md)

其中包括：

- ROCm SDK devel 安装
- `rocm_sdk init`
- `gfx1201` 编译目标
- VS2022 x64 Native Tools
- `hipcc`
- `clang-cl`
- `PYTORCH_ROCM_ARCH`
- ComfyUI embedded Python 缺少 `Python.h`
- ComfyUI embedded Python 缺少 `python312.lib`
- Wheel 构建方法
- 常见错误排查

---

## Security Notice

Wheel 文件属于可执行二进制代码。

社区提交的 wheel 不代表已经经过安全审计。

安装前建议：

- 确认提交者提供的构建环境
- 核对 SHA256
- 确认 wheel 来源
- 避免安装来源不明的二进制文件

仓库维护者不保证第三方 wheel 的安全性、稳定性或兼容性。

---

## Upstream

本仓库基于以下上游项目：

- SageAttention
- THU-ML
- AMD ROCm / TheRock

本仓库仅提供非官方 Windows ROCm 构建与社区分发。

---

## Credits

感谢：

- THU-ML / SageAttention
- SageAttention gfx12 native backend contributors
- AMD ROCm / TheRock contributors
- Community build contributors

---

## License

SageAttention is licensed under the Apache License 2.0.

This repository distributes unofficial Windows ROCm builds derived from SageAttention under the same Apache License 2.0.

See:

[LICENSE](LICENSE)

本项目与 THU-ML、SageAttention 官方项目以及 AMD 不存在官方隶属或支持关系。

---

## Disclaimer

This project is unofficial.

All trademarks, project names, GPU names, and software names belong to their respective owners.

Use community-provided binary wheels at your own risk.
