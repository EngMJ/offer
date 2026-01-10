# Web3 前端开发指南

> Web3 前端开发涉及区块链交互、钱包连接、智能合约调用等技术

---

## 1. Web3 基础概念

### 1.1 什么是 Web3？

Web3 是基于区块链技术的去中心化网络，与传统 Web2 的主要区别：

| 特性 | Web2 | Web3 |
|------|------|------|
| 数据存储 | 中心化服务器 | 分布式区块链 |
| 身份认证 | 用户名/密码 | 钱包地址/签名 |
| 资产所有权 | 平台控制 | 用户完全控制 |
| 交易中介 | 需要第三方 | 点对点交易 |

### 1.2 核心概念

- **钱包 (Wallet)**: 存储私钥、管理资产、签名交易的工具
- **智能合约 (Smart Contract)**: 部署在区块链上的自动执行程序
- **DApp**: 去中心化应用，前端与智能合约交互
- **Gas**: 执行交易需要支付的费用
- **ABI**: 应用二进制接口，描述智能合约的方法和事件

---

## 2. ethers.js 使用指南

### 2.1 安装与基础配置

```bash
npm install ethers
```

### 2.2 连接区块链

```typescript
import { ethers, BrowserProvider, JsonRpcProvider } from 'ethers';

// 连接到浏览器钱包（MetaMask）
async function connectBrowserWallet() {
  if (typeof window.ethereum === 'undefined') {
    throw new Error('请安装 MetaMask');
  }
  
  const provider = new BrowserProvider(window.ethereum);
  const signer = await provider.getSigner();
  const address = await signer.getAddress();
  
  return { provider, signer, address };
}

// 连接到 RPC 节点
const rpcProvider = new JsonRpcProvider('https://mainnet.infura.io/v3/YOUR_KEY');
```

### 2.3 读取区块链数据

```typescript
// 获取账户余额
async function getBalance(address: string) {
  const balance = await provider.getBalance(address);
  // 将 Wei 转换为 ETH
  return ethers.formatEther(balance);
}

// 获取区块信息
async function getBlock() {
  const block = await provider.getBlock('latest');
  console.log('区块号:', block.number);
  console.log('时间戳:', block.timestamp);
}

// 获取交易信息
async function getTransaction(txHash: string) {
  const tx = await provider.getTransaction(txHash);
  const receipt = await provider.getTransactionReceipt(txHash);
  return { tx, receipt };
}
```

### 2.4 与智能合约交互

```typescript
// 合约 ABI（简化示例）
const ERC20_ABI = [
  'function name() view returns (string)',
  'function symbol() view returns (string)',
  'function decimals() view returns (uint8)',
  'function balanceOf(address) view returns (uint256)',
  'function transfer(address to, uint256 amount) returns (bool)',
  'event Transfer(address indexed from, address indexed to, uint256 value)'
];

// 创建合约实例
const contractAddress = '0x...';
const contract = new ethers.Contract(contractAddress, ERC20_ABI, provider);

// 只读调用
async function getTokenInfo() {
  const name = await contract.name();
  const symbol = await contract.symbol();
  const decimals = await contract.decimals();
  return { name, symbol, decimals };
}

// 写入调用（需要 signer）
async function transferTokens(to: string, amount: string) {
  const contractWithSigner = contract.connect(signer);
  const tx = await contractWithSigner.transfer(
    to, 
    ethers.parseUnits(amount, 18)
  );
  // 等待交易确认
  const receipt = await tx.wait();
  return receipt;
}
```

### 2.5 监听事件

```typescript
// 监听合约事件
contract.on('Transfer', (from, to, value, event) => {
  console.log(`转账: ${from} -> ${to}, 金额: ${ethers.formatEther(value)}`);
});

// 查询历史事件
async function getTransferHistory() {
  const filter = contract.filters.Transfer();
  const events = await contract.queryFilter(filter, -10000); // 最近 10000 个区块
  return events;
}
```

---

## 3. viem 使用指南

viem 是一个更现代、类型安全的以太坊库：

### 3.1 安装

```bash
npm install viem
```

### 3.2 基础使用

```typescript
import { createPublicClient, createWalletClient, http, custom } from 'viem';
import { mainnet } from 'viem/chains';

// 创建公共客户端（只读）
const publicClient = createPublicClient({
  chain: mainnet,
  transport: http('https://mainnet.infura.io/v3/YOUR_KEY'),
});

// 创建钱包客户端（需要签名）
const walletClient = createWalletClient({
  chain: mainnet,
  transport: custom(window.ethereum),
});

// 获取余额
async function getBalance(address: `0x${string}`) {
  const balance = await publicClient.getBalance({ address });
  return balance;
}

// 读取合约
async function readContract() {
  const data = await publicClient.readContract({
    address: '0x...',
    abi: ERC20_ABI,
    functionName: 'balanceOf',
    args: ['0x...'],
  });
  return data;
}

// 写入合约
async function writeContract() {
  const [account] = await walletClient.getAddresses();
  
  const hash = await walletClient.writeContract({
    address: '0x...',
    abi: ERC20_ABI,
    functionName: 'transfer',
    args: ['0x...', 1000000000000000000n],
    account,
  });
  
  // 等待交易确认
  const receipt = await publicClient.waitForTransactionReceipt({ hash });
  return receipt;
}
```

