# Iroha-Daemon-Setup


#############===========AccountID will be use as just like address==========##################

 

### Installing Docker CE on Ubuntu 16.04-->

==>Docker is an application that allows to deploy programs that are run as containers. It was written in the popular Go programming language. This tutorial explains how to install Docker CE on Ubuntu 16.04.

Step 1: Updating all your software

==>First off, let's make sure that we are using a clean system. Run the apt updater.

apt-get update

Step 2: Set up the repository

==>Install packages to allow apt to use a repository over HTTPS

apt-get install apt-transport-https ca-certificates curl software-properties-common

==>Add Docker’s official GPG key

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

==>Use the following command to set up the stable repository.

add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

Step 3: Install Docker CE

apt-get update
apt-get install docker-ce

Step 4: Create a user

==>If you decide not to run Docker as the root user, you will need to create a non-root user.

==>Warning: The docker group grants privileges equivalent to the root user.

adduser user
usermod -aG docker user

Restart the Docker service.

systemctl restart docker

Step 5: Test Docker

==>Run the Docker hello-world container to test if the installation completed successfully.

docker run hello-world

==>You will see the following output.

    Hello from Docker!

   ## This message shows that your installation appears to be working correctly.

Step 6: Configure Docker to start on boot

==>Lastly, enable Docker to run when your system boots.

systemctl enable docker



#################================== Starting Iroha Node

### Getting Started

==>In this guide, we will create a very basic Iroha network, launch it, create a couple of transactions, and check the data written in the ledger. To keep things simple, we will use Docker.

### Prerequisites

==>For this guide, you need a machine with Docker installed


### Creating a Docker Network

==>To operate, Iroha requires a PostgreSQL database. Let’s start with creating a Docker network, so containers for Postgres and Iroha can run on the same virtual network and successfully communicate. In this guide we will call it iroha-network, but you can use any name. In your terminal write following command:

$ docker network create iroha-network

-->27c783228697b8c4ef09ecb9d874d01b48204841ab751b26c7d3e7c311d3cbc

### Starting PostgreSQL Container

==>Now we need to run PostgreSQL in a container, attach it to the network you have created before, and expose ports for communication:

$ docker run --name some-postgres \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=mysecretpassword \
-p 5432:5432 \
--network=iroha-network \
-d postgres:9.5 \
-c 'max_prepared_transactions=100'

-->Unable to find image 'postgres:9.5' locally
9.5: Pulling from library/postgres
0a4690c5d889: Pull complete 
723861590717: Pull complete 
db019468bdf4: Pull complete 
91cb81a60371: Pull complete 
a2a4ab07588d: Pull complete 
a7ccdc2a5f31: Pull complete 
93687df2bb93: Pull complete 
f00839cf3313: Pull complete 
18063c24e1d4: Pull complete 
7442baacf4a3: Pull complete 
8603551719b1: Pull complete 
f6d9c85570f1: Pull complete 
3ea2315d3fa1: Pull complete 
8d9fba94f577: Pull complete 
Digest: sha256:e49a97e5ac31cc778beea3bdefc104ff2002eb0e8d31cf5bb2b8ab912c23c2a8
Status: Downloaded newer image for postgres:9.5
3db6519a1e6b1e5fe2e9089a9404edd238b2a09cca280484d3731fbcff54da57


### Creating Blockstore

==>Before we run Iroha container, we may create a persistent volume to store files, storing blocks for the chain. It is done via the following command:

$ docker volume create blockstore

-->blockstore


### Preparing the configuration files

==>Now we need to configure our Iroha network. This includes creating a configuration file, generating keypairs for a users, writing a list of peers and creating a genesis block.


$ git clone -b master https://github.com/hyperledger/iroha --depth=1

### Starting Iroha Container
==>We are almost ready to launch our Iroha container. You just need to know the path to configuration files (from the step above).
Let’s start Iroha node in Docker container with the following command:

$ docker run --name iroha \
-d \
-p 50051:50051 \
-v $(pwd)/iroha/example:/opt/iroha_data \
-v blockstore:/tmp/block_store \
--network=iroha-network \
-e KEY='node0' \
hyperledger/iroha:latest

-->Unable to find image 'hyperledger/iroha:latest' locally
latest: Pulling from hyperledger/iroha
35b42117c431: Pull complete 
ad9c569a8d98: Pull complete 
293b44f45162: Pull complete 
0c175077525d: Pull complete 
099bef408ef2: Pull complete 
b303770cedcf: Pull complete 
e9a544750748: Pull complete 
9bc16eeea604: Pull complete 
294b79a4ac02: Pull complete 
Digest: sha256:42d571aabcdac32702232852ba9ccdfe6368448c4919fbcc2a18552d608e45ff
Status: Downloaded newer image for hyperledger/iroha:latest
04e1e1b26043bd52fa42afc4a6875eb09786b7714f9c8e284bad9ce8d0b3b2d2



