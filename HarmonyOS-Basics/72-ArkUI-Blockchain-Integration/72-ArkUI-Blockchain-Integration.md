# ArkUI Blockchain Integration

## Introduction

Blockchain integration enables decentralized applications (DApps) and Web3 functionality in ArkUI applications. This guide covers wallet integration, smart contract interaction, and blockchain data visualization.

## Wallet Integration

### Wallet Connection Manager

```typescript
interface WalletProvider {
  name: string;
  isInstalled: boolean;
  connect(): Promise<string[]>;
  disconnect(): Promise<void>;
  signMessage(message: string): Promise<string>;
  sendTransaction(tx: Transaction): Promise<string>;
}

interface Transaction {
  to: string;
  value: string;
  data?: string;
  gasLimit?: string;
  gasPrice?: string;
}

class WalletManager {
  private providers = new Map<string, WalletProvider>();
  private currentProvider: WalletProvider | null = null;
  private accounts: string[] = [];

  registerProvider(name: string, provider: WalletProvider): void {
    this.providers.set(name, provider);
  }

  async connectWallet(providerName: string): Promise<boolean> {
    const provider = this.providers.get(providerName);
    if (!provider || !provider.isInstalled) {
      return false;
    }

    try {
      this.accounts = await provider.connect();
      this.currentProvider = provider;
      return true;
    } catch (error) {
      console.error("Wallet connection failed:", error);
      return false;
    }
  }

  async disconnectWallet(): Promise<void> {
    if (this.currentProvider) {
      await this.currentProvider.disconnect();
      this.currentProvider = null;
      this.accounts = [];
    }
  }

  getConnectedAccount(): string | null {
    return this.accounts.length > 0 ? this.accounts[0] : null;
  }

  async signMessage(message: string): Promise<string | null> {
    if (!this.currentProvider) return null;

    try {
      return await this.currentProvider.signMessage(message);
    } catch (error) {
      console.error("Message signing failed:", error);
      return null;
    }
  }

  async sendTransaction(transaction: Transaction): Promise<string | null> {
    if (!this.currentProvider) return null;

    try {
      return await this.currentProvider.sendTransaction(transaction);
    } catch (error) {
      console.error("Transaction failed:", error);
      return null;
    }
  }
}
```

### Smart Contract Interface

```typescript
interface ContractABI {
  name: string;
  type: "function" | "event" | "constructor";
  inputs: ContractInput[];
  outputs?: ContractOutput[];
  stateMutability?: "pure" | "view" | "nonpayable" | "payable";
}

interface ContractInput {
  name: string;
  type: string;
  indexed?: boolean;
}

interface ContractOutput {
  name: string;
  type: string;
}

class SmartContract {
  private address: string;
  private abi: ContractABI[];
  private walletManager: WalletManager;

  constructor(
    address: string,
    abi: ContractABI[],
    walletManager: WalletManager
  ) {
    this.address = address;
    this.abi = abi;
    this.walletManager = walletManager;
  }

  async call(functionName: string, params: any[] = []): Promise<any> {
    const functionABI = this.abi.find(
      (item) => item.name === functionName && item.type === "function"
    );

    if (!functionABI) {
      throw new Error(`Function ${functionName} not found in ABI`);
    }

    if (
      functionABI.stateMutability === "view" ||
      functionABI.stateMutability === "pure"
    ) {
      return this.callReadFunction(functionName, params);
    } else {
      return this.callWriteFunction(functionName, params);
    }
  }

  private async callReadFunction(
    functionName: string,
    params: any[]
  ): Promise<any> {
    const data = this.encodeFunction(functionName, params);

    // Simulate calling a read function
    // In real implementation, this would make an RPC call
    return this.simulateReadCall(data);
  }

  private async callWriteFunction(
    functionName: string,
    params: any[]
  ): Promise<string> {
    const data = this.encodeFunction(functionName, params);

    const transaction: Transaction = {
      to: this.address,
      data,
      gasLimit: "100000",
      gasPrice: "20000000000", // 20 gwei
    };

    const txHash = await this.walletManager.sendTransaction(transaction);
    if (!txHash) {
      throw new Error("Transaction failed");
    }

    return txHash;
  }

  private encodeFunction(functionName: string, params: any[]): string {
    // Simplified function encoding
    // In real implementation, would use proper ABI encoding
    const signature = this.getFunctionSignature(functionName);
    const encodedParams = this.encodeParameters(params);
    return signature + encodedParams;
  }

  private getFunctionSignature(functionName: string): string {
    // Return function selector (first 4 bytes of keccak256 hash)
    return "0x" + functionName.substring(0, 8).padEnd(8, "0");
  }

  private encodeParameters(params: any[]): string {
    // Simplified parameter encoding
    return params
      .map((param) => {
        if (typeof param === "string") {
          return param.replace("0x", "").padStart(64, "0");
        } else if (typeof param === "number") {
          return param.toString(16).padStart(64, "0");
        }
        return "0".repeat(64);
      })
      .join("");
  }

  private async simulateReadCall(data: string): Promise<any> {
    // Simulate blockchain read call
    return {
      success: true,
      result: Math.floor(Math.random() * 1000000),
    };
  }
}
```

