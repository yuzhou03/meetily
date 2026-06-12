# Meetily 音频捕获技术术语表

本文档系统整理了 [audio-capture.md](file:///home/zy/repo/opensource/meetily/docs/audio/audio-capture.md) 与 [audio-capture-zh.md](file:///home/zy/repo/opensource/meetily/docs/audio/audio-capture-zh.md) 中出现的全部技术术语,以及 [微软Teams音频处理技术调研报告](file:///home/zy/repo/opensource/meetily/docs/audio/微软Teams音频处理技术调研报告：回声消除与降噪.md) 中的回声消除与降噪相关术语,按照主题分类,便于快速查阅。

- **术语 (Term)**:文档中出现的英文 / 缩写形式。
- **全称 (Full Form)**:缩写的完整展开 (如适用)。
- **中文解释**:准确且简明的中文释义。

## 1. 应用与框架

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Tauri | Tauri Framework | 基于 Rust 与 WebView 的桌面应用开发框架,可将 Web 前端与 Rust 后端组合为原生应用。 |
| Tauri Bridge | Tauri Bridge Layer | Tauri 提供的 IPC 桥接层,负责在 Web 前端 (TypeScript) 与 Rust 后端之间传递消息与事件。 |
| Tauri Command | Tauri Command | 在 Rust 中通过 `#[tauri::command]` 标注、由前端 `invoke` 调用的异步函数。 |
| Frontend | Frontend (TypeScript / React) | 应用的前端层,使用 TypeScript 与 React 实现 UI 与交互。 |
| FastAPI | FastAPI (Python Web Framework) | 高性能 Python Web 框架,Meetily 用其实现转写后处理 HTTP 服务。 |
| Backend | Backend Service | 后端服务,在本项目中既指 Tauri 的 Rust 后端,也是 Python FastAPI 服务的统称。 |

## 2. 操作系统与平台

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| macOS | macOS (Apple Desktop OS) | Apple 桌面操作系统,部分音频能力 (Core Audio Tap、CoreML) 仅在该平台可用。 |
| Windows | Microsoft Windows | 微软桌面操作系统,使用 WASAPI 作为音频后端。 |
| Linux | Linux | 开源操作系统,使用 ALSA / PulseAudio / PipeWire 等音频栈。 |
| Info.plist | iOS / macOS Application Property List | macOS / iOS 应用的属性清单文件,用于声明使用麦克风、系统音频所需的用途说明。 |
| NSMicrophoneUsageDescription | NSMicrophoneUsageDescription | macOS / iOS `Info.plist` 中描述麦克风用途的键,系统会在申请权限时展示该字符串。 |
| NSAudioCaptureUsageDescription | NSAudioCaptureUsageDescription | macOS `Info.plist` 中描述系统音频 (Core Audio Tap) 用途的键,用于 macOS 14.4+ 权限弹窗。 |
| Entitlements | macOS Entitlements | macOS 应用的权限描述文件,声明进程可使用的系统能力。 |
| Privileged | Privileged Process / Privileged API | 拥有更高系统权限的进程 / API,普通用户态应用通常无法调用。 |

## 3. 音频后端与宿主 API

| 术语 | 全称 | 中文解释 |
| --- | --- | --- |
| Core Audio | Apple Core Audio | Apple 的底层音频框架,提供音频设备管理、流与进程 tap 等能力。 |
| Core Audio Tap | Apple Core Audio Process Tap | 通过 `with_mono_global_tap_excluding_processes` 等 API 创建的系统音频捕获通道,macOS 14.4+ 公开。 |
| AVFoundation | Apple AVFoundation | Apple 的媒体框架,提供音视频采集与播放高层 API。 |
| ScreenCaptureKit | Apple ScreenCaptureKit (SCK) | Apple 提供的屏幕 / 系统音频捕获框架,需屏幕录制权限,延迟通常高于 Core Audio Tap。 |
| CPAL | Cross-Platform Audio Library | Rust 跨平台音频抽象库,封装各平台音频后端,作为 Meetily 麦克风采集的统一入口。 |
| WASAPI | Windows Audio Session API | Windows 上的音频会话 API,提供共享 / 独占模式与 loopback 录制能力。 |
| ALSA | Advanced Linux Sound Architecture | Linux 内核原生的低层音频驱动架构。 |
| PulseAudio | PulseAudio | Linux 桌面常用的声音服务,提供高级路由与会话管理。 |
| PipeWire | PipeWire | Linux 的新一代多媒体服务,逐步替代 PulseAudio。 |
| Loopback | Audio Loopback | 音频回环录制,将系统输出作为输入进行捕获 (如 WASAPI loopback)。 |
| cpal::default_host | cpal::default_host | CPAL 库中获取当前平台默认音频宿主的函数。 |
| cidre | cidre (Rust Binding) | Apple Core Audio / AVFoundation 的 Rust 绑定,Meetily 用其实现 Core Audio Tap。 |

## 4. 音频基础概念

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| PCM | Pulse Code Modulation | 脉冲编码调制,未压缩的数字音频编码方式,WAV 文件通常以 PCM 存储。 |
| WAV | Waveform Audio File Format | 由微软与 IBM 定义的常见未压缩音频文件格式,文件后缀为 `.wav`。 |
| MP3 | MPEG-1 Audio Layer III | 广泛使用的有损压缩音频格式,Meetily 通过 FFmpeg 支持导出。 |
| Sample | Audio Sample | 音频采样,数字音频在某一时刻的振幅数值。 |
| Sample Rate | Sample Rate | 采样率,每秒采集的样本数 (Hz);Meetily 管线目标采样率为 48 kHz。 |
| Frame | Audio Frame | 多声道音频在同一时刻的样本集合,例如立体声一帧包含两个样本。 |
| Float32 | 32-bit Floating Point | 32 位浮点音频样本格式,Meetily 内部处理采用此格式。 |
| Mono | Monaural | 单声道音频,仅包含一个声道。 |
| Stereo | Stereophonic | 立体声音频,包含左右两个声道。 |
| Multi-channel | Multi-channel Audio | 多声道音频,例如 5.1、7.1 声道系统。 |
| Latency | Latency | 延迟,音频从输入到可用的时间差。 |
| Jitter | Jitter | 抖动,音频延迟的波动;蓝牙设备抖动通常较大。 |
| DC Offset | Direct Current Offset | 直流偏置,音频信号中存在的非零平均值,需要通过高通滤波去除。 |
| Rumble | Low Frequency Rumble | 极低频隆隆声,通常由设备或环境引起,通过高通滤波抑制。 |
| Loudness | Perceived Loudness | 人耳主观响度,需通过归一化处理保持一致。 |
| Peak | Peak Amplitude | 信号瞬时最大幅度,用于限幅与削波检测。 |
| RMS | Root Mean Square | 均方根值,反映信号平均能量,常用于电平计量与闪避决策。 |
| Clipping | Audio Clipping | 削波,信号幅度超出可表示范围时被截断,产生失真。 |
| Hard Clamping | Hard Clamping | 硬截断,直接将超过阈值的样本设为上限 / 下限。 |
| Soft Scaling | Soft Scaling | 软缩放,按比例缩小信号以避免削波。 |
| Mix-down | Mix-down | 多声道混缩为更少声道,例如将立体声合并为单声道。 |
| Mixing | Audio Mixing | 将多路音频按比例叠加为一路输出。 |
| Ducking | Audio Ducking | 闪避,当主源 (如麦克风) 有声时自动压低副源 (如系统音频) 的音量。 |
| Crossfade | Crossfade | 交叉淡入淡出,使两段音频平滑过渡。 |
| Gap Detection | Audio Gap Detection | 音频数据缺失检测,用于在传输中断时插入静音或触发回退。 |
| Adaptive Buffering | Adaptive Buffering | 根据设备延迟与抖动动态调整缓冲大小与超时。 |
| Adaptive Mixer | Adaptive Audio Mixer | 具备自适应缓冲与间隙处理的混音器,典型实现为 `FFmpegAudioMixer`。 |
| Buffer Overflow | Buffer Overflow | 缓冲区溢出,数据量超过缓冲区容量时发生,需丢弃旧样本。 |
| Buffer Timeout | Buffer Timeout | 缓冲区等待新数据的最长时间,超过即触发间隙处理。 |
| Sample Rate Mismatch | Sample Rate Mismatch | 不同设备返回的采样率与管线目标不一致,需通过重采样统一。 |
| Resampling | Sample Rate Conversion | 采样率转换,例如将 44.1 kHz 转换为 48 kHz。 |
| Mono Mix-down | Mono Mix-down | 多声道降为单声道的混缩过程。 |
| Buffer Stats | Buffer Statistics | 混音器内部的状态统计,包括已接收块数、间隙数、插入静音量等。 |
| Far-end Signal | Far-end Signal | 远端信号,指来自远程通话方的音频信号,在发送到扬声器之前被截取作为回声消除的参考信号。 |
| Near-end Signal | Near-end Signal | 近端信号,指本地麦克风采集到的混合信号,包含近端语音、回声与背景噪声。 |
| Frame Size / Frame Shift | Frame Size / Frame Shift | 帧长与帧移,帧长为每帧的时长 (如 20ms),帧移为相邻帧之间的时间偏移 (如 10ms)。 |
| Lookahead Window | Lookahead Window | 前瞻窗口,处理当前帧时可额外查看的未来音频范围 (如 40ms),用于提升处理质量但增加延迟。 |
| SNR | Signal-to-Noise Ratio | 信噪比,信号功率与噪声功率的比值,单位 dB;信噪比越低,语音增强难度越大。 |
| Causal Processing | Causal Processing | 因果处理,仅使用当前与过去帧的信息进行处理,不使用未来信息,满足实时通信要求。 |

## 5. 音频处理与算法

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| VAD | Voice Activity Detection | 语音活动检测,用于判断音频中是否包含人声。 |
| ContinuousVadProcessor | Continuous VAD Processor | 持续运行 VAD 的处理器,实时输出语音段。 |
| Silero VAD | Silero Voice Activity Detector | 由 Silero AI 发布的轻量 VAD 模型,Meetily 通过 `silero_rs` 调用。 |
| silero_rs | silero-rs (Rust Crate) | Silero VAD 的 Rust 绑定。 |
| extract_speech_16k | extract_speech_16k | 在 16 kHz 单声道音频上抽取语音片段的函数。 |
| RNNoise | Recurrent Neural Network Noise Suppression | Mozilla 发布的使用 RNN 的实时噪声抑制算法。 |
| nnnoiseless | nnnoiseless (Rust Crate) | RNNoise 的 Rust 绑定。 |
| Noise Suppression | Noise Suppression | 噪声抑制,降低背景噪声以提升语音清晰度。 |
| NoiseSuppressionProcessor | Noise Suppression Processor | 噪声抑制处理器,对 RNNoise 的封装。 |
| Loudness Normalization | Loudness Normalization | 响度归一化,统一混音后音频的整体响度。 |
| LoudnessNormalizer | Loudness Normalizer | 响度归一化器,基于峰值或 RMS 进行归一化。 |
| High-Pass Filter | High-Pass Filter | 高通滤波器,去除直流偏置与极低频成分。 |
| HighPassFilter | High-Pass Filter Implementation | Meetily 内部的高通滤波器实现。 |
| Speaker Diarization | Speaker Diarization | 说话人分离,将转写文本按不同说话人分段。 |
| audio_to_mono | audio_to_mono | 将多声道音频转换为单声道的工具函数。 |
| resampler.rs | Resampler (audio_v2) | `audio_v2` 下的占位重采样器,留作未来支持动态采样率。 |
| normalizer.rs | Normalizer (audio_v2) | `audio_v2` 下的占位响度归一化模块。 |
| limiter.rs | Limiter (audio_v2) | `audio_v2` 下的占位限幅器,防止信号超过阈值。 |
| sync.rs | Sync (audio_v2) | `audio_v2` 下的占位时钟同步模块。 |
| compatibility.rs / LegacyBridge | Compatibility / Legacy Bridge | `audio_v2` 中用于新旧管线并存的桥接模块。 |
| AudioMode::Hybrid | Audio Mode (Hybrid) | 旧版与新版管线并行的运行模式。 |
| rubato | rubato (Rust Crate) | 高质量音频重采样库,Meetily 使用其 SincFixedIn 算法。 |
| SincFixedIn | Sinc Interpolation Fixed Input | 基于 Sinc 插值的固定输入重采样算法。 |
| MAS | Microsoft Audio Stack | 微软音频栈,嵌入 Windows 10/11 的 DSP 技术集合,包含 AEC、NS、DR、AGC 等模块,针对 Surface 与认证耳机深度优化。 |
| AEC | Acoustic Echo Cancellation | 声学回声消除,消除扬声器声音被麦克风拾取后产生的回声。 |
| NS | Noise Suppression (Abbreviation) | 噪声抑制的缩写形式,通过信号处理或深度学习降低背景噪声。 |
| DR | Dereverberation | 去混响,减少房间声学反射 (混响) 对语音质量的影响。 |
| AGC | Automatic Gain Control | 自动增益控制,动态调整语音音量,使输出电平保持在合理范围内。 |
| Voice Isolation | Voice Isolation | 语音隔离,基于声纹的个性化语音分离,只传输目标用户的清晰语音,抑制背景人声干扰。 |
| NLMS | Normalized Least Mean Squares | 归一化最小均方算法,经典自适应滤波算法,用于建立房间脉冲响应模型以消除回声。 |
| Adaptive Filter | Adaptive Filter | 自适应滤波器,通过误差信号迭代更新滤波器系数,用于回声消除中的回声路径估计。 |
| RIR | Room Impulse Response | 房间脉冲响应,描述房间声学特性的数学模型,反映声音从声源到麦克风的传播路径。 |
| ERLE | Echo Return Loss Enhancement | 回声返回损耗增强,衡量回声消除效果的核心指标,目标值 > 40dB,数值越大消除效果越好。 |
| ERL | Echo Return Loss | 回声返回损耗,回声信号相对于原始信号的衰减量。 |
| DT | Double Talk | 双向通话,通话双方同时说话的场景,要求 AEC 算法在消除回声的同时保留近端语音。 |
| Spectral Subtraction | Spectral Subtraction | 频谱减法,经典降噪算法,估计噪声功率谱并从带噪语音谱中减去;易产生 "音乐噪声" 伪影。 |
| Wiener Filtering | Wiener Filtering | 维纳滤波,基于最小均方误差准则的最优线性滤波方法,需要准确的噪声与语音功率谱估计。 |
| DeepVQE | Deep Voice Quality Enhancement | 微软研究院 2023 年发布的端到端联合处理模型,单模型同时执行 AEC、NS、DR 三大任务;7.5M 参数,CPU 推理 3.66ms/帧;已成功部署到 Microsoft Teams 生产环境。 |
| DeepVQE-S | DeepVQE Small | DeepVQE 的轻量化版本,仅 0.8M 参数,推理速度 < 1ms/帧 (Intel i7),适合边缘设备部署。 |
| Cross-Attention Alignment | Cross-Attention Alignment | 交叉注意力对齐,DeepVQE 中用于实现远端参考信号与麦克风信号精确时间同步的机制,无需单独的延迟估计模块。 |
| Mask Estimation | Mask Estimation | 掩码估计,深度学习语音增强中估计时频掩码以分离目标语音与干扰成分。 |
| Voice Profile | Voice Profile | 语音指纹 / 声纹,用户独特的声学特征签名,用于语音隔离技术中的个性化降噪。 |
| Residual Echo Suppression | Residual Echo Suppression | 残差回声抑制,AEC 后处理步骤,用于消除自适应滤波器未能完全消除的残余回声。 |
| Speech Enhancement | Speech Enhancement | 语音增强,综合运用降噪、去混响、回声消除等技术提升语音清晰度与可懂度的处理过程。 |
| Beamforming | Beamforming | 波束形成,利用多麦克风阵列实现定向拾音,增强目标方向信号并抑制其它方向噪声。 |
| Spatial Filtering | Spatial Filtering | 空间滤波,基于声源空间位置信息抑制非目标方向的噪声与干扰。 |
| BiGRU | Bidirectional Gated Recurrent Unit | 双向门控循环单元,DeepVQE 中使用的循环神经网络层,用于捕获音频信号的时序依赖关系。 |
| CNN | Convolutional Neural Network | 卷积神经网络,DeepVQE 编码器使用的残差 CNN 块,用于提取音频频谱特征。 |
| INT8 Quantization | INT8 Quantization | 8 位整数量化,将浮点模型参数转换为 8 位整数以减小模型体积与推理延迟,精度损失通常 < 1%。 |
| Perceptual Loss | Perceptual Loss | 感知损失,在训练深度学习音频模型时优化主观听觉质量而非仅最小化数值误差的损失函数。 |
| Adversarial Training | Adversarial Training | 对抗训练,通过引入对抗样本提升模型鲁棒性的训练策略。 |
| Steady-state Noise | Steady-state Noise | 稳态噪声,持续性背景噪声 (如风扇、空调),特征相对稳定,适合深度抑制。 |
| Transient Noise | Transient Noise | 瞬态噪声,短时突发性噪声 (如键盘敲击、关门声),需要精准检测与消除。 |
| Music Noise Artifact | Music Noise Artifact | 音乐噪声伪影,频谱减法等传统降噪算法产生的不自然听觉伪影,表现为断续的音调噪声。 |

## 6. 设备与硬件

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Microphone | Microphone | 麦克风,输入声音的物理设备。 |
| System Audio | System Audio | 系统音频,即操作系统或应用正在播放的声音输出。 |
| Wired Device | Wired Audio Device | 有线音频设备,如 USB 麦克风、3.5 mm 接口耳机,延迟较低。 |
| Bluetooth Device | Bluetooth Audio Device | 蓝牙音频设备,如 AirPods、蓝牙耳机,延迟与抖动较大。 |
| Unknown Device | Unknown Device | 设备类型无法识别时使用的保守默认类别。 |
| AirPods | Apple AirPods | Apple 出品的蓝牙耳机,在 Meetily 中通过名称启发式识别。 |
| USB | Universal Serial Bus | 通用串行总线,常见的有线音频接口。 |
| Headset | Headset | 头戴式耳机 (含麦克风)。 |
| Virtual Audio Cable | Virtual Audio Cable | 虚拟音频线缆,Windows 上的软件方案,允许将应用输出作为输入捕获。 |
| Latency (Wired) | 20 – 50 ms | 有线设备的自适应缓冲超时范围。 |
| Latency (Bluetooth) | 80 – 200 ms | 蓝牙设备的自适应缓冲超时范围。 |
| Latency (Unknown) | 80 – 180 ms | 未知设备的保守缓冲超时范围。 |

## 7. 数据结构与流

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Audio Chunk | Audio Chunk | 一段连续的音频样本,通常以 1024 帧为单位在流中传递。 |
| Audio Stream | Audio Stream | 由 CPAL / Core Audio / ScreenCaptureKit 等后端提供的实时音频流。 |
| AudioStreamManager | Audio Stream Manager | 负责启动、停止、转发多个音频流的协调器。 |
| Ring Buffer | Ring Buffer | 环形缓冲区,数据写入到尾部、读取从头部开始,到达末尾后回绕。 |
| AudioMixerRingBuffer | Audio Mixer Ring Buffer | 负责累积麦克风与系统音频样本的环形缓冲。 |
| SourceBuffer | Source Buffer | 自适应混音器中为单个音频源维护的缓冲与超时。 |
| Audio Pipeline | Audio Pipeline | 音频处理管线,从采集到混音、VAD、转写、保存的完整链路。 |
| AudioPipelineManager | Audio Pipeline Manager | 管线的入口管理器,负责创建与启动 `AudioPipeline`。 |
| AudioPipeline::run | Audio Pipeline Run Loop | 管线的主循环,持续接收、混音与转发音频块。 |
| AudioCapture | Audio Capture Processor | 音频采集处理器,负责格式转换、混缩、电平计量、降噪等。 |
| AudioCaptureBackend | Audio Capture Backend Enum | 表示当前所选音频后端的枚举 (Core Audio / ScreenCaptureKit / CPAL)。 |
| BACKEND_CONFIG | Backend Config (Mutex) | 全局互斥锁保护的后端配置,可通过设置界面运行时切换。 |
| get_current_backend | get_current_backend | 读取当前后端配置的函数。 |
| ProfessionalAudioMixer | Professional Audio Mixer | 基于比例软缩放的混音器,避免削波。 |
| FFmpegAudioMixer | FFmpeg-style Audio Mixer | 仿 FFmpeg 设计的自适应分源混音器,含间隙检测与 RMS 闪避。 |
| AudioMixer | Audio Mixer | 自适应混音器内部的 RMS 闪避与硬截断实现。 |
| extract_window | extract_window | 从环形缓冲中抽取对齐窗口的函数。 |
| mix_window | mix_window | 对齐窗口进行混音的函数。 |
| BufferStats | Buffer Statistics | 混音器暴露的诊断统计结构。 |
| Stream Wrapper | Stream Wrapper | 封装不同后端音频流的统一接口。 |
| No-op Stream | No-operation Stream | 不实际处理音频的空操作流,用于触发权限弹窗。 |
| Source | Audio Source | 混音器中的一种输入源,例如麦克风或系统音频。 |
| Sink | Audio Sink | 混音器的输出目标,例如转写发送端或保存器。 |
| Speaker | Speaker | 扬声器,音频输出设备。 |
| Audio Device | Audio Device | Meetily 内部对音频输入/输出设备的抽象结构。 |
| DeviceType | Device Type | 设备类型,标识输入或输出,以及其它自定义分类。 |
| AudioStream::create_with_backend | Audio Stream Create With Backend | 按指定后端创建音频流的工厂函数。 |
| create_core_audio_stream | Create Core Audio Stream | 创建 Core Audio tap 流并转发到处理器的函数。 |
| start_streams | Start Streams | 启动多个音频采集流的函数。 |
| stop_streams | Stop Streams | 停止音频采集流的函数。 |
| drain pipeline | Drain Pipeline | 录制结束时排空管线中残余数据的清理步骤。 |
| stop_streams_and_force_flush | Stop Streams And Force Flush | 停止流并强制刷新缓冲区的复合函数。 |
| flush | Flush Buffer | 强制将缓冲区内容写入下游或磁盘。 |
| transcript-update | Transcript Update Event | 转写更新事件,Tauri 向前端推送新片段的通知。 |
| recording-saved | Recording Saved Event | 录制保存完成事件,携带最终文件路径。 |
| SystemAudioEvent | System Audio Event | 系统音频活动事件,例如开始 / 停止播放。 |
| DeviceEvent | Device Event | 设备添加 / 移除 / 状态变化事件。 |
| DeviceEvent::DeviceAdded | Device Added Event | 新设备接入事件。 |
| DeviceEvent::DeviceRemoved | Device Removed Event | 设备移除事件。 |
| Device Hot-plug | Device Hot-plug | 设备热插拔,设备在运行时接入或拔出系统。 |
| Hot-plug | Hot-plug | 运行时插拔设备,无需重启应用。 |

## 8. 权限与系统集成

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Permission | Permission | 系统授予应用访问受限资源的能力,例如麦克风、系统音频。 |
| Permission Dialog | Permission Dialog | 系统弹出的权限申请对话框,需用户手动授权。 |
| Microphone Permission | Microphone Permission | 麦克风访问权限。 |
| System Audio Permission | System Audio Permission | 系统音频捕获权限,macOS 14.4+ 上由 Core Audio Tap 自动触发。 |
| Screen Recording Permission | Screen Recording Permission | 屏幕录制权限,使用 ScreenCaptureKit 时必需。 |
| trigger_audio_permission | trigger_audio_permission | 通过空操作 CPAL 流触发麦克风权限弹窗的 Rust 函数。 |
| trigger_microphone_permission | trigger_microphone_permission (Tauri Command) | 前端可调用的麦克风权限触发命令。 |
| trigger_system_audio_permission | trigger_system_audio_permission | 触发系统音频权限的 Rust 函数。 |
| check_screen_recording_permission | check_screen_recording_permission | 检查屏幕录制权限是否已授予的命令。 |
| get_audio_devices | get_audio_devices (Tauri Command) | 前端调用的设备枚举命令。 |
| default_input_device | default_input_device | 获取系统默认输入设备的 CPAL API。 |
| CoreAudioCapture::new | CoreAudioCapture::new | 创建 Core Audio 采集实例的构造函数。 |
| CoreAudioCapture::stream | CoreAudioCapture::stream | 获取 Core Audio 流的函数。 |
| SystemAudioCapture | System Audio Capture | 系统音频采集抽象。 |
| SystemAudioStreamManager | System Audio Stream Manager | 系统音频流管理器。 |
| list_system_audio_devices | list_system_audio_devices | 列出系统音频设备的函数。 |
| check_system_audio_permissions | check_system_audio_permissions | 检查系统音频权限状态的函数。 |
| Property Listener | Core Audio Property Listener | Core Audio 的属性变化监听器,用于响应设备状态变更。 |
| DEVICE_IS_RUNNING_SOMEWHERE | Device Is Running Somewhere | Core Audio 设备的属性键,标识是否有进程正在使用该设备。 |
| MacOSSystemAudioDetector | macOS System Audio Detector | macOS 上检测系统音频活动的探测器。 |
| Onboarding | Onboarding | 用户首次使用应用时的引导流程,包括权限申请与功能介绍。 |
| Crash Recovery | Crash Recovery | 崩溃恢复,通过增量保存的检查点恢复部分录制。 |
| Toast | UI Toast Notification | UI 上的轻量提示气泡,用于展示错误或状态信息。 |

## 9. 录制、转写与存储

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Recording Manager | Recording Manager | 录制会话的生命周期协调器。 |
| RecordingState | Recording State | 录制共享状态对象,跨组件传递录制元数据。 |
| Recording Lifecycle | Recording Lifecycle | 录制会话从启动到停止再到保存的完整生命周期。 |
| Recording Session | Recording Session | 一次完整的录制会话,包含音频与对应的转写。 |
| start_recording | start_recording | 启动录制会话的 Tauri 命令。 |
| start_recording_with_devices_and_meeting | start_recording_with_devices_and_meeting | 携带设备与会议上下文的启动录制命令。 |
| start_recording_with_defaults_and_auto_save | start_recording_with_defaults_and_auto_save | 使用默认设备并启用自动保存的快捷启动。 |
| stop_recording | stop_recording | 停止录制会话的 Tauri 命令。 |
| Recording Saver | Recording Saver | 录制保存器,负责落盘音频与转写。 |
| RecordingSaver::start_accumulation | Start Accumulation | 录制保存器开始累积数据的函数。 |
| RecordingSaver::stop_and_save | Stop And Save | 停止录制并生成最终文件的函数。 |
| Incremental Saver | Incremental Saver | 增量保存器,周期性将部分音频写入磁盘。 |
| Checkpoint | Checkpoint | 检查点,崩溃时仍可恢复的中间状态文件。 |
| WAV File | WAV File | 录制输出的标准 PCM WAV 文件。 |
| MP3 Export | MP3 Export | 通过 FFmpeg 将 WAV 重新编码为 MP3 的导出流程。 |
| Audio Encoder | Audio Encoder | 音频编码器,本项目使用 FFmpeg。 |
| ffmpeg-sidecar | ffmpeg-sidecar (Rust Crate) | 在 Rust 应用中捆绑并调用 FFmpeg 的工具库。 |
| FFmpeg | FFmpeg | 开源多媒体处理框架,支持音视频转码、合并、滤镜等。 |
| Transcription | Transcription | 音频转写,将语音转换为文本的过程。 |
| Transcript | Transcript | 转写结果文本。 |
| Whisper | OpenAI Whisper | OpenAI 开源的本地语音转写模型。 |
| Parakeet | NVIDIA Parakeet | NVIDIA 推出的低延迟语音转写模型。 |
| localWhisper | Local Whisper Provider | Meetily 内部的本地 Whisper 转写 provider 标识。 |
| Provider | Transcription Provider | 转写 provider,标识使用哪种转写后端 (Whisper / Parakeet)。 |
| Transcription Engine | Transcription Engine | 转写引擎,负责实际的语音到文本推理。 |
| validate_transcription_model_ready | Validate Transcription Model Ready | 检查转写模型是否加载完成的函数。 |
| get_or_init_transcription_engine | Get Or Init Transcription Engine | 懒加载并获取转写引擎单例的函数。 |
| start_transcription_task | Start Transcription Task | 启动转写工作任务的函数。 |
| Transcription Worker | Transcription Worker | 负责持续消费音频块并输出转写的工作线程。 |
| Worker Pool | Worker Pool | 线程池,用于并发处理转写任务。 |
| Engine Abstraction | Engine Abstraction | 对不同转写后端的统一抽象层。 |
| JSON | JavaScript Object Notation | 文本数据交换格式,Meetily 用其存储转写结果。 |
| SQLite | SQLite | 嵌入式关系型数据库,Meetily 用其存储录制元数据。 |
| sqlx | sqlx (Rust Crate) | Rust 异步 SQL 工具包,支持 SQLite / Postgres / MySQL 等。 |
| Summary | Meeting Summary | 会议纪要摘要,由 Python 后端基于转写生成。 |
| Mind Map | Mind Map | 思维导图,由 Python 后端基于转写生成。 |
| Speaker Diarization | Speaker Diarization | 说话人分离,见第 5 节。 |

## 10. 桌面运行时与硬件加速

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Metal | Apple Metal | Apple 的低层 GPU API,Whisper 在 macOS 上的硬件加速后端之一。 |
| CoreML | Apple Core ML | Apple 的机器学习推理框架,可在 macOS / iOS 上加速模型推理。 |
| CUDA | NVIDIA CUDA | NVIDIA GPU 的并行计算平台与编程模型。 |
| HIP | AMD HIP | AMD 的 GPU 编程接口,功能上对应 CUDA。 |
| Vulkan | Vulkan | 跨平台低层 GPU API,可作为 Whisper 加速后端。 |
| CPU | Central Processing Unit | 中央处理器,作为推理的回退后端。 |
| GPU | Graphics Processing Unit | 图形处理器,常用于并行推理。 |
| NPU | Neural Processing Unit | 神经网络处理单元,专用 AI 推理加速器,如 Intel AMX、AMD Ryzen AI、Qualcomm Hexagon 等。 |
| Intel AMX | Intel Advanced Matrix Extensions | 英特尔高级矩阵扩展指令集,用于优化深度学习推理性能。 |
| AMD Ryzen AI | AMD Ryzen AI NPU | AMD 的嵌入式 AI 加速单元,可为音频处理等任务提供低功耗推理加速。 |
| Qualcomm Hexagon | Qualcomm Hexagon DSP | 高通 Hexagon 数字信号处理器,用于移动端与边缘设备的 AI 推理加速。 |
| NVIDIA TensorRT | NVIDIA TensorRT | NVIDIA 的深度学习推理优化引擎与运行时,支持模型优化、量化与高效推理部署。 |
| DSP | Digital Signal Processor | 数字信号处理器,专用于实时信号处理的硬件,认证音频设备可通过硬件 DSP 卸载 AEC 等处理。 |

## 11. Rust 编程概念

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Async / Await | Asynchronous Programming | Rust 异步编程模型,用于并发处理 I/O 与定时任务。 |
| Tokio | Tokio (Async Runtime) | Rust 中最常用的异步运行时。 |
| tokio::spawn | tokio::spawn | Tokio 中派生新异步任务的函数。 |
| Future | Future | Rust 中表示尚未完成的异步计算。 |
| mpsc | Multi-Producer Single-Consumer | 多生产者单消费者通道,用于在线程 / 任务间传递消息。 |
| UnboundedChannel | Unbounded Channel | 无界通道,允许任意长度积压。 |
| Channel | Channel | 并发原语,用于在任务间传递数据。 |
| Mutex | Mutex (Mutual Exclusion) | 互斥锁,保护共享数据避免竞争。 |
| Mutex-protected | Mutex-protected | 由互斥锁保护的共享变量。 |
| Arc | Atomic Reference Count | 原子引用计数智能指针,允许多线程共享所有权。 |
| Crate | Rust Crate | Rust 的代码包单元,对应一个 `Cargo.toml` 依赖。 |
| sqlx | sqlx (Rust Crate) | 见第 9 节。 |
| re-export | Re-export | 通过 `pub use` 重新导出其它模块的符号。 |

## 12. 前端相关术语

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| TypeScript | TypeScript | 带类型系统的 JavaScript 超集,Meetily 前端的主要开发语言。 |
| React | React | 用于构建 UI 的 JavaScript 库。 |
| recordingService.ts | recordingService.ts | 前端录制服务模块,封装 Tauri 命令调用。 |
| Level Meter | Level Meter | 电平表,UI 上显示当前音量的可视化组件。 |
| Device Picker | Device Picker | 设备选择器,UI 上让用户选择输入 / 输出设备的控件。 |
| RMS / Peak Metering | RMS / Peak Metering | 基于均方根与峰值的电平计量,见第 4 节。 |
| AudioLevelMonitor | Audio Level Monitor | 电平监控器,持续采集并推送当前电平。 |
| SimpleLevelMonitor | Simple Level Monitor | 简化版电平监控器。 |

## 13. 系统音频监控与设备事件

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| System Audio Activity | System Audio Activity | 系统音频活动,指是否有应用正在播放音频。 |
| AudioDeviceMonitor | Audio Device Monitor | 监听音频设备添加 / 移除 / 状态变化的后台任务。 |
| device_monitor.rs | Device Monitor Module | 实现 `AudioDeviceMonitor` 的 Rust 模块。 |
| level_monitor.rs | Level Monitor Module | 实现 `AudioLevelMonitor` 的 Rust 模块。 |
| simple_level_monitor.rs | Simple Level Monitor Module | 简化版电平监控模块。 |
| system_detector.rs | System Audio Detector Module | macOS 系统音频活动检测模块。 |
| permissions.rs | Permissions Module | 权限申请相关函数集合。 |
| list_audio_devices | list_audio_devices | 跨平台设备枚举函数。 |
| configure_windows_audio | configure_windows_audio | Windows 平台特定音频配置。 |
| configure_linux_audio | configure_linux_audio | Linux 平台特定音频配置。 |
| configure_macos_audio | configure_macos_audio | macOS 平台特定音频配置。 |
| get_safe_recording_devices_macos | get_safe_recording_devices_macos | macOS 上挑选稳定录制设备的辅助函数。 |

## 14. 错误处理术语

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| Permission Denied | Permission Denied | 权限被拒,用户拒绝授权时的错误状态。 |
| No Input Device | No Input Device | 系统未检测到可用输入设备。 |
| Buffer Overflow | Buffer Overflow | 见第 4 节,音频缓冲溢出。 |
| Sample Rate Mismatch | Sample Rate Mismatch | 见第 4 节,采样率不匹配。 |
| Device Disconnected | Device Disconnected | 录制过程中设备被拔出的事件。 |
| Stream Build Failure | Stream Build Failure | 音频流构建失败,例如权限不足或后端不可用。 |
| Transcription Engine Not Loaded | Transcription Engine Not Loaded | 转写引擎未加载,通常因模型缺失。 |
| Crash Mid-recording | Crash Mid-recording | 录制过程中应用崩溃。 |
| bail! | anyhow::bail! | anyhow 宏,用于在错误时立即返回。 |
| Ok(false) | Ok(false) | 权限申请返回的"用户未授权"语义值。 |
| Toast | UI Toast | 见第 8 节,用于展示错误信息。 |
| Crash Recovery | Crash Recovery | 见第 8 节,崩溃后的状态恢复。 |

## 15. 关键 API / 函数索引

| 术语 | 出现位置 | 用途 |
| --- | --- | --- |
| `start_recording_with_devices_and_meeting` | `recording_commands.rs` | 携带设备与会议上下文的启动录制命令。 |
| `stop_recording` | `recording_commands.rs` | 停止录制命令。 |
| `get_audio_devices` | `recording_commands.rs` | 设备枚举命令。 |
| `trigger_microphone_permission` | `recording_commands.rs` | 触发麦克风权限命令。 |
| `RecordingManager::start_recording` | `recording_manager.rs` | 启动录制生命周期。 |
| `AudioStream::create_with_backend` | `stream.rs` | 按后端创建音频流。 |
| `AudioPipelineManager::start` | `pipeline.rs` | 启动音频管线。 |
| `AudioPipeline::run` | `pipeline.rs` | 管线主循环。 |
| `AudioMixerRingBuffer` | `pipeline.rs` | 环形缓冲混音器。 |
| `ProfessionalAudioMixer` | `pipeline.rs` | 比例软缩放混音器。 |
| `FFmpegAudioMixer` | `ffmpeg_mixer.rs` | 自适应分源混音器。 |
| `SourceBuffer` | `ffmpeg_mixer.rs` | 单源缓冲。 |
| `AudioMixer` | `ffmpeg_mixer.rs` | RMS 闪避与硬截断实现。 |
| `InputDeviceKind::detect` | `device_detection.rs` | 设备类型检测。 |
| `InputDeviceKind::buffer_timeout` | `device_detection.rs` | 自适应缓冲超时范围。 |
| `AudioCaptureBackend` | `capture/backend_config.rs` | 后端枚举。 |
| `BACKEND_CONFIG` | `capture/backend_config.rs` | 全局后端配置。 |
| `get_current_backend` | `capture/backend_config.rs` | 读取当前后端。 |
| `CoreAudioCapture::new` | `capture/core_audio.rs` | 创建 Core Audio 采集实例。 |
| `CoreAudioCapture::stream` | `capture/core_audio.rs` | 获取 Core Audio 流。 |
| `SystemAudioCapture::start_system_audio_capture` | `capture/system.rs` | 启动系统音频采集。 |
| `trigger_system_audio_permission` | `permissions.rs` | 触发系统音频权限。 |
| `trigger_audio_permission` | `permissions.rs` | 触发麦克风权限。 |
| `MacOSSystemAudioDetector::start` | `system_detector.rs` | 启动系统音频活动检测。 |
| `RecordingSaver::start_accumulation` | `recording_saver.rs` | 录制保存器开始累积。 |
| `RecordingSaver::stop_and_save` | `recording_saver.rs` | 停止录制并保存。 |
| `IncrementalSaver` | `incremental_saver.rs` | 增量保存器。 |
| `AudioDeviceMonitor` | `device_monitor.rs` | 设备热插拔监听器。 |
| `AudioLevelMonitor::start_monitoring` | `level_monitor.rs` | 启动电平监控。 |
| `validate_transcription_model_ready` | `transcription/engine.rs` | 检查转写模型。 |
| `get_or_init_transcription_engine` | `transcription/engine.rs` | 懒加载转写引擎。 |
| `start_transcription_task` | `transcription/worker.rs` | 启动转写任务。 |
| `recordingService.ts` | `frontend/src/services` | 前端录制服务模块。 |
| `lib.rs` (Tauri commands) | `frontend/src-tauri/src/lib.rs` | Tauri 命令入口文件。 |

## 16. 其它常用缩写

| 术语 | 全称 / 说明 | 中文解释 |
| --- | --- | --- |
| API | Application Programming Interface | 应用程序编程接口。 |
| OS | Operating System | 操作系统。 |
| DB | Database | 数据库,本项目使用 SQLite。 |
| HTTP | Hypertext Transfer Protocol | 超文本传输协议,前后端通信协议。 |
| IPC | Inter-Process Communication | 进程间通信,Tauri 前后端通过 IPC 桥接。 |
| UI | User Interface | 用户界面。 |
| UX | User Experience | 用户体验。 |
| ML | Machine Learning | 机器学习,转写与降噪的基础。 |
| RNN | Recurrent Neural Network | 循环神经网络,RNNoise 使用的网络结构。 |
| CNN | Convolutional Neural Network | 卷积神经网络,见第 5 节。 |
| BiGRU | Bidirectional Gated Recurrent Unit | 双向门控循环单元,见第 5 节。 |
| MSE | Mean Squared Error | 均方误差,深度学习模型训练中最常用的损失函数之一。 |
| GPU | Graphics Processing Unit | 图形处理器,见第 10 节。 |
| SDK | Software Development Kit | 软件开发工具包。 |
| N/A | Not Applicable | 不适用。 |
| bail! | anyhow::bail! | anyhow 提供的快速返回错误的宏。 |
