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
| 电源 | 5V/3A 充电宝 | 提供稳定供电 |

### 2.2 系统连接结构图
示意图如下所示：

![视觉采集系统整体结构图](doc/figures/whole_structure.png)

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
```

### 3.3 视频采集流程
1. 打开树莓派并启动采集脚本；
2. 确认双目视频流均输出正常；
3. 根据实验环境调整曝光、增益；
4. 保存视频流（或录制 rosbag）到 `/data/raw/`；
5. 检查帧同步与时间戳。

采集命令示例（Python）：
```bash
python3 capture_dual_imx477.py --output ./data/raw/sequence_01/
```

若使用 ROS：
```bash
rosbag record /camera/left/image_raw /camera/right/image_raw
```

---

### 3.4 视频示例
下方为系统整体采集流程的短视频演示  
[点击下载系统采集演示视频 (video1.mp4)](doc/video/video1.mp4)



## 4. 相机标定流程

### 4.1 标定设备与环境
- 标定板：9×6 棋盘格，方格边长 25 mm；
- 拍摄距离：0.5–1.0 m；
- 照明：均匀光照，避免反光；
- 若为水下标定，应在相同折射环境下进行。

### 4.2 标定方法（基于 ROS）
```bash
rosrun camera_calibration cameracalibrator.py   --size 9x6 --square 0.025   left:=/camera/left/image_raw right:=/camera/right/image_raw   left_camera:=/camera/left right_camera:=/camera/right
```

### 4.3 标定结果
- 输出文件：`config/left.yaml` 与 `config/right.yaml`
- 包含内参矩阵、畸变系数、旋转和平移矩阵：
  ```yaml
  camera_matrix:
    data: [fx, 0, cx, 0, fy, cy, 0, 0, 1]
  distortion_coefficients:
    data: [k1, k2, p1, p2, k3]
  ```

---

## 5. Ground Truth（GT）生成流程

### 5.1 目标
基于采集的双目序列生成高精度轨迹 Ground Truth，用于 SLAM 结果评估。

### 5.2 方法一：基于 VINS-Fusion
1. 导入双目序列；
2. 运行轨迹估计；
3. 导出轨迹：
   ```bash
   vins_node output_gt.csv
   ```
4. 转换格式：
   ```
   timestamp tx ty tz qx qy qz qw
   ```

### 5.3 方法二：基于 ORB-SLAM3
1. 使用标定文件与双目图像运行 ORB-SLAM3；
2. 导出轨迹文件：
   ```
   KeyFrameTrajectory.txt
   ```
3. 重采样与时间对齐；
4. 存储在 `data/gt/sequence_01_gt.txt`。

---

## 6. 数据整理与后处理
- 对采集数据进行帧同步、亮度一致性检测；
- 删除模糊帧与异常曝光帧；
- 可执行的示例脚本：
  ```bash
  python3 scripts/clean_dataset.py --input ./data/raw --output ./data/processed
  ```

---

## 7. 附录
- **标定工具**：ROS camera_calibration / Kalibr  
- **轨迹评估工具**：evo、rpg_trajectory_evaluation  
- **数据命名规范**：
  ```
  sequence_01/
    left/
    right/
    imu/
    gt/
  ```

---

© Chengdong Zhu, 2025.  
该文档用于实验记录与系统说明，未经许可请勿转载。