## Blockchain Data Visualization

### Transaction History Component

```typescript
interface BlockchainTransaction {
  hash: string
  from: string
  to: string
  value: string
  timestamp: number
  status: 'pending' | 'confirmed' | 'failed'
  gasUsed?: string
  gasPrice?: string
}

@Component
struct TransactionHistory {
  @State private transactions: BlockchainTransaction[] = []
  @State private loading: boolean = false
  @State private selectedTx: BlockchainTransaction | null = null
  private walletManager = new WalletManager()

  build() {
    Column() {
      this.buildHeader()
      this.buildTransactionList()
      if (this.selectedTx) {
        this.buildTransactionDetails()
      }
    }
    .padding(16)
  }

  aboutToAppear() {
    this.loadTransactions()
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Transaction History')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      Button('Refresh')
        .onClick(() => this.loadTransactions())
        .enabled(!this.loading)
    }
    .width('100%')
    .margin({ bottom: 16 })
  }

  @Builder
  private buildTransactionList() {
    if (this.loading) {
      Row() {
        LoadingProgress()
          .width(24)
          .height(24)

        Text('Loading transactions...')
          .margin({ left: 8 })
      }
      .justifyContent(FlexAlign.Center)
      .width('100%')
      .padding(20)
    } else {
      List() {
        ForEach(this.transactions, (tx: BlockchainTransaction) => {
          ListItem() {
            this.buildTransactionItem(tx)
          }
          .onClick(() => this.selectedTx = tx)
        })
      }
      .height(400)
      .divider({ strokeWidth: 1, color: '#E0E0E0' })
    }
  }

  @Builder
  private buildTransactionItem(tx: BlockchainTransaction) {
    Row() {
      Column() {
        Row() {
          Text(this.formatHash(tx.hash))
            .fontSize(14)
            .fontWeight(FontWeight.Medium)
            .flexGrow(1)

          this.buildStatusBadge(tx.status)
        }
        .width('100%')
        .margin({ bottom: 4 })

        Row() {
          Text(`From: ${this.formatAddress(tx.from)}`)
            .fontSize(12)
            .fontColor('#666')
            .flexGrow(1)

          Text(this.formatValue(tx.value))
            .fontSize(14)
            .fontWeight(FontWeight.Bold)
        }
        .width('100%')
        .margin({ bottom: 4 })

        Text(`To: ${this.formatAddress(tx.to)}`)
          .fontSize(12)
          .fontColor('#666')
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)
    }
    .padding(12)
    .width('100%')
  }

  @Builder
  private buildStatusBadge(status: string) {
    Text(status.toUpperCase())
      .fontSize(10)
      .fontColor(this.getStatusColor(status))
      .backgroundColor(this.getStatusBackgroundColor(status))
      .padding({ horizontal: 6, vertical: 2 })
      .borderRadius(10)
  }

  @Builder
  private buildTransactionDetails() {
    Column() {
      Row() {
        Text('Transaction Details')
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Button('×')
          .fontSize(20)
          .backgroundColor(Color.Transparent)
          .fontColor('#666')
          .onClick(() => this.selectedTx = null)
      }
      .margin({ bottom: 16 })

      if (this.selectedTx) {
        this.buildDetailRow('Hash', this.selectedTx.hash)
        this.buildDetailRow('From', this.selectedTx.from)
        this.buildDetailRow('To', this.selectedTx.to)
        this.buildDetailRow('Value', this.formatValue(this.selectedTx.value))
        this.buildDetailRow('Status', this.selectedTx.status)
        this.buildDetailRow('Timestamp', this.formatTimestamp(this.selectedTx.timestamp))

        if (this.selectedTx.gasUsed) {
          this.buildDetailRow('Gas Used', this.selectedTx.gasUsed)
        }

        if (this.selectedTx.gasPrice) {
          this.buildDetailRow('Gas Price', this.selectedTx.gasPrice)
        }
      }
    }
    .backgroundColor('#f8f9fa')
    .borderRadius(8)
    .padding(16)
    .margin({ top: 16 })
  }

  @Builder
  private buildDetailRow(label: string, value: string) {
    Row() {
      Text(label)
        .fontSize(14)
        .fontWeight(FontWeight.Medium)
        .width(100)

      Text(value)
        .fontSize(14)
        .flexGrow(1)
        .textOverflow({ overflow: TextOverflow.Ellipsis })
        .maxLines(1)
    }
    .margin({ bottom: 8 })
    .width('100%')
  }

  private async loadTransactions(): Promise<void> {
    this.loading = true

    try {
      // Simulate loading transaction data
      await new Promise(resolve => setTimeout(resolve, 1000))

      this.transactions = this.generateMockTransactions()
    } catch (error) {
      console.error('Failed to load transactions:', error)
    } finally {
      this.loading = false
    }
  }

  private generateMockTransactions(): BlockchainTransaction[] {
    const statuses: ('pending' | 'confirmed' | 'failed')[] = ['pending', 'confirmed', 'failed']

    return Array.from({ length: 10 }, (_, index) => ({
      hash: `0x${Math.random().toString(16).substring(2, 66)}`,
      from: `0x${Math.random().toString(16).substring(2, 42)}`,
      to: `0x${Math.random().toString(16).substring(2, 42)}`,
      value: (Math.random() * 10).toFixed(4),
      timestamp: Date.now() - index * 3600000,
      status: statuses[Math.floor(Math.random() * statuses.length)],
      gasUsed: Math.floor(Math.random() * 100000).toString(),
      gasPrice: '20000000000'
    }))
  }

  private formatHash(hash: string): string {
    return `${hash.substring(0, 10)}...${hash.substring(hash.length - 8)}`
  }

  private formatAddress(address: string): string {
    return `${address.substring(0, 6)}...${address.substring(address.length - 4)}`
  }

  private formatValue(value: string): string {
    return `${parseFloat(value).toFixed(4)} ETH`
  }

  private formatTimestamp(timestamp: number): string {
    return new Date(timestamp).toLocaleString()
  }

  private getStatusColor(status: string): string {
    switch (status) {
      case 'confirmed': return '#FFFFFF'
      case 'pending': return '#FFFFFF'
      case 'failed': return '#FFFFFF'
      default: return '#000000'
    }
  }

  private getStatusBackgroundColor(status: string): string {
    switch (status) {
      case 'confirmed': return '#34C759'
      case 'pending': return '#FF9500'
      case 'failed': return '#FF3B30'
      default: return '#E0E0E0'
    }
  }
}
```

