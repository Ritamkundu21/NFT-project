# NFT-project

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Generating the NFT Collection](#generating-the-nft-collection)
4. [Storing Items on IPFS](#storing-items-on-ipfs)
5. [Deploying the Smart Contract](#deploying-the-smart-contract)
6. [Minting NFTs](#minting-nfts)
7. [Transferring NFTs to Polygon amoy](#transferring-nfts-to-polygon-amoy)
8. [Testing on amoy](#testing-on-amoy)
9. [Conclusion](#conclusion)

## Introduction

This project involves creating an NFT collection, storing the items on IPFS, deploying an ERC721 or ERC1155 contract to the sepolia Ethereum Testnet, and transferring the NFTs to the Polygon amoy network using the FxPortal Bridge. The project utilizes Hardhat, Ethers.js, and various other tools to streamline the process.

## Prerequisites

- Node.js and npm installed
- Hardhat installed
- Ethers.js installed
- An account on Pinata.cloud
- Access to DALLE 2 or Midjourney for image generation
- Metamask wallet for sepolia and amoy networks

## Generating the NFT Collection

Use DALLE 2 or Midjourney to generate a 5-item collection. Each item should have a unique prompt. 

Ensure that you have access to the prompts used for generating the images, as they will be returned by the `promptDescription` function in your smart contract.

## Storing Items on IPFS

1. Create an account on [Pinata.cloud](https://pinata.cloud).
2. Upload each item to IPFS using Pinata and obtain the CID for each image.
3. Store the CIDs securely, as they will be used in your smart contract.

## Deploying the Smart Contract

1. Write a smart contract (ERC721 or ERC1155) with a `promptDescription` function that returns the prompt used for generating the images.
2. Deploy the contract to the sepolia Ethereum Testnet using Hardhat.

### Example Smart Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFTCollection is ERC721 {
    string[] public prompts;
    mapping(uint256 => string) private _tokenURIs;

    constructor(string[] memory _prompts) ERC721("MyNFTCollection", "MNFTC") {
        prompts = _prompts;
    }

    function mint(address to, uint256 tokenId, string memory tokenURI) public {
        _mint(to, tokenId);
        _setTokenURI(tokenId, tokenURI);
    }

    function _setTokenURI(uint256 tokenId, string memory tokenURI) internal virtual {
        _tokenURIs[tokenId] = tokenURI;
    }

    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        return _tokenURIs[tokenId];
    }

    function promptDescription(uint256 tokenId) public view returns (string memory) {
        return prompts[tokenId];
    }
}
```

### Deploy Script

```javascript
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();

    const prompts = ["prompt1", "prompt2", "prompt3", "prompt4", "prompt5"];
    const MyNFTCollection = await ethers.getContractFactory("MyNFTCollection");
    const myNFTCollection = await MyNFTCollection.deploy(prompts);

    console.log("Contract deployed to address:", myNFTCollection.address);
}

main().catch((error) => {
    console.error(error);
    process.exit(1);
});
```

## Minting NFTs

Create a Hardhat script to batch mint all NFTs using ERC721A.

### Batch Mint Script

```javascript
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    const contractAddress = "YOUR_CONTRACT_ADDRESS";
    const MyNFTCollection = await ethers.getContractFactory("MyNFTCollection");
    const myNFTCollection = MyNFTCollection.attach(contractAddress);

    const cids = ["cid1", "cid2", "cid3", "cid4", "cid5"];
    const batchSize = cids.length;

    for (let i = 0; i < batchSize; i++) {
        await myNFTCollection.mint(deployer.address, i, cids[i]);
        console.log(`Minted token ${i} with CID: ${cids[i]}`);
    }
}

main().catch((error) => {
    console.error(error);
    process.exit(1);
});
```

## Transferring NFTs to Polygon amoy

1. Approve the NFTs for transfer.
2. Deposit the NFTs to the FxPortal Bridge.
3. Use Hardhat to script the batch transfer of all NFTs.

### Approve and Transfer Script

```javascript
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    const contractAddress = "YOUR_CONTRACT_ADDRESS";
    const MyNFTCollection = await ethers.getContractFactory("MyNFTCollection");
    const myNFTCollection = MyNFTCollection.attach(contractAddress);

    const fxPortalAddress = "FXPORTAL_CONTRACT_ADDRESS"; // Update with the actual address

    for (let tokenId = 0; tokenId < 5; tokenId++) {
        await myNFTCollection.approve(fxPortalAddress, tokenId);
        console.log(`Approved token ${tokenId} for transfer`);
    }

    // Deposit function and further bridge interaction should be implemented here
}

main().catch((error) => {
    console.error(error);
    process.exit(1);
});
```

## Testing on amoy

After the NFTs are transferred to Polygon amoy, check the balance of the NFTs on the amoy network to ensure the transfer was successful.

### Balance Check Script

```javascript
const { ethers } = require("hardhat");

async function main() {
    const [deployer] = await ethers.getSigners();
    const amoyProvider = new ethers.providers.JsonRpcProvider("https://rpc-amoy.maticvigil.com/");
    const contractAddress = "YOUR_CONTRACT_ADDRESS_ON_amoy";
    const MyNFTCollection = await ethers.getContractFactory("MyNFTCollection", deployer);
    const myNFTCollection = MyNFTCollection.attach(contractAddress);

    const balance = await myNFTCollection.balanceOf(deployer.address);
    console.log(`Balance on amoy: ${balance}`);
}

main().catch((error) => {
    console.error(error);
    process.exit(1);
});
```

## Conclusion

This README provides a comprehensive guide for creating, deploying, minting, and transferring an NFT collection across Ethereum and Polygon networks. Follow each step carefully, and ensure all scripts are properly configured with your contract addresses and other necessary parameters.
