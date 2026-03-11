# ADR-001: 使用独立 PM 仓库协调三端

## 状态
已采纳 (2026-03-11)

## 背景
二狗产品分三端（Web/Android/鸿蒙），技术栈完全不同，但共享 API 接口、系统 Prompt、工具定义。

## 决策
创建独立的 `ergouPM` 仓库作为协调中心，存放共享定义和同步记录。

## 理由
- 三端技术栈差异大（Rust/Kotlin/ArkTS），Monorepo 收益低
- 个人开发者，git submodule 增加不必要复杂度
- 共享的是"定义"而非"代码"，文档仓库足够

## 后果
- 需手动保持 sync-log.md 更新
- 各端 CLAUDE.md 需引用 ergouPM 路径