---

## 4. wagmi Hooks（React）

wagmi 是 React 的 Web3 Hooks 库：

### 4.1 安装与配置

```bash
npm install wagmi viem @tanstack/react-query
```

```typescript
// config.ts
import { createConfig, http } from 'wagmi';
import { mainnet, sepolia } from 'wagmi/chains';
import { injected, walletConnect } from 'wagmi/connectors';

export const config = createConfig({
  chains: [mainnet, sepolia],
  connectors: [
    injected(),
    walletConnect({ projectId: 'YOUR_PROJECT_ID' }),
  ],
  transports: {
    [mainnet.id]: http(),
    [sepolia.id]: http(),
  },
});
```

```tsx
// App.tsx
import { WagmiProvider } from 'wagmi';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { config } from './config';

const queryClient = new QueryClient();

function App() {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <YourApp />
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

### 4.2 常用 Hooks

```tsx
import { 
  useAccount, 
  useConnect, 
  useDisconnect,
  useBalance,
  useReadContract,
  useWriteContract,
  useWaitForTransactionReceipt
} from 'wagmi';

function WalletConnect() {
  // 账户信息
  const { address, isConnected, chain } = useAccount();
  
  // 连接钱包
  const { connect, connectors } = useConnect();
  
  // 断开连接
  const { disconnect } = useDisconnect();
  
  // 获取余额
  const { data: balance } = useBalance({
    address,
  });

  if (isConnected) {
    return (
      <div>
        <p>地址: {address}</p>
        <p>余额: {balance?.formatted} {balance?.symbol}</p>
        <button onClick={() => disconnect()}>断开连接</button>
      </div>
    );
  }

  return (
    <div>
      {connectors.map((connector) => (
        <button key={connector.id} onClick={() => connect({ connector })}>
          {connector.name}
        </button>
      ))}
    </div>
  );
}
```

### 4.3 合约交互

```tsx
import { useReadContract, useWriteContract, useWaitForTransactionReceipt } from 'wagmi';

const contractConfig = {
  address: '0x...' as const,
  abi: ERC20_ABI,
};

function TokenBalance() {
  const { address } = useAccount();
  
  // 读取合约
  const { data: balance, isLoading } = useReadContract({
    ...contractConfig,
    functionName: 'balanceOf',
    args: [address],
  });
  
  // 写入合约
  const { writeContract, data: hash, isPending } = useWriteContract();
  
  // 等待交易确认
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({
    hash,
  });

  const handleTransfer = () => {
    writeContract({
      ...contractConfig,
      functionName: 'transfer',
      args: ['0x...', 1000000000000000000n],
    });
  };

  return (
    <div>
      <p>代币余额: {balance?.toString()}</p>
      <button onClick={handleTransfer} disabled={isPending || isConfirming}>
        {isPending ? '签名中...' : isConfirming ? '确认中...' : '转账'}
      </button>
      {isSuccess && <p>交易成功!</p>}
    </div>
  );
}
```

---

## 5. 钱包连接

### 5.1 MetaMask 连接

```typescript
async function connectMetaMask() {
  if (typeof window.ethereum === 'undefined') {
    throw new Error('请安装 MetaMask');
  }
  
  // 请求连接
  const accounts = await window.ethereum.request({
    method: 'eth_requestAccounts',
  });
  
  return accounts[0];
}

// 监听账户变化
window.ethereum.on('accountsChanged', (accounts: string[]) => {
  console.log('账户变更:', accounts[0]);
});

// 监听链变化
window.ethereum.on('chainChanged', (chainId: string) => {
  console.log('链变更:', chainId);
  window.location.reload(); // 推荐重新加载页面
});
```

### 5.2 WalletConnect

```typescript
import { createWeb3Modal, defaultWagmiConfig } from '@web3modal/wagmi';
import { mainnet, sepolia } from 'wagmi/chains';

const projectId = 'YOUR_WALLETCONNECT_PROJECT_ID';

const metadata = {
  name: 'My DApp',
  description: 'My Web3 Application',
  url: 'https://mydapp.com',
  icons: ['https://mydapp.com/icon.png'],
};

const chains = [mainnet, sepolia];
const config = defaultWagmiConfig({ chains, projectId, metadata });

createWeb3Modal({ wagmiConfig: config, projectId, chains });
```

### 5.3 多钱包支持

```tsx
import { useConnect } from 'wagmi';

function WalletSelector() {
  const { connectors, connect, isPending } = useConnect();

  return (
    <div className="wallet-list">
      {connectors.map((connector) => (
        <button
          key={connector.uid}
          onClick={() => connect({ connector })}
          disabled={isPending}
        >
          <img src={connector.icon} alt={connector.name} />
          <span>{connector.name}</span>
        </button>
      ))}
    </div>
  );
}
```

---

## 6. 交易与签名

### 6.1 发送交易

```typescript
import { ethers } from 'ethers';

