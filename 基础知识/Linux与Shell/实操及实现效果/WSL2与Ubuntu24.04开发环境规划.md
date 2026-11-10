---
type: 环境规划
status: 待验证
created: 2026-10-25
updated: 2026-10-26
tags:
  - 具身智能
  - WSL2
  - Ubuntu24.04
  - ROS2
---

# WSL2 与 Ubuntu 24.04 开发环境规划

[[具身智能/任务规划/MCU到VLA项目驱动转型任务规划|返回任务规划]] · [[具身智能/具身智能索引|具身智能索引]]

## 采用的主环境

用户于 2026-10-26 确认正在使用 **Windows + WSL2 + Ubuntu 24.04 LTS**。ROS 2 配套采用 **Jazzy**，与原始路线图一致。Jazzy 官方支持 Ubuntu 24.04，支持期到 2029 年 5 月。[Open Robotics 官方发布说明](https://www.openrobotics.org/blog/2024/5/ros-jazzy-jalisco-released)

本页以现有 WSL2 发行版为基础安排开发任务。ROS 2、图形、CUDA 与训练环境是否可用，按各自的实际运行结果登记；本次仅修改笔记。

| 所在位置 | 主要职责 | 工具安排 |
| --- | --- | --- |
| Windows | Obsidian、资料浏览、文件管理、录屏 | 当前知识库、Windows Terminal、VS Code |
| WSL2 的 Ubuntu 24.04 | 主项目开发、编译、ROS 2、轻量仿真、数据处理与策略实验 | ROS 2 Jazzy、Git、CMake、系统 Python、独立训练环境 |
| WSL2 中可用的本机 GPU | 在显存允许时训练和运行视觉策略 | Windows NVIDIA 驱动、适配的 PyTorch CUDA 构建，实际运算验收 |
| 其他 GPU 资源，按需评估 | 超出本机显存或计算能力的实验 | 在出现明确需求后选择设备与预算 |

## 代码、笔记与文件位置

代码仓库与频繁读取的数据优先放在 WSL2 的 Linux 文件系统，例如 `~/projects/arm_ai_control`。Obsidian 知识库继续保留在 Windows；需要交换截图、日志、视频时，再通过 `/mnt/c/...` 等挂载路径访问 Windows 文件。[Microsoft：跨文件系统工作的建议](https://learn.microsoft.com/en-us/windows/wsl/filesystems)

VS Code 可通过 WSL 扩展在 Windows 界面中编辑 Linux 项目。记录实际工作目录和运行环境，避免在 Windows Python 与 WSL Python 之间切换后混淆依赖。

## 先核对现有 WSL2

在 Windows PowerShell 记录：

```powershell
wsl --version
wsl -l -v
```

确认实际使用的发行版名称，以及列表中的 `VERSION` 为 `2`。发行版不一定叫 `Ubuntu-24.04`，后续命令按实际名称填写。[Microsoft：WSL 基本命令](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)

在 Ubuntu 内记录：

```bash
cat /etc/os-release
uname -r
python3 --version
```

| 项目 | 起步安排 | 验收重点 |
| --- | --- | --- |
| 发行版 | 继续使用现有 Ubuntu 24.04 | 系统版本、架构、WSL 内核有记录 |
| CPU 与内存 | 先使用现有配置，出现资源瓶颈后再调整 | Windows 与 WSL 均能正常响应 |
| 资源限制 | 需要时再使用 Windows 用户目录下的 `.wslconfig` | 修改后记录实际生效值 |
| 磁盘 | 记录 Linux 与 Windows 的可用空间 | 下载数据、训练与导出备份前检查余量 |
| 图形 | 使用 WSLg 运行 Linux 图形应用 | RViz 或最小 MuJoCo 窗口实测 |
| 网络 | 先验收同一 WSL2 发行版内的节点与服务 | 下载、节点通信、跨机器通信分别记录 |
| 外设 | 使用串口或相机时单独验证 WSL 连接方式 | 设备可见、权限、持续采集与重连 |
| 备份 | Git、环境记录、关键阶段发行版导出 | 代码与环境能够恢复 |

`.wslconfig` 的 CPU、内存等设置面向 WSL2，不按传统桌面虚拟机分配“虚拟显卡”。是否调整资源，以实际机器情况为准。[Microsoft：WSL 高级配置](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)

## ROS 2 与机器学习环境分开管理

ROS 2 Jazzy 优先使用 Ubuntu 24.04 对应的软件包与系统 Python。MuJoCo、PyTorch、LeRobot 按选定示例的版本创建独立 Python 环境；不替换系统 Python。

记录各环境的 Python、依赖和代码 commit。LeRobot 的版本要求以最终选定的 release 或 commit 文档为准；此前记录的安装要求不直接视为当前锁定配置。[LeRobot 官方安装文档](https://github.com/huggingface/lerobot/blob/main/docs/source/installation.mdx)

ROS 2 与策略进程分别管理依赖，通过明确的观测和动作接口协作。策略直接导入 ROS 2 Python 库时核对 Python ABI 与依赖兼容性；分进程通信时定义时间戳、单位、状态顺序、请求对应关系、超时与失败行为。

## WSLg 图形与 MuJoCo 仿真

WSLg 提供 Linux GUI 应用与 OpenGL 图形支持，先用它运行 RViz 或最小 MuJoCo 场景。Windows 显卡驱动、WSL 版本、渲染后端和应用依赖都可能影响结果，分别记录实际运行表现。[Microsoft：在 WSL 中运行 Linux GUI 应用](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps)

MuJoCo 从一个模型和少量关节开始，再增加夹爪、物体与相机。图形出现问题时先检查 WSLg 和 OpenGL；离屏软件渲染可作为备选，但需要实际测试依赖与速度。可尝试在导入 MuJoCo 前为对应进程设置 `MUJOCO_GL=osmesa`，并安装适用的 OSMesa 依赖。[MuJoCo 官方渲染说明](https://mujoco.readthedocs.io/en/3.2.2/programming/)

WSLg 能显示画面、MuJoCo 能生成图像、PyTorch 能使用 CUDA，分别登记验收结果。

## CUDA 与训练安排

WSL2 支持在适配的 NVIDIA 硬件与 Windows 驱动上运行 CUDA。优先验证本机 GPU，再决定能训练多大的策略。驱动安装在 Windows；不要在 WSL 内另外安装 Linux NVIDIA 显示驱动。CUDA Toolkit 仅在项目需要编译 CUDA 代码或扩展时，按相应文档配置。[NVIDIA：CUDA on WSL 官方指南](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)

先在 WSL2 中检查驱动可见性：

```bash
nvidia-smi
```

若命令不在 PATH 中，可按 NVIDIA 文档检查 `/usr/lib/wsl/lib/nvidia-smi`。驱动可见后，在准备使用的 PyTorch 环境中执行一次实际运算：

```python
import torch

print("torch:", torch.__version__)
print("cuda runtime:", torch.version.cuda)
available = torch.cuda.is_available()
print("cuda available:", available)
if available:
    print("device:", torch.cuda.get_device_name(0))
    x = torch.ones((32, 32), device="cuda")
    y = x @ x
    torch.cuda.synchronize()
    print("result:", y[0, 0].item())
```

保存输出，确认实际运算成功后再开始 GPU 训练。此前提到的 RTX 3060 型号与显存仍需核对，不能据型号直接确定模型、图像尺寸或 batch size。

| 路径 | 适合完成的事 | 进入条件 |
| --- | --- | --- |
| WSL2 CPU | 张量练习、数据检查、小模型与短流程验证 | 可先推进，不等待大模型环境 |
| WSL2 本机 GPU | 显存允许的 ACT 等策略训练、推理与评估 | CUDA 运算通过，依赖和资源已核对 |
| 其他 GPU 设备或远程资源 | 本机资源不足的训练与推理 | 已有明确实验目标、资源与预算 |

项目代码、数据版本、配置和权重保持可追溯；是否采用外部算力根据实际实验决定。[Microsoft：WSL GPU 计算](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gpu-compute)

## 网络与硬件接入

首阶段在同一 WSL2 发行版内运行 ROS 2 节点。连接局域网机器人或其他机器时，再验证 IP、端口、ROS_DOMAIN_ID、DDS 发现和防火墙。支持条件满足时，可评估镜像网络；它支持多播，但仍需实测 ROS 2 发现与双向通信。[Microsoft：WSL 网络模式](https://learn.microsoft.com/en-us/windows/wsl/networking)

USB 串口、USB 相机或其他设备按所选硬件核对连接方式，必要时采用 WSL USB 连接方案。先验证持续读取、权限与重连，再进入机器人控制任务。[Microsoft：在 WSL 中连接 USB 设备](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)

## 首月验收清单

- [ ] 记录 Windows、WSL、Ubuntu 的版本、发行版名称与资源情况。
- [ ] Linux 项目目录、网络与 Windows 文件交换可用。
- [ ] Git、CMake、基础编译工具与 Python 可用，完成一个小程序运行。
- [ ] ROS 2 Jazzy 的 talker / listener 在同一 WSL2 发行版通信。
- [ ] 通过 WSLg 实际运行 RViz 或轻量 MuJoCo 示例，保存输出和画面。
- [ ] 有 NVIDIA GPU 时，在 WSL2 中记录 `nvidia-smi` 和 PyTorch CUDA 实际运算结果；尚未通过时继续 CPU 小实验。
- [ ] 保存依赖、运行命令、代码版本与发行版导出备份。

WSL2 + Ubuntu 24.04 由用户确认已在使用，上述各项仍按实际验收更新。

## 环境备份与恢复

关键阶段使用 `wsl --export` 保存发行版备份，配合 Git 和依赖记录使用。先结束所用发行版内的训练、仿真和写入任务，再按需停止该发行版并导出；不要把全局关闭 WSL 作为每次运行项目前的固定步骤。[Microsoft：WSL 导出与导入命令](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)

示例在 Windows PowerShell 执行，发行版名称用 `wsl -l -v` 查到的实际值替换，备份目录需先存在：

```powershell
wsl --export <实际发行版名称> "D:\WSL-Backups\arm-ai-baseline.tar"
```

恢复验证时另建发行版，保留正在使用的项目环境；任务计划中的“干净环境复现”可使用新 Python 环境或另行导入的发行版完成。

## 后续版本复查

Jazzy 支持期到 2029 年 5 月，本年度计划使用 Ubuntu 24.04 + Jazzy 作为基线。每季度或遇到依赖问题时核对模型、仿真与 ROS 2 的兼容性，再决定是否调整版本。[Open Robotics 官方支持期说明](https://www.openrobotics.org/blog/2024/5/ros-jazzy-jalisco-released)

- [ ] 每季度记录一次依赖与岗位版本要求的变化，并保存继续固定基线或升级的决定。

## 环境记录模板

```text
记录日期：
Windows 版本：
WSL 版本 / 内核 / WSLg 版本：
发行版名称 / Ubuntu 版本与架构：
宿主机 CPU / 内存 / 磁盘余量：
WSL 资源限制与实际可用资源：
ROS_DISTRO / 系统 Python：
MuJoCo 版本与渲染后端：
策略环境 Python / PyTorch / LeRobot：
代码仓库与 commit：
Windows NVIDIA 驱动 / GPU 型号 / 显存：
nvidia-smi / PyTorch CUDA 实际运算结果：
已通过的启动命令：
当前未通过项与下一步：
发行版备份位置：
```

## 资料与核对来源

环境方案更新日期：2026-10-26。实际安装与运行时按选定版本复查。

- [Open Robotics：Jazzy、Ubuntu 24.04 与支持期](https://www.openrobotics.org/blog/2024/5/ros-jazzy-jalisco-released)
- [Microsoft：WSL 基本命令、发行版导出与导入](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)
- [Microsoft：WSL Linux GUI 应用](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps)
- [Microsoft：WSL GPU 计算](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gpu-compute)
- [NVIDIA：CUDA on WSL 官方指南](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)
- [Microsoft：WSL 文件系统](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- [Microsoft：WSL 高级配置](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)
- [Microsoft：WSL 网络模式](https://learn.microsoft.com/en-us/windows/wsl/networking)
- [Microsoft：WSL USB 设备连接](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)
- [LeRobot 官方安装文档](https://github.com/huggingface/lerobot/blob/main/docs/source/installation.mdx)
- [MuJoCo 官方 OpenGL 与离屏渲染说明](https://mujoco.readthedocs.io/en/3.2.2/programming/)
