# ArkUI Blockchain Integration

## Introduction

Blockchain integration in ArkUI enables decentralized applications (DApps), cryptocurrency transactions, smart contract interactions, and distributed data storage. This guide covers Web3 integration, wallet connectivity, and blockchain transaction management.

## Web3 Integration Framework

```typescript
interface BlockchainNetwork {
  id: string;
  name: string;
  chainId: number;
  rpcUrl: string;
  currency: string;
  explorer: string;
}

interface WalletConnection {
  address: string;
  network: BlockchainNetwork;
  balance: string;
  isConnected: boolean;
}

interface Transaction {
  hash: string;
  from: string;
  to: string;
  value: string;
  gasPrice: string;
  gasLimit: string;
  status: "pending" | "confirmed" | "failed";
  timestamp: number;
}

interface SmartContract {
  address: string;
  abi: any[];
  network: BlockchainNetwork;
}

class BlockchainManager {
  private provider: any = null;
  private signer: any = null;
  private connection: WalletConnection | null = null;
  private contracts = new Map<string, SmartContract>();
  private transactions: Transaction[] = [];

  async connectWallet(networkId: string): Promise<WalletConnection | null> {
    try {
      if (!this.isWeb3Available()) {
        throw new Error("Web3 provider not available");
      }

      const provider = await this.getWeb3Provider();
      const accounts = await provider.request({
        method: "eth_requestAccounts",
      });

      if (accounts.length === 0) {
        throw new Error("No accounts available");
      }

      const network = this.getNetworkById(networkId);
      const balance = await provider.request({
        method: "eth_getBalance",
        params: [accounts[0], "latest"],
      });

      this.connection = {
        address: accounts[0],
        network,
        balance: this.formatBalance(balance),
        isConnected: true,
      };

      this.provider = provider;
      this.setupEventListeners();

      return this.connection;
    } catch (error) {
      console.error("Wallet connection failed:", error);
      return null;
    }
  }

  async disconnectWallet(): Promise<void> {
    this.connection = null;
    this.provider = null;
    this.signer = null;
  }

  async sendTransaction(to: string, value: string): Promise<string | null> {
    if (!this.connection || !this.provider) {
      throw new Error("Wallet not connected");
    }

    try {
      const txParams = {
        from: this.connection.address,
        to,
        value: this.parseValue(value),
        gas: "0x5208", // Standard gas limit for ETH transfer
      };

      const txHash = await this.provider.request({
        method: "eth_sendTransaction",
        params: [txParams],
      });

      const transaction: Transaction = {
        hash: txHash,
        from: this.connection.address,
        to,
        value,
        gasPrice: "0",
        gasLimit: "21000",
        status: "pending",
        timestamp: Date.now(),
      };

      this.transactions.push(transaction);
      this.monitorTransaction(txHash);

      return txHash;
    } catch (error) {
      console.error("Transaction failed:", error);
      return null;
    }
  }

  async callContractMethod(
    contractId: string,
    method: string,
    params: any[]
  ): Promise<any> {
    const contract = this.contracts.get(contractId);
    if (!contract || !this.provider) {
      throw new Error("Contract or provider not available");
    }

    try {
      const data = this.encodeMethodCall(method, params, contract.abi);

      const result = await this.provider.request({
        method: "eth_call",
        params: [
          {
            to: contract.address,
            data,
          },
          "latest",
        ],
      });

      return this.decodeResult(result, method, contract.abi);
    } catch (error) {
      console.error("Contract call failed:", error);
      throw error;
    }
  }

  async executeContractTransaction(
    contractId: string,
    method: string,
    params: any[],
    value?: string
  ): Promise<string | null> {
    const contract = this.contracts.get(contractId);
    if (!contract || !this.connection || !this.provider) {
      throw new Error("Contract, connection, or provider not available");
    }

    try {
      const data = this.encodeMethodCall(method, params, contract.abi);

      const txParams = {
        from: this.connection.address,
        to: contract.address,
        data,
        value: value ? this.parseValue(value) : "0x0",
      };

      const txHash = await this.provider.request({
        method: "eth_sendTransaction",
        params: [txParams],
      });

      const transaction: Transaction = {
        hash: txHash,
        from: this.connection.address,
        to: contract.address,
        value: value || "0",
        gasPrice: "0",
        gasLimit: "0",
        status: "pending",
        timestamp: Date.now(),
      };

      this.transactions.push(transaction);
      this.monitorTransaction(txHash);

      return txHash;
    } catch (error) {
      console.error("Contract transaction failed:", error);
      return null;
    }
  }

  addContract(id: string, contract: SmartContract): void {
    this.contracts.set(id, contract);
  }

  getConnection(): WalletConnection | null {
    return this.connection;
  }

  getTransactions(): Transaction[] {
    return [...this.transactions].sort((a, b) => b.timestamp - a.timestamp);
  }

  private isWeb3Available(): boolean {
    return typeof (globalThis as any).ethereum !== "undefined";
  }

  private async getWeb3Provider(): Promise<any> {
    return (globalThis as any).ethereum;
  }

  private getNetworkById(id: string): BlockchainNetwork {
    const networks: Record<string, BlockchainNetwork> = {
      ethereum: {
        id: "ethereum",
        name: "Ethereum Mainnet",
        chainId: 1,
        rpcUrl: "https://mainnet.infura.io",
        currency: "ETH",
        explorer: "https://etherscan.io",
      },
      polygon: {
        id: "polygon",
        name: "Polygon",
        chainId: 137,
        rpcUrl: "https://polygon-rpc.com",
        currency: "MATIC",
        explorer: "https://polygonscan.com",
      },
    };

    return networks[id] || networks.ethereum;
  }

  private formatBalance(balance: string): string {
    // Convert from wei to ETH
    const ethValue = parseInt(balance, 16) / Math.pow(10, 18);
    return ethValue.toFixed(4);
  }

  private parseValue(value: string): string {
    // Convert ETH to wei
    const weiValue = parseFloat(value) * Math.pow(10, 18);
    return "0x" + Math.floor(weiValue).toString(16);
  }

  private setupEventListeners(): void {
    if (!this.provider) return;

    this.provider.on("accountsChanged", (accounts: string[]) => {
      if (accounts.length === 0) {
        this.disconnectWallet();
      } else if (this.connection) {
        this.connection.address = accounts[0];
      }
    });

    this.provider.on("chainChanged", (chainId: string) => {
      console.log("Chain changed to:", chainId);
      // Handle network change
    });
  }

  private async monitorTransaction(txHash: string): Promise<void> {
    const checkStatus = async () => {
      try {
        const receipt = await this.provider.request({
          method: "eth_getTransactionReceipt",
          params: [txHash],
        });

        const transaction = this.transactions.find((tx) => tx.hash === txHash);
        if (transaction) {
          if (receipt) {
            transaction.status =
              receipt.status === "0x1" ? "confirmed" : "failed";
          } else {
            // Transaction still pending, check again later
            setTimeout(checkStatus, 5000);
          }
        }
      } catch (error) {
        console.error("Error checking transaction status:", error);
      }
    };

    setTimeout(checkStatus, 2000);
  }

  private encodeMethodCall(method: string, params: any[], abi: any[]): string {
    // Simplified method encoding - in real implementation, use a proper ABI encoder
    const methodSignature = this.getMethodSignature(method, abi);
    const encodedParams = this.encodeParameters(params);
    return methodSignature + encodedParams;
  }

  private getMethodSignature(method: string, abi: any[]): string {
    // Simplified signature generation
    return "0x" + method.slice(0, 8).padEnd(8, "0");
  }

  private encodeParameters(params: any[]): string {
    // Simplified parameter encoding
    return params
      .map((p) => {
        if (typeof p === "string") {
          return p.padStart(64, "0");
        }
        return p.toString(16).padStart(64, "0");
      })
      .join("");
  }

  private decodeResult(result: string, method: string, abi: any[]): any {
    // Simplified result decoding
    return result;
  }
}
```

