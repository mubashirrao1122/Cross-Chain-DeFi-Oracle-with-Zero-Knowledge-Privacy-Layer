# Deployment Guide

This guide provides detailed instructions for deploying the Cross-Chain DeFi Oracle with Zero-Knowledge Privacy Layer system to various environments.

## Deployment Environments

The system can be deployed to the following environments:

- **Development**: Local development environment
- **Testnet**: Public blockchain testnets (Goerli, Sepolia, Devnet, etc.)
- **Staging**: Production-like environment with isolated data
- **Production**: Live environment with real assets

## Prerequisites

Before deployment, ensure you have:

- Access credentials for deployment targets
- Required private keys and seed phrases (securely stored)
- Infrastructure provisioned (if applicable)
- Security audit completed for production deployments
- Monitoring systems configured

## Deployment Configuration

### Environment Variables

Create environment-specific configuration files with the following variables:

```bash
# Network configuration
NETWORK_TYPE=mainnet|testnet|local
ETHEREUM_RPC_URL=https://ethereum-rpc.example.com
SOLANA_RPC_URL=https://solana-rpc.example.com
POLKADOT_RPC_URL=https://polkadot-rpc.example.com

# Security keys (use secure key management)
DEPLOYER_PRIVATE_KEY=xxx
OPERATOR_PRIVATE_KEY=xxx
VALIDATOR_PRIVATE_KEY=xxx

# Service configuration
ORACLE_VALIDATOR_COUNT=5
BRIDGE_CONFIRMATION_BLOCKS=20
ZKP_PROVING_PARALLELISM=4
LAYER2_BATCH_SIZE=1000

# API and infrastructure
API_RATE_LIMIT=1000
DATABASE_URL=postgres://user:password@localhost:5432/dbname
REDIS_URL=redis://localhost:6379
IPFS_GATEWAY=https://ipfs.example.com

# Monitoring and logging
MONITORING_ENDPOINT=https://metrics.example.com
LOG_LEVEL=info
```

### Network Configuration

Create network-specific configuration files for each blockchain:

```json
// config/ethereum/mainnet.json
{
  "rpcUrl": "https://mainnet.infura.io/v3/YOUR_API_KEY",
  "chainId": 1,
  "confirmations": 12,
  "contracts": {
    "bridge": "0x...",
    "oracle": "0x...",
    "verifier": "0x..."
  },
  "gasSettings": {
    "maxFeePerGas": "100000000000",
    "maxPriorityFeePerGas": "2000000000"
  }
}
```

## Deployment Process

### 1. Smart Contract Deployment

#### Ethereum Contracts

```bash
# Compile contracts
npx hardhat compile

# Deploy to testnet
npx hardhat run scripts/deploy-ethereum.js --network goerli

# Verify contracts
npx hardhat verify --network goerli DEPLOYED_CONTRACT_ADDRESS "Constructor Arg 1" "Constructor Arg 2"
```

#### Solana Programs

```bash
# Build programs
anchor build

# Deploy to devnet
anchor deploy --provider.cluster devnet

# Upgrade existing program
solana program deploy --program-id PROGRAM_ID ./target/deploy/program.so
```

#### Cross-Chain Configuration

After deploying contracts to all chains, configure the cross-chain mappings:

```bash
# Set up chain connections
node scripts/setup-cross-chain-config.js --config config/deployment/mainnet.json
```

### 2. Backend Services Deployment

#### Using Docker Compose (Development/Staging)

```bash
# Build service images
docker-compose build

# Deploy services
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

#### Using Kubernetes (Production)

```bash
# Apply configuration
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/configmap.yaml

# Deploy core services
kubectl apply -f k8s/oracle
kubectl apply -f k8s/bridge
kubectl apply -f k8s/zkp
kubectl apply -f k8s/layer2

# Deploy API and frontend
kubectl apply -f k8s/api
kubectl apply -f k8s/frontend

# Verify deployment
kubectl get pods -n cross-chain-oracle-zkp
```

### 3. ZK Trusted Setup

For production deployments, perform a trusted setup ceremony:

```bash
# Generate initial parameters
npx snarkjs powersoftau new bn128 15 pot15_0000.ptau -v

# Contribute to the ceremony (repeat with multiple participants)
npx snarkjs powersoftau contribute pot15_0000.ptau pot15_0001.ptau --name="First contribution" -v

# Apply random beacon
npx snarkjs powersoftau beacon pot15_0001.ptau pot15_beacon.ptau 0102030405060708090a0b0c0d0e0f10 10 -n="Final Beacon"

# Prepare phase 2
npx snarkjs powersoftau prepare phase2 pot15_beacon.ptau pot15_final.ptau -v

# Generate circuit-specific setup
npx snarkjs groth16 setup circuit.r1cs pot15_final.ptau circuit_0000.zkey

# Contribute to phase 2 (repeat with multiple participants)
npx snarkjs zkey contribute circuit_0000.zkey circuit_0001.zkey --name="Second contribution" -v

# Finalize setup
npx snarkjs zkey beacon circuit_0001.zkey circuit_final.zkey 0102030405060708090a0b0c0d0e0f10 10 -n="Final Beacon phase2"

# Export verification key
npx snarkjs zkey export verificationkey circuit_final.zkey verification_key.json
```

### 4. Oracle Network Setup

```bash
# Initialize oracle network
node scripts/oracle/initialize-network.js --config config/oracle/mainnet.json

