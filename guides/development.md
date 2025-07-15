# Development Guide

This guide provides comprehensive instructions for setting up a development environment and contributing to the Cross-Chain DeFi Oracle with Zero-Knowledge Privacy Layer project.

## Development Environment Setup

### Prerequisites

- **Node.js**: v16.x or later
- **Rust**: 1.60.0 or later
- **Go**: 1.18 or later
- **Solidity Compiler**: 0.8.x
- **Docker**: Latest version
- **Python**: 3.9 or later (for testing scripts)

### Installation

1. Clone the repository:

```bash
git clone https://github.com/mubashirrao1122/cross-chain-oracle-zkp.git
cd cross-chain-oracle-zkp
```

2. Install dependencies:

```bash
# Install JavaScript dependencies
npm install

# Install Rust components
cd zkp-core
cargo build
cd ..

# Set up development databases
docker-compose up -d
```

3. Configure environment:

```bash
# Copy example configuration
cp .env.example .env

# Edit the .env file with your settings
vim .env
```

4. Install development tools:

```bash
# Install Hardhat for Ethereum development
npm install -g hardhat

# Install Anchor for Solana development
npm install -g @project-serum/anchor

# Install ZK tools
npm install -g circom snarkjs
```

## Project Structure

```
├── contracts/                # Smart contracts for all components
│   ├── ethereum/             # Ethereum contracts
│   ├── solana/               # Solana programs
│   └── substrate/            # Substrate pallets
├── zkp-core/                 # Zero-knowledge proof system
│   ├── circuits/             # ZK circuit definitions
│   ├── prover/               # Proof generation code
│   └── verifier/             # Proof verification code
├── bridge/                   # Cross-chain bridge implementation
│   ├── relayer/              # Bridge relayer service
│   └── adapters/             # Chain-specific adapters
├── oracle/                   # Oracle system implementation
│   ├── validators/           # Validator node software
│   ├── aggregators/          # Data aggregation services
│   └── connectors/           # External data source connectors
├── layer2/                   # Layer 2 scaling solution
│   ├── processor/            # Transaction processing engine
│   ├── state-manager/        # State management system
│   └── prover/               # Layer 2 proof generation
├── sdk/                      # Client SDKs
│   ├── js/                   # JavaScript/TypeScript SDK
│   ├── python/               # Python SDK
│   └── rust/                 # Rust SDK
├── api/                      # API services
│   ├── rest/                 # REST API implementation
│   ├── graphql/              # GraphQL API implementation
│   └── rpc/                  # JSON-RPC API implementation
├── frontend/                 # Web interface
├── tests/                    # Test suite
├── docs/                     # Documentation
└── tools/                    # Development and deployment tools
```

## Development Workflow

### 1. Setting Up Local Blockchain Networks

```bash
# Start local Ethereum network
npx hardhat node

# Start local Solana validator
solana-test-validator

# Start local Substrate node
./scripts/start-substrate-node.sh
```

### 2. Deploying Contracts for Development

```bash
# Deploy Ethereum contracts
npx hardhat run scripts/deploy-ethereum.js --network localhost

# Deploy Solana programs
anchor deploy

# Deploy to Substrate
./scripts/deploy-substrate.sh
```

### 3. Running ZK Proof System

```bash
# Generate circuit
cd zkp-core
npm run build:circuits

# Start prover service
npm run start:prover

# Start verifier service
npm run start:verifier
```

### 4. Starting Development Services

```bash
# Start all services in development mode
npm run dev

# Or start individual components
npm run dev:oracle
npm run dev:bridge
npm run dev:zkp
npm run dev:layer2
```

### 5. Running Tests

```bash
# Run all tests
npm test

# Run specific test categories
npm run test:unit
npm run test:integration
npm run test:e2e

# Run specific component tests
npm run test:oracle
npm run test:bridge
npm run test:zkp
npm run test:layer2
```

## Development Standards

### Code Style

The project follows strict coding standards enforced by linters:

```bash
# Run linters
npm run lint

# Fix automatic linting issues
npm run lint:fix
```

For specific languages:

- **JavaScript/TypeScript**: ESLint with Airbnb style guide
- **Solidity**: Solhint with project-specific rules
- **Rust**: Rustfmt and Clippy
- **Go**: Gofmt and golangci-lint

### Git Workflow

1. Create a feature branch from `develop`:

```bash
git checkout develop
git pull
git checkout -b feature/your-feature-name
```

2. Make changes and commit using conventional commit format:

```bash
# Example commit messages:
git commit -m "feat: implement zero-knowledge proof verification"
git commit -m "fix: resolve cross-chain message duplication issue"
git commit -m "docs: update API documentation"
```

3. Push changes and create a pull request to `develop`:

```bash
git push -u origin feature/your-feature-name
```

4. After code review and approval, the branch will be merged.

### Documentation

All code should be thoroughly documented:

- **Smart Contracts**: NatSpec documentation for all public functions
- **API Endpoints**: OpenAPI/Swagger documentation
- **Functions and Classes**: JSDoc, RustDoc, or equivalent
- **Architecture Changes**: Updated diagrams and design docs

## Testing Guidelines

### Unit Testing

- Every component should have comprehensive unit tests
- Mock external dependencies
- Aim for >90% code coverage

Example unit test (JavaScript):

```javascript
describe('Oracle Validator', () => {
  it('should correctly aggregate price data from multiple sources', async () => {
    // Arrange
    const mockDataSources = [
      { getPrice: async () => ({ value: '1950.25', timestamp: Date.now() }) },
      { getPrice: async () => ({ value: '1955.50', timestamp: Date.now() }) },
      { getPrice: async () => ({ value: '1952.75', timestamp: Date.now() }) }
    ];
    
    const validator = new OracleValidator(mockDataSources);
    
    // Act
    const result = await validator.aggregatePrice('ETH/USD');
    
    // Assert
    expect(result.value).to.equal('1952.75'); // Should be median value
    expect(result.sources).to.equal(3);
    expect(result.timestamp).to.be.a('number');
  });
});
```

### Integration Testing

- Test interactions between components
- Use local blockchain networks
- Test realistic scenarios

Example integration test:

```javascript
describe('Cross-Chain Bridge Integration', () => {
  let ethereumBridge, solanaBridge, user;

  before(async () => {
    // Set up test environment
    ethereumBridge = await deployEthereumBridge();
    solanaBridge = await deploySolanaBridge();
    user = await createTestUser();
  });

  it('should transfer tokens from Ethereum to Solana', async () => {
    // Arrange
    const amount = ethers.utils.parseEther('10');
    await fundUserWithEth(user.ethereumAddress, amount.mul(2));
    
    // Act
    const tx = await ethereumBridge.connect(user.ethereumWallet).transferToSolana(
      USDC_ADDRESS,
      amount,
      user.solanaAddress
    );
    await tx.wait();
    
    // Wait for relayers to process
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    // Assert
    const solanaBalance = await getSolanaTokenBalance(user.solanaAddress, USDC_SPL_ADDRESS);
    expect(solanaBalance.toString()).to.equal(amount.toString());
  });
});
```

### Security Testing

- Run automated security analyzers
- Conduct regular internal audits
- Consider formal verification for critical components

```bash
# Run security analysis
npm run security:analyze

# Run formal verification (where applicable)
npm run verify:contracts
```

## Debugging Tools

### Smart Contract Debugging

- Use Hardhat's debugging capabilities:

```bash
npx hardhat test --debug
```

- For Solana, use Anchor's logs:

```bash
solana logs | grep BPF
```

### ZK Proof Debugging

- Debug circuits with:

```bash
npm run debug:circuit
```

- Visualize circuit constraints:

```bash
npm run visualize:circuit
```

### Transaction Tracing

- Trace Ethereum transactions:

```bash
npx hardhat trace --hash 0x123...
```

- For cross-chain operations, use the debugging dashboard:

```bash
npm run start:debug-dashboard
```

## Performance Optimization

### Gas Optimization for Ethereum

- Use the gas reporter to analyze contract gas usage:

```bash
REPORT_GAS=true npx hardhat test
```

### ZK Proof Optimization

- Optimize circuit constraints:

```bash
npm run optimize:circuits
```

- Benchmark proof generation:

```bash
npm run benchmark:prover
```

## Contribution Guidelines

1. **Select an Issue**: Pick an open issue from the GitHub repository
2. **Discuss Approach**: Comment on the issue to discuss your approach
3. **Implementation**: Follow the development workflow above
4. **Testing**: Ensure all tests pass and add new tests for your feature
5. **Documentation**: Update relevant documentation
6. **Pull Request**: Submit a PR with a clear description of changes
7. **Code Review**: Address feedback from maintainers
8. **Merge**: Once approved, your PR will be merged

For major changes, please open an issue first to discuss what you would like to change.

## Additional Resources

- [Project Roadmap](../roadmap.md)
- [Architecture Decision Records](../architecture/decisions/)
- [Known Issues and Workarounds](../troubleshooting.md)
- [External Dependencies Documentation](../references.md)