async function sendTransaction(to: string, value: string) {
  const tx = await signer.sendTransaction({
    to,
    value: ethers.parseEther(value),
  });
  
  console.log('交易哈希:', tx.hash);
  
  // 等待确认
  const receipt = await tx.wait();
  console.log('交易确认，区块:', receipt.blockNumber);
  
  return receipt;
}
```

### 6.2 消息签名

```typescript
// 个人签名（用于登录验证）
async function signMessage(message: string) {
  const signature = await signer.signMessage(message);
  return signature;
}

// 验证签名
function verifySignature(message: string, signature: string) {
  const recoveredAddress = ethers.verifyMessage(message, signature);
  return recoveredAddress;
}

// 登录流程示例
async function login() {
  const address = await signer.getAddress();
  const nonce = await fetchNonceFromServer(address);
  const message = `登录验证\nNonce: ${nonce}`;
  
  const signature = await signMessage(message);
  
  // 发送到后端验证
  const token = await verifyOnServer(address, message, signature);
  return token;
}
```

### 6.3 EIP-712 类型化数据签名

```typescript
const domain = {
  name: 'My DApp',
  version: '1',
  chainId: 1,
  verifyingContract: '0x...',
};

const types = {
  Permit: [
    { name: 'owner', type: 'address' },
    { name: 'spender', type: 'address' },
    { name: 'value', type: 'uint256' },
    { name: 'nonce', type: 'uint256' },
    { name: 'deadline', type: 'uint256' },
  ],
};

const value = {
  owner: '0x...',
  spender: '0x...',
  value: 1000000000000000000n,
  nonce: 0,
  deadline: Math.floor(Date.now() / 1000) + 3600,
};

async function signTypedData() {
  const signature = await signer.signTypedData(domain, types, value);
  return signature;
}
```

---

## 7. 智能合约交互

### 7.1 ERC-20 代币

```typescript
const ERC20_ABI = [
  'function name() view returns (string)',
  'function symbol() view returns (string)',
  'function decimals() view returns (uint8)',
  'function totalSupply() view returns (uint256)',
  'function balanceOf(address) view returns (uint256)',
  'function transfer(address to, uint256 amount) returns (bool)',
  'function allowance(address owner, address spender) view returns (uint256)',
  'function approve(address spender, uint256 amount) returns (bool)',
  'function transferFrom(address from, address to, uint256 amount) returns (bool)',
];

class ERC20Token {
  private contract: ethers.Contract;
  
  constructor(address: string, provider: ethers.Provider) {
    this.contract = new ethers.Contract(address, ERC20_ABI, provider);
  }
  
  async getInfo() {
    const [name, symbol, decimals, totalSupply] = await Promise.all([
      this.contract.name(),
      this.contract.symbol(),
      this.contract.decimals(),
      this.contract.totalSupply(),
    ]);
    return { name, symbol, decimals, totalSupply };
  }
  
  async balanceOf(address: string) {
    return this.contract.balanceOf(address);
  }
  
  async transfer(signer: ethers.Signer, to: string, amount: bigint) {
    const contractWithSigner = this.contract.connect(signer);
    return contractWithSigner.transfer(to, amount);
  }
  
  async approve(signer: ethers.Signer, spender: string, amount: bigint) {
    const contractWithSigner = this.contract.connect(signer);
    return contractWithSigner.approve(spender, amount);
  }
}
```

### 7.2 ERC-721 NFT

```typescript
const ERC721_ABI = [
  'function name() view returns (string)',
  'function symbol() view returns (string)',
  'function tokenURI(uint256 tokenId) view returns (string)',
  'function balanceOf(address owner) view returns (uint256)',
  'function ownerOf(uint256 tokenId) view returns (address)',
  'function safeTransferFrom(address from, address to, uint256 tokenId)',
  'function approve(address to, uint256 tokenId)',
  'function setApprovalForAll(address operator, bool approved)',
  'function isApprovedForAll(address owner, address operator) view returns (bool)',
];

async function getNFTMetadata(contractAddress: string, tokenId: number) {
  const contract = new ethers.Contract(contractAddress, ERC721_ABI, provider);
  const tokenURI = await contract.tokenURI(tokenId);
  
  // 获取元数据
  const response = await fetch(tokenURI);
  const metadata = await response.json();
  
  return metadata;
}
```

---

## 8. 安全最佳实践

### 8.1 常见安全问题

1. **私钥暴露**: 永远不要在前端代码中存储私钥
2. **重放攻击**: 使用 nonce 和时间戳防止签名重放
3. **钓鱼攻击**: 验证连接的网站和合约地址
4. **前端篡改**: 使用多重验证，不要仅依赖前端

### 8.2 安全实践

```typescript
// 1. 验证合约地址
const VERIFIED_CONTRACTS = {
  USDC: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
  USDT: '0xdAC17F958D2ee523a2206206994597C13D831ec7',
};

function isVerifiedContract(address: string) {
  return Object.values(VERIFIED_CONTRACTS).includes(address.toLowerCase());
}