## DeFi Integration

### Token Balance Display

```typescript
interface TokenInfo {
  address: string
  symbol: string
  name: string
  decimals: number
  balance: string
  price?: number
}

@Component
struct TokenBalanceView {
  @State private tokens: TokenInfo[] = []
  @State private totalValue: number = 0
  @State private loading: boolean = false
  private walletManager = new WalletManager()

  build() {
    Column() {
      this.buildSummary()
      this.buildTokenList()
    }
    .padding(16)
  }

  aboutToAppear() {
    this.loadTokenBalances()
  }

  @Builder
  private buildSummary() {
    Column() {
      Text('Total Portfolio Value')
        .fontSize(16)
        .fontColor('#666')

      Text(`$${this.totalValue.toFixed(2)}`)
        .fontSize(32)
        .fontWeight(FontWeight.Bold)
        .margin({ top: 8 })
    }
    .backgroundColor('#f8f9fa')
    .borderRadius(12)
    .padding(20)
    .width('100%')
    .margin({ bottom: 20 })
  }

  @Builder
  private buildTokenList() {
    Column() {
      Text('Token Balances')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 16 })

      if (this.loading) {
        LoadingProgress()
          .width(40)
          .height(40)
      } else {
        ForEach(this.tokens, (token: TokenInfo) => {
          this.buildTokenItem(token)
        })
      }
    }
  }

  @Builder
  private buildTokenItem(token: TokenInfo) {
    Row() {
      Column() {
        Text(token.symbol)
          .fontSize(16)
          .fontWeight(FontWeight.Bold)

        Text(token.name)
          .fontSize(12)
          .fontColor('#666')
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)

      Column() {
        Text(this.formatBalance(token.balance, token.decimals))
          .fontSize(16)
          .fontWeight(FontWeight.Bold)

        if (token.price) {
          Text(`$${(parseFloat(token.balance) * token.price).toFixed(2)}`)
            .fontSize(12)
            .fontColor('#666')
        }
      }
      .alignItems(HorizontalAlign.End)
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#ffffff')
    .borderRadius(8)
    .margin({ bottom: 8 })
    .shadow({ radius: 2, color: '#00000010', offsetY: 1 })
  }

  private async loadTokenBalances(): Promise<void> {
    this.loading = true

    try {
      // Simulate loading token data
      await new Promise(resolve => setTimeout(resolve, 1000))

      this.tokens = [
        {
          address: '0x...',
          symbol: 'ETH',
          name: 'Ethereum',
          decimals: 18,
          balance: '2.5',
          price: 2000
        },
        {
          address: '0x...',
          symbol: 'USDC',
          name: 'USD Coin',
          decimals: 6,
          balance: '1000.0',
          price: 1
        }
      ]

      this.totalValue = this.tokens.reduce((total, token) => {
        return total + (parseFloat(token.balance) * (token.price || 0))
      }, 0)
    } catch (error) {
      console.error('Failed to load token balances:', error)
    } finally {
      this.loading = false
    }
  }

  private formatBalance(balance: string, decimals: number): string {
    const value = parseFloat(balance)
    if (value === 0) return '0'
    if (value < 0.001) return '< 0.001'
    return value.toFixed(Math.min(decimals, 6))
  }
}
```

