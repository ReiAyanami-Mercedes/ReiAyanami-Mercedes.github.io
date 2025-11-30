# 📂 Project Log: ZZZ-Lite Character Pipeline
**日期:** Day 2 - Day 3 (Morning)
**当前阶段:** 风格化渲染开发 (Stylized Rendering Dev) & 工程重构 (Refactoring)
**项目状态:** 🟢 **Stable (稳定)** - 核心渲染管线跑通，版本控制已修复。

---

### ✅ 1. 已完成里程碑 (Achievements)

*   **渲染管线开发 (Rendering Pipeline):**
    *   [x] **URP HLSL 框架:** 抛弃 Shader Graph，搭建了基于 URP 的纯代码 Shader 基础框架。
    *   [x] **NPR 卡通渲染:** 实现了基于 `Half-Lambert` + `Smoothstep` 的硬边缘光照模型 (Cel-Shading)。
    *   [x] **参数化控制:** 在 Shader GUI 中暴露了 `Shadow Color` (阴影色), `Threshold` (阈值), `Smoothness` (羽化) 等核心参数。
    *   [x] **特殊材质支持:** 开发了 **`ZZZ_Stockings`** 专用 Shader，利用 Fresnel + ZWrite On 实现半透明黑丝透肉效果。

*   **资产标准化 (Asset Standardization):**
    *   [x] **材质分离:** 完成了角色多材质球 (Face, Body, Hair) 的拆分与独立配置。
    *   [x] **贴图修正:** 解决了 Unity 默认导入贴图导致的“黑边/变黑”问题 (Alpha Is Transparency 修正)。
    *   [x] **色彩空间:** 将项目色彩空间统一为 **Linear**，配合 ACES ToneMapping 提升画面通透度。

*   **工程与版本控制 (Infrastructure):**
    *   [x] **路径重构:** 将项目从桌面（含中文路径）迁移至 `D:\Projects`，彻底解决了 DCC 软件 (Maya) 的读写权限与乱码问题。
    *   [x] **Git 仓库重建:** 解决了 `.gitignore` 失效导致的 `Library` 文件夹上传事故 (124MB 大文件报错)，通过 `git init` 重建仓库，实现了 100% 纯净上传。

---

### 🐛 2. 遇到的问题与解决方案 (Issue Tracker)

| ID | 问题描述 (Issue) | 原因分析 (Root Cause) | 解决方案 (Solution) | 状态 |
| :--- | :--- | :--- | :--- | :--- |
| **006** | **Maya 无法导出/报错** | 项目路径包含中文字符（桌面），导致 Maya Namespace 识别错误。 | **项目迁移**至全英文路径；Obj 导入时禁用 Namespace。 | ✅ 已解决 |
| **007** | **Git Push 502/RPC Error** | `.gitignore` 配置晚于 `git add`，导致 `Library` 缓存被追踪，单文件超 100MB。 | 删除 `.git` 文件夹，**重建仓库 (Re-init)**，重新配置忽略规则并强制推送。 | ✅ 已解决 |
| **008** | **角色渲染全黑** | 1. 材质 BaseColor 默认为灰；2. 贴图 Alpha 通道混合错误。 | 将 BaseColor 设为纯白；关闭非透明贴图的 **Alpha Is Transparency** 选项。 | ✅ 已解决 |
| **009** | **半透明穿帮 (腿部)** | 透明 Shader 默认关闭深度写入 (ZWrite Off)。 | 在 Shader Pass 中强制开启 **`ZWrite On`**。 | ✅ 已解决 |
| **010** | **分支混乱 (Master/Main)** | 本地默认 Master 与 远程 Main 不一致。 | 使用 `git branch -m` 重命名本地分支，并更新 GitHub 默认分支设置。 | ✅ 已解决 |

---

### 📉 3. 遗留的技术债 (Technical Debt)

*   **面部阴影不完美 (Face Shadow):** 目前面部使用通用卡通光照，特定角度会出现“猪腰子脸”或不美观的阴影形状。
    *   *计划:* Day 3 下午使用 **Houdini** 烘焙 SDF 贴图来解决。
*   **骨骼动画缺失:** 目前角色仍为静态网格 (Static Mesh)。
    *   *计划:* 渲染效果完善后，后期统一处理动画绑定。

---

### 🚀 4. 下一步行动计划 (Next Steps)

**当前优先级: Critical (关键)**

1.  **Houdini SDF 流程:**
    *   导入角色头部模型到 Houdini。
    *   构建 VDB SDF 算子。
    *   将 SDF 数据烘焙至贴图 (Texture Baking)。
2.  **Shader 升级:**
    *   修改 `ZZZ_Character` Shader，增加 SDF 采样逻辑。
    *   实现基于 SDF 的可控面部阴影 (Face Rendering)。

---

### 📝 个人备注 (Personal Note)
> "Technical Art is about solving problems."
> 这里的每一个报错，都是通往大厂路上的垫脚石。Git 仓库的重建虽然惊险，但彻底理解了 `.gitignore` 的生效机制。渲染效果的从无到有，验证了 HLSL 编写能力。
> **下午目标：攻克 Houdini！**

---

你可以把这个直接复制，新建一个 `Log_Day03.md` 传上去！
去睡个好觉吧，我的大工程师！下午见！👋😴
