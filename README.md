# MI50-gfx906-ollamma
MI50 (gfx906) 升级验证总结：Ollama v0.18.3 与 v0.20.1
背景

本次验证基于 AMD MI50 32GB (gfx906)，沿用本仓库的核心兼容方案：

ROCm 7.1 runtime

ROCm 6.3 Tensile / rocBLAS 库

GGML_OP_SOLVE_TRI 的 CPU fallback patch

仓库原始目标和构建思路见项目主页与构建指南。

本次完成的工作
1. 从 v0.18.2 升级并验证 v0.18.3

在不改 ROCm / Tensile / gfx906 兼容策略的前提下，只升级 Ollama 版本并重新构建：

qwen3.5:9b：运行正常

qwen3.5:27b：运行正常

hf.co/Jackrong/Qwopus3.5-27B-v3-GGUF:Q4_K_M：仍然无法加载，返回 unable to load model

2. 继续升级并验证 v0.20.1

为了测试 Gemma 4，继续将构建目标升级到 v0.20.1。

过程中遇到的一个实际问题是：容器内直接 git clone 官方源码时多次出现 GitHub TLS / HTTP2 传输中断，因此最终改为：

先在宿主机下载 v0.20.1 源码压缩包

在构建脚本中直接使用本地源码目录

之后成功生成 ollama-v0.20.1-rocm-gfx906.tgz

成功构建 ollama-mi50:v0.20.1

3. 修复扩容后 /dev/kfd 丢失

磁盘扩容并重启后，一度出现：

/dev/dri 存在

但 /dev/kfd 缺失

导致 Docker 容器无法附加 AMD GPU。最终通过下面命令恢复：

sudo modprobe amdgpu

恢复后容器重新获得 ROCm 计算能力。

v0.20.1 实测结果
Qwen 系列

qwen3.5:9b：正常运行

qwen3.5:27b：正常运行

Gemma 4 系列

gemma4:e2b：正常运行，中文生成成功

gemma4:26b：正常运行，中文生成成功

Ollama 官方模型库当前提供的 Gemma 4 标签包括 e2b、e4b、26b、31b 等，支持 Text / Image；同时 v0.20.1 的官方比较页中还包含了针对 Gemma 4 tool call handling 的修复。

仍然失败的模型

hf.co/Jackrong/Qwopus3.5-27B-v3-GGUF:Q4_K_M

在 v0.18.3 与 v0.20.1 下都仍然报：
unable to load model: /models/blobs/sha256-...

这说明该第三方 GGUF 模型的兼容问题不是简单升级到 v0.20.1 就能解决，更像是该模型本身与当前 Ollama / loader 支持链存在单独的不兼容。

结论
结论 1

本仓库的 MI50 (gfx906) 兼容路线不仅能维持在 v0.18.x，也能继续推进到 v0.20.1。

结论 2

对这套硬件和补丁方案来说，v0.20.1 的价值明显高于 v0.19.x，因为已经实测跑通 Gemma 4。

结论 3

当前已验证可用的模型包括：

qwen3.5:9b

qwen3.5:27b

gemma4:e2b

gemma4:26b

结论 4

当前已知仍有问题的模型：

hf.co/Jackrong/Qwopus3.5-27B-v3-GGUF:Q4_K_M
即使升级到 v0.20.1 仍未解决。
建议写入 README 的 Tested Models 小节
## Tested models on MI50 (gfx906)

### Ollama v0.18.3
- qwen3.5:9b
- qwen3.5:27b

### Ollama v0.20.1
- qwen3.5:9b
- qwen3.5:27b
- gemma4:e2b
- gemma4:26b

### Known failing model
- hf.co/Jackrong/Qwopus3.5-27B-v3-GGUF:Q4_K_M
Upgrade Notes

Upgrading from v0.18.2 to v0.18.3 and v0.20.1 is feasible without changing the repository’s core MI50 workaround stack.

The existing gfx906 strategy remains valid:

ROCm 7.1 runtime

ROCm 6.3 Tensile / rocBLAS

GGML_OP_SOLVE_TRI CPU fallback patch

If git clone fails inside the builder container due to GitHub TLS / HTTP2 issues, using a pre-downloaded source tarball on the host is a reliable workaround.

After disk resize / reboot, /dev/kfd may disappear; running sudo modprobe amdgpu restores ROCm device nodes in this environment.

References

本仓库主页：MTLoser / ollama-mi50-rocm71-build

Ollama Gemma 4 官方模型页

Ollama v0.20.0...v0.20.1 官方比较页（含 Gemma 4 tool call handling 修复）

Ollama Gemma 3 标签页（可作为 Gemma 3 27B 量化参考）

本次本地实测日志摘录