// 2. 交易前确认
async function safeTransfer(to: string, amount: string) {
  // 检查地址有效性
  if (!ethers.isAddress(to)) {
    throw new Error('无效的地址');
  }
  
  // 检查余额
  const balance = await provider.getBalance(signer.getAddress());
  const value = ethers.parseEther(amount);
  if (balance < value) {
    throw new Error('余额不足');
  }
  
  // 显示确认对话框
  const confirmed = await showConfirmDialog({
    to,
    amount,
    estimatedGas: await provider.estimateGas({ to, value }),
  });
  
  if (!confirmed) return;
  
  return signer.sendTransaction({ to, value });
}

// 3. 错误处理
async function handleTransaction(txPromise: Promise<any>) {
  try {
    const tx = await txPromise;
    const receipt = await tx.wait();
    
    if (receipt.status === 0) {
      throw new Error('交易失败');
    }
    
    return receipt;
  } catch (error: any) {
    if (error.code === 'ACTION_REJECTED') {
      console.log('用户拒绝签名');
    } else if (error.code === 'INSUFFICIENT_FUNDS') {
      console.log('余额不足');
    } else {
      console.error('交易错误:', error);
    }
    throw error;
  }
}
```

### 8.3 Gas 优化

```typescript
// 估算 Gas
async function estimateGas(tx: ethers.TransactionRequest) {
  const gasEstimate = await provider.estimateGas(tx);
  return gasEstimate;
}

// 获取当前 Gas 价格
async function getGasPrice() {
  const feeData = await provider.getFeeData();
  return {
    gasPrice: feeData.gasPrice,
    maxFeePerGas: feeData.maxFeePerGas,
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  };
}

// 设置合理的 Gas 限制
async function sendWithOptimalGas(tx: ethers.TransactionRequest) {
  const gasEstimate = await provider.estimateGas(tx);
  const feeData = await provider.getFeeData();
  
  return signer.sendTransaction({
    ...tx,
    gasLimit: (gasEstimate * 120n) / 100n, // 增加 20% 缓冲
    maxFeePerGas: feeData.maxFeePerGas,
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  });
}
```

---

## 9. 常见面试题

### 9.1 什么是 Gas？如何计算交易费用？

Gas 是以太坊网络上执行操作所需的计算单位。交易费用 = Gas Used × Gas Price。

- **Gas Limit**: 愿意支付的最大 Gas 量
- **Gas Price**: 每单位 Gas 的价格（Gwei）
- **EIP-1559 后**: Base Fee + Priority Fee

### 9.2 解释 ERC-20 和 ERC-721 的区别？

| 特性 | ERC-20 | ERC-721 |
|------|--------|---------|
| 类型 | 同质化代币 | 非同质化代币（NFT） |
| 可分割 | 是 | 否 |
| 唯一性 | 无 | 每个代币唯一 |
| 用途 | 货币、积分 | 艺术品、游戏物品 |

### 9.3 如何处理链切换？

```typescript
// 请求切换链
async function switchChain(chainId: number) {
  try {
    await window.ethereum.request({
      method: 'wallet_switchEthereumChain',
      params: [{ chainId: `0x${chainId.toString(16)}` }],
    });
  } catch (error: any) {
    // 如果链不存在，添加链
    if (error.code === 4902) {
      await addChain(chainId);
    }
  }
}
```

### 9.4 什么是签名验证？如何实现登录？

签名验证是通过私钥对消息签名，然后用公钥/地址验证签名的过程。常用于无密码登录：

1. 服务器生成随机 nonce
2. 用户用钱包签名包含 nonce 的消息
3. 服务器验证签名恢复的地址与用户地址匹配
4. 验证通过后发放 JWT token

---

### 9.5 ethers.js v5 和 v6 有什么区别？

| 特性 | ethers.js v5 | ethers.js v6 |
|------|--------------|--------------|
| 导入方式 | `import { ethers } from 'ethers'` | 按需导入：`import { BrowserProvider } from 'ethers'` |
| Provider | `new ethers.providers.Web3Provider()` | `new BrowserProvider()` |
| BigNumber | `ethers.BigNumber.from()` | 原生 `BigInt` |
| 格式化 | `ethers.utils.formatEther()` | `ethers.formatEther()` |
| 解析 | `ethers.utils.parseEther()` | `ethers.parseEther()` |
| 包大小 | 较大 | 更小，支持 Tree Shaking |

```typescript
// v5
const provider = new ethers.providers.Web3Provider(window.ethereum);
const balance = ethers.utils.formatEther(await provider.getBalance(address));

// v6
const provider = new BrowserProvider(window.ethereum);
const balance = ethers.formatEther(await provider.getBalance(address));
```

---

### 9.6 viem 和 ethers.js 如何选择？

| 特性 | ethers.js | viem |
|------|-----------|------|
| 类型安全 | 一般 | 优秀（完整 TypeScript） |
| 包大小 | 中等 | 更小 |
| API 风格 | 面向对象 | 函数式 |
| 学习曲线 | 较低 | 较高 |
| 社区生态 | 更成熟 | 快速增长 |
| 与 wagmi 集成 | 需额外配置 | 原生集成 |

**选择建议：**
- 新项目 + React → viem + wagmi
- 需要快速开发 → ethers.js
- 严格类型要求 → viem
- 非 React 项目 → ethers.js 或 viem 均可

---

### 9.7 如何处理多链 DApp 开发？

```typescript
import { mainnet, polygon, arbitrum, optimism } from 'viem/chains';

