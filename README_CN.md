# LLM Hub 国内源版（基于 Omar-afk801/LLM-Hub-FOSS）

## 这个版本做了什么
- 底子：Omar-afk801/LLM-Hub-FOSS —— 已去广告、已解锁全部高级功能、Release 带现成 APK，且保留 LTX-Video 2B 0.9.8 图生视频（含骁龙 8 Gen 3 的 v75 QNN 包，768x512 / 25 帧）。
- 中文界面：FOSS 4.5.x 已自带 values-zh/strings.xml（1061 条），无需从 huxu7755 合并。
- 模型下载源：huggingface.co 全部换成 https://hf-mirror.com（共 438 处：428 + 7 + 3），并默认打开 DEBUG_PREMIUM。
- CI：JDK 17 + Android SDK/NDK → assembleDebug → apksigner 记录指纹 → 自动创建 Release。

## 改了哪些文件（路径即仓库内路径）
| 文件 | 改动 |
| --- | --- |
| android/app/src/main/java/com/llmhub/llmhub/data/ModelData.kt | 428 处域名替换；保留 LTXV_QNN_REPO / LTXV_GGUF_REPO / v75 定义 |
| android/app/src/main/java/com/llmhub/llmhub/data/ModelDownloader.kt | 7 处替换 + 镜像不支持登录的提示文案 |
| android/app/src/main/java/com/llmhub/llmhub/viewmodels/ModelDownloadViewModel.kt | 3 处替换，含视频包 downloadUrl 硬编码 |
| android/app/build.gradle.kts | DEBUG_PREMIUM 默认 true；覆盖默认 debug signingConfig 指向固定密钥 |
| .github/workflows/main.yml | 构建 APK + 固定签名 + 自动递增 versionCode + 自动创建 Release |
| .gitignore | 忽略 android/app/ci-signing.p12 |

## 固定签名（重要）

自 v4.5.1-cn2 起，CI 从仓库 Actions Secret `SIGNING_KEYSTORE_BASE64` 恢复同一个 PKCS12 密钥
（`llmhub-cn.p12`，alias `llmhub-cn`），覆盖 AGP 自带的临时 debug keystore。
证书 SHA-256：`72:23:D0:38:A8:B1:43:15:87:0F:66:DA:ED:FB:17:F0:FF:8A:D0:C2:06:84:15:16:23:D9:D6:BD:3D:F0:EA:E7`

- 之后改代码 → push 新 v* 标签 → 新包签名不变 → **直接覆盖安装，已下载的模型不丢**
- 本地自己编译：把同一个 `llmhub-cn.p12` 放到 `android/app/ci-signing.p12`，签名同样一致
- 每次构建自动递增 versionCode（169 + run_number），避免覆盖安装被版本拦住
- Release 里附 `cert-sha256.txt`，可自行核对指纹
- ⚠️ 私钥只存在 Secret 和你的本地备份里，**丢了就永远签不出同签名的包，务必备份**
- ⚠️ 从 v4.5.1-cn1（临时 debug 签名）升级到本版，**仍需卸载一次**；之后就不用了

## 使用
1. 装 APK（固定签名，与 Play 官方版签名不同，Play 版需先卸载）。
2. 打开 Video Generator，选 LTX-Video 2B 0.9.8 的 Snapdragon NPU 后端，8 Gen 3 自动对应 v75。
3. 模型约 7~8GB，从 HF-Mirror 下载，建议留 10GB 存储、连 WiFi、注意发热。

## 触发 Release
push 一个 v* 标签，或在 Actions 里手动运行并填 tag 名。

## 已知限制
- HF-Mirror 不支持账号登录：Gemma / Llama 等 gated 模型需自备 HF Token 并切回官方源；LTX-Video QNN 包与 calcuis/ltxv-gguf 均为公开仓库，不受影响。
- v75 是固定形状编译包，输出固定 768x512 / 25 帧，不可调分辨率。
- 上游许可证为 PolyForm Noncommercial 1.0.0，仅限个人非商业使用，请勿分发或商用。
- 视频模型必须在 App 内下载，手动导入入口只支持 .task/.litertlm/.mnn/.gguf 等聊天与绘图模型。
