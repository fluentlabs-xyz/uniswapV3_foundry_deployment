## Foundry

Based on:

https://www.youtube.com/watch?v=SeiaiiviEhM

https://github.com/MarcusWentz/uniswapV3_hardhat_deployment


### Install Uniswap V3 contracts directly

```shell
forge install https://github.com/Uniswap/v3-core --no-commit
forge install https://github.com/Uniswap/v3-periphery --no-commit
forge install https://github.com/Brechtpd/base64 --no-commit
forge install https://github.com/Uniswap/solidity-lib --no-commit
```

## Install Specific OpenZeppelin Version for Uniswap V3 with NPM

Uniswap V3 depends on an outdated version of OpenZeppelin defined in package.json:

https://github.com/Uniswap/v3-periphery/blob/main/package.json

which is added to the package.json in this project with:

```shell
npm install @openzeppelin/contracts@3.4.2-solc-0.7
```

then moved into Foundry path lib with these commands:

```shell
npm i
mkdir -p lib/openzeppelin-contracts
cp -r node_modules/@openzeppelin/contracts lib/openzeppelin-contracts/
```

## Deploy Uniswap V3 With Forge Commands Directly 

### Step 1: Deploy UniswapV3Factory 

Inside file:

lib/v3-core/contracts/UniswapV3Factory.sol:UniswapV3Factory

add the following line to the contract:

```solidity
bytes32 public constant POOL_INIT_CODE_HASH = keccak256(abi.encodePacked(type(UniswapV3Pool).creationCode));
```

this will be used later updating: 
```
-SwapRouter 
-NonfungibleTokenPositionDescriptor
-NonfungiblePositionManager
```
which uses library PoolAddress with CREATE2 to compute pool contract addresses:

https://github.com/uniswap/v3-periphery/blob/main/contracts/libraries/PoolAddress.sol#L33-L47

reference: 

https://ethereum.stackexchange.com/a/156409

Script:

```shell
forge create lib/v3-core/contracts/UniswapV3Factory.sol:UniswapV3Factory  \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

### Step 2: Deploy SwapRouter 

Inside file:

lib/v3-periphery/contracts/libraries/PoolAddress.sol

go to:

https://github.com/uniswap/v3-periphery/blob/main/contracts/libraries/PoolAddress.sol#L6

then modify POOL_INIT_CODE_HASH to be the value you read from UniswapV3Factory after it was deployed:

```solidity
bytes32 internal constant POOL_INIT_CODE_HASH =  <UniswapV3Factory_POOL_INIT_CODE_HASH>;
```

Note: swaps will fail if POOL_INIT_CODE_HASH is not set correctly.

```shell
forge create lib/v3-periphery/contracts/SwapRouter.sol:SwapRouter \
--constructor-args-path src/deployConstructor/SwapRouter.txt \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

### Step 3: Deploy NFTDescriptor

```shell
forge create lib/v3-periphery/contracts/libraries/NFTDescriptor.sol:NFTDescriptor  \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

### Step 4: Deploy NonfungibleTokenPositionDescriptor

Use the --libraries flag in forge to link library NFTDescriptor to NonfungibleTokenPositionDescriptor

Example:

https://github.com/foundry-rs/foundry/issues/4587#issuecomment-1522159970

Inside file:

lib/v3-periphery/contracts/libraries/PoolAddress.sol

go to:

https://github.com/uniswap/v3-periphery/blob/main/contracts/libraries/PoolAddress.sol#L6

then modify POOL_INIT_CODE_HASH to be the value you read from UniswapV3Factory after it was deployed:

```solidity
bytes32 internal constant POOL_INIT_CODE_HASH =  <UniswapV3Factory_POOL_INIT_CODE_HASH>;
```

Note: adding liquidity will fail if POOL_INIT_CODE_HASH is not set correctly.

Script:

```shell
forge create lib/v3-periphery/contracts/NonfungibleTokenPositionDescriptor.sol:NonfungibleTokenPositionDescriptor  \
--constructor-args-path src/deployConstructor/NonfungibleTokenPositionDescriptor.txt \
--libraries lib/v3-periphery/contracts/libraries/NFTDescriptor.sol:NFTDescriptor:<contract_address> \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

### Step 5: Deploy NonfungiblePositionManager

Inside file:

lib/v3-periphery/contracts/libraries/PoolAddress.sol

go to:

https://github.com/uniswap/v3-periphery/blob/main/contracts/libraries/PoolAddress.sol#L6

then modify POOL_INIT_CODE_HASH to be the value you read from UniswapV3Factory after it was deployed:

```solidity
bytes32 internal constant POOL_INIT_CODE_HASH =  <UniswapV3Factory_POOL_INIT_CODE_HASH>;
```

Note: adding liquidity will fail if POOL_INIT_CODE_HASH is not set correctly.

Script:

```shell
forge create lib/v3-periphery/contracts/NonfungiblePositionManager.sol:NonfungiblePositionManager  \
--constructor-args-path src/deployConstructor/NonfungiblePositionManager.txt \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

## Deployments 

### Fluent Testnet Sepolia

#### UniswapV3Factory

https://testnet.fluentscan.xyz/address/0xE8eb488bEe284ed5b9657D5fc928f90F40BC2d57

#### SwapRouter

https://testnet.fluentscan.xyz/address/0xd8DEaA13846ae4f2f20f1e7B82AC60D5130BF5Cd

#### NFTDescriptor

https://testnet.fluentscan.xyz/address/0x41Ae7549023a7F0b6Cb7FE4d1807487b18cbAe10

#### NonfungibleTokenPositionDescriptor

https://testnet.fluentscan.xyz/address/0xc26ea02fb53594952b64559278bD0622555584e4

#### NonfungiblePositionManager

https://testnet.fluentscan.xyz/address/0x61B00321567CBcBcb02f2a86742b48C381aCD0d6