## Wallet Integration Component

```typescript
@Component
struct WalletConnect {
  @State private connection: WalletConnection | null = null
  @State private isConnecting: boolean = false
  @State private selectedNetwork: string = 'ethereum'
  @State private transactions: Transaction[] = []

  private blockchainManager = new BlockchainManager()

  build() {
    Column() {
      this.buildHeader()

      if (this.connection) {
        this.buildWalletInfo()
        this.buildTransactions()
      } else {
        this.buildConnectWallet()
      }
    }
    .width('100%')
    .padding(16)
  }

  @Builder
  private buildHeader() {
    Row() {
      Text('Blockchain Wallet')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .flexGrow(1)

      if (this.connection) {
        Button('Disconnect')
          .onClick(() => this.disconnectWallet())
          .backgroundColor('#FF3B30')
          .fontColor('#FFFFFF')
          .fontSize(14)
      }
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildConnectWallet() {
    Column() {
      Text('🔗')
        .fontSize(48)
        .margin({ bottom: 16 })

      Text('Connect Your Wallet')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Text('Connect your Web3 wallet to interact with blockchain')
        .fontSize(14)
        .fontColor('#666666')
        .textAlign(TextAlign.Center)
        .margin({ bottom: 24 })

      this.buildNetworkSelector()

      Button(this.isConnecting ? 'Connecting...' : 'Connect Wallet')
        .onClick(() => this.connectWallet())
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .width('100%')
        .height(48)
        .enabled(!this.isConnecting)
        .margin({ top: 16 })

      if (this.isConnecting) {
        LoadingProgress()
          .width(32)
          .height(32)
          .margin({ top: 16 })
      }
    }
    .width('100%')
    .justifyContent(FlexAlign.Center)
    .height(400)
  }

  @Builder
  private buildNetworkSelector() {
    Column() {
      Text('Select Network')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      Row() {
        Button('Ethereum')
          .onClick(() => this.selectedNetwork = 'ethereum')
          .backgroundColor(this.selectedNetwork === 'ethereum' ? '#007AFF' : '#F0F0F0')
          .fontColor(this.selectedNetwork === 'ethereum' ? '#FFFFFF' : '#333333')
          .flexGrow(1)
          .margin({ right: 8 })

        Button('Polygon')
          .onClick(() => this.selectedNetwork = 'polygon')
          .backgroundColor(this.selectedNetwork === 'polygon' ? '#007AFF' : '#F0F0F0')
          .fontColor(this.selectedNetwork === 'polygon' ? '#FFFFFF' : '#333333')
          .flexGrow(1)
      }
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildWalletInfo() {
    Column() {
      Row() {
        Column() {
          Text(this.connection!.network.name)
            .fontSize(16)
            .fontWeight(FontWeight.Bold)

          Text(this.formatAddress(this.connection!.address))
            .fontSize(14)
            .fontColor('#666666')
        }
        .alignItems(HorizontalAlign.Start)
        .flexGrow(1)

        Column() {
          Text(this.connection!.balance)
            .fontSize(20)
            .fontWeight(FontWeight.Bold)
            .fontColor('#007AFF')

          Text(this.connection!.network.currency)
            .fontSize(12)
            .fontColor('#666666')
        }
        .alignItems(HorizontalAlign.End)
      }
      .width('100%')
      .padding(16)
      .backgroundColor('#FFFFFF')
      .borderRadius(8)
      .margin({ bottom: 16 })

      this.buildActions()
    }
  }

  @Builder
  private buildActions() {
    Row() {
      Button('Send')
        .onClick(() => this.openSendDialog())
        .backgroundColor('#34C759')
        .fontColor('#FFFFFF')
        .flexGrow(1)
        .margin({ right: 8 })

      Button('Receive')
        .onClick(() => this.openReceiveDialog())
        .backgroundColor('#007AFF')
        .fontColor('#FFFFFF')
        .flexGrow(1)
        .margin({ right: 8 })

      Button('Contract')
        .onClick(() => this.openContractDialog())
        .backgroundColor('#FF9500')
        .fontColor('#FFFFFF')
        .flexGrow(1)
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildTransactions() {
    if (this.transactions.length === 0) return

    Column() {
      Text('Recent Transactions')
        .fontSize(18)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      ForEach(this.transactions.slice(0, 5), (tx: Transaction) => {
        this.buildTransactionItem(tx)
      })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildTransactionItem(transaction: Transaction) {
    Row() {
      Column() {
        Row() {
          Text(this.getTransactionTypeIcon(transaction))
            .fontSize(16)
            .margin({ right: 8 })

          Text(`${transaction.value} ${this.connection!.network.currency}`)
            .fontSize(14)
            .fontWeight(FontWeight.Medium)
        }

        Text(this.formatAddress(transaction.to))
          .fontSize(12)
          .fontColor('#666666')
          .margin({ top: 2 })
      }
      .alignItems(HorizontalAlign.Start)
      .flexGrow(1)

      Column() {
        Text(this.getStatusText(transaction.status))
          .fontSize(12)
          .fontColor(this.getStatusColor(transaction.status))
          .backgroundColor(this.getStatusBackgroundColor(transaction.status))
          .padding({ horizontal: 8, vertical: 4 })
          .borderRadius(4)

        Text(new Date(transaction.timestamp).toLocaleTimeString())
          .fontSize(10)
          .fontColor('#999999')
          .margin({ top: 4 })
      }
      .alignItems(HorizontalAlign.End)
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(6)
    .margin({ bottom: 8 })
    .onClick(() => this.openTransactionDetails(transaction))
  }

  private async connectWallet(): Promise<void> {
    this.isConnecting = true

    try {
      const connection = await this.blockchainManager.connectWallet(this.selectedNetwork)
      if (connection) {
        this.connection = connection
        this.loadTransactions()
      }
    } finally {
      this.isConnecting = false
    }
  }

  private async disconnectWallet(): Promise<void> {
    await this.blockchainManager.disconnectWallet()
    this.connection = null
    this.transactions = []
  }

  private loadTransactions(): void {
    this.transactions = this.blockchainManager.getTransactions()

    // Update transactions periodically
    setInterval(() => {
      this.transactions = this.blockchainManager.getTransactions()
    }, 5000)
  }

  private formatAddress(address: string): string {
    return `${address.slice(0, 6)}...${address.slice(-4)}`
  }

  private getTransactionTypeIcon(transaction: Transaction): string {
    return transaction.from === this.connection?.address ? '📤' : '📥'
  }

  private getStatusText(status: string): string {
    const statusMap = {
      pending: 'Pending',
      confirmed: 'Confirmed',
      failed: 'Failed'
    }
    return statusMap[status] || status
  }

  private getStatusColor(status: string): string {
    const colors = {
      pending: '#FF9500',
      confirmed: '#34C759',
      failed: '#FF3B30'
    }
    return colors[status] || '#8E8E93'
  }

  private getStatusBackgroundColor(status: string): string {
    const colors = {
      pending: '#FFF3E0',
      confirmed: '#E8F5E8',
      failed: '#FFEBEE'
    }
    return colors[status] || '#F0F0F0'
  }

  private openSendDialog(): void {
    // Implementation for send dialog
    console.log('Open send dialog')
  }

  private openReceiveDialog(): void {
    // Implementation for receive dialog
    console.log('Open receive dialog')
  }

  private openContractDialog(): void {
    // Implementation for contract interaction dialog
    console.log('Open contract dialog')
  }

  private openTransactionDetails(transaction: Transaction): void {
    // Implementation for transaction details
    console.log('Transaction details:', transaction.hash)
  }
}
```

