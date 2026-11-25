# LLM Hub 国内源版（基于 Omar-afk801/LLM-Hub-FOSS）

## 这个版本做了什么
- 底子：Omar-afk801/LLM-Hub-FOSS —— 已去广告、已解锁全部高级功能、Releases 带现成 APK，且保留 LTX-Video 2B 0.9.8 图生视频（含骁龙 8 Gen 3 的 v75 QNN 包，768x512 / 25 帧）。
- 中文界面：FOSS 4.5.x 已自带 values-zh/strings.xml（1061 条），无需从 huxu7755 合并。
- 本次改动：把所有模型下载源从 huggingface.co 换成 https://hf-mirror.com（共 439 处），并在 local.properties / build.gradle.kts 默认打开 DEBUG_PREMIUM。
- CI：合并 FOSS 的自动发 Release 与 huxu7755 的 SDK/NDK 安装配置，push v* 标签或手动运行即出 APK。

## 改了哪些文件（路径即仓库内路径）
| 文件 | 改动 |
| --- | --- |
| android/app/src/main/java/com/llmhub/llmhub/data/ModelData.kt | 428 处 HF 域名替换；含 LTXV_QNN_REPO / LTXV_GGUF_REPO / v75 定义 |
| android/app/src/main/java/com/llmhub/llmhub/data/ModelDownloader.kt | 8 处替换 + 镜像不支持登录的提示文案 |
| android/app/src/main/java/com/llmhub/llmhub/viewmodels/ModelDownloadViewModel.kt | 3 处替换，含视频包 downloadUrl 硬编码 |
| android/app/build.gradle.kts | DEBUG_PREMIUM 默认值 false -> true |
| .github/workflows/main.yml | 新建：构建 APK + 自动创建 Release |

## 固定签名（重要）

自 v4.5.1-cn2 起，CI 从仓库 Actions Secret  恢复同一个 PKCS12 密钥
（，alias ），覆盖 AGP 自带的临时 debug keystore。
证书 SHA-256：

- 之后改代码 → push 新 v* 标签 → 新包签名不变 → **直接覆盖安装，已下载的模型不丢**
- 本地自己编译：把同一个  放到 ，签名同样一致
- 每次构建自动递增 versionCode（169 + run_number），避免覆盖安装被版本拦住
- Release 里附 ，可自行核对指纹
- ⚠️ 私钥只存在 Secret 和你的本地备份里，**丢了就永远签不出同签名的包，务必备份**

## 使用
1. 装 APK（debug 签名，与 Play 版签名不同，需先卸载 Play 版）。
2. 打开 Video Generator，选 LTX-Video 2B 0.9.8 的 Snapdragon NPU 后端，8 Gen 3 自动对应 v75。
3. 模型约 7~8GB，从 HF-Mirror 下载，建议留 10GB 存储、连 WiFi、注意发热。

## 已知限制
- HF-Mirror 不支持账号登录：Gemma / Llama 等 gated 模型无法用镜像下载，需自备 HF Token 并切回官方源；LTX-Video QNN 包与 calcuis/ltxv-gguf 均为公开仓库，不受影响。
- v75 是固定形状编译包，输出固定 768x512 / 25 帧，不可调分辨率。
- 上游许可证为 PolyForm Noncommercial 1.0.0，仅限个人非商业使用，请勿分发或商用。
- 视频模型必须在 App 内下载，手动导入入口只支持 .task/.litertlm/.mnn/.gguf 等聊天与绘图模型。

## 触发 Release
push 一个 v* 标签，或在 Actions 里手动运行「构建并发布国内源版 APK」并填 tag 名。
