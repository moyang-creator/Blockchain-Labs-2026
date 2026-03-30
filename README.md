# Blockchain-Labs-2026
# 区块链金融课程 - Arbitrum Orbit L3 实验实验室

**课程名称**：《区块链金融》  
**实验主题**：自建 Arbitrum Orbit 链（L2 / L3）并部署金融智能合约  
**实验目标**：让学生亲手部署自己的区块链，理解派生链（L2/L3）的技术原理，以及稳定币发行、DeFi 借贷和真实世界资产（RWA）代币化在区块链金融中的实际应用。

---

## 一、实验概述

本次实验提供两种部署方式：
- **主力方案（推荐）**：本地 Docker 部署 Arbitrum Orbit L3 DevNet（完全免费）
- **保底方案**：Remix IDE + Arbitrum Sepolia 测试网（公共免费 L2）

学生可根据自身情况选择，优先推荐使用**本地 Docker 部署 L3**，以更好地体验自定义链的优势。

---

## 二、L2 与 L3 对比说明（重要）

| 项目                  | **L3（推荐，本地部署）**                          | **L2（公共测试网）**                              |
|-----------------------|--------------------------------------------------|--------------------------------------------------|
| **Settlement Layer** | Arbitrum Sepolia（L2）                           | Ethereum Sepolia（L1）                           |
| **层级**             | Layer 3（构建在 L2 之上）                        | Layer 2（直接构建在 Ethereum 之上）              |
| **Gas 费用**         | 极低，几乎为 0                                   | 较低，但高于 L3                                  |
| **安全性**           | 继承 Arbitrum L2 + Ethereum L1                   | 直接继承 Ethereum L1                             |
| **部署方式**         | 本地 Docker（完全免费）                          | Remix + 公共测试网（免费）                       |
| **适用场景**         | 高频交易、专用金融应用（如 RWA、借贷）           | 通用金融应用                                     |
| **教学价值**         | 更高，能深刻理解“自建区块链”和派生链概念        | 适合入门，操作更简单                             |

**建议**：大部分学生优先完成 L3 本地部署。有余力的同学可同时体验 L2 与 L3，并对比差异。

---

## 三、实验工具准备

- Docker Desktop（必须）
- MetaMask 钱包
- Remix IDE（https://remix.ethereum.org/）
- Node.js（可选，用于 Hardhat 进阶部署）

---

## 四、实验步骤

### 步骤 1：本地 Docker 部署 Arbitrum Orbit L3（主力推荐方案）

1. 安装并启动 **Docker Desktop**（确保正在运行）。
2. 打开终端（PowerShell / Terminal），执行以下命令：

   ```bash
   # 创建并进入工作目录
   mkdir orbit-l3-finance && cd orbit-l3-finance

   # 一键初始化 Orbit 链项目
   npx create-orbit-chain@latest