==>If you started the node successfully you would see the container id in the same console where you started the container.
Let’s look in details what this command does:

 ==>>
 
  docker run --name iroha \ creates a container iroha
   -d \ runs container in the background
   -p 50051:50051 \ exposes a port for communication with a client (we will use this later)
    -v YOUR_PATH_TO_CONF_FILES:/opt/iroha_data \ is how we pass our configuration files to docker container. The example directory is indicated in the code block above.
   -v blockstore:/tmp/block_store \ adds persistent block storage (Docker volume) to a container, so that the blocks aren’t lost after we stop the container
   --network=iroha-network \ adds our container to previously created iroha-network for communication with PostgreSQL server
  -e KEY='node0' \ - here please indicate a key name that will identify the node allowing it to confirm operations. The keys should be placed in the #    directory with configuration files mentioned above.
   hyperledger/iroha:latest is a reference to the image pointing to the latest release

==>You can check the logs by running 

$ docker logs iroha.


### CLI guide: sending your first transactions and queries



==>You can interact with Iroha using various ways. You can use our client libraries to write code in various programming languages (e.g. Java, Python, Javascript, Swift) which communicates with Iroha. Alternatively, you can use iroha-cli – our command-line tool for interacting with Iroha. As a part of this guide, let’s get familiar with iroha-cli



==>Open a new terminal (note that Iroha container and irohad should be up and running) and attach to an iroha docker container:

$ docker exec -it iroha /bin/bash



==>Now you are in the interactive shell of Iroha’s container again. We need to launch iroha-cli and pass an account name of the desired user. In our example, the account admin is already created in the test domain. Let’s use this account to work with Iroha.

$ iroha-cli -account_name admin@test


### Creating the First Transaction

==>You can see the interface of iroha-cli now. Let’s create a new asset, add some asset to the admin account and transfer it to other account. To achieve this, please choose option 1. New transaction (tx) by writing tx or 1 to a console.

Now you can see a list of available commands. Let’s try creating a new asset. Select 14. Create Asset (crt_ast). Now enter a name for your asset, for example coolcoin. Next, enter a Domain ID. In our example we already have a domain test, so let’s use it. Then we need to enter an asset precision – the amount of numbers in a fractional part. Let’s set precision to 2.

Congratulations, you have created your first command and added it to a transaction! You can either send it to Iroha or add some more commands 1. Add one more command to the transaction (add). Let’s add more commands, so we can do everything in one shot. Type add.

Now try adding some coolcoins to our account. Select 16. Add Asset Quantity (add_ast_qty), enter asset ID – coolcoin#test, integer part and coolcoin#test, integer part and precision. For example, to add 200.50 precision. For example, to add 200.50 coolcoins, we need to enter integer coolcoins, we need to enter integer part as 20050 and precision as part as 20050 and precision as 2, so it becomes 200.50.



==>Let’s transfer 100.50 coolcoins from admin@test to test@test by adding one more command and choosing 5. Transfer Assets (tran_ast). Enter Source Account and Destination Account, in our case admin@test and test@test, Asset ID (coolcoin#test), integer part and precision (10050 and 2 accordingly).
Now we need to send our transaction to Iroha peer (2. Send to Iroha peer (send)). Enter peer address (in our case localhost) and port (50051). Now your transaction is submitted and you can see your transaction hash. You can use it to check transaction’s status.
Go back to a terminal where irohad is running. You can see logs of your transaction.
Yay! You have submitted your first transaction to Iroha.


### Creating the First Query

==>Now let’s check if coolcoins were successfully transferred from admin@test to test@test. Choose 2. New query (qry). 8. Get Account's Assets (get_acc_ast)can help you to check if test@test now has coolcoin. Form a query in a similar way you did with commands you did with commands and 1. Send to Iroha peer (send). Now you can see information about how many coolcoin does test@test have. It will look similar to this:

[2018-03-21 12:33:23.179275525][th:36][info] QueryResponseHandler [Account Assets]
[2018-03-21 12:33:23.179329199][th:36][info] QueryResponseHandler -Account Id:- test@test
[2018-03-21 12:33:23.179338394][th:36][info] QueryResponseHandler -Asset Id- coolcoin#test
[2018-03-21 12:33:23.179387969][th:36][info] QueryResponseHandler -Balance- 100.50

Isn’t that awesome? You have submitted your first query to Iroha and got a response!




### Being Badass

==>Let’s try being badass and cheat Iroha. For example, let’s transfer more coolcoins than admin@test has. Try to transfer 100000.00 coolcoins from admin@test to test@test. Again, proceed to 1. New transaction (tx), 5. Transfer Assets (tran_ast), enter Source Account and Destination Account, in our case admin@test and test@test, Asset ID (coolcoin#test), integer part and precision (10000000 and 2 accordingly). Send a transaction to Iroha peer as you did before. Well, it says

[2018-03-21 12:58:40.791297963][th:520][info] TransactionResponseHandler Transaction successfully sent
Congratulation, your transaction was accepted for processing.
Its hash is fc1c23f2de1b6fccbfe1166805e31697118b57d7bb5b1f583f2d96e78f60c241

==>Your transaction was accepted for processing. Does it mean that we had successfully cheated Iroha? Let’s try to see transaction’s status. Choose 3. New transaction status request (st) and enter transaction’s hash which you can get in the console after the previous command. Let’s send it to Iroha. It replies with:
Transaction has not passed stateful validation.
Apparently no. Our transaction was not accepted because it did not pass stateful validation and coolcoins were not transferred. You can check the status of admin@test and test@test with queries to be sure (like we did earlier).
