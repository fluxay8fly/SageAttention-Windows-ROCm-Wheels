# Contributing

感谢你为 SageAttention Windows ROCm Wheels 提交社区构建。

本仓库欢迎社区贡献不同：

- GPU architecture
- Python version
- PyTorch version
- ROCm version
- SageAttention version

组合下的 Windows ROCm SageAttention wheels。

---

## Supported Contributions

欢迎提交例如：

```text
gfx1100
gfx1101
gfx1102
gfx1200
gfx1201
```

以及不同：

```text
Python 3.11
Python 3.12
Python 3.13
PyTorch ROCm versions
ROCm versions
SageAttention versions
```

---

## Contribution Types

目前接受：

1. 新的预编译 wheel
2. 已有 wheel 的实机验证结果
3. 新 GPU architecture 支持
4. 新 PyTorch / ROCm 版本构建
5. BUILDING 文档修正
6. 安装和兼容性说明修正
7. 已知问题报告

---

# Wheel Submission Requirements

提交新的 wheel 时必须提供完整构建信息。

至少包括：

```text
SageAttention version:
Source repository:
Source commit:
Python version:
PyTorch version:
ROCm version:
GPU:
GPU architecture:
Windows version:
Compiler:
Wheel filename:
SHA256:
```

示例：

```text
SageAttention version: 2.2.0
Source repository: thu-ml/SageAttention / gfx12 branch
Source commit: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Python version: 3.12.10
PyTorch version: 2.9.1+rocm7.2.1
ROCm version: 7.2.1
GPU: AMD Radeon RX 9070
GPU architecture: gfx1201
Windows version: Windows 11 x64
Compiler: AMD clang 22 + Visual Studio 2022
Wheel filename: sageattention-xxxxxxxx.whl
SHA256: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

# Required Tests

提交 wheel 前至少完成以下测试。

## Python

```bash
python --version
```

---

## PyTorch / ROCm

```bash
python -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

请提交完整输出。

---

## SageAttention Import

```bash
python -c "import sageattention; print(sageattention.__file__)"
```

---

## Native Extension

如果 wheel 包含 gfx12 native backend：

```bash
python -c "import torch; import sageattention._qattn_gfx12_native as m; print('GFX12 native loaded:', m)"
```

---

## Native Backend Status

如果适用：

```bash
python -c "from sageattention.core import GFX12_NATIVE_ENABLED; print('GFX12_NATIVE_ENABLED =', GFX12_NATIVE_ENABLED)"
```

预期：

```text
GFX12_NATIVE_ENABLED = True
```

---

# Hardware Validation

如果实际在 GPU 上运行过，请提供：

```text
Tested GPU:
GPU architecture:
Application:
Workload:
Result:
Known issues:
```

例如：

```text
Tested GPU: AMD Radeon RX 9070
GPU architecture: gfx1201
Application: ComfyUI
Workload: MiniMax H3
Result: Working
Known issues: None observed
```

---

# Build Status

提交的 wheel 会按照验证程度标记状态。

| Status | Meaning |
|---|---|
| ✅ Maintainer Tested | 仓库维护者已在对应硬件实机验证 |
| ✅ Community Tested | 提交者已报告在对应硬件正常运行 |
| ⚠️ Build Only | 编译成功，但未完成实际 GPU workload 验证 |
| ❌ Broken | 已知无法正常导入或运行 |

> [!NOTE]
> `Community Tested` 不等于维护者独立验证。

---

# SHA256

所有 binary wheel 必须提供 SHA256。

PowerShell：

```powershell
Get-FileHash .\sageattention-xxx.whl -Algorithm SHA256
```

示例：

```text
Algorithm : SHA256
Hash      : XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Path      : sageattention-xxx.whl
```

提交时请提供完整 64-character SHA256。

---

# Source Commit

必须提供构建 wheel 所使用的准确 source commit。

例如：

```text
66f5e64c9e36084c863a4480e570069245e58f90
```

不建议只写：

```text
latest
main
master
gfx12 branch
```

因为这些引用未来可能发生变化。

准确 commit SHA 有助于：

- 重现构建
- 对比不同 wheel
- 排查 bug
- 确认 binary 来源

---

# Wheel Naming

推荐在文件或 Release Asset 名称中体现关键兼容信息。

例如：

```text
sageattention-2.2.0+gfx1201.rocm721.torch291-cp312-cp312-win_amd64.whl
```