# Register initial validators
node scripts/oracle/register-validators.js --validators validators.json

# Configure data sources
node scripts/oracle/configure-data-sources.js --sources data-sources.json

# Test oracle network
node scripts/oracle/test-oracle-request.js
```

### 5. Bridge Network Setup

```bash
# Initialize bridge relayers
node scripts/bridge/initialize-relayers.js --config config/bridge/mainnet.json

# Configure bridge limits and parameters
node scripts/bridge/configure-bridge.js --limits bridge-limits.json

# Test bridge functionality
node scripts/bridge/test-bridge-transfer.js
```

### 6. Layer 2 Deployment

```bash
# Initialize Layer 2 state
node scripts/layer2/initialize-state.js --config config/layer2/mainnet.json

# Deploy prover service
kubectl apply -f k8s/layer2/prover.yaml

# Start batch processor
kubectl apply -f k8s/layer2/batch-processor.yaml
```

## Post-Deployment Verification

### 1. Smoke Tests

Run basic functionality tests to verify the deployment:

```bash
# Run smoke tests
npm run test:smoke -- --network mainnet
```

### 2. Security Verification

Verify security settings and access controls:

```bash
# Verify contract permissions
node scripts/security/verify-permissions.js --network mainnet

# Check for common vulnerabilities
node scripts/security/vulnerability-scan.js --network mainnet
```

### 3. Performance Benchmarking

Measure system performance metrics:

```bash
# Run performance benchmarks
npm run benchmark -- --network mainnet
```

## Monitoring Setup

### 1. Logging Configuration

Configure centralized logging:

```bash
# Deploy logging stack
kubectl apply -f k8s/monitoring/logging

# Verify logs are flowing
curl -X GET "https://logging.example.com/api/status"
```

### 2. Metrics and Alerting

Set up monitoring and alerting:

```bash
# Deploy monitoring stack
kubectl apply -f k8s/monitoring/prometheus
kubectl apply -f k8s/monitoring/grafana

# Configure alerts
kubectl apply -f k8s/monitoring/alerts
```

### 3. Dashboard Setup

Set up monitoring dashboards:

- Oracle Network Health
- Bridge Transaction Status
- ZK Proof Generation Metrics
- Layer 2 Performance
- API Usage and Latency

## Backup and Recovery

### 1. Database Backups

Configure regular database backups:

```bash
# Set up automated backups
kubectl apply -f k8s/backup/database-backup.yaml

# Test backup and restore procedure
node scripts/admin/test-backup-restore.js
```

### 2. Contract State Backups

Regularly back up critical on-chain state:

```bash
# Back up contract state
node scripts/admin/backup-contract-state.js --network mainnet
```

### 3. Recovery Procedures

Document and test recovery procedures:

- Oracle network recovery
- Bridge emergency shutdown and restart
- Layer 2 state reconstruction
- Full system recovery from backups

## Upgrade Procedures

### 1. Smart Contract Upgrades

For upgrading smart contracts:

```bash
# Deploy new implementation
npx hardhat run scripts/upgrade/deploy-implementation.js --network mainnet

# Prepare upgrade transaction
npx hardhat run scripts/upgrade/prepare-upgrade.js --network mainnet

# Execute upgrade
npx hardhat run scripts/upgrade/execute-upgrade.js --network mainnet
```

### 2. Backend Service Upgrades

For zero-downtime backend upgrades:

```bash
# Update service image
docker build -t oracle-service:new .

# Push to registry
docker push registry.example.com/oracle-service:new

# Rolling update
kubectl set image deployment/oracle-service oracle-service=registry.example.com/oracle-service:new

# Verify update
kubectl rollout status deployment/oracle-service
```

### 3. Database Migrations

For database schema upgrades:

```bash
# Apply migrations
npm run migrate:up

# Verify migration
npm run migrate:status
```

## Troubleshooting

### Common Deployment Issues

1. **RPC Connection Failures**
   - Check network connectivity
   - Verify RPC endpoint is correct and accessible
   - Check authentication if using private RPC

2. **Contract Deployment Failures**
   - Verify gas settings are appropriate
   - Check constructor arguments
   - Ensure account has sufficient funds

3. **Service Startup Failures**
   - Check environment variables
   - Verify service dependencies are available
   - Check container resource limits

### Rollback Procedures

In case of deployment failures:

```bash
# Rollback contracts
npx hardhat run scripts/rollback/rollback-contracts.js --network mainnet

# Rollback services
kubectl rollout undo deployment/service-name

# Restore database
node scripts/admin/restore-database.js --backup backup_id
```

## Compliance and Audit

### 1. Deployment Auditing

Record all deployment activities:

```bash
# Generate deployment report
node scripts/admin/generate-audit-report.js --deployment mainnet-2025-07-08
```

### 2. Regulatory Compliance

Verify compliance requirements:

```bash
# Run compliance checks
node scripts/compliance/verify-compliance.js --region global
```

### 3. Post-Deployment Audit

Conduct a thorough post-deployment audit:

- Contract addresses and versions
- Access control settings
- Parameter configurations
- Integration points
- Security settings

## Additional Resources

- [Infrastructure Setup Guide](infrastructure-setup.md)
- [Disaster Recovery Plan](disaster-recovery.md)
- [Compliance Requirements](compliance-requirements.md)
- [Monitoring Guidelines](monitoring-guidelines.md)