// 1. 定义支持的链
const SUPPORTED_CHAINS = {
  [mainnet.id]: {
    name: 'Ethereum',
    rpc: 'https://eth.llamarpc.com',
    contracts: {
      usdc: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
    },
  },
  [polygon.id]: {
    name: 'Polygon',
    rpc: 'https://polygon.llamarpc.com',
    contracts: {
      usdc: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
    },
  },
};

// 2. 根据链获取合约地址
function getContractAddress(chainId: number, contract: string) {
  return SUPPORTED_CHAINS[chainId]?.contracts[contract];
}

// 3. 创建多链 Provider
function createProvider(chainId: number) {
  const chain = SUPPORTED_CHAINS[chainId];
  if (!chain) throw new Error('Unsupported chain');
  return new JsonRpcProvider(chain.rpc);
}

// 4. 监听链切换
window.ethereum.on('chainChanged', (chainId: string) => {
  const numericChainId = parseInt(chainId, 16);
  if (!SUPPORTED_CHAINS[numericChainId]) {
    alert('请切换到支持的网络');
  }
});
```

---

### 9.8 什么是 Multicall？如何优化批量请求？

Multicall 将多个合约调用打包成一次请求，减少 RPC 调用次数。

```typescript
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';

const client = createPublicClient({
  chain: mainnet,
  transport: http(),
});

// 批量读取多个代币余额
async function getMultipleBalances(userAddress: string, tokenAddresses: string[]) {
  const results = await client.multicall({
    contracts: tokenAddresses.map(address => ({
      address: address as `0x${string}`,
      abi: ERC20_ABI,
      functionName: 'balanceOf',
      args: [userAddress],
    })),
  });
  
  return results.map((result, i) => ({
    token: tokenAddresses[i],
    balance: result.status === 'success' ? result.result : 0n,
  }));
}

// ethers.js 使用 Multicall 合约
import { Contract } from 'ethers';

const MULTICALL_ADDRESS = '0xcA11bde05977b3631167028862bE2a173976CA11';
const MULTICALL_ABI = [
  'function aggregate(tuple(address target, bytes callData)[] calls) view returns (uint256 blockNumber, bytes[] returnData)',
];

async function multicall(calls: { target: string; callData: string }[]) {
  const multicall = new Contract(MULTICALL_ADDRESS, MULTICALL_ABI, provider);
  const { returnData } = await multicall.aggregate(calls);
  return returnData;
}
```

---

### 9.9 如何处理代币授权（Approve）？

```typescript
// 检查授权额度
async function checkAllowance(
  tokenAddress: string,
  ownerAddress: string,
  spenderAddress: string
) {
  const contract = new Contract(tokenAddress, ERC20_ABI, provider);
  const allowance = await contract.allowance(ownerAddress, spenderAddress);
  return allowance;
}

// 授权代币
async function approveToken(
  tokenAddress: string,
  spenderAddress: string,
  amount: bigint
) {
  const contract = new Contract(tokenAddress, ERC20_ABI, signer);
  
  // 无限授权（不推荐，但常用）
  const MAX_UINT256 = ethers.MaxUint256;
  
  // 或精确授权（更安全）
  const tx = await contract.approve(spenderAddress, amount);
  await tx.wait();
  return tx;
}

// 最佳实践：先检查再授权
async function ensureAllowance(
  tokenAddress: string,
  spenderAddress: string,
  requiredAmount: bigint
) {
  const ownerAddress = await signer.getAddress();
  const currentAllowance = await checkAllowance(tokenAddress, ownerAddress, spenderAddress);
  
  if (currentAllowance < requiredAmount) {
    // 有些代币要求先设为 0 再授权新额度
    if (currentAllowance > 0n) {
      const resetTx = await approveToken(tokenAddress, spenderAddress, 0n);
      await resetTx.wait();
    }
    await approveToken(tokenAddress, spenderAddress, requiredAmount);
  }
}
```

---

### 9.10 什么是 EIP-1559？对 Gas 费有什么影响？

EIP-1559 改变了以太坊的 Gas 机制：

| 特性 | 旧机制 | EIP-1559 |
|------|--------|----------|
| 费用模型 | Gas Price | Base Fee + Priority Fee |
| 费用去向 | 全部给矿工 | Base Fee 销毁，Priority Fee 给验证者 |
| 费用可预测性 | 低 | 高 |
| 用户体验 | 需要猜测 Gas Price | 钱包自动估算 |

```typescript
// 获取 EIP-1559 费用数据
async function getEIP1559Fee() {
  const feeData = await provider.getFeeData();
  
  return {
    // 基础费用（由协议自动计算）
    baseFeePerGas: feeData.gasPrice,
    // 最大费用（用户愿意支付的最高价格）
    maxFeePerGas: feeData.maxFeePerGas,
    // 优先费（给验证者的小费）
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  };
}

