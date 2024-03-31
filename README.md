### QuantumNexus
Flash loans are a powerful tool in decentralized finance (DeFi) that allow users to borrow assets without collateral as long as the borrowed amount is repaid within the same transaction block. Flash loans are often used for arbitrage opportunities, collateral swapping, and other sophisticated DeFi strategies.

With this code we want to achieve transaction Atomicity with Zero colateral loans using DeFi Liquidity Providers for Crypto Token Arbitrage across two different exchanges.

Let me put it in simple words, we will get a loan from a DeFi Liquidity provider using a smart-contract of 1,000.00 USDT with zero collateral required because we are using smart-contracts, then use it on an Exchange A to buy the Token A at 0.9 USDT and then use Exchange B to sell Token A at 1.0 USDT, then we repay the load of 1,000.00 USDT and we keep profit of 10% or 100.00 USDT on our wallet.   

All of this in one simgle trasnsaction and without having 1,000.00 USDT nor needing collateral for it.

Welcome to the future of Lending and Borrowing....

### Logical Functionality Diagram

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/4d7dd311-b840-4d30-9fed-d51b6c1b4599)


### Architecture Overview
![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/5a34357c-cc15-473a-8bfa-5fb9d2221a68)


### Requirements Definition

#### The OS Used:
- OS: Ubuntu 20.04 LTS x64
- <code>Linux flash1 5.4.0-170-generic #188-Ubuntu SMP Wed Jan 10 09:51:01 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux </code>

#### Software Used:
- SysAdmin experience with Linux based on Debian Environments is important specially to troubleshoot and RCA (Root Cause Analysis).
- Understanding of Blockchain and Smart Contracts: solid grasp of blockchain technology and how smart contracts work.
- Ethereum and Hardhat Knowledge: Familiarity with Ethereum and the Hardhat development environment is essential.
- If you’re new to Hardhat, consider going through their official documentation first here https://hardhat.org/getting-started
- Node.js,Yarn and NPM: Ensure you have Node.js and NPM (Node Package Manager) installed on your server.
- Remix ETH IDE - Remix Online IDE is a powerful toolset for developing, deploying, debugging, and testing Ethereum and EVM-compatible smart contracts. - https://remix.ethereum.org
- Infura - https://www.infura.io
- Alchemy - https://alchemy.com
- MetaMask - https://metamask.io
- Solidity Programming Skills - https://soliditylang.org
- OpenZeppelin - https://www.openzeppelin.com

### Project Execution Procedure
#### Step 1 - Update OS
<code> $ apt update -y && apt upgrade -y </code>

Reboot afterwards

#### Step 2 - Initialize New HardHat Project
Open your terminal and navigate to your desired project directory. Run the following commands:

<code>npm install --save-dev hardhat</code>

<code>npx hardhat</code>

Follow the prompts to create a new Hardhat project. Choose the default settings for simplicity.

#### Step 3 - Install Dependencies
We’ll need to install some additional dependencies for our project. Open your terminal and run the following commands:

<code>yarn add --dev @nomiclabs/hardhat-ethers@npm:hardhat-deploy-ethers ethers @nomiclabs/hardhat-etherscan @nomiclabs/hardhat-waffle chai ethereum-waffle hardhat hardhat-contract-sizer hardhat-deploy hardhat-gas-reporter prettier prettier-plugin-solidity solhint solidity-coverage dotenv</code>

<code>yarn add @aave/core-v3</code>

#### Step 4: Project Structure
Your project directory should now have the following structure:

```
- contracts/
  - FlashLoanArbitrage.sol
  - Dex.sol
- deploy/
  - 00-deployDex.js
  - 01-deployFlashLoanArbitrage.js  
- scripts/
- test/
- hardhat.config.js
- package.json
- README.md
```

create <code>.env</code> file, add both <code>SEPOLIA_RPC_URL</code> and <code>PRIVATE_KEY</code> by your proper credentials as follows:

<code>SEPOLIA_RPC_URL=<KEY></code>

<code>PRIVATE_KEY=<KEY></code>

