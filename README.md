# virtual_audience_simulator

观众模拟器版本 1.0
请根据这些提示使用 Python 实现一个桌面应用。按照目录结构创建必要的模块、类和文件, 必要时适当添加。

### 一、总体要求
- 使用 Python 3.10+。

- 使用 Pygame 创建窗口、绘制 2D 图形并处理用户输入和图形更新。

- 使用 pyaudio 或 sounddevice 采集麦克风音频，至少实现基于音量阈值的简单反应逻辑。

- 采用 面向对象设计，将系统拆分为多个可维护的模块和类。

- 提供一个简单友好的用户界面，包括：

- 场景选择（首版可以固定为一个舞台背景）。

- 选择虚拟观众性格：友好、冷漠或挑剔（影响观众的随机行为）。

- 开始/停止演讲按钮。

- 演讲时间统计显示。

- 可在本地持久化演讲记录（例如使用 CSV 文件），保存日期、时长和观众设置。

- 确保项目可在 Windows、macOS 和 Linux 上运行。

### 二、目录结构
请创建如下项目目录结构，使用包来组织模块：

```text

virtual_audience_simulator/
├── main.py                # 程序入口，负责启动 UI 和主循环
├── requirements.txt       # 依赖列表（pygame、pyaudio 或 sounddevice、numpy 等）
├── README.md              # 项目说明，介绍运行方法
├── data/
│   ├── audience_images/   # 存放观众角色的不同表情图片（如 smile.png, clap.png 等）
│   ├── backgrounds/       # 存放背景图片，例如 stage.png
│   └── logs/              # 用于保存演讲记录的 CSV 文件
└── simulator/
    ├── __init__.py
    ├── ui.py              # UI 管理器，管理按钮和文本渲染
    ├── audience.py        # 定义单个观众类 Audience
    ├── audience_manager.py# 管理观众集合和布局
    ├── audio.py           # 音频捕获和处理逻辑
    ├── stats.py           # 统计记录器，用于记录演讲时长和音量数据
    ├── settings.py        # 配置选项和常量
    └── utils.py           # 工具函数，例如加载图片、文本渲染等
```

说明：

- main.py：程序入口，负责初始化 Pygame 环境、加载资源、创建各模块实例，并运行主循环。

- requirements.txt：列出项目依赖，如 pygame, pyaudio 或 sounddevice, numpy 等。

- simulator/ui.py：负责构建 UI（按钮、文本），处理用户点击事件，调用其它模块方法。可使用简单按钮类实现，提供绘制方法和碰撞检测。

- simulator/audience.py：定义 Audience 类，包含观众的图像资源、状态、位置及动画控制逻辑。应实现 update() 和 draw() 方法。

- simulator/audience_manager.py：定义 AudienceManager 类，用于根据用户设定创建观众列表，控制观众布局和统一更新/绘制观众集合。

- simulator/audio.py：封装音频采集逻辑。使用线程或异步方式读取麦克风数据并计算 RMS 音量；当音量超过或低于阈值时向其它模块发送事件或调用回调。

- simulator/stats.py：实现 StatisticsRecorder 类，负责记录演讲开始时间、结束时间，统计平均音量，保存结果到 CSV 文件。

- simulator/settings.py：定义常量和配置，例如窗口大小、默认阈值、观众性格参数（不同性格对应不同随机行为概率）。

- simulator/utils.py：包含图像加载辅助函数、文本渲染函数（例如加载 TTF 字体并绘制到 Pygame surface）。

### 三、核心模块实现要点
1. UI 管理器（simulator/ui.py）
- 使用 Pygame 创建窗口，设置窗口标题和图标。

- 定义简单的 Button 类：包含矩形区域、文本标签、回调函数；在 draw() 中绘制矩形和文字，在 handle_event() 中检测鼠标点击并触发回调。

- 创建所需按钮：选择观众性格、开始、停止；通过回调函数修改观众管理器配置或开始/停止计时。

- 显示演讲时长：可在每帧渲染统计文本，如“演讲时间：00:01:23”。

2. 观众相关模块
Audience 类（simulator/audience.py）
属性：

- images: dict，键为状态名称（如 normal, clap, bored），值为对应的 Pygame Surface。

- state: 当前状态。