// 发送 EIP-1559 交易
async function sendEIP1559Transaction(to: string, value: bigint) {
  const feeData = await provider.getFeeData();
  
  const tx = await signer.sendTransaction({
    to,
    value,
    type: 2, // EIP-1559 交易类型
    maxFeePerGas: feeData.maxFeePerGas,
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  });
  
  return tx;
}
```

---

### 9.11 如何处理交易状态和错误？

```typescript
// 交易状态枚举
enum TxStatus {
  PENDING = 'pending',
  CONFIRMING = 'confirming',
  CONFIRMED = 'confirmed',
  FAILED = 'failed',
}

// 完整的交易处理流程
async function handleTransaction(
  txPromise: Promise<ethers.TransactionResponse>,
  onStatusChange: (status: TxStatus, data?: any) => void
) {
  try {
    onStatusChange(TxStatus.PENDING);
    
    const tx = await txPromise;
    onStatusChange(TxStatus.CONFIRMING, { hash: tx.hash });
    
    const receipt = await tx.wait();
    
    if (receipt.status === 0) {
      onStatusChange(TxStatus.FAILED, { receipt });
      throw new Error('Transaction reverted');
    }
    
    onStatusChange(TxStatus.CONFIRMED, { receipt });
    return receipt;
    
  } catch (error: any) {
    // 常见错误处理
    const errorMessages: Record<string, string> = {
      ACTION_REJECTED: '用户取消了交易',
      INSUFFICIENT_FUNDS: '余额不足',
      UNPREDICTABLE_GAS_LIMIT: '无法估算 Gas，交易可能会失败',
      NONCE_EXPIRED: 'Nonce 已过期，请刷新页面',
      REPLACEMENT_UNDERPRICED: '替换交易的 Gas 价格过低',
    };
    
    const message = errorMessages[error.code] || error.message;
    onStatusChange(TxStatus.FAILED, { error: message });
    throw error;
  }
}

// React Hook 示例
function useTransaction() {
  const [status, setStatus] = useState<TxStatus | null>(null);
  const [txHash, setTxHash] = useState<string | null>(null);
  const [error, setError] = useState<string | null>(null);
  
  const execute = async (txPromise: Promise<ethers.TransactionResponse>) => {
    setError(null);
    return handleTransaction(txPromise, (newStatus, data) => {
      setStatus(newStatus);
      if (data?.hash) setTxHash(data.hash);
      if (data?.error) setError(data.error);
    });
  };
  
  return { status, txHash, error, execute };
}
```

---

### 9.12 什么是 Account Abstraction（账户抽象）？EIP-4337 解决什么问题？

Account Abstraction 允许用户使用智能合约作为账户，而不仅仅是 EOA（外部拥有账户）。

**EIP-4337 的优势：**
- 无需私钥即可操作（社交恢复）
- 可以使用任意代币支付 Gas
- 批量交易
- 自定义签名验证
- 账户可升级

```typescript
// 使用 permissionless.js 实现账户抽象
import { createSmartAccountClient } from 'permissionless';
import { signerToSimpleSmartAccount } from 'permissionless/accounts';
import { createPimlicoClient } from 'permissionless/clients/pimlico';

// 1. 创建 Paymaster 客户端（代付 Gas）
const pimlicoClient = createPimlicoClient({
  transport: http('https://api.pimlico.io/v2/sepolia/rpc?apikey=YOUR_KEY'),
});

// 2. 创建智能账户
const simpleAccount = await signerToSimpleSmartAccount({
  signer: localSigner, // 可以是任何签名者
  factoryAddress: '0x...',
  entryPoint: '0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789',
});

// 3. 创建智能账户客户端
const smartAccountClient = createSmartAccountClient({
  account: simpleAccount,
  bundlerTransport: http('https://bundler.pimlico.io'),
  paymaster: pimlicoClient,
});

// 4. 发送 UserOperation（无需 ETH 支付 Gas）
const txHash = await smartAccountClient.sendTransaction({
  to: '0x...',
  value: parseEther('0.1'),
  data: '0x...',
});
```

---

### 9.13 如何实现 DApp 的离线签名和批量交易？

```typescript
// 离线签名（用于后续广播）
async function signOfflineTransaction(tx: ethers.TransactionRequest) {
  // 获取当前 nonce
  const nonce = await provider.getTransactionCount(await signer.getAddress());
  const feeData = await provider.getFeeData();
  
  const signedTx = await signer.signTransaction({
    ...tx,
    nonce,
    chainId: (await provider.getNetwork()).chainId,
    maxFeePerGas: feeData.maxFeePerGas,
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  });
  
  return signedTx; // 可以保存后稍后广播
}

// 广播已签名交易
async function broadcastSignedTx(signedTx: string) {
  const tx = await provider.broadcastTransaction(signedTx);
  return tx;
}

