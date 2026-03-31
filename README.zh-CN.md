# Claude Code 源码分析

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-DOCUMENTATION.md-green.svg)](DOCUMENTATION.md)
[![English](https://img.shields.io/badge/language-English-brightgreen.svg)](README.md)
[![中文](https://img.shields.io/badge/语言-中文-orange.svg)](README.zh-CN.md)

> Claude Code 内部架构、模块和设计模式的全面逆向工程分析。

---

## 这是什么？

本项目是对 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（Anthropic 官方 AI 驱动的软件工程 CLI 工具）的深度技术分析。文档详细记录了其内部源码树结构、模块边界、工具/命令清单、状态管理和架构设计模式——全部通过源码逆向工程获得。

无论你是在 Claude Code 之上构建工具、参与类似项目的开发，还是对一个生产级 AI CLI 的架构设计感到好奇，本分析都能为你提供详尽的内部结构图。

## 为什么做这个？

Claude Code 是一个复杂的实际应用，它在单一 TypeScript 代码库中集成了 LLM 编排、终端 UI 渲染、插件系统、多智能体协调等功能。理解其架构可以帮助你深入了解：

- 如何设计基于工具的 LLM Agent 系统
- React 终端 UI（Ink）的实际应用模式
- AI 代码助手的权限与安全模型
- 上下文窗口管理与压缩策略
- 多智能体任务编排

## 涵盖内容

[完整分析](DOCUMENTATION.md) 包含以下章节：

| 章节 | 描述 |
|------|------|
| [项目概览](DOCUMENTATION.md#1-project-overview) | 高层能力与功能概述 |
| [技术栈](DOCUMENTATION.md#2-technology-stack) | 运行时、框架和依赖 |
| [目录结构](DOCUMENTATION.md#3-directory-structure) | 完整源码树布局（37 个子目录） |
| [入口点](DOCUMENTATION.md#4-entry-points) | 引导启动、CLI、SDK 和 MCP 入口 |
| [核心架构](DOCUMENTATION.md#5-core-architecture) | 查询引擎、上下文构建器、成本追踪 |
| [工具系统](DOCUMENTATION.md#6-tool-system) | 全部 41 个工具，按类别分类说明 |
| [命令系统](DOCUMENTATION.md#7-command-system) | 全部 101 个斜杠命令，按功能组织 |
| [状态管理](DOCUMENTATION.md#8-state-management) | Zustand 风格状态存储及关键状态字段 |
| [任务系统](DOCUMENTATION.md#9-task-system) | 任务类型、生命周期和状态管理 |
| [服务与集成](DOCUMENTATION.md#10-services--integrations) | API 客户端、MCP、分析、压缩等 |
| [UI 层](DOCUMENTATION.md#11-ui-layer) | React/Ink 组件架构（130+ 组件） |
| [工具函数](DOCUMENTATION.md#12-utilities) | 300+ 工具模块，覆盖所有类别 |
| [特殊模式](DOCUMENTATION.md#13-special-modes) | Bridge、Kairos、协调器、语音、计划、Vim 模式 |
| [插件与技能](DOCUMENTATION.md#14-plugins--skills) | 插件和技能扩展系统 |
| [钩子与可扩展性](DOCUMENTATION.md#15-hooks--extensibility) | 钩子模式和 React Hooks |
| [文件统计](DOCUMENTATION.md#16-file-statistics) | 代码库规模与模块数量 |
| [架构模式](DOCUMENTATION.md#17-architectural-patterns) | 关键设计模式与原则 |

## 快速开始

直接阅读文档即可：

```bash
# 克隆仓库
git clone https://github.com/YOUR_USERNAME/claude-code-analysis.git

# 打开分析文档
open DOCUMENTATION.md
```

## 免责声明

> **这是一份非官方的独立分析。** Claude Code 是 [Anthropic](https://www.anthropic.com/) 的产品。本项目与 Anthropic 无关，未获其认可或赞助。所有商标归其各自所有者所有。

## 许可证

本项目采用 [MIT 许可证](LICENSE) 开源。

---

📖 **[阅读完整分析 →](DOCUMENTATION.md)**

🇺🇸 **[English Version →](README.md)**