至少应该能够明确判断：

```text
SageAttention version
Python version
PyTorch version
ROCm version
GPU architecture
Windows architecture
```

> [!NOTE]
> Python wheel 标准 tag，例如：
>
> ```text
> sageattention-2.2.0+gfx1201.rocm721.torch291-cp312-cp312-win_amd64.whl
> ```
>
> 并不会表示 PyTorch、ROCm 或 GPU architecture 兼容性。
>
> 因此发布说明中仍然必须单独标注这些信息。

---

# Binary Distribution

大型 `.whl` 文件不建议直接提交到 Git repository history。

推荐：

```text
Source repository
    README.md
    BUILDING.md
    CONTRIBUTING.md
    LICENSE

GitHub Releases
    .whl
    SHA256
    build information
```

社区贡献者可以先通过 Issue / Pull Request 提供：

- 构建信息
- 测试输出
- SHA256
- source commit
- wheel 信息

审核完成后再加入对应 GitHub Release。

---

# Do Not Submit Standalone `.pyd` Files

不推荐仅提交：

```text
_qattn_gfx12_native.cp312-win_amd64.pyd
```

作为正式发布物。

原因包括：

- 缺少完整 Python package
- 缺少 package metadata
- pip 无法正确管理
- Python code 与 native binary 可能不是同一个 commit
- 容易出现 ABI mismatch
- 无法清楚表达 PyTorch / ROCm compatibility

正式发布优先使用：

```text
.whl
```

---

# Binary Compatibility

Native extension 与以下环境存在二进制兼容关系：

```text
Python
PyTorch
ROCm
GPU architecture
Windows architecture
Compiler / runtime
```

请不要声称一个 wheel：

```text
works on all AMD GPUs
works on all ROCm versions
works on all PyTorch versions
```

除非已经完成相应验证。

建议只声明：

```text
Built against
```

和：

```text
Tested on
```

的实际环境。

---

# Security Requirements

Wheel 是可以执行 native code 的二进制文件。

因此：

- 不接受来源无法说明的 wheel
- 不接受 source commit 无法确认的构建
- 不接受缺少 SHA256 的 binary
- 不接受经过未知修改但未说明来源的 binary
- 不接受声称来自第三方但无法确认 source 的 binary

仓库维护者可能拒绝任何无法合理验证来源的 binary submission。

---

# Licensing

提交者必须确认：

1. 构建所使用的源码允许 binary redistribution
2. 上游 License 要求得到保留
3. 必要 attribution 得到保留
4. 提交者有权分发该 binary

对于 SageAttention 派生构建，应保留对应上游许可信息。

---

# Submission Template

提交新的 wheel 时可以直接复制以下模板：

```markdown
## Wheel Information

**SageAttention version:**

**Source repository:**

**Source commit:**

**Python version:**

**PyTorch version:**

**ROCm version:**

**GPU:**

**GPU architecture:**

**Windows version:**

**Compiler:**

**Wheel filename:**

**SHA256:**

---

## Build Command

```text
Paste build command here
```

---

## PyTorch / ROCm

```text
Paste output of:

python -c "import torch; print(torch.__version__); print(torch.version.hip)"
```

---

## SageAttention Import

```text
Paste output here
```

---

## Native Backend Test

```text
Paste output here
```

---

## Hardware Validation

**Tested GPU:**

**Application:**

**Workload:**

**Result:**

**Known issues:**

---

## Additional Notes

Add any additional information here.
```

---

# Pull Requests

对于文档、构建脚本和 compatibility table 修改，欢迎直接提交 Pull Request。

请尽量：

- 一次 PR 只处理一个明确主题
- 描述修改原因
- 提供相关测试信息
- 避免无关格式化修改
- 不删除其他 community build 的验证信息

---

# Corrections

如果发现某个 wheel：

- 无法导入
- ABI 不兼容
- GPU architecture 标注错误
- SHA256 不一致
- Release metadata 错误
- 实际 workload 无法运行

欢迎提交 Issue。

请附带：

```text
Python version
PyTorch version
ROCm version
GPU
GPU architecture
Wheel filename
完整错误信息
```

---

# Code of Conduct

请保持技术讨论聚焦于：

- 构建
- 兼容性
- 性能
- Bug
- 文档

对于不同硬件和软件环境的结果，请优先提供可验证的日志和测试数据。
