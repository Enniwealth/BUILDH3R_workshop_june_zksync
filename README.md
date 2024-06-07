# Deploying My First Contract on zkSync and Understanding Paymasters

## Introduction
Hello,
I'm excited to share the profound knowledge i got from the BUILDH3R Zksync workshop. I learnt a ton about smart contracts and how they are deployed on zksync using the zksync extension on metamask and I'll like to walk you through the things i learnt in my journey into the world of Zksync.
Welcome to your first experience deploying a smart contract on the zkSync blockchain! This guide will walk you through the steps to build and deploy your first contract on zkSync and introduce you to the concept of Paymasters, which allow you to subsidize transaction fees using your own tokens.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Setting Up the Environment](#setting-up-the-environment)
3. [Deploying Your First Contract](#deploying-your-first-contract)
4. [Introducing Paymasters](#introducing-paymasters)
   - [What is a Paymaster?](#what-is-a-paymaster)
   - [Using Paymasters to Pay Fees with Your Own Token](#using-paymasters-to-pay-fees-with-your-own-token)
5. [Conclusion](#conclusion)

## Getting Started

Before we dive into the technical details, let's ensure you have everything set up for a smooth deployment process.

### Prerequisites

- **Browser Wallet**: Ensure you have a browser wallet like MetaMask configured to connect to the zkSync Sepolia Testnet. Follow the instructions [here](https://docs.zksync.io/build/connect-to-zksync) to set it up.
- **Testnet ETH**: Fund your wallet with zkSync Sepolia Testnet ETH using one of the available faucets. This is essential to pay for gas during the contract deployment.
- **Basic Solidity Knowledge**: Familiarity with writing and deploying smart contracts using Solidity is recommended.

## Setting Up the Environment

To deploy your first contract on zkSync, you'll need to prepare your development environment. Follow these steps:

1. **Install Node.js and npm**: Make sure you have Node.js and npm installed on your machine. You can download and install them from [nodejs.org](https://nodejs.org/).

2. **Install zkSync CLI**: Install the zkSync command-line interface (CLI) to interact with the zkSync network.
   ```bash
   npm install -g zkSync-cli
   ```
   or you can just use remix for the simplicity (https://remix.ethereum.org/#lang=en&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.26+commit.8a97fa7a.js)

3. **Initialize a New Project**: Create a new directory for your project and initialize it with `zkSync-cli`.
   ```bash
   mkdir zkSyncFirstContract
   cd zkSyncFirstContract
   zkSync-cli init
   ```

4. **Install Hardhat and zkSync Plugins**:
   ```bash
   npm install --save-dev hardhat @matterlabs/hardhat-zksync-deploy @matterlabs/hardhat-zksync-solc
   ```

## Deploying Your First Contract

Now, let's write and deploy a simple Solidity contract on the zkSync network.

1. **Create a Solidity Contract**: In your project directory, create a new file `contracts/HelloWorld.sol` with the following content:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.0;

   contract HelloWorld {
       string public message;

       constructor(string memory initMessage) {
           message = initMessage;
       }

       function setMessage(string memory newMessage) public {
           message = newMessage;
       }
   }
   ```

2. **Compile the Contract**: Use Hardhat to compile your contract.
   ```bash
   npx hardhat compile
   ```

3. **Deploy the Contract**: Create a deployment script `scripts/deploy.js`:
   ```javascript
   const { zkSyncDeploy } = require("@matterlabs/hardhat-zksync-deploy");

   async function main() {
       const [deployer] = await ethers.getSigners();

       console.log("Deploying contracts with the account:", deployer.address);

       const HelloWorld = await ethers.getContractFactory("HelloWorld");
       const helloWorld = await HelloWorld.deploy("Hello, zkSync!");

       await helloWorld.deployed();

       console.log("HelloWorld deployed to:", helloWorld.address);
   }

   main()
       .then(() => process.exit(0))
       .catch((error) => {
           console.error(error);
           process.exit(1);
       });
   ```

   Run the deployment script:
   ```bash
   npx hardhat run scripts/deploy.js --network zksync
   ```

Congratulations! You've successfully deployed your first contract on the zkSync blockchain.

## Introducing Paymasters

### What is a Paymaster?

In the zkSync ecosystem, Paymasters represent a groundbreaking approach to managing transaction fees. They are specialized smart contracts designed to subsidize transaction costs for other accounts. This feature is particularly beneficial for dApp developers who wish to enhance their platform's user experience by covering transaction fees on behalf of their users, thus making certain transactions effectively free for end-users.

### Using Paymasters to Pay Fees with Your Own Token

This section will guide you through leveraging a Paymaster to pay transaction fees with your ERC20 token.

1. **Understanding Paymasters**:
   Paymasters validate and pay for transaction fees based on specific criteria. They can be configured to accept various tokens or currencies, making them flexible for different use cases.

2. **Smart Contract for Paymaster**:
   You need a smart contract that implements the Paymaster logic. Typically, a Paymaster contract will have two main functions:
   - **`validateAndPayForPaymasterTransaction`**: This function checks the transaction parameters and covers the transaction fee if the conditions are met.
   - **`postTransaction` (optional)**: This function can perform additional actions after the transaction is executed.

3. **Deploying a Paymaster**:
   Before deploying your Paymaster, ensure you have an ERC20 token contract deployed. Refer to the previous tutorials on deploying contracts and creating an ERC20 token.

   Here's a simplified example of a Paymaster smart contract:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.0;

   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   import "@zksync/contracts/Paymaster.sol";

   contract SimplePaymaster is Paymaster {
       IERC20 public token;
       address public owner;

       constructor(address tokenAddress) {
           token = IERC20(tokenAddress);
           owner = msg.sender;
       }

       function validateAndPayForPaymasterTransaction(
           address from,
           uint256 amount,
           address to
       ) external payable override {
           // Validation logic
           require(token.balanceOf(from) >= amount, "Insufficient token balance");
           require(msg.value >= tx.gasprice * tx.gaslimit, "Insufficient ETH for gas");

           // Pay for the transaction fee using the provided ETH
           payable(to).transfer(msg.value);
       }

       function postTransaction() external override {
           // Optional post-transaction logic
       }

       // Additional functions for managing the paymaster can be added here
   }
   ```

4. **Configuring Your Paymaster**:
   Adjust your Paymaster contract to fit your specific needs. You can set up logic to validate transactions based on various criteria, such as the token balance of the user or specific transaction parameters.

5. **Deploy and Use the Paymaster**:
   Deploy your Paymaster smart contract just like any other Solidity contract. Once deployed, you can configure your dApp to utilize the Paymaster for handling transaction fees, enhancing the user experience by allowing them to use your ERC20 token to cover transaction costs.

## Conclusion

Deploying your first contract on zkSync is a significant milestone. By leveraging Paymasters, you can further enhance your dApp's functionality and user experience. Whether you're subsidizing transaction fees or integrating innovative fee payment mechanisms, zkSync provides the tools to make your application stand out.

For further information and advanced usage of zkSync features, check out the official [zkSync documentation](https://zksync.io/docs/).

---

Feel free to reach out if you have any questions or need additional support. Happy building on zkSync!