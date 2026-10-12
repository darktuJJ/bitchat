<img width="256" height="256" alt="icon_128x128@2x" src="https://github.com/user-attachments/assets/90133f83-b4f6-41c6-aab9-25d0859d2a47" />

## bitchat

一款去中心化的点对点消息应用，采用双传输架构：本地蓝牙 Mesh 网络用于离线通信，基于互联网的 Nostr 协议用于全球通信。无需账号、无需手机号、没有中心服务器。它就是那个“小群聊”。

[bitchat.free](http://bitchat.free)

📲 [App Store](https://apps.apple.com/us/app/bitchat-mesh/id6748219622)

## 许可证

本项目已发布到公有领域。详情请参阅 [LICENSE](LICENSE) 文件。

## 功能特性

- **双传输架构**：蓝牙 Mesh 用于离线通信 + Nostr 协议用于互联网消息
- **基于位置的频道**：通过全球 Nostr 中继、使用 geohash 坐标的地理聊天室
- **智能消息路由**：自动选择最佳传输方式（蓝牙 → Nostr 回退）
- **去中心化 Mesh 网络**：基于低功耗蓝牙（Bluetooth LE）的自动节点发现和多跳消息中继
- **隐私优先**：无账号、无手机号、无持久标识符
- **私信端到端加密**：Mesh 使用 [Noise Protocol](https://noiseprotocol.org)，Nostr 使用 NIP-17
- **IRC 风格命令**：熟悉的 `/slap`、`/msg`、`/who` 风格界面
- **通用应用**：原生支持 iOS 和 macOS
- **紧急擦除**：三击即可立即清除所有数据
- **性能优化**：LZ4 消息压缩、自适应省电模式和优化的网络处理

## [技术架构](https://deepwiki.com/permissionlesstech/bitchat)

BitChat 采用**混合消息架构**，包含两个互补的传输层：

### 蓝牙 Mesh 网络（离线）

- **本地通信**：在蓝牙范围内直接点对点通信
- **多跳中继**：消息经由附近设备转发（最多 7 跳）
- **无需互联网**：在灾难场景下完全离线可用
- **Noise Protocol 加密**：具备前向保密的端到端加密
- **二进制协议**：针对低功耗蓝牙限制优化的紧凑数据包格式
- **自动发现**：节点发现与连接管理
- **自适应功耗**：针对电池优化的占空比调度

### Nostr 协议（互联网）

- **全球覆盖**：通过互联网中继与全球用户连接
- **位置频道**：使用 geohash 坐标的地理聊天室
- **290+ 中继网络**：分布于全球，保障可靠性
- **NIP-17 加密**：采用 gift-wrap 封装的私信，保护互联网隐私
- **临时密钥**：每个 geohash 区域使用全新的加密身份

### 频道类型

#### `mesh #bluetooth`

- **传输方式**：低功耗蓝牙 Mesh 网络
- **范围**：多跳范围内的本地设备
- **互联网**：不需要
- **适用场景**：离线通信、抗议活动、灾难、偏远地区

#### 位置频道（`block #dr5rsj7`、`neighborhood #dr5rs`、`country #dr`）

- **传输方式**：基于互联网的 Nostr 协议
- **范围**：由 geohash 精度定义的地理区域
  - `block`（7 个字符）：街区级别
  - `neighborhood`（6 个字符）：区/社区
  - `city`（5 个字符）：城市级别
  - `province`（4 个字符）：州/省
  - `region`（2 个字符）：国家/大区域
- **互联网**：需要（连接到 Nostr 中继）
- **适用场景**：基于位置的社区聊天、本地活动、区域讨论

### 私信路由

私信采用**智能传输选择**：

1. **蓝牙优先**（可用时首选）

   - 通过已建立的 Noise 会话直接连接
   - 最快、最私密的方式

2. **Nostr 回退**（蓝牙不可用时）

   - 使用接收方的 Nostr 公钥
   - 采用 NIP-17 gift-wrap 保护隐私
   - 经由全球中继网络路由

3. **智能排队**（两者都不可用时）
   - 消息排队等待，直到有可用的传输方式
   - 连接建立后自动投递

详细的协议文档请参阅[技术白皮书](WHITEPAPER.md)。

## 安装配置

### 方式一：使用 Xcode

   ```bash
   cd bitchat
   open bitchat.xcodeproj
   ```

   要在真机上运行，需要先做几步准备：
   - 复制本地配置：`cp Configs/Local.xcconfig.example Configs/Local.xcconfig`
   - 在新建的 `Configs/Local.xcconfig` 中填入你的开发者团队 ID（Developer Team ID）
      - Bundle ID 将被设置为 `chat.bitchat.<team_id>`（除非你另行设置）
   - Entitlements 需要手动更新（TODO：自动化）：
      - 搜索并将 `group.chat.bitchat` 替换为 `group.<your_bundle_id>`（例如 `group.chat.bitchat.ABC123`）

### 方式二：使用 `just`

   ```bash
   brew install just
   ```

想在 macOS 上试用：`just run` 会完成配置并从源码运行。
之后运行 `just clean`，即可恢复到原始状态，以便进行移动端应用的构建和开发。

## 本地化

- 应用的基础资源位于 `bitchat/Localization/Base.lproj/`。新文案添加到 `Localizable.strings`，复数规则添加到 `Localizable.stringsdict`。
- 分享扩展的字符串单独存放在 `bitchatShareExtension/Localization/Base.lproj/Localizable.strings`。
- 优先使用能描述意图的键名（`app_info.features.offline.title`），并尽可能复用已有的键。
- 运行 `xcodebuild -project bitchat.xcodeproj -scheme "bitchat (macOS)" -configuration Debug CODE_SIGNING_ALLOWED=NO build`，对本地化更新进行编译检查。