## NFT Gallery

### NFT Display Component

```typescript
interface NFTMetadata {
  tokenId: string
  name: string
  description: string
  image: string
  attributes: { trait_type: string; value: string }[]
  contractAddress: string
}

@Component
struct NFTGallery {
  @State private nfts: NFTMetadata[] = []
  @State private selectedNFT: NFTMetadata | null = null
  @State private loading: boolean = false

  build() {
    Column() {
      this.buildHeader()
      this.buildNFTGrid()

      if (this.selectedNFT) {
        this.buildNFTDetails()
      }
    }
    .padding(16)
  }

  aboutToAppear() {
    this.loadNFTs()
  }

  @Builder
  private buildHeader() {
    Text('My NFT Collection')
      .fontSize(24)
      .fontWeight(FontWeight.Bold)
      .margin({ bottom: 20 })
  }

  @Builder
  private buildNFTGrid() {
    if (this.loading) {
      LoadingProgress()
        .width(40)
        .height(40)
    } else {
      Grid() {
        ForEach(this.nfts, (nft: NFTMetadata) => {
          GridItem() {
            this.buildNFTCard(nft)
          }
        })
      }
      .columnsTemplate('1fr 1fr')
      .columnsGap(16)
      .rowsGap(16)
    }
  }

  @Builder
  private buildNFTCard(nft: NFTMetadata) {
    Column() {
      Image(nft.image)
        .width('100%')
        .height(150)
        .objectFit(ImageFit.Cover)
        .borderRadius({ topLeft: 8, topRight: 8 })

      Column() {
        Text(nft.name)
          .fontSize(14)
          .fontWeight(FontWeight.Bold)
          .maxLines(1)
          .textOverflow({ overflow: TextOverflow.Ellipsis })

        Text(`#${nft.tokenId}`)
          .fontSize(12)
          .fontColor('#666')
          .margin({ top: 4 })
      }
      .padding(12)
      .alignItems(HorizontalAlign.Start)
    }
    .backgroundColor('#ffffff')
    .borderRadius(8)
    .shadow({ radius: 4, color: '#00000020', offsetY: 2 })
    .onClick(() => this.selectedNFT = nft)
  }

  @Builder
  private buildNFTDetails() {
    if (!this.selectedNFT) return

    Column() {
      Row() {
        Text('NFT Details')
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Button('×')
          .backgroundColor(Color.Transparent)
          .fontColor('#666')
          .onClick(() => this.selectedNFT = null)
      }
      .margin({ bottom: 16 })

      Image(this.selectedNFT.image)
        .width(200)
        .height(200)
        .objectFit(ImageFit.Cover)
        .borderRadius(12)
        .margin({ bottom: 16 })

      Text(this.selectedNFT.name)
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Text(this.selectedNFT.description)
        .fontSize(14)
        .fontColor('#666')
        .margin({ bottom: 16 })

      Text('Attributes')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      ForEach(this.selectedNFT.attributes, (attr) => {
        Row() {
          Text(attr.trait_type)
            .fontSize(14)
            .flexGrow(1)

          Text(attr.value)
            .fontSize(14)
            .fontWeight(FontWeight.Medium)
        }
        .padding(8)
        .backgroundColor('#f0f0f0')
        .borderRadius(4)
        .margin({ bottom: 4 })
      })
    }
    .backgroundColor('#ffffff')
    .borderRadius(12)
    .padding(20)
    .margin({ top: 20 })
    .shadow({ radius: 8, color: '#00000020', offsetY: 4 })
  }

  private async loadNFTs(): Promise<void> {
    this.loading = true

    try {
      // Simulate loading NFT data
      await new Promise(resolve => setTimeout(resolve, 1000))

      this.nfts = this.generateMockNFTs()
    } catch (error) {
      console.error('Failed to load NFTs:', error)
    } finally {
      this.loading = false
    }
  }

  private generateMockNFTs(): NFTMetadata[] {
    return Array.from({ length: 6 }, (_, index) => ({
      tokenId: (index + 1).toString(),
      name: `Cool NFT #${index + 1}`,
      description: `This is a unique digital collectible with special properties.`,
      image: `https://picsum.photos/200/200?random=${index}`,
      attributes: [
        { trait_type: 'Background', value: 'Blue' },
        { trait_type: 'Eyes', value: 'Laser' },
        { trait_type: 'Rarity', value: 'Rare' }
      ],
      contractAddress: '0x...'
    }))
  }
}
```

## Conclusion

Blockchain integration in ArkUI applications enables:

- Seamless wallet connectivity and transaction management
- Smart contract interaction and DeFi functionality
- Real-time blockchain data visualization
- NFT gallery and marketplace features
- Decentralized identity and authentication
- Cross-chain asset management

These features enable building comprehensive Web3 applications with native blockchain functionality integrated into the HarmonyOS ecosystem.