## Smart Contract Integration

```typescript
interface ContractMethod {
  name: string
  inputs: Array<{ name: string; type: string }>
  outputs: Array<{ name: string; type: string }>
  stateMutability: 'view' | 'pure' | 'nonpayable' | 'payable'
}

@Component
struct SmartContractInterface {
  @State private selectedContract: string = ''
  @State private contractMethods: ContractMethod[] = []
  @State private methodResults: Map<string, any> = new Map()
  @State private isExecuting: boolean = false

  private blockchainManager = new BlockchainManager()

  aboutToAppear() {
    this.setupContracts()
  }

  build() {
    Column() {
      Text('Smart Contracts')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 20 })

      this.buildContractSelector()

      if (this.selectedContract) {
        this.buildMethodsList()
      }
    }
    .width('100%')
    .padding(16)
  }

  @Builder
  private buildContractSelector() {
    Column() {
      Text('Select Contract')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Button('ERC-20 Token')
        .onClick(() => this.selectContract('erc20'))
        .backgroundColor(this.selectedContract === 'erc20' ? '#007AFF' : '#F0F0F0')
        .fontColor(this.selectedContract === 'erc20' ? '#FFFFFF' : '#333333')
        .width('100%')
        .margin({ bottom: 8 })

      Button('NFT Contract')
        .onClick(() => this.selectContract('nft'))
        .backgroundColor(this.selectedContract === 'nft' ? '#007AFF' : '#F0F0F0')
        .fontColor(this.selectedContract === 'nft' ? '#FFFFFF' : '#333333')
        .width('100%')
    }
    .alignItems(HorizontalAlign.Start)
    .margin({ bottom: 20 })
  }

  @Builder
  private buildMethodsList() {
    Column() {
      Text('Contract Methods')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 12 })

      ForEach(this.contractMethods, (method: ContractMethod) => {
        this.buildMethodCard(method)
      })
    }
    .alignItems(HorizontalAlign.Start)
  }

  @Builder
  private buildMethodCard(method: ContractMethod) {
    Column() {
      Row() {
        Text(method.name)
          .fontSize(16)
          .fontWeight(FontWeight.Bold)
          .flexGrow(1)

        Text(method.stateMutability)
          .fontSize(12)
          .fontColor('#666666')
          .backgroundColor('#F0F0F0')
          .padding({ horizontal: 8, vertical: 4 })
          .borderRadius(4)
      }
      .width('100%')
      .margin({ bottom: 8 })

      if (method.inputs.length > 0) {
        Text('Inputs:')
          .fontSize(14)
          .fontWeight(FontWeight.Medium)
          .margin({ bottom: 4 })

        ForEach(method.inputs, (input: any) => {
          Text(`${input.name} (${input.type})`)
            .fontSize(12)
            .fontColor('#666666')
            .margin({ left: 16, bottom: 2 })
        })
      }

      Button(method.stateMutability === 'view' ? 'Call' : 'Execute')
        .onClick(() => this.executeMethod(method))
        .backgroundColor(method.stateMutability === 'view' ? '#34C759' : '#FF9500')
        .fontColor('#FFFFFF')
        .width('100%')
        .margin({ top: 8 })
        .enabled(!this.isExecuting)

      const result = this.methodResults.get(method.name)
      if (result !== undefined) {
        Text(`Result: ${JSON.stringify(result)}`)
          .fontSize(12)
          .fontColor('#007AFF')
          .margin({ top: 8 })
          .padding(8)
          .backgroundColor('#F0F8FF')
          .borderRadius(4)
      }
    }
    .width('100%')
    .padding(16)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .margin({ bottom: 12 })
  }

  private setupContracts(): void {
    // Add sample ERC-20 contract
    this.blockchainManager.addContract('erc20', {
      address: '0x1234567890123456789012345678901234567890',
      abi: [], // Simplified - would contain full ABI
      network: {
        id: 'ethereum',
        name: 'Ethereum',
        chainId: 1,
        rpcUrl: '',
        currency: 'ETH',
        explorer: ''
      }
    })

    // Add sample NFT contract
    this.blockchainManager.addContract('nft', {
      address: '0x0987654321098765432109876543210987654321',
      abi: [], // Simplified - would contain full ABI
      network: {
        id: 'ethereum',
        name: 'Ethereum',
        chainId: 1,
        rpcUrl: '',
        currency: 'ETH',
        explorer: ''
      }
    })
  }

  private selectContract(contractId: string): void {
    this.selectedContract = contractId
    this.loadContractMethods(contractId)
  }

  private loadContractMethods(contractId: string): void {
    if (contractId === 'erc20') {
      this.contractMethods = [
        {
          name: 'balanceOf',
          inputs: [{ name: 'account', type: 'address' }],
          outputs: [{ name: 'balance', type: 'uint256' }],
          stateMutability: 'view'
        },
        {
          name: 'transfer',
          inputs: [{ name: 'to', type: 'address' }, { name: 'amount', type: 'uint256' }],
          outputs: [{ name: 'success', type: 'bool' }],
          stateMutability: 'nonpayable'
        }
      ]
    } else if (contractId === 'nft') {
      this.contractMethods = [
        {
          name: 'ownerOf',
          inputs: [{ name: 'tokenId', type: 'uint256' }],
          outputs: [{ name: 'owner', type: 'address' }],
          stateMutability: 'view'
        },
        {
          name: 'mint',
          inputs: [{ name: 'to', type: 'address' }],
          outputs: [],
          stateMutability: 'payable'
        }
      ]
    }
  }

  private async executeMethod(method: ContractMethod): Promise<void> {
    this.isExecuting = true

    try {
      // Simplified parameter preparation
      const params = this.prepareMethodParams(method)

      let result: any
      if (method.stateMutability === 'view' || method.stateMutability === 'pure') {
        result = await this.blockchainManager.callContractMethod(this.selectedContract, method.name, params)
      } else {
        result = await this.blockchainManager.executeContractTransaction(this.selectedContract, method.name, params)
      }

      this.methodResults.set(method.name, result)
    } catch (error) {
      console.error('Method execution failed:', error)
      this.methodResults.set(method.name, `Error: ${error.message}`)
    } finally {
      this.isExecuting = false
    }
  }

  private prepareMethodParams(method: ContractMethod): any[] {
    // Simplified parameter preparation
    return method.inputs.map(input => {
      switch (input.type) {
        case 'address':
          return '0x1234567890123456789012345678901234567890'
        case 'uint256':
          return '1000000000000000000' // 1 token with 18 decimals
        default:
          return ''
      }
    })
  }
}
```

## Conclusion

Blockchain integration in ArkUI enables:

- Web3 wallet connectivity and transaction management
- Smart contract interaction with read and write operations
- Multi-network support for different blockchain platforms
- Real-time transaction monitoring and status updates
- Decentralized application development capabilities

These blockchain features allow developers to create sophisticated DApps with cryptocurrency payments, NFT marketplaces, and decentralized data storage solutions.