Open <code>hardhat.config.js</code>, and update it with the details below:

    require("@nomiclabs/hardhat-waffle")
    require("hardhat-deploy")
    require("dotenv").config()

    /**
     * @type import('hardhat/config').HardhatUserConfig
     */

    const SEPOLIA_RPC_URL =
        process.env.SEPOLIA_RPC_URL || "<RPC_URL+KEY>"
    const PRIVATE_KEY = process.env.PRIVATE_KEY || "<KEY>" 

    module.exports = {
        defaultNetwork: "hardhat",
        networks: {
        // DEFINES THE MAIN NETWORK
            hardhat: {
                // // If you want to do some forking, uncomment this
                // forking: {
                //   url: MAINNET_RPC_URL
                // }
                chainId: 31337,
            },
            localhost: {
                chainId: 31337,
            },
        // DEFINES THE TEST NETWORK
            sepolia: {
                url: SEPOLIA_RPC_URL,
                accounts: PRIVATE_KEY !== undefined ? [PRIVATE_KEY] : [],
                //   accounts: {
                //     mnemonic: MNEMONIC,
                //   },
                saveDeployments: true,
                chainId: 11155111,
            },
        },
        namedAccounts: {
            deployer: {
                default: 0, // here this will by default take the first account as deployer
                1: 0, // similarly on mainnet it will take the first account as deployer. Note though that depending on how hardhat network are configured, the account 0 on one network can be different than on another
            },
            player: {
                default: 1,
            },
        },
        solidity: {
            compilers: [
                {
                    version: "0.8.7",
                },
                {
                    version: "0.8.10",
                },
            ],
        },
        mocha: {
            timeout: 500000, // 500 seconds max for running tests
        },
    }

    

Additionally, create a new file called <code>helper-hardhat-config.js</code> in the root directory and add the following needed details keeping in ming that we’re using the Sepolia test network for all save addresses:
1. PoolAddressesProvider,
2. daiAddress,
3. usdcAddress

Here’s how this file should look:

```const { ethers } = require('hardhat');

const networkConfig = {
  default: {
    name: 'hardhat',
  },
  11155111: {
    name: 'sepolia',
    PoolAddressesProvider: '0x0496275d34753A48320CA58103d5220d394FF77F',
    daiAddress:'0x68194a729C2450ad26072b3D33ADaCbcef39D574',
    usdcAddress:'0xda9d4f9b69ac6C22e444eD9aF0CfC043b7a7f53f',
  },
};

module.exports = {
  networkConfig
} 
```
After all the adjustments made above, here’s how our project structure should look:
```
- contracts/
  - FlashLoanArbitrage.sol
  - Dex.sol
- deploy/
  - 00-deployDex.js
  - 01-deployFlashLoanArbitrage.js  
- scripts/
- test/
-.env
- hardhat.config.js
-helper-hardhat-config
- package.json
- README.md
```
#### Step 5: Contracts

In this tutorial, we’ll be working with two smart contracts:

1. <code>Dex.sol:</code> - This contract simulates a decentralized exchange where arbitrage opportunities occur.
2. <code>FlashLoanArbitrage.sol:</code> - This contract handles flash loans and arbitrage operations.

Let’s dive into these contracts and understand each line of code. First, we’ll explore FlashLoanArbitrage.sol.

<code>Dex.sol</code>

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.10;

import {IERC20} from "@aave/core-v3/contracts/dependencies/openzeppelin/contracts/IERC20.sol";

