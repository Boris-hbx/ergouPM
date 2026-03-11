# 二狗 (Ergou) 项目管理中心

本仓库是二狗产品三端的协调中心，存放共享定义、API 规范和同步记录。

## 三端工程
| 端 | 路径 | 技术栈 | 说明 |
|---|---|---|---|
| Web 后端 + 前端 | `C:\Project\ergouWeb\` | Rust (Axum) + Tauri | 服务器 + 桌面客户端 |
| Android | `C:\Project\ergou\` | Kotlin + Jetpack Compose | 安卓客户端 |
| HarmonyOS | `C:\Project\ergouh\` | ArkTS + ArkUI | 鸿蒙客户端 |

## 本仓库职责
- **api/**：API 接口规范（唯一来源）
- **llm/**：系统 Prompt + 工具定义（唯一来源）
- **docs/**：产品路线图、架构决策记录
- **sync-log.md**：跨端变更同步记录

## 工作流
1. 修改共享定义 → 先在本仓库更新
2. 在 sync-log.md 记录变更 → 标注需同步的端
3. 各端按 sync-log 去实现
4. 实现完打勾 ✓
