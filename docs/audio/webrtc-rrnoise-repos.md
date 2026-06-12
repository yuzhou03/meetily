## RNNoise 降噪 + WebRTC AEC3 官方 GitHub 地址

### 1. RNNoise 官方仓库
- **GitHub 主仓库**：https://github.com/xiph/rnnoise 
- **GitLab 官方镜像**：https://gitlab.xiph.org/xiph/rnnoise 

**说明**：RNNoise 由 Xiph.Org 基金会开发维护，采用 BSD-3-Clause 开源协议，完全免费商用。编译时会自动从 Xiph 服务器下载预训练的神经网络模型文件。

---

### 2. WebRTC AEC3 相关仓库
WebRTC AEC3 是 WebRTC 项目中的一个核心模块，没有单独的官方仓库，位于 WebRTC 主仓库的特定路径下。

- **Google 官方主仓库**：https://chromium.googlesource.com/external/webrtc.git
- **GitHub 官方镜像**：https://github.com/webrtc-sdk/webrtc 
- **AEC3 模块在仓库中的路径**：`modules/audio_processing/aec3/` 

---

### 3. 推荐的第三方单独提取仓库
如果不想集成完整的 WebRTC 项目，可以使用这些社区维护的单独 AEC3 提取版本：
- **C++ 独立版**：https://github.com/ewan-xu/AEC3 
- **Rust 移植版**：https://github.com/RubyBit/aec3-rs 
- **Unity 集成版**：https://github.com/xue-fei/aec3-unity 

---

### 4. 最佳实践
对于你之前提到的"RNNoise 降噪 + WebRTC AEC3"组合方案，建议：
1. 使用 RNNoise 官方仓库进行降噪处理
2. 使用 WebRTC 官方镜像仓库提取 AEC3 模块，或直接使用上述第三方独立版
3. 按照"先回声消除，后降噪"的顺序处理音频流，以获得最佳效果