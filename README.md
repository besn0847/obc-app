# Open Blockchain Containerized App Set-up
This post gives a quick start-up guide for those who want to understand how [Open Blockchain](https://github.com/openblockchain/obc-docs) works. This code has been provided by IBM to the [Hyperledger](https://www.hyperledger.org/) Foundation. 

One of the main issues  of the set-up provided by IBM is the requirement to set-up a complete development environment which makes hard to quickly test on your own laptop and deploy your first chaincode (aka Smart Contract) to exchange some "coins".

This guide will help you to set-up your own distributed ledger with 4 peers. It only assumes you have a Docker environment running with at least 2GB of RAM as a peer takes about 300 to 400 Mb to run.

## Context
We will deploy one chaincode to exchange coins between A and B. At the beginning A will have 100 coins and B 200. Each time a transaction will run, it will move 10 coins from A to B. This is the chaincode sample #2 from [here](https://github.com/openblockchain/obc-peer/tree/master/openchain/example/chaincode/chaincode_example02).

On the infrastructure side, we won't run a secured distributed ledger which requires a membership service (see [here](https://github.com/openblockchain/obc-docs/blob/master/whitepaper.md)). We will run 4 validation peers in Docker containers and 1 CLI peer to deploy, invoke & query the chaincode mentionned above.

Once deployed, the chaincode is running on each peer so 4 new containers will be started as we will see. 

## Sequence
Here is the overall sequence done in this tutorial :

* Deploy the OpenBlockChain (OBC) app on the Docker host (4 validating peers + 1 client peer)
* Deploy the ChainCode (CC) on the distributed ledger
* Invoke the Chaincode to move money from A to B (once or more)
* Query the Chaincode to get the A balance
	
So something quite simple which should take no more than 5 to 10 minutes to set-up.

## Quickstart
* Step 0 : Set-up your docker host :
	* Need more than 2 GB of memory
	* Make sure the Docker daemon listen on TCP port 4243
	* Make sure there is no TLS enabled on the Docker host

If you use docker-machine, the Docker VM will look like this :
*

		docker-machine create --driver virtualbox --engine-opt host=tcp://0.0.0.0:4243 --engine-env DOCKER_TLS=no --virtualbox-memory "2048" openblockchain

And to point to the Docker host :
*

		set DOCKER_HOST=tcp://<your_local_docker_IP_addr>:4243

* Step 1 : Clone the repo : 

		git clone https://github.com/besn0847/obc-app.git

* Step 2 : Bootstrap the environment : 

		cd obc-app && docker-compose up -d

* Step 3 : Check everything is running fine :

		docker-compose ps
		    Name              Command               State   Ports
		    -----------------------------------------------------
		    cli    /bin/sh -c while(true); do ...   Up
		    vp0    obc-peer peer                    Up
		    vp1    obc-peer peer                    Up
		    vp2    obc-peer peer                    Up
		    vp3    obc-peer peer                    Up

It now time to deploy the Chaincode on the distributed ledger :

* Step 4 : Deploy the chaincode by connecting to the Client peer and then exit 

		> docker exec -t -i cli /bin/bash
		> obc-peer chaincode deploy -p github.com/openblockchain/obc-peer/openchain/example/chaincode/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'
		> exit

The chaincode deploy action will return the unique ID of the code in the ledger; this will be needed to invoke and query it. It looks like this : 

		e58eb9f63a3434fd50d35b24897a88a0d7375a82021bacfc4e8299ddbaa6af7bfb7fe03df53b842ba6b512b16c3550e2c8f4537493408a3e6e8bd8eb19fc7523

So now the blockchain has been initialized and A has 100 while B has 200. You should see now the 4 new micro services running the chaincode.

* Step 5 : Verify the chaincode has been successfully deployed with 4 new containers 

		> docker ps -q
		    d0bc608ec943
		    8e12f136da30
		    ed4202be7dfa
		    f392c898573f
		    2657dee21823
		    12b6fb5436d4
		    40a19aa8ef8a
		    801d30618a50
		    654158c5a3e0

* Step 6 : Invoke the chaincode to transfer some coins from A to B

		> docker exec -t -i cli /bin/bash
		> obc-peer chaincode invoke -n <your_blockchain_id> -c '{"Function": "invoke", "Args": ["a", "b", "10"]}'
		> exit

It will return a long unique ID to confirm the invocation. Now let's just finally check the balance for A.

* Step 7 : Check balance for A

		> docker exec -t -i cli /bin/bash
		> obc-peer chaincode query -n <your_blockchain_id> -c '{"Function": "query", "Args": ["a"]}'
		> exit

It will return value 90. You can try again step 6 multiple times and see the results.

## Conclusion
Docker seems really to be the platform of choice to run OpenBlockChain. It still needs some improvement because the image size are insane but it is very easy to set-up its own lab and start deploying your own chaincodes (only in GO for now).

The overall design is very nice with chaincode isolation in containers as any good microservice making things easy to scale and deploy.