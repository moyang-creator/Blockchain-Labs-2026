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
   ```

   - 项目名称输入：`my-finance-l3`
   - 选择 **Arbitrum Nitro**
   - 选择 **Devnet**

3. 启动本地 L3 链：

   ```bash
   cd my-finance-l3
   docker compose up -d
   ```

4. 查看链信息：

   ```bash
   # 查看 RPC 和 Chain ID
   docker compose logs sequencer | grep -E "RPC|Chain ID"
   ```

   常见信息：
   - **RPC URL**：`http://localhost:8547`
   - **Chain ID**：通常为 `412346`（或启动时显示的值）

5. 将链添加到 MetaMask：
   - 网络名称：`Local Orbit L3`
   - RPC URL：`http://localhost:8547`
   - Chain ID：`412346`
   - 货币符号：`ETH`

### 步骤 2：使用 Remix IDE 部署金融合约

1. 打开 [Remix IDE](https://remix.ethereum.org/)
2. Environment 选择 **Injected Provider - MetaMask**，连接你的本地 L3 链
3. 新建以下三个文件，复制下方合约代码：

   - `StudentStablecoin.sol`
   - `SimpleLendingPool.sol`
   - `RWASimulation.sol`

4. 编译并依次部署合约（推荐顺序：稳定币 → 借贷池 → RWA）

### 步骤 3（可选进阶）：使用 Hardhat 部署

```bash
npm install
# 配置 .env 文件后执行
npx hardhat run scripts/deploy-all.js --network orbitDevnet
```

---

## 五、三个金融合约代码

### 1. contracts/StudentStablecoin.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

contract StudentStablecoin is ERC20, Ownable, Pausable {
    constructor() ERC20("StudentUSD", "sUSD") Ownable(msg.sender) {
        _mint(msg.sender, 1_000_000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) external onlyOwner whenNotPaused {
        _mint(to, amount);
        emit StablecoinMinted(to, amount);
    }

    function burn(uint256 amount) external whenNotPaused {
        _burn(msg.sender, amount);
        emit StablecoinBurned(msg.sender, amount);
    }

    function pause() external onlyOwner { _pause(); emit ContractPaused(msg.sender); }
    function unpause() external onlyOwner { _unpause(); emit ContractUnpaused(msg.sender); }

    event StablecoinMinted(address indexed to, uint256 amount);
    event StablecoinBurned(address indexed from, uint256 amount);
    event ContractPaused(address indexed by);
    event ContractUnpaused(address indexed by);
}
```

### 2. contracts/SimpleLendingPool.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SimpleLendingPool is Ownable {
    IERC20 public stablecoin;
    mapping(address => uint256) public deposits;
    mapping(address => uint256) public lastDepositTime;
    uint256 public constant INTEREST_RATE = 5; // 年化 5%

    constructor(address _stablecoin) Ownable(msg.sender) {
        stablecoin = IERC20(_stablecoin);
    }

    function deposit(uint256 amount) external {
        require(amount > 0, "Amount must be > 0");
        stablecoin.transferFrom(msg.sender, address(this), amount);
        deposits[msg.sender] += amount;
        lastDepositTime[msg.sender] = block.timestamp;
        emit Deposited(msg.sender, amount);
    }

    function withdraw(uint256 amount) external {
        require(deposits[msg.sender] >= amount, "Insufficient deposit");
        uint256 interest = calculateInterest(msg.sender);
        uint256 total = amount + interest;
        deposits[msg.sender] -= amount;
        if (deposits[msg.sender] == 0) delete lastDepositTime[msg.sender];
        stablecoin.transfer(msg.sender, total);
        emit Withdrawn(msg.sender, amount, interest);
    }

    function calculateInterest(address user) public view returns (uint256) {
        if (lastDepositTime[user] == 0) return 0;
        uint256 timePassed = block.timestamp - lastDepositTime[user];
        return (deposits[user] * INTEREST_RATE * timePassed) / (365 days * 100);
    }

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 principal, uint256 interest);
}
```

### 3. contracts/RWASimulation.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract RWASimulation is ERC721, Ownable {
    uint256 private _nextTokenId;
    mapping(uint256 => uint256) public assetValue;
    mapping(uint256 => string) public assetDescription;

    constructor() ERC721("RealWorldAsset", "RWA") Ownable(msg.sender) {}

    function mintRWA(address to, uint256 valueInUSD, string memory description) external onlyOwner {
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
        assetValue[tokenId] = valueInUSD;
        assetDescription[tokenId] = description;
        emit RWAMinted(tokenId, to, valueInUSD, description);
    }

    function transferFrom(address from, address to, uint256 tokenId) public override {
        super.transferFrom(from, to, tokenId);
        emit RWATransferred(tokenId, from, to, assetValue[tokenId]);
    }

    event RWAMinted(uint256 indexed tokenId, address indexed owner, uint256 valueInUSD, string description);
    event RWATransferred(uint256 indexed tokenId, address from, address to, uint256 valueInUSD);
}
```

---

## 六、实验作业要求

1. 你的 L3 链信息（RPC URL、Chain ID）
2. 三个合约的部署地址
3. 观察记录（Gas 费用、交易速度、Events 等）
4. 思考题：
   - L3 与 L2 在金融场景中的优势与权衡是什么？
   - Events 在区块链金融监管中起什么作用？
   - RWA 合约用于真实资产时还需增加哪些功能？

---

**实验提示**：本地部署虽然只能在自己电脑上访问，但能让你真正理解区块链底层技术，是目前最适合高校课堂的免费方案。

祝实验顺利！

---