- pos: 二元元组，屏幕坐标。

- timer: 用于控制随机行为的计时器。

方法：

- update(delta_time, audio_event): 根据时间增量和音频事件更新状态；当 audio_event 表示用户音量高或低时，改变状态；定期进行随机切换。

- draw(surface): 在指定的 surface 上绘制当前状态的图像。

- AudienceManager 类（simulator/audience_manager.py）
  - 管理 Audience 实例的集合；根据用户设置创建 1 个观众（版本 1.0）并确定其位置；
  
  - 提供 update_all(delta_time, audio_event) 和 draw_all(surface) 用于统一更新/绘制；
  
  - 根据性格设置调整每个观众的状态切换概率（例如友好时更倾向于鼓掌）。

3. 音频处理（simulator/audio.py）
- 使用 sounddevice 或 pyaudio 采集麦克风数据。

- 启动一个后台线程实时读取音频流，并计算每个时间片的 RMS 音量；

- 将音量传入队列或回调函数；主线程可从队列中读取当前音量并判断是否超过阈值；

- 提供开始/停止录音的方法；版本 1.0 可以不保存音频文件。

4. 统计记录（simulator/stats.py）
- 在用户点击“开始”时记录当前时间，点击“停止”时记录结束时间并计算总时长；

- 可记录平均音量或最大音量；

- 将记录写入 CSV 文件，如 data/logs/sessions.csv，包含字段：日期、时长、性格设置等。

5. 主程序（main.py）
- 初始化 Pygame、加载配置与资源；

- 创建 AudienceManager、AudioProcessor、StatisticsRecorder 和 UI 按钮；

进入主循环：

- 处理 Pygame 事件，包括退出、按钮点击等；

- 从 AudioProcessor 获取最新音量事件；

- 调用 AudienceManager.update_all() 更新观众状态；

- 调用 AudienceManager.draw_all() 以及 UI.draw() 绘制内容；

- 刷新屏幕并控制帧率（30–60 FPS）。

6. 配置文件与资源
在 requirements.txt 中列出项目依赖：

```text
pygame==2.5.0
sounddevice==0.4.6  # 或 pyaudio，如果需要
numpy>=1.23.0
将图片资源放在 data/audience_images 和 data/backgrounds 目录中。可使用占位图片，如白色笑脸、小人剪影等；确保文件命名与代码读取一致。

在 settings.py 中指定音量阈值、观众性格参数（例如友好：鼓掌概率 0.6，无聊概率 0.1；挑剔：鼓掌概率 0.2，无聊概率 0.5）等常量。
```

### 四、开发顺序建议
- 环境搭建：安装依赖，确认 Pygame 和音频库可正常运行；构建项目目录。

- 基础渲染：实现 main.py 初始窗口和主循环，加载背景图并绘制到屏幕；实现最简单的 UI 按钮并响应点击事件。

- 观众模块：实现 Audience 和 AudienceManager，加载观众图片并绘制一个观众。先通过计时器随机切换表情，测试动画效果。

- 音频采集：实现 AudioProcessor，在后台线程中获取音量数据；简单阈值触发观众状态切换。

- 统计模块：实现演讲时间统计和 CSV 记录功能；在 UI 上显示计时。

- 功能集成：将各模块组合到 main.py 中，完成版本 1.0 原型；测试按钮逻辑、观众反应、音频采集和统计记录。

- 测试与优化：测试软件在不同系统上的兼容性，优化性能，改善用户体验；在此基础上可准备迭代计划，为多观众、复杂反馈等扩展做准备。


### 五、注意事项
- 线程安全：音频采集可能在后台线程运行，注意在主线程读取音量时避免竞争条件；可使用线程安全队列。

- 资源释放：在程序退出时释放麦克风、关闭 Pygame；捕获异常并确保文件句柄已关闭。

- 跨平台问题：音频库在不同系统上的配置可能不同。需要在文档中注明安装方法，例如 Linux 上可能要安装 portaudio。

- 未来兼容性：请保持代码的模块化和可扩展性，便于后续添加多观众、语音情感分析和网络同步等功能。

- 按照以上提示，AI 编程助手应能生成一个结构清晰、易于运行的版本 1.0 实现，为公开演讲练习提供基本的虚拟观众模拟功能。