// 批量交易（通过 Multicall 合约）
const MULTICALL3_ADDRESS = '0xcA11bde05977b3631167028862bE2a173976CA11';
const MULTICALL3_ABI = [
  'function aggregate3(tuple(address target, bool allowFailure, bytes callData)[] calls) payable returns (tuple(bool success, bytes returnData)[] returnData)',
];

async function batchTransactions(calls: { target: string; data: string }[]) {
  const multicall = new Contract(MULTICALL3_ADDRESS, MULTICALL3_ABI, signer);
  
  const tx = await multicall.aggregate3(
    calls.map(call => ({
      target: call.target,
      allowFailure: false,
      callData: call.data,
    }))
  );
  
  return tx;
}
```

---

### 9.14 什么是 SIWE（Sign-In with Ethereum）？如何实现？

SIWE 是基于以太坊签名的标准化登录协议（EIP-4361）。

```typescript
import { SiweMessage } from 'siwe';

// 前端：创建并签名消息
async function createSiweMessage(address: string, statement: string) {
  const domain = window.location.host;
  const origin = window.location.origin;
  
  // 从后端获取 nonce
  const nonce = await fetch('/api/nonce').then(res => res.text());
  
  const message = new SiweMessage({
    domain,
    address,
    statement,
    uri: origin,
    version: '1',
    chainId: 1,
    nonce,
    issuedAt: new Date().toISOString(),
    expirationTime: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(),
  });
  
  return message.prepareMessage();
}

async function signInWithEthereum() {
  const address = await signer.getAddress();
  const message = await createSiweMessage(address, '登录到 My DApp');
  const signature = await signer.signMessage(message);
  
  // 发送到后端验证
  const response = await fetch('/api/verify', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message, signature }),
  });
  
  return response.json();
}

// 后端：验证签名
import { SiweMessage } from 'siwe';

async function verifySignature(message: string, signature: string) {
  const siweMessage = new SiweMessage(message);
  
  try {
    const { data: fields } = await siweMessage.verify({ signature });
    
    // 验证 nonce（从数据库获取）
    const storedNonce = await getNonceFromDB(fields.address);
    if (fields.nonce !== storedNonce) {
      throw new Error('Invalid nonce');
    }
    
    // 验证过期时间
    if (new Date(fields.expirationTime) < new Date()) {
      throw new Error('Message expired');
    }
    
    return fields.address;
  } catch (error) {
    throw new Error('Signature verification failed');
  }
}
```

---

### 9.15 如何优化 Web3 DApp 的性能和用户体验？

```typescript
// 1. 使用 WebSocket Provider 实时更新
const wsProvider = new WebSocketProvider('wss://eth-mainnet.g.alchemy.com/v2/YOUR_KEY');

// 2. 缓存合约实例
const contractCache = new Map<string, Contract>();

function getContract(address: string, abi: any[]) {
  if (!contractCache.has(address)) {
    contractCache.set(address, new Contract(address, abi, provider));
  }
  return contractCache.get(address)!;
}

// 3. 使用 React Query 缓存数据
import { useQuery } from '@tanstack/react-query';

function useTokenBalance(tokenAddress: string, userAddress: string) {
  return useQuery({
    queryKey: ['tokenBalance', tokenAddress, userAddress],
    queryFn: async () => {
      const contract = getContract(tokenAddress, ERC20_ABI);
      return contract.balanceOf(userAddress);
    },
    staleTime: 30_000, // 30 秒内不重新请求
    refetchInterval: 60_000, // 每分钟自动刷新
  });
}

// 4. 乐观更新
function useTransfer() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async ({ to, amount }: { to: string; amount: bigint }) => {
      const contract = getContract(TOKEN_ADDRESS, ERC20_ABI).connect(signer);
      const tx = await contract.transfer(to, amount);
      return tx.wait();
    },
    onMutate: async ({ amount }) => {
      // 乐观更新：立即减少余额显示
      await queryClient.cancelQueries(['tokenBalance']);
      const previousBalance = queryClient.getQueryData(['tokenBalance']);
      queryClient.setQueryData(['tokenBalance'], (old: bigint) => old - amount);
      return { previousBalance };
    },
    onError: (err, variables, context) => {
      // 出错时回滚
      queryClient.setQueryData(['tokenBalance'], context?.previousBalance);
    },
    onSettled: () => {
      // 交易完成后刷新真实数据
      queryClient.invalidateQueries(['tokenBalance']);
    },
  });
}

