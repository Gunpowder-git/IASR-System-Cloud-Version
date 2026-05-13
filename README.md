# IASR System · Streamlit Community Cloud 版

> 面向低空经济的「感知 → 指标 → 事件 → 预警路由」一体化原型系统。  
> 本版本针对 **Streamlit Community Cloud 在线部署** 做了轻量化处理：支持网页上传视频/图片、在线分析、下载中文 CSV / JSON / 标注后视频 / 完整结果包。

## ✨ Features

### 1. 视频态势识别
- 上传本地视频（mp4 / mov / mkv）
- YOLO 目标检测与跟踪
- 支持跳帧、缩放、最大处理帧数，适合云端演示
- 输出标注后视频 `annotated_h264.mp4`

### 2. 中文化交通指标
系统会生成更易读的中文指标表：

| 指标 | 含义 |
|---|---|
| 流量(辆/分钟) | 通过计数线的车辆数量 |
| ROI内车辆数 | 指定区域内车辆数量 |
| 占有率(%) | 车辆框面积 / ROI面积 |
| 平均相对速度(px/s) | 基于目标轨迹估计的相对速度 |
| 拥堵指数(0-100) | 综合流量、占有率、速度得到的态势分数 |
| 能见度分数 | 基于画面清晰度/对比度的代理指标 |

### 3. 事件输出与联动派单
系统会把识别结果转成统一事件，并模拟分发到对应对象：

| 事件类型 | 示例路由对象 |
|---|---|
| 鸟群风险 | 运行/飞行管控 |
| 敏感区域入侵 | 安防/监管 |
| 低能见度风险 | 应急/运行调度 |
| 疑似异常停车 | 交管/运营 |
| 疑似作物病害 | 农业运维/巡检人员 |

输出文件包括：
- `events.json`
- `events_zh.csv`
- `dispatch_log_zh.csv`
- 事件证据关键帧

### 4. 农业扩展接口（MVP）
- 支持上传作物/叶片图片
- 基于颜色与纹理启发式规则判断「疑似病害/黄化/枯斑」
- 输出农业指标表、证据图、事件表、派单日志

> 注意：农业模块是 MVP 演示接口，不等同于专业病害诊断。

### 5. 云端下载体验
每次运行都会生成独立目录，不覆盖历史结果。页面支持下载：
- 中文指标表 CSV
- 事件 JSON
- 中文事件表 CSV
- 中文派单日志 CSV
- 标注后视频 MP4
- 本次运行完整结果包 ZIP

## 🚀 在线部署到 Streamlit Community Cloud

### 1. 准备 GitHub 仓库
把本项目文件推送到 GitHub，根目录建议保持：

```text
IASR-System/
  app.py
  perception_core.py
  event_engine.py
  agriculture_core.py
  requirements.txt
  .streamlit/config.toml
  models/.gitkeep
  README.md
```

不要提交 `.pt` 模型权重文件：

```text
*.pt
models/*.pt
```

本 Cloud 版本默认允许运行时自动下载 `yolov8n.pt`。

### 2. 在 Streamlit Community Cloud 创建应用
1. 登录 Streamlit Community Cloud
2. 选择你的 GitHub 仓库
3. 入口文件选择：

```text
app.py
```

4. Python 版本建议选择 **3.11 或 3.12**
5. 点击 Deploy

首次运行可能需要下载 YOLO 权重，加载会稍慢；之后同一实例内会缓存模型。

## 🖥️ 本地运行

### 1. 创建虚拟环境

Windows PowerShell：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

macOS / Linux：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 2. 启动

```bash
python -m streamlit run app.py
```

浏览器打开终端显示的地址，一般是：

```text
http://localhost:8501
```

## 🧭 使用方法

### 视频分析
1. 打开网页
2. 上传短视频（建议 10–20 秒，Cloud 版建议小于 80 MB）
3. 选择场景预设
4. 调整运行模式：
   - `max_frames`：最多处理帧数
   - `frame_skip`：跳帧处理
   - `resize_width`：处理宽度
5. 可选配置：
   - ROI 区域
   - 车辆计数线
   - 敏感/禁飞区多边形
6. 点击「开始视频分析」
7. 在各 Tab 查看结果并下载文件

### 农业扩展分析
1. 进入「农业扩展」Tab
2. 上传作物/叶片图片
3. 调整疑似异常区域阈值
4. 点击「开始农业扩展分析」
5. 下载农业指标表、事件表、派单日志或完整结果包

## 📦 输出文件

每次运行都会写入一个独立目录，例如：

```text
/tmp/iasr_outputs/runs/video_20260512_153000/
```

常见输出：

```text
annotated_h264.mp4          # 标注后视频
metrics.csv                 # 原始指标表
metrics_zh.csv              # 中文指标表
events.json                 # 原始事件 JSON
events_zh.csv               # 中文事件表
dispatch_log_zh.csv         # 中文联动派单日志
run_config.json             # 本次运行参数
evidence/                   # 事件证据截图
```

Cloud 环境的文件是临时的，请在页面上及时下载。

## ⚙️ 重要配置

### `.streamlit/config.toml`

```toml
[server]
maxUploadSize = 100

[browser]
gatherUsageStats = false
```

### 依赖说明

Cloud 版本使用 `opencv-python-headless` 与 `imageio-ffmpeg`，因此不需要 `packages.txt` 安装系统级 `ffmpeg`。这样可以避免 Streamlit Community Cloud 上的 apt 依赖冲突。

## ❓常见问题

### 1. 首次运行很慢？
Cloud 版本默认会在首次视频分析时下载 YOLO 权重，并初始化模型。建议先用短视频测试。

### 2. 视频上传后处理很慢？
建议：
- 视频长度控制在 10–20 秒
- `max_frames` 设置为 300 左右
- `frame_skip` 设置为 3 或更高
- `resize_width` 设置为 768 或更低

### 3. 模型下载失败？
可能是网络波动。可以重试部署/重启应用。本地版本可手动把 `yolov8n.pt` 放到项目根目录或 `models/` 目录。

### 4. 农业病害判断准确吗？
当前只是 MVP 级启发式判断，用于展示「农业扩展接口」和事件联动闭环，不用于真实农事诊断。

## 🧩 Roadmap

- 多路摄像头/无人机视频源管理
- 更完善的事件时间线与空间热力图
- 接入真实气象/水位/雷达数据
- 细分农业病害模型插件
- 对接真实平台 API：派单、消息推送、处置闭环

## License

MIT License

## Acknowledgements

- Ultralytics YOLO
- Streamlit
- OpenCV Headless / imageio-ffmpeg
- Tongji University
