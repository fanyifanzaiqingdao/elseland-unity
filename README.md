# Elseland Unity migration

独立 Unity 原型工程。现有 Expo、Three.js、后端仓库均未修改。
目标已确定为整个 App 外壳和游戏都迁移到 Unity，不再以 Expo 嵌入 Unity 为目标。
当前仍是分批实现中的原型，不是全部游戏的移植成品。

## 环境

- Unity Editor **6000.3.26f1 Apple Silicon**，在 Unity Hub 安装。
- iOS 导出需要安装 **iOS Build Support**，并在 Mac 安装兼容的 Xcode。
- 在 Hub 完成登录和许可证激活。只安装 Unity Hub 不包含 Editor。
- 包依赖为 URP 17.3.0、Test Framework 1.6.0、Nakama Unity SDK 3.22.1；首次导入需联网解析。
- 客户端架构从 QFramework 核心开始增量接入；现有 UI Toolkit 界面暂不替换为 UIKit。
- Unity 导入后生成的 `Packages/packages-lock.json` 和 `.meta` 应纳入版本控制。

## 打开与运行

1. Unity Hub -> Projects -> Add project from disk，选择本目录。
2. 等待依赖安装及脚本编译完成。
3. 选择菜单 **Elseland -> Create or Open Starter Scene**。
4. 点击 Play。启动场景与 URP 配置由编辑器脚本生成，不会覆盖已有 Boot 场景。
5. 启动进入 Home；底部有 Home / Plaza / Planet / Me 四个入口。
6. Home 可浏览推荐卡片、启动本地小游戏或在线游戏；Planet 屏幕方向按钮或 WASD/方向键控制角色。
7. 照顾与换装操作自动保存；Me 展示本地游客数据。

## App 外壳

- Home：历史优先的推荐卡片流，接入在线随机分页、本地高尔夫/滑雪、前后切换和详情区纵向滑动。在线封面加载失败时显示标题回退；尚不是 Expo 的卡片内可玩 WebView。
- Plaza：可选远程游戏列表与当前列表标题搜索（最多 100 条），不是完整分类/分页实现。
- Planet：保留小屋/草地、照顾、换装和移动原型；切到其他页时暂停角色移动并隐藏场景内容。
- Planet 海岸：从地图前往潮汐海岸，在沙滩靠近贝壳可拾取；走到海边可抛竿，看到浮标咬钩后及时收竿。鱼与贝壳保存在本地活动存档，贝壳点每日重生。
- Me：本地游客资料与数值，以及 Nakama 连接/设备登录状态。云存档尚未启用。
- `Assets/Elseland/Resources/AppConfig.json` 的 `apiBase` 默认指向 dev API，用于 Home 自动加载推荐。离线测试可置空，Home 仍保留本地游戏。
- 如需连接其他环境，在该文件填入经确认的 HTTPS API 根路径（含 `/api`）。
- 请求为只读 `GET /games?limit=100&offset=0&sort=new&platform=mobile`，返回格式对照 Expo Game[]。
- 在线推荐游戏在 iOS 通过原生 WKWebView 全屏打开；Editor/其他平台通过系统浏览器打开。iOS 设备构建及第三方 Web 游戏兼容性尚待验证。
- 不在 AppConfig.json 存令牌、AWS 凭据或其他密钥。

## QFramework 与 Nakama

- 正式账号、权威数据、离线存档与迁移决策见 [数据权威与存档方案](docs/data-authority-and-saves.md)。该文档是目标契约，不表示云同步已实现。
- `Assets/ThirdParty/QFramework/QFramework.cs` 是官方 MIT 核心架构快照；来源提交与接入边界见 [联网说明](docs/networking.md)。
- Nakama 官方 SDK 通过 Unity Package Manager 锁定在 v3.22.1。`ElselandArchitecture` 目前注册 `NakamaConnection` 与 `PlanetConstructionSystem`；星球建造通过 Command 提交并用 Event 更新世界，其他玩法和 UI 逐步迁移，不做整包替换。
- `Assets/Elseland/Resources/AppConfig.json` 默认 `nakamaEnabled: false`，因此离线启动不受影响。连接服务器的配置及注意事项见 [联网说明](docs/networking.md)。
- 设备登录成功只建立 Nakama 玩家身份；角色、建筑、背包和小游戏记录仍保存在本机，尚无云同步或跨设备账号合并。

首次运行的角色和家具是程序化原型，不是已导入的正式美术资源。
本地测试食物初始 5 份，尚未接入商店、补给或旧存档。
UI 三语文本已写入 Words.cs；默认字体的中日文字形覆盖尚待验证，
发布前需要接入授权清晰的 CJK 字体资源并配置 UI Toolkit 回退字体。

## 测试

编辑器 **Window -> General -> Test Runner -> EditMode -> Run All**。
覆盖喂食、重复点击、存档失败、数值上限、语言、帽子保存和损坏文件保护。

安装 Editor 后也可以执行：

```sh
~/.unity/bin/unity test /Users/mima1234/Documents/code/elser-project-total/elseland-unity --mode EditMode --output /tmp/elseland-unity-tests.xml
```

## iOS

菜单 **Elseland -> Export iOS Prototype** 导出到 Builds/iOS。
在 Xcode 配置你自己的开发团队和签名后运行。
原型使用独立 Bundle ID `ai.elser.elseland.unityprototype`，避免覆盖已有 App。
此导出是独立 Unity App 原型，不是 Expo 集成，不会上传 TestFlight/App Store/AWS。

## 安全边界

- 存档位于 Application.persistentDataPath/unity-companion-v1.json。
- 不读取或覆盖 WebView localStorage，不自动连接现网 Nakama 账号；只有明确启用 Nakama 配置后才尝试连接指定服务器。
- 所有数值是原型规则，不宣称与网页现有经济系统一致。
- 服务端鉴权、可信奖励、旧存档迁移未实现；不能把本地数值当线上资产。
- 全量 App 和游戏迁移状态见 [迁移记录](docs/migration.md)。
# elseland-unity
