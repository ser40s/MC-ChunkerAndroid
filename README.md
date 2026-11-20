Chunker Android

把 Chunker 的 Minecraft 世界转换引擎带到 Android，让手机也能完成 Java 版 ↔ 基岩版 世界存档转换。

---

📖 简介

Chunker 是优秀的开源世界转换工具，但官方只有 Windows / Linux / macOS 桌面版，没有 Android 版。本项目将它移植到手机，实现完整闭环：

选择世界 → 选择目标格式/版本 → 转换 → 导出

Chunker 本体是纯 Java 转换引擎（Java 17），桌面前端只是 Electron 壳，核心与 GUI 完全解耦——这是移植可行的根本前提。

---

✨ 特性（规划）

· 📱 纯 Android 原生体验（Kotlin）
· 🔄 Bedrock ↔ Java 双向转换，支持版本升降
· 📂 SAF 选择世界目录，自动复制到私有目录处理
· 📊 实时进度条 + 可导出日志
· 🔋 前台服务 + WakeLock，熄屏不中断
· 🧠 内存预检、低内存模式、OOM 提示
· 📦 转换全程本地，不上传世界数据

---

🏗️ 技术路线

采用 两步走 策略：

路线 A：内置 JRE 进程沙箱（首选）

· 直接使用官方 chunker-cli.jar，零引擎改动
· jlink 裁剪精简 JRE 打包进 APK，独立进程运行
· ✅ 完全绕开 ART 兼容问题，有 Chunkoid 生产验证
· ⚠️ APK 体积较大（约 30–80MB）

路线 B：纯 Java 库嵌入（深化）

· 将 cli 模块引入 Android 工程，ART 直接运行
· 需重写入口（去除 System.exit、stdout、File 路径依赖），处理 R8/Guava/LevelDB 兼容
· ✅ 体积小、启动快
· ⚠️ 无公开先例，存在未知风险

先 A 后 B：A 以最低风险最快交付，B 作为后续瘦身提速方向。

---

🗺️ 里程碑

阶段 内容 工作量
M1 · PoC 工程骨架、jlink 裁剪、真机冒烟转换 3～5 人日
M2 · 产品化 UI、SAF、前台服务、内存/空间预检、输出管理 2～4 周
M3 · 深化（可选） JRE 再裁剪、路线 B 评估、高级功能 2～3 周
M4 · 发布 版权声明、测试矩阵、GitHub Release 1 周

验收标准：50+ 真实世界样本成功率 ≥95%；连续 10 次转换无崩溃；覆盖主流 arm64 真机与 Android 8～15。

---

🔍 关键纠偏

初版方案曾把 LevelDB 崩溃归因于"JNI 缺 .so"，经核实不成立——Chunker 使用的是纯 Java LevelDB 实现，无 JNI 依赖。真正需排查的是：

· Guava：33.x-jre 部分类在 Android 不可用，需换 -android 变体
· R8：LevelDB 通过反射加载实现类，易被裁剪误删，需加 keep 规则
· ART 运行时：纯 Java LevelDB 对文件锁、mmap 的处理差异

---

⚠️ 主要风险

风险 应对
LevelDB 在 ART 兼容异常 路线 A 进程沙箱绕开，M1 单独冒烟
大世界内存/存储不足 -Xmx 限制、低内存模式、空间预检
后台进程被杀 前台服务 + WakeLock + 断点校验
APK 体积过大 jdeps 收窄、rootfs 压缩、按 ABI 拆包

---

📜 许可证

· 基于 Chunker（MIT） 移植，保留原始版权声明
· 参考 Chunkoid（GPL-3.0）思路，壳层独立实现
· 转换全程本地，不上传世界数据

---

🙏 致谢

· [HiveGamesOSS/Chunker](https://github.com/HiveGamesOSS/Chunker) — 转换引擎
· [Dicecan/Chunkoid](https://github.com/Dicecan/Chunkoid) — Android 先例参考

---

🚧 规划阶段 — M1 概念验证待启动。欢迎提交 Issue / PR，尤其期待真实世界样本测试与 ART 兼容性侦察的帮助。

让世界转换，装进口袋。 📱⛏️