contract Dex {
    address payable public owner;

    IERC20 private dai;
    IERC20 private usdc;

    // exchange rate indexes
    uint256 dexARate = 90;
    uint256 dexBRate = 100;

    // keeps track of individuals' dai balances
    mapping(address => uint256) public daiBalances;

    // keeps track of individuals' USDC balances
    mapping(address => uint256) public usdcBalances;

    constructor(address  _daiAddress, address _usdcAddress) {
        owner = payable(msg.sender);
        dai = IERC20(_daiAddress);
        usdc = IERC20(_usdcAddress);
    }

    function depositUSDC(uint256 _amount) external {
        usdcBalances[msg.sender] += _amount;
        uint256 allowance = usdc.allowance(msg.sender, address(this));
        require(allowance >= _amount, "Check the token allowance");
        usdc.transferFrom(msg.sender, address(this), _amount);
    }

    function depositDAI(uint256 _amount) external {
        daiBalances[msg.sender] += _amount;
        uint256 allowance = dai.allowance(msg.sender, address(this));
        require(allowance >= _amount, "Check the token allowance");
        dai.transferFrom(msg.sender, address(this), _amount);
    }

    function buyDAI() external {
        uint256 daiToReceive = ((usdcBalances[msg.sender] / dexARate) * 100) *
            (10**12);
        dai.transfer(msg.sender, daiToReceive);
    }

    function sellDAI() external {
        uint256 usdcToReceive = ((daiBalances[msg.sender] * dexBRate) / 100) /
            (10**12);
        usdc.transfer(msg.sender, usdcToReceive);
    }

    function getBalance(address _tokenAddress) external view returns (uint256) {
        return IERC20(_tokenAddress).balanceOf(address(this));
    }

    function withdraw(address _tokenAddress) external onlyOwner {
        IERC20 token = IERC20(_tokenAddress);
        token.transfer(msg.sender, token.balanceOf(address(this)));
    }

    modifier onlyOwner() {
        require(
            msg.sender == owner,
            "Only the contract owner can call this function"
        );
        _;
    }

    receive() external payable {}
}
```
Read the Dex Contract Explanation:

The <code>Dex.sol</code> contract simulates a decentralized exchanges. Let's dive into its key features:

- Dex Contract: The main contract defines storage variables for the owner, DAI and USDC addresses, and instances of IERC20 for DAI and USDC.
- Token Deposits: depositUSDC and depositDAI functions allow users to deposit USDC and DAI tokens, updating their balances.
- Token Exchange: buyDAI and sellDAI functions simulate token exchanges, buying DAI with USDC and selling DAI for USDC based on exchange rates.
- Balance Tracking: The contract tracks individual user balances for DAI and USDC with mappings.
- Token Withdrawal: The withdraw function enables the contract owner to withdraw tokens from the contract.

<code>FlashLoanArbitrage.sol</code>

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.10;

import {FlashLoanSimpleReceiverBase} from "@aave/core-v3/contracts/flashloan/base/FlashLoanSimpleReceiverBase.sol";
import {IPoolAddressesProvider} from "@aave/core-v3/contracts/interfaces/IPoolAddressesProvider.sol";
import {IERC20} from "@aave/core-v3/contracts/dependencies/openzeppelin/contracts/IERC20.sol";

interface IDex {
    function depositUSDC(uint256 _amount) external;

    function depositDAI(uint256 _amount) external;

    function buyDAI() external;

    function sellDAI() external;
}

contract FlashLoanArbitrage is FlashLoanSimpleReceiverBase {
    address payable owner;
    // Dex contract address
    address private dexContractAddress =
        0x81EA031a86EaD3AfbD1F50CF18b0B16394b1c076;

    IERC20 private dai;
    IERC20 private usdc;
    IDex private dexContract;

    constructor(address _addressProvider,address  _daiAddress, address _usdcAddress)
        FlashLoanSimpleReceiverBase(IPoolAddressesProvider(_addressProvider))
    {
        owner = payable(msg.sender);

        dai = IERC20(_daiAddress);
        usdc = IERC20(_usdcAddress);
        dexContract = IDex(dexContractAddress);
    }

    /**
        This function is called after your contract has received the flash loaned amount
     */
    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external override returns (bool) {
        //
        // This contract now has the funds requested.
        // Your logic goes here.
        //

        // Arbirtage operation
        dexContract.depositUSDC(1000000000); // 1000 USDC
        dexContract.buyDAI();
        dexContract.depositDAI(dai.balanceOf(address(this)));
        dexContract.sellDAI();

        // At the end of your logic above, this contract owes
        // the flashloaned amount + premiums.
        // Therefore ensure your contract has enough to repay
        // these amounts.

        // Approve the Pool contract allowance to *pull* the owed amount
        uint256 amountOwed = amount + premium;
        IERC20(asset).approve(address(POOL), amountOwed);

        return true;
    }

    function requestFlashLoan(address _token, uint256 _amount) public {
        address receiverAddress = address(this);
        address asset = _token;
        uint256 amount = _amount;
        bytes memory params = "";
        uint16 referralCode = 0;

        POOL.flashLoanSimple(
            receiverAddress,
            asset,
            amount,
            params,
            referralCode
        );
    }

    function approveUSDC(uint256 _amount) external returns (bool) {
        return usdc.approve(dexContractAddress, _amount);
    }

    function allowanceUSDC() external view returns (uint256) {
        return usdc.allowance(address(this), dexContractAddress);
    }

    function approveDAI(uint256 _amount) external returns (bool) {
        return dai.approve(dexContractAddress, _amount);
    }

    function allowanceDAI() external view returns (uint256) {
        return dai.allowance(address(this), dexContractAddress);
    }

    function getBalance(address _tokenAddress) external view returns (uint256) {
        return IERC20(_tokenAddress).balanceOf(address(this));
    }

    function withdraw(address _tokenAddress) external onlyOwner {
        IERC20 token = IERC20(_tokenAddress);
        token.transfer(msg.sender, token.balanceOf(address(this)));
    }

    modifier onlyOwner() {
        require(
            msg.sender == owner,
            "Only the contract owner can call this function"
        );
        _;
    }

    receive() external payable {}
}
```
Read the <code>FlashLoanArbitrage.sol</code> Contract Explanation:

The <code>FlashLoanArbitrage.sol</code> contract is the core of our arbitrage strategy. 

It utilizes Aave flash loans to execute profitable trades. Let's break down the key aspects of the contract:

- Imports and Interfaces: Import the required contracts and interfaces from Aave and OpenZeppelin. These include <code>FlashLoanSimpleReceiverBase</code>, <code>IPoolAddressesProvider</code>, and <code>IERC20</code>.
- IDex Interface: Define the interface for the decentralized exchange (DEX) where arbitrage takes place. Methods like <code>depositUSDC</code>, <code>depositDAI</code>, <code>buyDAI</code>, and <code>sellDAI</code> are defined.
- FlashLoanArbitrage Contract: The main contract, inheriting from, initializes addresses and contracts for DAI, USDC, and the DEX. It implements the <code>executeOperation</code> function that executes the arbitrage operation, buying DAI at a lower rate and selling it at a higher rate.
- Flash Loan Execution: The arbitrage operation is executed within the <code>executeOperation</code> function. Funds are deposited, DAI is bought, deposited, and then sold.
- Loan Repayment: The contract repays the flash loan amount plus the premium to Aave by approving the Pool contract to pull the owed amount.
- Flash Loan Request: The <code>requestFlashLoan</code> function initiates the flash loan by calling <code>flashLoanSimple</code> from the POOL contract.
- Token Approval and Allowance: Functions like <code>approveUSDC</code>, <code>approveDAI</code>, <code>allowanceUSDC</code>, and <code>allowanceDAI</code> are included for approving tokens and checking allowances for the DEX.
- Balance Inquiry and Withdrawal: The <code>getBalance</code> function checks the balance of a token. The <code>withdraw</code> function allows the contract owner to withdraw tokens.

#### Step 6: Deploying Smart Contracts
Deploying your smart contracts is the next crucial step. Let’s take a look at the deployment scripts.

<code>00-deployDex.js</code>

The deployment script for the <code>Dex.sol</code> contract:

```
const { network } = require("hardhat")
const { networkConfig } = require("../helper-hardhat-config")

module.exports = async ({ getNamedAccounts, deployments }) => {
  const { deploy, log } = deployments
  const { deployer } = await getNamedAccounts()
  const chainId = network.config.chainId
  const arguments = [networkConfig[chainId]["daiAddress"],networkConfig[chainId]["usdcAddress"]]

  const dex = await deploy("Dex", {
    from: deployer,
    args: arguments,
    log: true,
  })

  log("Dex contract deployed at : ", dex.address)
}

module.exports.tags = ["all", "dex"]
```

<code>01-deployFlashLoanArbitrage.js</code>

The deployment script for the <code>FlashLoanArbitrage.sol</code> contract:

```
const { network } = require("hardhat")
const { networkConfig } = require("../helper-hardhat-config")

module.exports = async ({ getNamedAccounts, deployments }) => {
  const { deploy, log } = deployments
  const { deployer } = await getNamedAccounts()
  const chainId = network.config.chainId
  const arguments = [networkConfig[chainId]["PoolAddressesProvider"],networkConfig[chainId]["daiAddress"],networkConfig[chainId]["usdcAddress"]]

  const dex = await deploy("FlashLoanArbitrage", {
    from: deployer,
    args: arguments,
    log: true,
  })

  log("FlashLoanArbitrage contract deployed at : ", dex.address)
}

module.exports.tags = ["all", "FlashLoanArbitrage"]
```

Let’s start by deploying the <code>Dex.sol</code> contract:

<code>yarn hardhat deploy --tags dex --network sepolia</code>

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/36acecfd-e769-47dc-81ec-bfbe05ddbca3)


The Dex contract address is <code>“0x81EA031a86EaD3AfbD1F50CF18b0B16394b1c076”</code> which is added to the <code>FlashLoanArbitrage</code> contract

Then we deploy <code>FlashLoanArbitrage.sol</code> by running the command below:

<code>yarn hardhat deploy --tags FlashLoanArbitrage --network sepolia</code>

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/e5692b22-1ce0-4cc0-88dc-08375bd9b1a5)

which outputs the FlashLoanArbitrage contract address below:
<code>“0xF05f88dAC4ADa7E11603ee06386086F05121C9F3”</code>

#### Step 7: Testing Smart Contracts
Let’s now text out the contracts using Remix IDE, but to be more specific here’s where the Flash Loan Arbitrage:

```
// exchange rate indexes
uint256 dexARate = 90;
uint256 dexBRate = 100;
```

Here, we buy <code>1DAI</code> at 0.90 and sell it at 100.00 .
When coping the code to Remix IDE, consider change the imports below in both contracts accordingly:

```
import {IERC20} from "@aave/core-v3/contracts/dependencies/openzeppelin/contracts/IERC20.sol";
import {FlashLoanSimpleReceiverBase} from "https://github.com/aave/aave-v3-core/blob/master/contracts/flashloan/base/FlashLoanSimpleReceiver.sol";
import {IPoolAddressesProvider} from "https://github.com/aave/aave-v3-core/blob/master/contracts/interfaces/IPoolAddressesProvider.sol";
```

Add liquidity to <code>Dex.sol</code>:
- USDC 1500
- DAI 1500

  ![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/0e433102-26b6-4be3-ac61-382f034d9921)![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/e01bdc88-2be6-48a5-b68f-7cc04d8e1e0b)

Approve:
- USDC 1000000000
- DAI 1200000000000000000000

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/d5cb2dc2-649f-4c12-ac1b-99516f19819b)

Request Loan — USDC (6 decimal):

1. 0xda9d4f9b69ac6C22e444eD9aF0CfC043b7a7f53f,1000000000 // 1,000 USDC

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/46c75898-c4d0-45cd-a853-dc58631f23fa)

Let’s view our transaction on ehterscan

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/1ea4dd3f-cab2-45d4-88c1-0e312595525d)

Here’s the transactions explanation:

1 - Transfer 1000 USDC from the aave LendingPool contract toFlashLoanArbitrage contract,

2 - Deposit 1000 USDC from the FlashLoanArbitrage contract to Dex contract,

3 - Purchase of DAI From Dex contract to FlashLoanArbitrage contract,

4 - Deposit of the DAI amount in the Dex contract,

5 - Transfer 1,111.1111 USDC from Dex contract to FlashLoanArbitrage,

6 - Repay 1000 USDC + 0.05% of flashloan fee (1000.5 USDC)

After a successfull tranaction, we’ll chek back our balalnce wich increases up to 110.611100 USDC

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/bd9cf566-b0e6-4d8b-aedd-fbad6031a0e6)

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/7b98b780-c13d-4c74-8959-b90d848818df)

Then we Witdrawn and Import the token and we will have the 110.6 USDT made on our MetaMask Wallet as follows:

![image](https://github.com/tHeStRyNg/QuantumNexus/assets/118682909/fc619271-ac7a-4727-ab14-706ac77b3908)