// 5. 预加载常用数据
async function prefetchUserData(address: string) {
  await Promise.all([
    queryClient.prefetchQuery(['ethBalance', address], () => provider.getBalance(address)),
    queryClient.prefetchQuery(['tokenBalance', USDC_ADDRESS, address], () => 
      getContract(USDC_ADDRESS, ERC20_ABI).balanceOf(address)
    ),
  ]);
}
```

---

### 9.16 Web3 前端常见安全问题及防范

| 攻击类型 | 描述 | 防范措施 |
|----------|------|----------|
| 签名钓鱼 | 诱骗用户签名恶意交易 | 显示明确的签名内容、验证合约地址 |
| 授权钓鱼 | 诱骗用户授权无限额度 | 只授权必要额度、定期检查授权 |
| 前端劫持 | 篡改前端代码 | CSP、SRI、定期审计 |
| RPC 劫持 | 恶意 RPC 返回虚假数据 | 多 RPC 验证、使用知名提供商 |
| 重放攻击 | 重复使用签名 | 使用 nonce、设置过期时间 |

```typescript
// 安全检查清单
const securityChecks = {
  // 1. 验证合约地址
  verifyContractAddress: (address: string) => {
    const knownContracts = ['0x...', '0x...'];
    return knownContracts.includes(address.toLowerCase());
  },
  
  // 2. 限制授权额度
  getSafeApprovalAmount: (requiredAmount: bigint) => {
    // 授权额度 = 需要量的 1.1 倍，而非无限
    return (requiredAmount * 110n) / 100n;
  },
  
  // 3. 检查交易模拟
  simulateTransaction: async (tx: any) => {
    try {
      await provider.estimateGas(tx);
      return { success: true };
    } catch (error: any) {
      return { success: false, error: error.message };
    }
  },
  
  // 4. 检查授权状态
  checkApprovals: async (owner: string) => {
    // 使用 Revoke.cash API 或自建索引
    const approvals = await fetch(`https://api.revoke.cash/v1/${owner}/approvals`);
    return approvals.json();
  },
};
```

---

### 9.17 解释 Provider、Signer、Contract 的关系？

```
┌─────────────────────────────────────────────────────────┐
│                      区块链网络                          │
└─────────────────────────────────────────────────────────┘
                           ▲
                           │ RPC 请求
                           │
┌─────────────────────────────────────────────────────────┐
│                      Provider                            │
│  • 连接区块链节点                                         │
│  • 只读操作（查询余额、交易、区块等）                       │
│  • 不能签名或发送交易                                      │
└─────────────────────────────────────────────────────────┘
                           ▲
                           │ 继承/扩展
                           │
┌─────────────────────────────────────────────────────────┐
│                       Signer                             │
│  • 拥有 Provider 的所有功能                               │
│  • 可以签名消息和交易                                      │
│  • 代表一个以太坊账户                                      │
└─────────────────────────────────────────────────────────┘
                           │
                           │ 连接
                           ▼
┌─────────────────────────────────────────────────────────┐
│                      Contract                            │
│  • 智能合约的 JavaScript 抽象                             │
│  • 只连接 Provider → 只能读取                             │
│  • 连接 Signer → 可以读取和写入                           │
└─────────────────────────────────────────────────────────┘
```

```typescript
// 只读 Contract（连接 Provider）
const readOnlyContract = new Contract(address, abi, provider);
await readOnlyContract.balanceOf(userAddress); // ✅ 可以
await readOnlyContract.transfer(to, amount);   // ❌ 错误

// 可写 Contract（连接 Signer）
const writableContract = new Contract(address, abi, signer);
// 或
const writableContract = readOnlyContract.connect(signer);
await writableContract.transfer(to, amount);   // ✅ 可以
```

---

### 9.18 如何处理大数（BigInt）和精度问题？

```typescript
// 以太坊使用 18 位小数精度
const ONE_ETH = 1000000000000000000n; // 10^18 wei
const ONE_USDC = 1000000n;             // 10^6 (USDC 是 6 位精度)

// 1. 格式化显示
function formatTokenAmount(amount: bigint, decimals: number): string {
  const divisor = 10n ** BigInt(decimals);
  const integerPart = amount / divisor;
  const fractionalPart = amount % divisor;
  
  const fractionalStr = fractionalPart.toString().padStart(decimals, '0');
  const trimmedFractional = fractionalStr.replace(/0+$/, '');
  
  return trimmedFractional 
    ? `${integerPart}.${trimmedFractional}`
    : integerPart.toString();
}

// 2. 解析用户输入
function parseTokenAmount(input: string, decimals: number): bigint {
  const [integer, fraction = ''] = input.split('.');
  const paddedFraction = fraction.padEnd(decimals, '0').slice(0, decimals);
  return BigInt(integer + paddedFraction);
}

// 3. 使用 ethers.js 工具函数
import { formatUnits, parseUnits } from 'ethers';

// 格式化
formatUnits(1000000000000000000n, 18); // "1.0"
formatUnits(1500000n, 6);               // "1.5"

// 解析
parseUnits('1.5', 18); // 1500000000000000000n
parseUnits('1.5', 6);  // 1500000n

// 4. 安全的数学运算
function safeMultiply(a: bigint, b: bigint, decimals: number): bigint {
  return (a * b) / (10n ** BigInt(decimals));
}

function safeDivide(a: bigint, b: bigint, decimals: number): bigint {
  return (a * (10n ** BigInt(decimals))) / b;
}

// 5. 处理价格计算（避免精度丢失）
function calculatePrice(
  amountIn: bigint,
  reserveIn: bigint,
  reserveOut: bigint
): bigint {
  // Uniswap V2 公式：amountOut = (amountIn * reserveOut) / (reserveIn + amountIn)
  const numerator = amountIn * reserveOut;
  const denominator = reserveIn + amountIn;
  return numerator / denominator;
}
```

---

