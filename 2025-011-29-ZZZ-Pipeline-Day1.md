---
layout: post
title: "[DevLog] ZZZ风格化角色管线开发 | Day 01-02"
date: 2025-11-29 16:00:00
tags: [Unity, URP, Shader, ZZZ]
---

# [DevLog] ZZZ风格化角色管线开发 | Day 01-02: 从资产清洗到自定义URP Shader

> **项目代号:** ZZZ-Pipeline-Lite
> **核心目标:** 构建一套基于 Unity URP & Houdini 的风格化角色生产管线，复刻《绝区零》渲染表现。
> **当前进度:** 基础设施搭建 ✅ / 资产标准化 ✅ / 基础 Cel-Shading 实现 ✅

---

## 🛠️ 一、 基础设施与工程规范 (Infrastructure)

在大厂的管线标准中，工程规范（Engineering Standard）的重要性不亚于算法实现。为了避免后期的“屎山”代码，我在项目初期花费了大量精力进行环境配置。

### 1.1 版本控制与大文件管理
*   **Git LFS 配置:** 针对游戏开发中常见的二进制大文件（`.fbx`, `.png`, `.psd`, `.dll`），配置了 Git LFS (Large File Storage) 追踪，避免仓库体积膨胀。
*   **忽略文件管理:** 配置了 Unity 专用的 `.gitignore`，过滤掉 `Library`, `Temp`, `Logs` 等临时文件，保持仓库纯净。
*   **网络环境:** 解决了国内 GitHub 连接不稳定的问题（Proxy 配置），确保 CI/CD 流程的潜在可行性。

### 1.2 目录结构规范
采用了 **源文件 (Arts)** 与 **引擎资产 (Assets)** 分离的策略：
*   `D:/Projects/ZZZ-Pipeline-Lite/Arts/`：存放 Maya/Blender/Houdini 的源文件。
*   `D:/Projects/ZZZ-Pipeline-Lite/UnityProject/`：仅存放经过标准化处理后导入引擎的资产。
*   **迁移复盘:** 遭遇了 Maya 对中文路径/桌面路径支持不佳导致的 Namespace 死锁问题，最终将项目完整迁移至 D 盘根目录全英文路径下，彻底解决了 DCC 软件的文件读写权限问题。

---

## 🎨 二、 DCC 资产流水线 (DCC Pipeline)

本次使用的测试资产源自 MMD 格式（.pmx），这类资产存在单位不统一、垃圾节点多、骨骼结构非标等问题。为了将其接入 Unity 管线，我制定了一套**“资产标准化清洗流程”**。

### 2.1 痛点与解决方案
*   **问题 A: 单位比例失调 (Scale Issue)**
    *   **现象:** Blender 导出的模型在 Unity 中表现为“进击的巨人”（放大100倍）或“微米人”。
    *   **解决:** 统一了 Maya 和 Unity 的单位标准（1 Unit = 1 Meter）。在 Maya 导出前执行 **Freeze Transformations**，确保 Scale 重置为 `(1,1,1)`，避免物理计算异常。
*   **问题 B: 垃圾数据残留**
    *   **现象:** 模型携带大量 MMD 专用的 RigidBody 刚体和 Joint 节点，导致层级混乱且消耗性能。
    *   **解决:** 在 Maya 大纲视图（Outliner）中进行了手动清洗，删除了除 Mesh 和主要 Bone 以外的所有辅助节点。
*   **问题 C: 轴向修正**
    *   **现象:** 导入后模型面朝 Z 轴负方向。
    *   **解决:** 在 Maya 中修正旋转 Y=180 度并冻结变换，确保在 Unity 左手坐标系中 `Forward` 方向正确。

### 2.2 妥协与迭代
由于源资产骨骼在转换过程中存在丢失风险，为了优先验证**渲染管线 (Rendering Pipeline)**，我采取了**“静态网格 (Static Mesh) 先行”**的策略。暂时以无骨骼的 `SM_Character` 导入 Unity，优先保证材质和 Shader 的开发进度。这也符合敏捷开发中 MVP (Minimum Viable Product) 的思路。

---

## 💻 三、 渲染开发：自定义 HLSL Shader

这是本阶段的核心突破。摒弃了 Shader Graph，直接使用 **HLSL** 编写基于 URP (Universal Render Pipeline) 的自定义 Shader，以获得对渲染流程的完全控制权。

### 3.1 URP Shader 框架搭建
创建了 `ZZZ_Character.shader`，构建了标准的 URP Pass 结构：
*   **Tags:** `"RenderPipeline" = "UniversalPipeline"`
*   **HLSL引用:** 引入 `Core.hlsl` 和 `Lighting.hlsl` 库。
*   **结构体:** 定义了 `Attributes` (OS) 和 `Varyings` (CS/WS) 数据流。

### 3.2 实现二次元阶梯光照 (Cel-Shading)
为了实现《绝区零》硬朗的卡通阴影效果，我抛弃了传统的 PBR 漫反射计算，编写了自定义的光照模型：

```c
// 核心代码片段：卡通渲染计算
half4 frag(Varyings input) : SV_Target
{
    // ... (采样贴图与获取光源) ...

    // 1. 半兰伯特 (Half-Lambert) 计算
    // 将 NdotL 从 [-1, 1] 映射到 [0, 1]，防止背光面死黑，使皮肤更通透
    float NdotL = dot(normal, lightDir);
    float halfLambert = NdotL * 0.5 + 0.5;

    // 2. 风格化阈值切割 (Stylized Threshold)
    // 使用 smoothstep 实现硬边缘过渡，_ShadowThreshold 控制阴影面积
    float ramp = smoothstep(_ShadowThreshold - _ShadowSmoothness, _ShadowThreshold + _ShadowSmoothness, halfLambert);

    // 3. 阴影着色
    // 不使用黑色阴影，而是使用 Lerp 混合环境色 (_ShadowColor)，提升画面空气感
    float3 lightIntensity = lerp(_ShadowColor.rgb, float3(1,1,1), ramp);

    // ... (最终合成) ...
}
