# Open Blockchain Containerized App Set-up
This post gives a quick start-up guide for those who want to understand how [Open Blockchain](https://github.com/openblockchain/obc-docs)) works. This code has been provided by IBM to the [Hyperledger](https://www.hyperledger.org/) Foundation. 

One of the main issue of the set-up provided is the requirement to set-up a complete development environment which makes hard to quickly test on your own laptop and deploy your first chaincode (aka Smart Contract) to exchange some "coins".

This guide will help you to set-up your own distributed ledger with 4 peers. It only assumes you have a Docker environment running with at least 2GB of RAM as a peer takes about 300 to 400 Mb to run.

## Context
We will deploy one chaincode to exchange coins between A and B. At the beginning A will have 100 coins and B 200. Each time a transction will be run, it will move 10 coins from A to B. This is the chaincode sample #2 from [here](https://github.com/openblockchain/obc-peer/tree/master/openchain/example/chaincode/chaincode_example02).

On the infrastructure side, we won't run a secured distributed ledger which requires a membership service (see [here](https://github.com/openblockchain/obc-docs/blob/master/whitepaper.md)). We will run 4 validation peers in Docker containers and 1 CLI peer to deploy, invoke & query the chaincode mentionned above.

One deployed, the chaincode is deployed on each peer to 4 new containers will be deployed as we will see. There are rather small since they don't exceed 10mb.

## Sequence
Here is the overall sequence done in this tutorial :
	* 