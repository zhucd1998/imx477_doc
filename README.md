# IMX477 双目视频采集系统说明文档

## 1. 系统概述
本系统旨在构建一个基于 **Arducam IMX477 双目相机与 Raspberry Pi 4B** 的视频采集平台，用于水下及实验环境中的图像采集、标定与 Ground Truth（GT）数据生成。  
系统主要用于为后续的视觉 SLAM、图像增强与特征匹配研究提供高质量实验数据。

---

## 2. 硬件组成与连接

### 2.1 硬件清单
| 模块 | 型号 | 主要功能 |
|------|------|----------|
| 主控板 | Raspberry Pi 4B（8GB） | 图像采集与数据存储 |
| 相机模块 | Arducam IMX477 ×2 | 双目同步视频采集 |
| 相机接口板 | Arducam UC-512 + UC-517 | 双路 CSI 接口转换与同步控制 |
| 存储 | SSD 500GB（USB3.0 接口） | 数据缓存与录制 |
| 电源 | 5V/5A 稳压电源 | 提供稳定供电 |
| 其他 | HDMI 显示器、键鼠、散热系统 | 实验调试辅助设备 |

### 2.2 系统连接结构图
示意图（可在 `docs/figures/system_architecture.png` 中替换为你的结构图）：

![系统结构图](docs/figures/system_architecture.png)

连接说明：
- 两个 IMX477 模块分别连接至 UC-517，再通过 UC-512 与树莓派的 CSI0 / CSI1 接口连接；
- SSD 通过 USB3.0 连接；
- 外接显示器和键鼠用于系统配置与采集监控。

---

## 3. 采集系统配置

### 3.1 软件环境
| 项目 | 版本 | 说明 |
|------|------|------|
| 操作系统 | Raspberry Pi OS (64-bit) | 官方镜像 |
| Python | 3.13.2 | 控制与脚本 |
| OpenCV | 4.5+ | 视频流采集与图像预览 |
| ROS | Noetic (可选) | 若需录制为 rosbag 格式 |
| Arducam SDK | 最新版 | IMX477 驱动及控制库 |

### 3.2 依赖安装
```bash
sudo apt update
sudo apt install python3-opencv
pip install numpy
