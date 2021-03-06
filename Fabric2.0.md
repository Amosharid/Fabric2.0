# [](#tutorial-1-hyperledger-fabric-v20---create-a-development-business-network-on-ubuntu-1604-lts)Tutorial 1. Hyperledger Fabric V2.0 - Create a Development Business Network on Ubuntu 16.04 LTS

This article is a tutorial that guides you on how to create a Hyperledger Fabric v.2.0 business network on Ubuntu 16.04 LTS using the development tools that are found in the Hyperledger Fabric repository.

We will go through the process of setting up the Hyperledger Fabric prerequisites and later on we define and start the Hyperledger Fabric blockchain network between three organizations.

 - [Tutorial 1: Hyperledger Fabric V2.0 - Create a Development Business Network on Ubuntu 16.04 LTS](#tutorial-1-hyperledger-fabric-v20---create-a-development-business-network-on-ubuntu-1604-lts)
	 - [Recommended Reading](#recommended-reading)
-   [Setup Your Environment](#setup-your-environment)
    -   [Utility Packages](#utility-packages)
    -   [Docker](#docker)
    -   [Docker Compose](#docker-compose)
    -   [Go language](#go-language)
    -   [NodeJS](#nodejs)
-   [Retrieve Artifacts from Hyperledger Fabric Repsitories](#retrieve-artifacts-from-hyperledger-fabric-repsitories)
-   [Create Hyperledger Fabric Business Network](#create-hyperledger-fabric-business-network)
    -   [Create Project Directory](#create-project-directory)
    -   [Generate Peer and Orderer Certificates](#generate-peer-and-orderer-certificates)
    -   [Create channel.tx and the Genesis Block Using the configtxgen Tool](#create-channeltx-and-the-genesis-block-using-the-configtxgen-tool)
        -   [Creating/Modifying  `configtx.yaml`](#creatingmodifying-configtxyaml)
        -   [Executing the configtxgen Tool](#executing-the-configtxgen-tool)
-   [Start the Hyperledger Fabric blockchain network](#start-the-hyperledger-fabric-blockchain-network)
    -   [Modifying the docker-compose yaml Files](#modifying-the-docker-compose-yaml-files)
    -   [Before running docker containers](#before-running-docker-containers)
    -   [Start the docker Containers](#start-the-docker-containers)
    -   [The channel](#the-channel)
        -   [Create the channel](#create-the-channel)
        -   [Join channel](#join-channel)
        -   [Update anchor peers](#update-anchor-peers)
    -   [Prepare chaincode](#prepare-chaincode)
    -   [Install chaincode](#install-chaincode)
    -   [Instantiate chaincode](#instantiate-chaincode)
    -   [Execute tutorial scenario](#execute-tutorial-scenario)
        -   [Query for initial state of the ledger](#query-for-initial-state-of-the-ledger)
        -   [Invoke chaincode](#invoke-chaincode)
        -   [Query for the new values](#query-for-the-new-values)
-   [Summary](#summary)
-   [What is Next?](#what-is-next)
-   [References?](#references)


## [](#recommended-reading)Recommended Reading

Before you start this tutorial, you may want to get familiar with the basic concepts of Hyperledger Fabric. Official Hyperledger Fabric documentation provides a comprehensive source of information related to Hyperledger Fabric configuration, modes of operation and prerequisites. We recommend to read the following articles and use them as the reference when going through this tutorial.

-   Hyperledger Fabric Glossary -  [http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html](http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html)
-   Hyperledger Fabric Model -  [http://hyperledger-fabric.readthedocs.io/en/latest/fabric_model.html](http://hyperledger-fabric.readthedocs.io/en/latest/fabric_model.html)
-   Hyperledger Fabric Prerequisities -  [http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html](http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)
-   Hyperledger Fabric Samples -  [https://github.com/hyperledger/fabric-samples](https://github.com/hyperledger/fabric-samples)
-   Building Your First Network -  [http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)


## [](setup-your-environment)Setup Your Environment

### [](#utility-packages)Utility Packages

### [](#docker)Docker

Docker is a tool for deploying, executing, and managing containers. Hyperledger Fabric is by default packaged as a set of Docker images and it is ready to be run as Docker container.

To install the Docker, we can go to  [Docker website](https://docs.docker.com/install/linux/docker-ce/ubuntu/):

**SETUP THE REPOSITORY**

 1. Update the `apt` package index:
	```
	$ sudo apt-get update
	```
 2. Install packages to allow `apt` to use a repository over HTTPS:
	```
	$ sudo apt-get install \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    gnupg-agent \
	    software-properties-common
	```
 3. Add Docker???s official GPG key:
	```
	$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	```
	Verify that you now have the key with the fingerprint `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`, by searching for the last 8 characters of the fingerprint.
	```
	$ sudo apt-key fingerprint 0EBFCD88


	pub   rsa4096 2017-02-22 [SCEA]
	      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
	uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
	sub   rsa4096 2017-02-22 [S]
	```
 4. Use the following command to set up the  **stable**  repository.
	 ```
	 $ sudo add-apt-repository \
	    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	    $(lsb_release -cs) \
	    stable"
	 ```

**INSTALL DOCKER ENGINE - COMMUNITY**
1.  Update the  `apt`  package index.
    ```
    $ sudo apt-get update
    ```
2.  Install the  _latest version_  of Docker Engine - Community and containerd, or go to the next step to install a specific version:
    ```
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
	 Adding your user to the ???docker??? group
	```
	$ sudo usermod -aG docker $USER
	```
	**After adding docker to current user group, please logout and login again.**

### [](#docker-compose)Docker Compose
Docker Compose is a tool for defining and running multi-container Docker applications. This is the case of the Hyperledger Fabric default setup.

Docker Compose is typically installed as a part of your Docker installation. If not, it is necessary to install it separately. Run  `docker-compose --version`  command to find out if Docker Compose is present on your system.

To install Docker Compose on your Ubuntu 16.04 LTS system, please follow the instructions describe below or  [follow the instruction how to install the docker compose in docker website](https://docs.docker.com/compose/install/#install-compose):

 1. Run this command to download the current stable release of Docker Compose:
	```
	$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
	```
>
2. Apply executable permissions to the binary:
	```
	$ sudo chmod +x /usr/local/bin/docker-compose
	```
	Check the docker-compose version:
	```
	$ docker-compose --version
	docker-compose version 1.25.4, build 8d51620a
	```
### [](#go-language)Go language

In this tutorial, we use the Go language as the basis of chaincode in Hyperledger Fabric. For Hyperledger Fabric, the Go language version is 1.13.x is required.

To install the Go language in the system, please follow the below instructions:

 1. Download the golang file.
	```
	$ wget https://dl.google.com/go/go1.13.8.linux-amd64.tar.gz
	```
 2. Extract it in home directory.
	```
	$ tar -xvf go1.13.8.linux-amd64.tar.gz
	```
 3. Create a `gopath` directory in **HOME directory**
	```
	$ mkdir $HOME/gopath
	```
	Then we need to set the **Environment Variables** for our Golang in `~/.bashrc` file such `GOPATH`, `GOROOT`, or `GOBIN`.
	```
	$ vim ~/.bashrc
	```
	Inside `~/.bashrc` file set the following commands:
	```
	# set GOPATH and GOROOT
	export GOPATH=$HOME/gopath
	export GOROOT=$HOME/go
	export GOBIN=$HOME/go/bin
	export PATH=$PATH:/$GOROOT/bin
	```
	Then run the command `source ~/.bashrc` to build the environment variables in the **.bashrc** file

### [](nodejs)NodeJS
If you will be developing applications for Hyperledger Fabric leveraging the Hyperledger Fabric SDK for Node.js, version 8 is supported from 8.9.4 and higher. Node.js version 10 is supported from 10.15.3 and higher.
Follow the instructions below to download and install **NodeJS**.
```
$ curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -

$ sudo apt-get install -y nodejs
```

## [](retrieve-artifacts-from-hyperledger-fabric-repositories)Retrieve Artifacts from Hyperledger Fabric Repositories
To download and install  **Hyperledger Fabric binaries**  specific to your platform. This include downloading of  `cryptogen`,  `configtxgen`,  `configtxlator`,  `fabric-ca-client`,  `get-docker-images.sh`,  `orderer`  and  `peer`  tools and placing them into  `bin`  directory in the directory of your choice. Besides, the script will download Hyperlerdger Docker images into your local Docker registry.

Execute the following command to download Hyperledger Fabric binaries and Docker images:

````
curl -sSL https://bit.ly/2ysbOFE | bash -s -- <fabric_version> <fabric-ca_version> <thirdparty_version>

curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.0.1 1.4.6 0.4.18
````


**RUN**
```
$ curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.0.1 1.4.6 0.4.18
```

After running the command above, it will show like this:
```
Clone hyperledger/fabric-samples repo

===> Cloning hyperledger/fabric-samples repo and checkout v2.0.1
Cloning into 'fabric-samples'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4820 (delta 0), reused 0 (delta 0), pack-reused 4816
Receiving objects: 100% (4820/4820), 1.71 MiB | 456.00 KiB/s, done.
Resolving deltas: 100% (2428/2428), done.
Checking connectivity... done.
error: pathspec 'v2.0.1' did not match any file(s) known to git.

Pull Hyperledger Fabric binaries

===> Downloading version 2.0.1 platform specific fabric binaries
===> Downloading:  https://github.com/hyperledger/fabric/releases/download/v2.0.1/hyperledger-fabric-linux-amd64-2.0.1.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   633  100   633    0     0   1240      0 --:--:-- --:--:-- --:--:--  1241
100 72.7M  100 72.7M    0     0   979k      0  0:01:16  0:01:16 --:--:-- 5113k
==> Done.
===> Downloading version 1.4.6 platform specific fabric-ca-client binary
===> Downloading:  https://github.com/hyperledger/fabric-ca/releases/download/v1.4.6/hyperledger-fabric-ca-linux-amd64-1.4.6.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   636  100   636    0     0   1363      0 --:--:-- --:--:-- --:--:--  1364
100 19.7M  100 19.7M    0     0   811k      0  0:00:24  0:00:24 --:--:-- 2225k
==> Done.

Pull Hyperledger Fabric docker images
.
.
.
===> List out hyperledger docker images
hyperledger/fabric-javaenv     2.0                 bb3837bf990f        12 days ago         505MB
hyperledger/fabric-javaenv     2.0.1               bb3837bf990f        12 days ago         505MB
hyperledger/fabric-javaenv     latest              bb3837bf990f        12 days ago         505MB
hyperledger/fabric-tools       2.0                 5c9a03790913        2 weeks ago         512MB
hyperledger/fabric-tools       2.0.1               5c9a03790913        2 weeks ago         512MB
hyperledger/fabric-tools       latest              5c9a03790913        2 weeks ago         512MB
hyperledger/fabric-peer        2.0                 5c7e5946f3dc        2 weeks ago         57.2MB
hyperledger/fabric-peer        2.0.1               5c7e5946f3dc        2 weeks ago         57.2MB
hyperledger/fabric-peer        latest              5c7e5946f3dc        2 weeks ago         57.2MB
hyperledger/fabric-orderer     2.0                 92bd220edcdd        2 weeks ago         39.7MB
hyperledger/fabric-orderer     2.0.1               92bd220edcdd        2 weeks ago         39.7MB
hyperledger/fabric-orderer     latest              92bd220edcdd        2 weeks ago         39.7MB
hyperledger/fabric-ccenv       2.0                 800087268d9b        2 weeks ago         529MB
hyperledger/fabric-ccenv       2.0.1               800087268d9b        2 weeks ago         529MB
hyperledger/fabric-ccenv       latest              800087268d9b        2 weeks ago         529MB
hyperledger/fabric-baseos      2.0                 74ff718f6f67        2 weeks ago         6.9MB
hyperledger/fabric-baseos      2.0.1               74ff718f6f67        2 weeks ago         6.9MB
hyperledger/fabric-baseos      latest              74ff718f6f67        2 weeks ago         6.9MB
hyperledger/fabric-ca          1.4                 3b96a893c1e4        3 weeks ago         150MB
hyperledger/fabric-ca          1.4.6               3b96a893c1e4        3 weeks ago         150MB
hyperledger/fabric-ca          latest              3b96a893c1e4        3 weeks ago         150MB
hyperledger/fabric-zookeeper   0.4                 ede9389347db        4 months ago        276MB
hyperledger/fabric-zookeeper   0.4.18              ede9389347db        4 months ago        276MB
hyperledger/fabric-zookeeper   latest              ede9389347db        4 months ago        276MB
hyperledger/fabric-kafka       0.4                 caaae0474ef2        4 months ago        270MB
hyperledger/fabric-kafka       0.4.18              caaae0474ef2        4 months ago        270MB
hyperledger/fabric-kafka       latest              caaae0474ef2        4 months ago        270MB
hyperledger/fabric-couchdb     0.4                 d369d4eaa0fd        4 months ago        261MB
hyperledger/fabric-couchdb     0.4.18              d369d4eaa0fd        4 months ago        261MB
hyperledger/fabric-couchdb     latest              d369d4eaa0fd        4 months ago        261MB
```
Then we can see the binary files and shell script file in the bin `~/fabric-samples/bin` directory.

```
blockchain@blockchain-make-doc:~/fabric-samples/bin$ ll
total 206892
drwxr-xr-x  2 blockchain blockchain     4096 Feb 25 22:56 ./
drwxrwxr-x 17 blockchain blockchain     4096 Mar 18 08:07 ../
-rwxr-xr-x  1 blockchain blockchain 20999000 Feb 26 22:05 configtxgen*
-rwxr-xr-x  1 blockchain blockchain 17448272 Feb 26 22:05 configtxlator*
-rwxr-xr-x  1 blockchain blockchain 13344644 Feb 26 22:05 cryptogen*
-rwxr-xr-x  1 blockchain blockchain 19116716 Feb 26 22:05 discover*
-rwxr-xr-x  1 blockchain blockchain 20702242 Feb 25 22:56 fabric-ca-client*
-rwxr-xr-x  1 blockchain blockchain 24603190 Feb 25 22:56 fabric-ca-server*
-rwxr-xr-x  1 blockchain blockchain 12352844 Feb 26 22:05 idemixgen*
-rwxr-xr-x  1 blockchain blockchain 32764776 Feb 26 22:05 orderer*
-rwxr-xr-x  1 blockchain blockchain 50501032 Feb 26 22:05 peer*
```
For your convenience, you can add this directory with Hyperledger Fabric binaries to your **PATH** environment variables `~/.bashrc` file.
```
$ vim ~/.bashrc
```
Inside `~/.bashrc` file set the following commands:
```
# set Hyperledger Fabric
export PATH=$PATH:$HOME/fabric-samples/bin
```
And then run the command `source ~/.bashrc` to rebuild the export **PATH** environment variables.
## Create Hyperledger Fabric Business Network
This tutorial describes creating a business network and deploying chaincode using the Hyperledger Fabric v1.2.0 code base (found here:  [https://github.com/hyperledger/fabric.git](https://github.com/hyperledger/fabric.git)). This tutorial exploits the Hyperledger Fabric v1.2.0 configuration toolset to create the business network consisting of the following items:

-   Three orderer nodes (HRD Center Organization)
-   OrdererType is Kafka (4 Kafka nodes and 3 zookeeper ensembles)
-   Two peerOrganizations (Coocon and Webcash Company)
    -   Coocon company has 2 peers
    -   Webcash company has 2 peers
-   Authentication Data Chaincode / Smart Contracts

At the end of this tutorial, you will have constructed a running instance of Hyperledger Fabric business network, as well as installed, instantiated, and executed chaincode.

For our tutorial we will make use of the Fabric deployment model based on Docker conatiners. Our network will run Docker containers running on our target Ubuntu 16.04 LTS (localhost).

### [](#create-project-directory)Create Project Directory
```
# Project path depends on your requirement
mkdir -p $GOPATH/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks
```
### [](#generate-peer-and-orderer-certificates)Generate Peer and Orderer Certificates

Nodes (such as peers and orderers) are permitted to access business networks using a membership service provider, which is typically in the form of a certificate authority. In this example, we use the development tool named  `cryptogen`  to generate the required certificates. We use a local MSP to store the certs, which are essentially a local directory structure, for each peer and orderer. In production environments, you can exploit the  `fabric ca`  toolset introducing full-featured certificate authorities to generate the certificates.

`crytogen`  tool uses a  `yaml`  configuration file as its configuration - based on the content of this file, the required certificates are generated. We are going to create  `crypto-config.yaml`  file for our configuration. We are going to define two organizations of peers and three orderer organization.

Reference: **Hyperledger Fabric Samples**  [crypto-config.yaml](https://github.com/hyperledger/fabric-samples/blob/master/first-network/crypto-config.yaml)
Create `crypto-config.yaml` file:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ vim crypto-config.yaml
```
Here is the listing of our  `crypto-config.yaml`  configuration file (for the purpose of simplicity, all comments are removed from this listing):
```
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer === KSHRD CENTER
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: kshrd.com.kh
    Specs:
      - Hostname: orderer1
      - Hostname: orderer2
      - Hostname: orderer3
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Coocon Organization
  # ---------------------------------------------------------------------------
  - Name: Coocon
    Domain: coocon.kshrd.com.kh
    Template:
      Count: 2
    Users:
      Count: 1

  # ---------------------------------------------------------------------------
  # Webcash Organization
  # ---------------------------------------------------------------------------
  - Name: Webcash
    Domain: webcash.kshrd.com.kh
    Template:
      Count: 2
    Users:
      Count: 1
```
To generate certificates, run the following command:
```
$ cryptogen generate --config=./crypto-config.yaml
```
After running of crytogen tool you should see the following output in console:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ cryptogen generate --config=./crypto-config.yaml
coocon.kshrd.com.kh
webcash.kshrd.com.kh
```
In addition, the new `crypto-config` directory has been created and contains various certificates and keys for **orderers** and **peers**.

First, let's check content under  `orderOrganizations`:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/crypto-config/ordererOrganizations$ tree
.
????????? kshrd.com.kh
    ????????? ca
    ???   ????????? ca.kshrd.com.kh-cert.pem
    ???   ????????? priv_sk
    ????????? msp
    ???   ????????? admincerts
    ???   ???   ????????? Admin@kshrd.com.kh-cert.pem
    ???   ????????? cacerts
    ???   ???   ????????? ca.kshrd.com.kh-cert.pem
    ???   ????????? tlscacerts
    ???       ????????? tlsca.kshrd.com.kh-cert.pem
    ????????? orderers
    ???   ????????? orderer1.kshrd.com.kh
    ???   ???   ????????? msp
    ???   ???   ???   ????????? admincerts
    ???   ???   ???   ???   ????????? Admin@kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? cacerts
    ???   ???   ???   ???   ????????? ca.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? keystore
    ???   ???   ???   ???   ????????? priv_sk
    ???   ???   ???   ????????? signcerts
    ???   ???   ???   ???   ????????? orderer1.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? tlscacerts
    ???   ???   ???       ????????? tlsca.kshrd.com.kh-cert.pem
    ???   ???   ????????? tls
    ???   ???       ????????? ca.crt
    ???   ???       ????????? server.crt
    ???   ???       ????????? server.key
    ???   ????????? orderer2.kshrd.com.kh
    ???   ???   ????????? msp
    ???   ???   ???   ????????? admincerts
    ???   ???   ???   ???   ????????? Admin@kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? cacerts
    ???   ???   ???   ???   ????????? ca.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? keystore
    ???   ???   ???   ???   ????????? priv_sk
    ???   ???   ???   ????????? signcerts
    ???   ???   ???   ???   ????????? orderer2.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? tlscacerts
    ???   ???   ???       ????????? tlsca.kshrd.com.kh-cert.pem
    ???   ???   ????????? tls
    ???   ???       ????????? ca.crt
    ???   ???       ????????? server.crt
    ???   ???       ????????? server.key
    ???   ????????? orderer3.kshrd.com.kh
    ???       ????????? msp
    ???       ???   ????????? admincerts
    ???       ???   ???   ????????? Admin@kshrd.com.kh-cert.pem
    ???       ???   ????????? cacerts
    ???       ???   ???   ????????? ca.kshrd.com.kh-cert.pem
    ???       ???   ????????? keystore
    ???       ???   ???   ????????? priv_sk
    ???       ???   ????????? signcerts
    ???       ???   ???   ????????? orderer3.kshrd.com.kh-cert.pem
    ???       ???   ????????? tlscacerts
    ???       ???       ????????? tlsca.kshrd.com.kh-cert.pem
    ???       ????????? tls
    ???           ????????? ca.crt
    ???           ????????? server.crt
    ???           ????????? server.key
    ????????? tlsca
    ???   ????????? priv_sk
    ???   ????????? tlsca.kshrd.com.kh-cert.pem
    ????????? users
        ????????? Admin@kshrd.com.kh
            ????????? msp
            ???   ????????? admincerts
            ???   ???   ????????? Admin@kshrd.com.kh-cert.pem
            ???   ????????? cacerts
            ???   ???   ????????? ca.kshrd.com.kh-cert.pem
            ???   ????????? keystore
            ???   ???   ????????? priv_sk
            ???   ????????? signcerts
            ???   ???   ????????? Admin@kshrd.com.kh-cert.pem
            ???   ????????? tlscacerts
            ???       ????????? tlsca.kshrd.com.kh-cert.pem
            ????????? tls
                ????????? ca.crt
                ????????? client.crt
                ????????? client.key

41 directories, 39 files
```
Second, let's check content under `peerOrganizations`:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/crypto-config/peerOrganizations$ tree
.
????????? coocon.kshrd.com.kh
???   ????????? ca
???   ???   ????????? ca.coocon.kshrd.com.kh-cert.pem
???   ???   ????????? priv_sk
???   ????????? msp
???   ???   ????????? admincerts
???   ???   ???   ????????? Admin@coocon.kshrd.com.kh-cert.pem
???   ???   ????????? cacerts
???   ???   ???   ????????? ca.coocon.kshrd.com.kh-cert.pem
???   ???   ????????? tlscacerts
???   ???       ????????? tlsca.coocon.kshrd.com.kh-cert.pem
???   ????????? peers
???   ???   ????????? peer0.coocon.kshrd.com.kh
???   ???   ???   ????????? msp
???   ???   ???   ???   ????????? admincerts
???   ???   ???   ???   ???   ????????? Admin@coocon.kshrd.com.kh-cert.pem
???   ???   ???   ???   ????????? cacerts
???   ???   ???   ???   ???   ????????? ca.coocon.kshrd.com.kh-cert.pem
???   ???   ???   ???   ????????? keystore
???   ???   ???   ???   ???   ????????? priv_sk
???   ???   ???   ???   ????????? signcerts
???   ???   ???   ???   ???   ????????? peer0.coocon.kshrd.com.kh-cert.pem
???   ???   ???   ???   ????????? tlscacerts
???   ???   ???   ???       ????????? tlsca.coocon.kshrd.com.kh-cert.pem
???   ???   ???   ????????? tls
???   ???   ???       ????????? ca.crt
???   ???   ???       ????????? server.crt
???   ???   ???       ????????? server.key
???   ???   ????????? peer1.coocon.kshrd.com.kh
???   ???       ????????? msp
???   ???       ???   ????????? admincerts
???   ???       ???   ???   ????????? Admin@coocon.kshrd.com.kh-cert.pem
???   ???       ???   ????????? cacerts
???   ???       ???   ???   ????????? ca.coocon.kshrd.com.kh-cert.pem
???   ???       ???   ????????? keystore
???   ???       ???   ???   ????????? priv_sk
???   ???       ???   ????????? signcerts
???   ???       ???   ???   ????????? peer1.coocon.kshrd.com.kh-cert.pem
???   ???       ???   ????????? tlscacerts
???   ???       ???       ????????? tlsca.coocon.kshrd.com.kh-cert.pem
???   ???       ????????? tls
???   ???           ????????? ca.crt
???   ???           ????????? server.crt
???   ???           ????????? server.key
???   ????????? tlsca
???   ???   ????????? priv_sk
???   ???   ????????? tlsca.coocon.kshrd.com.kh-cert.pem
???   ????????? users
???       ????????? Admin@coocon.kshrd.com.kh
???       ???   ????????? msp
???       ???   ???   ????????? admincerts
???       ???   ???   ???   ????????? Admin@coocon.kshrd.com.kh-cert.pem
???       ???   ???   ????????? cacerts
???       ???   ???   ???   ????????? ca.coocon.kshrd.com.kh-cert.pem
???       ???   ???   ????????? keystore
???       ???   ???   ???   ????????? priv_sk
???       ???   ???   ????????? signcerts
???       ???   ???   ???   ????????? Admin@coocon.kshrd.com.kh-cert.pem
???       ???   ???   ????????? tlscacerts
???       ???   ???       ????????? tlsca.coocon.kshrd.com.kh-cert.pem
???       ???   ????????? tls
???       ???       ????????? ca.crt
???       ???       ????????? client.crt
???       ???       ????????? client.key
???       ????????? User1@coocon.kshrd.com.kh
???           ????????? msp
???           ???   ????????? admincerts
???           ???   ???   ????????? User1@coocon.kshrd.com.kh-cert.pem
???           ???   ????????? cacerts
???           ???   ???   ????????? ca.coocon.kshrd.com.kh-cert.pem
???           ???   ????????? keystore
???           ???   ???   ????????? priv_sk
???           ???   ????????? signcerts
???           ???   ???   ????????? User1@coocon.kshrd.com.kh-cert.pem
???           ???   ????????? tlscacerts
???           ???       ????????? tlsca.coocon.kshrd.com.kh-cert.pem
???           ????????? tls
???               ????????? ca.crt
???               ????????? client.crt
???               ????????? client.key
????????? webcash.kshrd.com.kh
    ????????? ca
    ???   ????????? ca.webcash.kshrd.com.kh-cert.pem
    ???   ????????? priv_sk
    ????????? msp
    ???   ????????? admincerts
    ???   ???   ????????? Admin@webcash.kshrd.com.kh-cert.pem
    ???   ????????? cacerts
    ???   ???   ????????? ca.webcash.kshrd.com.kh-cert.pem
    ???   ????????? tlscacerts
    ???       ????????? tlsca.webcash.kshrd.com.kh-cert.pem
    ????????? peers
    ???   ????????? peer0.webcash.kshrd.com.kh
    ???   ???   ????????? msp
    ???   ???   ???   ????????? admincerts
    ???   ???   ???   ???   ????????? Admin@webcash.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? cacerts
    ???   ???   ???   ???   ????????? ca.webcash.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? keystore
    ???   ???   ???   ???   ????????? priv_sk
    ???   ???   ???   ????????? signcerts
    ???   ???   ???   ???   ????????? peer0.webcash.kshrd.com.kh-cert.pem
    ???   ???   ???   ????????? tlscacerts
    ???   ???   ???       ????????? tlsca.webcash.kshrd.com.kh-cert.pem
    ???   ???   ????????? tls
    ???   ???       ????????? ca.crt
    ???   ???       ????????? server.crt
    ???   ???       ????????? server.key
    ???   ????????? peer1.webcash.kshrd.com.kh
    ???       ????????? msp
    ???       ???   ????????? admincerts
    ???       ???   ???   ????????? Admin@webcash.kshrd.com.kh-cert.pem
    ???       ???   ????????? cacerts
    ???       ???   ???   ????????? ca.webcash.kshrd.com.kh-cert.pem
    ???       ???   ????????? keystore
    ???       ???   ???   ????????? priv_sk
    ???       ???   ????????? signcerts
    ???       ???   ???   ????????? peer1.webcash.kshrd.com.kh-cert.pem
    ???       ???   ????????? tlscacerts
    ???       ???       ????????? tlsca.webcash.kshrd.com.kh-cert.pem
    ???       ????????? tls
    ???           ????????? ca.crt
    ???           ????????? server.crt
    ???           ????????? server.key
    ????????? tlsca
    ???   ????????? priv_sk
    ???   ????????? tlsca.webcash.kshrd.com.kh-cert.pem
    ????????? users
        ????????? Admin@webcash.kshrd.com.kh
        ???   ????????? msp
        ???   ???   ????????? admincerts
        ???   ???   ???   ????????? Admin@webcash.kshrd.com.kh-cert.pem
        ???   ???   ????????? cacerts
        ???   ???   ???   ????????? ca.webcash.kshrd.com.kh-cert.pem
        ???   ???   ????????? keystore
        ???   ???   ???   ????????? priv_sk
        ???   ???   ????????? signcerts
        ???   ???   ???   ????????? Admin@webcash.kshrd.com.kh-cert.pem
        ???   ???   ????????? tlscacerts
        ???   ???       ????????? tlsca.webcash.kshrd.com.kh-cert.pem
        ???   ????????? tls
        ???       ????????? ca.crt
        ???       ????????? client.crt
        ???       ????????? client.key
        ????????? User1@webcash.kshrd.com.kh
            ????????? msp
            ???   ????????? admincerts
            ???   ???   ????????? User1@webcash.kshrd.com.kh-cert.pem
            ???   ????????? cacerts
            ???   ???   ????????? ca.webcash.kshrd.com.kh-cert.pem
            ???   ????????? keystore
            ???   ???   ????????? priv_sk
            ???   ????????? signcerts
            ???   ???   ????????? User1@webcash.kshrd.com.kh-cert.pem
            ???   ????????? tlscacerts
            ???       ????????? tlsca.webcash.kshrd.com.kh-cert.pem
            ????????? tls
                ????????? ca.crt
                ????????? client.crt
                ????????? client.key

82 directories, 78 files
```
### [](#create-channeltx-and-the-genesis-block-using-the-configtxgen-tool)Create channel.tx and the Genesis Block Using the configtxgen Tool

Now we generated the certificates and keys, we can now configure the  `configtx.yaml`  file. This yaml file serves as input to the  `configtxgen`  tool and generates the following important artifacts such as:

-   **channel.tx**

The channel creation transaction. This transaction lets you create the Hyperledger Fabric channel. The channel is the location where the ledger exists and the mechanism that lets peers join business networks.

-   **Genesis Block**

The Genesis block is the first block in our blockchain. It is used to bootstrap the ordering service and holds the channel configuration.

-   **Anchor peers transactions**

The anchor peer transactions specify each Org's Anchor Peer on this channel for communicating from one organization to other one.

#### [](#creatingmodifying-configtxyaml)Creating/Modifying  `configtx.yaml`

We recommend that you can start with the existing  `configtx.yaml`  in the  `fabric-sample`  project in the github as it contains a template that requires only minor modifications for our needs.

The  `configtx.yaml`  file is broken into several sections:

`Profile`: Profiles describe the organization structure of your network.

`Organization`: The details regarding individual organizations.

`Orderer`: The details regarding the Orderer parameters.

`Application`: Application defaults - not needed for this tutorial.

Create `configtx.yaml` file:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ vim configtx.yaml
```
Here is the listing of our  `configtx.yaml`  configuration file (for the purpose of simplicity, all comments are removed from this listing):

Reference: **Hyperledger Fabric Samples**  [configtx.yaml](https://github.com/hyperledger/fabric/blob/release-2.0/sampleconfig/configtx.yaml)
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
---
################################################################################
#
#   ORGANIZATIONS
#
################################################################################
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/kshrd.com.kh/msp
        Policies: &OrdererOrgPolicies
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
    - &Coocon
        Name: CooconMSP
        ID: CooconMSP
        MSPDir: crypto-config/peerOrganizations/coocon.kshrd.com.kh/msp
        AnchorPeers:
            - Host: peer0.coocon.kshrd.com.kh
              Port: 7051
        Policies: &CooconPolicies
            Readers:
                Type: Signature
                Rule: "OR('CooconMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('CooconMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('CooconMSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('CooconMSP.member')"

    - &Webcash
        Name: WebcashMSP
        ID: WebcashMSP
        MSPDir: crypto-config/peerOrganizations/webcash.kshrd.com.kh/msp
        AnchorPeers:
            - Host: peer0.webcash.kshrd.com.kh
              Port: 7051
        Policies: &WebcashPolicies
            Readers:
                Type: Signature
                Rule: "OR('WebcashMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('WebcashMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('WebcashMSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('WebcashMSP.member')"

################################################################################
#
#   CAPABILITIES
#
################################################################################
Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true
    Orderer: &OrdererCapabilities
        V2_0: true
    Application: &ApplicationCapabilities
        V2_0: true

################################################################################
#
#   APPLICATION
#
################################################################################
Application: &ApplicationDefaults
    ACLs: &ACLsDefault
        _lifecycle/CheckCommitReadiness: /Channel/Application/Writers

        # ACL policy for _lifecycle's "CommitChaincodeDefinition" function
        _lifecycle/CommitChaincodeDefinition: /Channel/Application/Writers

        # ACL policy for _lifecycle's "QueryChaincodeDefinition" function
        _lifecycle/QueryChaincodeDefinition: /Channel/Application/Readers

        # ACL policy for _lifecycle's "QueryChaincodeDefinitions" function
        _lifecycle/QueryChaincodeDefinitions: /Channel/Application/Readers

        #---Lifecycle System Chaincode (lscc) function to policy mapping for access control---#

        # ACL policy for lscc's "getid" function
        lscc/ChaincodeExists: /Channel/Application/Readers

        # ACL policy for lscc's "getdepspec" function
        lscc/GetDeploymentSpec: /Channel/Application/Readers

        # ACL policy for lscc's "getccdata" function
        lscc/GetChaincodeData: /Channel/Application/Readers

        # ACL Policy for lscc's "getchaincodes" function
        lscc/GetInstantiatedChaincodes: /Channel/Application/Readers

        #---Query System Chaincode (qscc) function to policy mapping for access control---#

        # ACL policy for qscc's "GetChainInfo" function
        qscc/GetChainInfo: /Channel/Application/Readers

        # ACL policy for qscc's "GetBlockByNumber" function
        qscc/GetBlockByNumber: /Channel/Application/Readers

        # ACL policy for qscc's  "GetBlockByHash" function
        qscc/GetBlockByHash: /Channel/Application/Readers

        # ACL policy for qscc's "GetTransactionByID" function
        qscc/GetTransactionByID: /Channel/Application/Readers

        # ACL policy for qscc's "GetBlockByTxID" function
        qscc/GetBlockByTxID: /Channel/Application/Readers

        #---Configuration System Chaincode (cscc) function to policy mapping for access control---#

        # ACL policy for cscc's "GetConfigBlock" function
        cscc/GetConfigBlock: /Channel/Application/Readers

        # ACL policy for cscc's "GetConfigTree" function
        cscc/GetConfigTree: /Channel/Application/Readers

        # ACL policy for cscc's "SimulateConfigTreeUpdate" function
        cscc/SimulateConfigTreeUpdate: /Channel/Application/Readers

        #---Miscellanesous peer function to policy mapping for access control---#

        # ACL policy for invoking chaincodes on peer
        peer/Propose: /Channel/Application/Writers

        # ACL policy for chaincode to chaincode invocation
        peer/ChaincodeToChaincode: /Channel/Application/Readers

        #---Events resource to policy mapping for access control###---#

        # ACL policy for sending block events
        event/Block: /Channel/Application/Readers

        # ACL policy for sending filtered block events
        event/FilteredBlock: /Channel/Application/Readers
    Organizations:
    Policies: &ApplicationDefaultPolicies
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
        Endorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ApplicationCapabilities

################################################################################
#
#   ORDERER
#
################################################################################
Orderer: &OrdererDefaults
    OrdererType: kafka
    Addresses:
        - orderer1.kshrd.com.kh:7050
        - orderer2.kshrd.com.kh:7050
        - orderer3.kshrd.com.kh:7050

    # Batch Timeout: The amount of time to wait before creating a batch.
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 500
        AbsoluteMaxBytes: 10 MB
        PreferredMaxBytes: 2 MB
    MaxChannels: 0

    Kafka:
        Brokers:
            - kafka1:9092
            - kafka2:9092
            - kafka3:9092
            - kafka4:9092
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
    Capabilities:
        <<: *OrdererCapabilities

################################################################################
#
#   CHANNEL
#
################################################################################
Channel: &ChannelDefaults
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ChannelCapabilities

################################################################################
#
#   PROFILES
#
################################################################################
Profiles:
    KSHRDCooconWebcashOrdererGenesis:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Addresses:
                - orderer1.kshrd.com.kh:7050
                - orderer2.kshrd.com.kh:7050
                - orderer3.kshrd.com.kh:7050
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            CooconWebConsortium:
                Organizations:
                    - *Coocon
                    - *Webcash

    CooconWebcashOrgChannel:
        <<: *ChannelDefaults
        Consortium: CooconWebConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Coocon
                - *Webcash
            Capabilities:
                <<: *ApplicationCapabilities
```
You can review the file or can modify it as necessary. However, the following items are key modifications:

-   The organizations that we specified in the profiles section are named exactly as we named them in the  `cryptogen`  tool and its  `crypto-config.yaml`  configuration file.
-   We modified the ID and Name fields to append MSP for the peers.
-   We modified the MSPDir to point to the output directories from the  `cryptogen tool`.

#### Executing the configtxgen Tool

You need to set the `FABRIC_CFG_PATH` to point to the `configtx.yaml` first:
```
export FABRIC_CFG_PATH=$PWD
```
Note:  `$PWD=/home/blockchain/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks`

To create orderer genesis block, run the following commands:
```
# Create the config directory to store the channel-artifacts
$ mkdir config

# Generate the Genesis Block
$ configtxgen -profile KSHRDCooconWebcashOrdererGenesis -outputBlock ./config/genesis.block -channelID ordererchannel
```
Output:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile KSHRDCooconWebcashOrdererGenesis -outputBlock ./config/genesis.block -channelID ordererchannel
2020-03-18 10:26:45.283 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-18 10:26:45.327 UTC [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: kafka
2020-03-18 10:26:45.327 UTC [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: /home/blockchain/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/configtx.yaml
2020-03-18 10:26:45.329 UTC [common.tools.configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2020-03-18 10:26:45.330 UTC [common.tools.configtxgen] doOutputBlock -> INFO 005 Writing genesis block
```
**After we created the orderer genesis block it is a time to create channel configuration transaction.**
```
# Generate channel configuration transaction
$ configtxgen -profile CooconWebcashOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID mychannel
```
Output:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile CooconWebcashOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID mychannel
2020-03-18 10:30:17.653 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-18 10:30:17.685 UTC [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/blockchain/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/configtx.yaml
2020-03-18 10:30:17.685 UTC [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
2020-03-18 10:30:17.687 UTC [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx
```
The last operation we are going to perform with  `configtxgen`  is the definition of anchor peers for our organizations. This is especially important if there are more peers belonging to a single organization.

Run the following two commands to define anchor peers for **each organization**. Note that the  `asOrg`  parameter refers to the MSP ID definitions in  `configtx.yaml`.
#### Generate Anchor Peer for Coocon Organization.
```
# Generate anchor peer transaction for Coocon Organization
$ configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/CooconMSPanchors.tx -channelID mychannel -asOrg CooconMSP
```
Output:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/CooconMSPanchors.tx -channelID mychannel -asOrg CooconMSP
2020-03-18 10:32:55.786 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-18 10:32:55.831 UTC [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/blockchain/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/configtx.yaml
2020-03-18 10:32:55.831 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-03-18 10:32:55.833 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update
```
#### Generate Anchor Peer for WebCash Organization.
```
# generate anchor peer transaction for Webcash Organization
$ configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/WebcashMSPanchors.tx -channelID mychannel -asOrg WebcashMSP
```
Output:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ configtxgen -profile CooconWebcashOrgChannel -outputAnchorPeersUpdate ./config/WebcashMSPanchors.tx -channelID mychannel -asOrg WebcashMSP
2020-03-18 10:34:49.753 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-18 10:34:49.815 UTC [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/blockchain/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/configtx.yaml
2020-03-18 10:34:49.815 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-03-18 10:34:49.816 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update
```
List files generated by `configtxgen` inside `config` folder.
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/config$ ll
total 32
drwxrwxr-x 2 blockchain blockchain 4096 Mar 18 10:34 ./
drwxrwxr-x 4 blockchain blockchain 4096 Mar 18 10:26 ../
-rw-r----- 1 blockchain blockchain 1666 Mar 18 10:30 channel.tx
-rw-r----- 1 blockchain blockchain  319 Mar 18 10:32 CooconMSPanchors.tx
-rw-r----- 1 blockchain blockchain 9745 Mar 18 10:26 genesis.block
-rw-r----- 1 blockchain blockchain  322 Mar 18 10:34 WebcashMSPanchors.tx
```
## []((#start-the-hyperledger-fabric-blockchain-network))Start the Hyperledger Fabric blockchain network
To start our network we will use  `docker-compose`  tool. Based on its configuration, we launch containers based on the Docker images we downloaded in the beginning.

### [](#modifying-the-docker-compose-yaml-files)Modifying the `docker-compose.yaml` Files

`docker-compose`  tools is using yaml configuration files where various aspects of the containers and their network connection are defined. You can start with configuration yaml file from scratch or leverage yaml configuration from the "first-network" example.

Inside your network directory, you should have a directory structure similar to the following:





# ????????????????????????????????????????????????





We have to create the `.env` file in current directory. There are many environment variables for using with the Docker Compose file.

Inside `.env` file set the following variable.
```
COMPOSE_PROJECT_NAME=net
```
The `docker-compose.yaml` content showed by the following:

Reference: **Hyperledger Fabric Samples**  [docker-compose-ca.yaml](https://github.com/hyperledger/fabric-samples/blob/release-1.4/first-network/docker-compose-ca.yaml) and other compose files.
```
#
# Copyright IBM Corp All Rights Reserved
#
# SPDX-License-Identifier: Apache-2.0
#
version: '2'

networks:
  byfn:

services:
#########################################################################################################
##### CA
#########################################################################################################
  ca.coocon.kshrd.com.kh:
    image: hyperledger/fabric-ca:1.4
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.coocon.kshrd.com.kh
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.coocon.kshrd.com.kh-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/priv_sk
      # - FABRIC_CA_SERVER_TLS_ENABLED=true
      # - FABRIC_CA_SERVER_PORT=7054

    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.coocon.kshrd.com.kh-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/priv_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.coocon.kshrd.com.kh
    networks:
      - byfn

  ca.webcash.kshrd.com.kh:
    image: hyperledger/fabric-ca:1.4
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.webcash.kshrd.com.kh
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.webcash.kshrd.com.kh-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/priv_sk
      # - FABRIC_CA_SERVER_TLS_ENABLED=true
      # - FABRIC_CA_SERVER_PORT=8054
    ports:
      - "8054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.webcash.kshrd.com.kh-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/priv_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.webcash.kshrd.com.kh
    networks:
      - byfn

#########################################################################################################
##### Zookeeper
#########################################################################################################
  zookeeper1:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper1
    restart: always
    environment:
      - ZOO_MY_ID=1
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    ports:
      - 2181
      - 2888
      - 3888
    networks:
      - byfn

  zookeeper2:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper2
    restart: always
    environment:
      - ZOO_MY_ID=2
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    ports:
      - 2181
      - 2888
      - 3888
    networks:
      - byfn

  zookeeper3:
    image: hyperledger/fabric-zookeeper
    container_name: zookeeper3
    restart: always
    environment:
      - ZOO_MY_ID=3
      - ZOO_SERVERS=server.1=zookeeper1:2888:3888 server.2=zookeeper2:2888:3888 server.3=zookeeper3:2888:3888
    ports:
      - 2181
      - 2888
      - 3888
    networks:
      - byfn

#########################################################################################################
##### Kafka
#########################################################################################################
  kafka1:
    image: hyperledger/fabric-kafka
    container_name: kafka1
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka1
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=1
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - byfn

  kafka2:
    image: hyperledger/fabric-kafka
    container_name: kafka2
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka2
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=2
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - byfn

  kafka3:
    image: hyperledger/fabric-kafka
    container_name: kafka3
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka3
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=3
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - byfn

  kafka4:
    image: hyperledger/fabric-kafka
    container_name: kafka4
#        restart: always
    environment:
      - KAFKA_ADVERTISED_HOST_NAME=kafka4
      - KAFKA_ADVERTISED_PORT=9092
      - KAFKA_BROKER_ID=4
      - KAFKA_MESSAGE_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_REPLICA_FETCH_MAX_BYTES=103809024 # 99 * 1024 * 1024 B
      - KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false
      - KAFKA_NUM_REPLICA_FETCHERS=1
      - KAFKA_DEFAULT_REPLICATION_FACTOR=2
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper1:2181,zookeeper2:2181,zookeeper3:2181
    ports:
      - 9092
    depends_on:
      - zookeeper1
      - zookeeper2
      - zookeeper3
    networks:
      - byfn


#########################################################################################################
##### Orderer
#########################################################################################################
  orderer1.kshrd.com.kh:
    container_name: orderer1.kshrd.com.kh
    image: hyperledger/fabric-orderer:2.0
    environment:
      - ORDERER_HOST=orderer1.kshrd.com.kh
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
      # Kafka Orderer Type
      - CONFIGTX_ORDERER_BATCHTIMEOUT=1s
      - CONFIGTX_ORDERER_ORDERERTYPE=kafka
      - CONFIGTX_ORDERER_KAFKA_BROKERS=[kafka1:9092,kafka2:9092,kafka3:9092,kafka4:9092]
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      # - ORDERER_GENERAL_GENESISPROFILE=SampleInsecureKafka
      # - ORDERER_ABSOLUTEMAXBYTES=${ORDERER_ABSOLUTEMAXBYTES}
      # - ORDERER_PREFERREDMAXBYTES=${ORDERER_PREFERREDMAXBYTES}
      - ORDERER_ABSOLUTEMAXBYTES=10 MB
      - ORDERER_PREFERREDMAXBYTES=512 KB

      # enabled TLS
      # - ORDERER_GENERAL_TLS_ENABLED=true
      # - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/orderer/tls/server.key
      # - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/orderer/tls/server.crt
      # - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    volumes:
      - ./config/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer1.kshrd.com.kh/:/etc/hyperledger/msp/orderer
      # - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      # - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer1.kshrd.com.kh/msp:/var/hyperledger/orderer/msp
      # - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer1.kshrd.com.kh/tls/:/var/hyperledger/orderer/tls
      # - orderer1.kshrd.com.kh:/var/hyperledger/production/orderer
    networks:
      - byfn
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - kafka4

  orderer2.kshrd.com.kh:
    container_name: orderer2.kshrd.com.kh
    image: hyperledger/fabric-orderer:2.0
    environment:
      - ORDERER_HOST=orderer2.kshrd.com.kh
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
      # Kafka Orderer Type
      - CONFIGTX_ORDERER_BATCHTIMEOUT=1s
      - CONFIGTX_ORDERER_ORDERERTYPE=kafka
      - CONFIGTX_ORDERER_KAFKA_BROKERS=[kafka1:9092,kafka2:9092,kafka3:9092,kafka4:9092]
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      # - ORDERER_GENERAL_GENESISPROFILE=SampleInsecureKafka
      # - ORDERER_ABSOLUTEMAXBYTES=${ORDERER_ABSOLUTEMAXBYTES}
      # - ORDERER_PREFERREDMAXBYTES=${ORDERER_PREFERREDMAXBYTES}
      - ORDERER_ABSOLUTEMAXBYTES=10 MB
      - ORDERER_PREFERREDMAXBYTES=512 KB

      # enabled TLS
      # - ORDERER_GENERAL_TLS_ENABLED=true
      # - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/orderer/tls/server.key
      # - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/orderer/tls/server.crt
      # - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 8050:7050
    volumes:
      - ./config/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer2.kshrd.com.kh/:/etc/hyperledger/msp/orderer
      # - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      # - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer2.kshrd.com.kh/msp:/var/hyperledger/orderer/msp
      # - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer2.kshrd.com.kh/tls/:/var/hyperledger/orderer/tls
      # - orderer2.kshrd.com.kh:/var/hyperledger/production/orderer
    networks:
      - byfn
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - kafka4

  orderer3.kshrd.com.kh:
    container_name: orderer3.kshrd.com.kh
    image: hyperledger/fabric-orderer:2.0
    environment:
      - ORDERER_HOST=orderer3.kshrd.com.kh
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
      # Kafka Orderer Type
      - CONFIGTX_ORDERER_BATCHTIMEOUT=1s
      - CONFIGTX_ORDERER_ORDERERTYPE=kafka
      - CONFIGTX_ORDERER_KAFKA_BROKERS=[kafka1:9092,kafka2:9092,kafka3:9092,kafka4:9092]
      - ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s
      - ORDERER_KAFKA_RETRY_SHORTTOTAL=30s
      - ORDERER_KAFKA_VERBOSE=true
      # - ORDERER_GENERAL_GENESISPROFILE=SampleInsecureKafka
      # - ORDERER_ABSOLUTEMAXBYTES=${ORDERER_ABSOLUTEMAXBYTES}
      # - ORDERER_PREFERREDMAXBYTES=${ORDERER_PREFERREDMAXBYTES}
      - ORDERER_ABSOLUTEMAXBYTES=10 MB
      - ORDERER_PREFERREDMAXBYTES=512 KB

      # enabled TLS
      # - ORDERER_GENERAL_TLS_ENABLED=true
      # - ORDERER_GENERAL_TLS_PRIVATEKEY=/etc/hyperledger/orderer/tls/server.key
      # - ORDERER_GENERAL_TLS_CERTIFICATE=/etc/hyperledger/orderer/tls/server.crt
      # - ORDERER_GENERAL_TLS_ROOTCAS=[/etc/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 9050:7050
    volumes:
      - ./config/:/etc/hyperledger/configtx
      - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer3.kshrd.com.kh/:/etc/hyperledger/msp/orderer
      # - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      # - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer3.kshrd.com.kh/msp:/var/hyperledger/orderer/msp
      # - ./crypto-config/ordererOrganizations/kshrd.com.kh/orderers/orderer3.kshrd.com.kh/tls/:/var/hyperledger/orderer/tls
      # - orderer3.kshrd.com.kh:/var/hyperledger/production/orderer
    networks:
      - byfn
    depends_on:
      - kafka1
      - kafka2
      - kafka3
      - kafka4


#########################################################################################################
##### Peer
#########################################################################################################
  peer0.coocon.kshrd.com.kh:
    container_name: peer0.coocon.kshrd.com.kh
    image: hyperledger/fabric-peer:2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG

      - CORE_PEER_ID=peer0.coocon.kshrd.com.kh
      - CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051
      # - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      # - CORE_PEER_CHAINCODEADDRESS=peer0.coocon.kshrd.com.kh:7052
      # - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      # - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.coocon.kshrd.com.kh:7053
      # - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.coocon.kshrd.com.kh:7051
      - CORE_PEER_LOCALMSPID=CooconMSP

      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0.coocon.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 7051:7051
      - 7052:7052
      - 7053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx

      # - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/msp:/etc/hyperledger/fabric/msp
      # - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/tls:/etc/hyperledger/fabric/tls
      # - peer0.coocon.kshrd.com.kh:/var/hyperledger/production
    depends_on:
      - couchdb0.coocon.kshrd.com.kh
    networks:
      - byfn
#########################################################################################################
##### CouchDB0 for coocon Peer0
#########################################################################################################
  couchdb0.coocon.kshrd.com.kh:
    container_name: couchdb0.coocon.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 5984:5984
    networks:
      - byfn

  peer1.coocon.kshrd.com.kh:
    container_name: peer1.coocon.kshrd.com.kh
    image: hyperledger/fabric-peer:2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG

      - CORE_PEER_ID=peer1.coocon.kshrd.com.kh
      - CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051
      # - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      # - CORE_PEER_CHAINCODEADDRESS=peer1.coocon.kshrd.com.kh:7052
      # - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      # - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.coocon.kshrd.com.kh:7053
      # - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.coocon.kshrd.com.kh:7051
      - CORE_PEER_LOCALMSPID=CooconMSP

      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1.coocon.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 8051:7051
      - 8052:7052
      - 8053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer1.coocon.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx

      # - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer1.coocon.kshrd.com.kh/msp:/etc/hyperledger/fabric/msp
      # - ./crypto-config/peerOrganizations/coocon.kshrd.com.kh/peers/peer1.coocon.kshrd.com.kh/tls:/etc/hyperledger/fabric/tls
      # - peer1.coocon.kshrd.com.kh:/var/hyperledger/production
    depends_on:
      - couchdb1.coocon.kshrd.com.kh
    networks:
      - byfn
#########################################################################################################
##### CouchDB1 for coocon Peer1
#########################################################################################################
  couchdb1.coocon.kshrd.com.kh:
    container_name: couchdb1.coocon.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 6984:5984
    networks:
      - byfn


  peer0.webcash.kshrd.com.kh:
    container_name: peer0.webcash.kshrd.com.kh
    image: hyperledger/fabric-peer:2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG

      - CORE_PEER_ID=peer0.webcash.kshrd.com.kh
      - CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051
      # - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      # - CORE_PEER_CHAINCODEADDRESS=peer0.webcash.kshrd.com.kh:7052
      # - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      # - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.webcash.kshrd.com.kh:7053
      # - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.webcash.kshrd.com.kh:7051
      - CORE_PEER_LOCALMSPID=WebcashMSP

      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0.webcash.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 9051:7051
      - 9052:7052
      - 9053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer0.webcash.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx

      # - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer0.webcash.kshrd.com.kh/msp:/etc/hyperledger/fabric/msp
      # - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer0.webcash.kshrd.com.kh/tls:/etc/hyperledger/fabric/tls
      # - peer0.webcash.kshrd.com.kh:/var/hyperledger/production
    depends_on:
      - couchdb0.webcash.kshrd.com.kh
    networks:
      - byfn
#########################################################################################################
##### CouchDB0 for webcash Peer0
#########################################################################################################
  couchdb0.webcash.kshrd.com.kh:
    container_name: couchdb0.webcash.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 7984:5984
    networks:
      - byfn

  peer1.webcash.kshrd.com.kh:
    container_name: peer1.webcash.kshrd.com.kh
    image: hyperledger/fabric-peer:2.0
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG

      - CORE_PEER_ID=peer1.webcash.kshrd.com.kh
      - CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051
      # - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      # - CORE_PEER_CHAINCODEADDRESS=peer1.webcash.kshrd.com.kh:7052
      # - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      # - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.webcash.kshrd.com.kh:7053
      # - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.webcash.kshrd.com.kh:7051
      - CORE_PEER_LOCALMSPID=WebcashMSP

      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1.webcash.kshrd.com.kh:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 10051:7051
      - 10052:7052
      - 10053:7053
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer1.webcash.kshrd.com.kh/msp:/etc/hyperledger/msp/peer
      - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/users:/etc/hyperledger/msp/users
      - ./config:/etc/hyperledger/configtx

      # - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer1.webcash.kshrd.com.kh/msp:/etc/hyperledger/fabric/msp
      # - ./crypto-config/peerOrganizations/webcash.kshrd.com.kh/peers/peer1.webcash.kshrd.com.kh/tls:/etc/hyperledger/fabric/tls
      # - peer1.webcash.kshrd.com.kh:/var/hyperledger/production
    depends_on:
      - couchdb1.webcash.kshrd.com.kh
    networks:
      - byfn
#########################################################################################################
##### CouchDB1 for webcash Peer1
#########################################################################################################
  couchdb1.webcash.kshrd.com.kh:
    container_name: couchdb1.webcash.kshrd.com.kh
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 8984:5984
    networks:
      - byfn



#########################################################################################################
##### CLI
#########################################################################################################
  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    # stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      # - FABRIC_LOGGING_SPEC=DEBUG
      - FABRIC_LOGGING_SPEC=DEBUG
      # - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051
      - CORE_PEER_LOCALMSPID=CooconMSP
      # - CORE_PEER_TLS_ENABLED=true
      # - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/tls/server.crt
      # - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/tls/server.key
      # - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/peers/peer0.coocon.kshrd.com.kh/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincodes/go:/opt/gopath/src/github.com/kshrdsmartcontract/go
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./config:/etc/hyperledger/configtx
      # - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
      # - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    networks:
      - byfn
    depends_on:
      # Orderer Organization HRD Center
      - orderer1.kshrd.com.kh
      - orderer2.kshrd.com.kh
      - orderer3.kshrd.com.kh
      # Coocon Organization
      - peer0.coocon.kshrd.com.kh
      - couchdb0.coocon.kshrd.com.kh
      - peer1.coocon.kshrd.com.kh
      - couchdb1.coocon.kshrd.com.kh
      # Webcash Organization
      - peer0.webcash.kshrd.com.kh
      - couchdb0.webcash.kshrd.com.kh
      - peer1.webcash.kshrd.com.kh
      - couchdb1.webcash.kshrd.com.kh
```
### Before running docker containers

We need to change CA Key File for each Organization's CA inside  `docker-composer.yaml`




# ????????????????????????????????????????????????




CA KeyFile can be found in `cryto-config/peerOrganizations` folder for each orginazation.

 1. Coocon CA
	```
	blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/crypto-config/peerOrganizations$ tree coocon.kshrd.com.kh/ca/
	coocon.kshrd.com.kh/ca/
	????????? ca.coocon.kshrd.com.kh-cert.pem
	????????? priv_sk

	0 directories, 2 files
	```
 2. WebCash CA
	```
	blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks/crypto-config/peerOrganizations$ tree webcash.kshrd.com.kh/ca
	webcash.kshrd.com.kh/ca
	????????? ca.webcash.kshrd.com.kh-cert.pem
	????????? priv_sk

	0 directories, 2 files
	```

### Start the docker Containers

After we have generated the certificates, the genesis block, the channel transaction configuration, and created or modified the appropriate yaml files, we are read to start our network. use the following command to start the network.

```
# With -d option = Run the docker container in background
$ docker-compose -f docker-compose.yaml up -d

# Or the Default file of the docker-compose is docker-compose.yaml so we don't need to specify the file name
$ docker-compose up -d
```
Output:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ docker-compose up -d
Creating network "net_byfn" with the default driver
Creating couchdb1.coocon.kshrd.com.kh  ... done
Creating couchdb1.webcash.kshrd.com.kh ... done
Creating zookeeper2                    ... done
Creating couchdb0.webcash.kshrd.com.kh ... done
Creating ca.coocon.kshrd.com.kh        ... done
Creating ca.webcash.kshrd.com.kh       ... done
Creating couchdb0.coocon.kshrd.com.kh  ... done
Creating zookeeper1                    ... done
Creating zookeeper3                    ... done
Creating peer1.coocon.kshrd.com.kh     ... done
Creating peer1.webcash.kshrd.com.kh    ... done
Creating peer0.webcash.kshrd.com.kh    ... done
Creating peer0.coocon.kshrd.com.kh     ... done
Creating kafka3                        ... done
Creating kafka2                        ... done
Creating kafka1                        ... done
Creating kafka4                        ... done
Creating orderer1.kshrd.com.kh         ... done
Creating orderer2.kshrd.com.kh         ... done
Creating orderer3.kshrd.com.kh         ... done
Creating cli                           ... done
```
You can use `docker ps` command to list the execusting containers. In our case, you should see the following containers executing:
-   2 peers in each organization
-   1 couchDB in each peer
-   3 orderer nodes
-   4 Kafka nodes
-   3 Zookeeper nodes
-   1 CLI

After running `docker-compose` and `docker ps` you can expect output similar to the following:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ docker ps
CONTAINER ID        IMAGE                            COMMAND                  CREATED              STATUS              PORTS                                                                       NAMES
90dd0a7989df        hyperledger/fabric-tools         "/bin/bash"              About a minute ago   Up About a minute                                                                               cli
2a3014382a2b        hyperledger/fabric-orderer:2.0   "orderer"                About a minute ago   Up About a minute   0.0.0.0:9050->7050/tcp                                                      orderer3.kshrd.com.kh
26b7699c60a1        hyperledger/fabric-orderer:2.0   "orderer"                About a minute ago   Up About a minute   0.0.0.0:8050->7050/tcp                                                      orderer2.kshrd.com.kh
ec60cf50f238        hyperledger/fabric-orderer:2.0   "orderer"                About a minute ago   Up About a minute   0.0.0.0:7050->7050/tcp                                                      orderer1.kshrd.com.kh
5c184f919260        hyperledger/fabric-kafka         "/docker-entrypoint.???"   About a minute ago   Up About a minute   9093/tcp, 0.0.0.0:32778->9092/tcp                                           kafka1
02f398d28ebc        hyperledger/fabric-kafka         "/docker-entrypoint.???"   About a minute ago   Up About a minute   9093/tcp, 0.0.0.0:32779->9092/tcp                                           kafka4
475411e65239        hyperledger/fabric-kafka         "/docker-entrypoint.???"   About a minute ago   Up About a minute   9093/tcp, 0.0.0.0:32780->9092/tcp                                           kafka2
6deff3388213        hyperledger/fabric-kafka         "/docker-entrypoint.???"   About a minute ago   Up About a minute   9093/tcp, 0.0.0.0:32777->9092/tcp                                           kafka3
dae02229fc07        hyperledger/fabric-peer:2.0      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.coocon.kshrd.com.kh
9dd1b96b0be7        hyperledger/fabric-peer:2.0      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.webcash.kshrd.com.kh
84093c280e48        hyperledger/fabric-peer:2.0      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.webcash.kshrd.com.kh
a94b7832dafd        hyperledger/fabric-peer:2.0      "peer node start"        About a minute ago   Up About a minute   0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.coocon.kshrd.com.kh
d277b9e16303        hyperledger/fabric-zookeeper     "/docker-entrypoint.???"   About a minute ago   Up About a minute   0.0.0.0:32773->2181/tcp, 0.0.0.0:32772->2888/tcp, 0.0.0.0:32771->3888/tcp   zookeeper3
c3c25a56139c        hyperledger/fabric-couchdb       "tini -- /docker-ent???"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0.coocon.kshrd.com.kh
0fad839212ba        hyperledger/fabric-zookeeper     "/docker-entrypoint.???"   About a minute ago   Up About a minute   0.0.0.0:32770->2181/tcp, 0.0.0.0:32769->2888/tcp, 0.0.0.0:32768->3888/tcp   zookeeper1
5c49bb0ed4ff        hyperledger/fabric-ca:1.4        "sh -c 'fabric-ca-se???"   About a minute ago   Up About a minute   0.0.0.0:8054->7054/tcp                                                      ca.webcash.kshrd.com.kh
878210b4cf93        hyperledger/fabric-zookeeper     "/docker-entrypoint.???"   About a minute ago   Up About a minute   0.0.0.0:32776->2181/tcp, 0.0.0.0:32775->2888/tcp, 0.0.0.0:32774->3888/tcp   zookeeper2
62b20550fc88        hyperledger/fabric-ca:1.4        "sh -c 'fabric-ca-se???"   About a minute ago   Up About a minute   0.0.0.0:7054->7054/tcp                                                      ca.coocon.kshrd.com.kh
265b0999b769        hyperledger/fabric-couchdb       "tini -- /docker-ent???"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb0.webcash.kshrd.com.kh
45c7b205d2fa        hyperledger/fabric-couchdb       "tini -- /docker-ent???"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb1.webcash.kshrd.com.kh
84f3cb3e34ef        hyperledger/fabric-couchdb       "tini -- /docker-ent???"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1.coocon.kshrd.com.kh
```
### The channel

After the Docker container start, we can use the  **Command Line Interface (CLI)**  or  **Fabric SDK**  to interact with the blockchain network. In this tutorial we use the  **CLI container**  to interact with the blockchain network. We can enter to the  **CLI container**  by using the following command:
```
$ docker exec -it cli bash
```
Output:
```
blockchain@blockchain-make-doc:~/gopath/src/kshrd/KSHRD-BLOCKCHAIN-NETWORK/networks$ docker exec -it cli bash
bash-5.0#
```
After you enter the CLI container, you can interact with the other peers by prefixing our peer commands with the appropriate environment variables. Usually, this means pointing to the certificates for that peer. You don't need to do that when interacting with peer0 of Coocon organization which is set as the default one in CLI container.

Here is the list of environment variables that need to be used as the prefix for peer commands when interacting with peer0 of Coocon organization, peer1 of Coocon organization, peer0 of Webcash organization and peer1 of Webcash organization respectively.

**Peer0 of Coocon Org**
```
# Peer0 of Coocon organization
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051
```
**Peer1 of Coocon Org**
```
# Peer1 of Coocon organization
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051
```
**Peer0 of WebCash Org**
```
# Peer0 of Webcash organization
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051
```
**Peer1 of WebCash Org**
```
# Peer1 of Webcash organization
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051
```
#### Create the channel

The first command that we issue is the  `peer create channel`  command. This command targets the orderer (where the channels must be created) and uses the  `channel.tx`  and the channel name that is created using the  `configtxgen`  tool. To create the channel, run the following command to create the channel:
```
# peer channel create \
   -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 \
   -c mychannel \
   -f /etc/hyperledger/configtx/channel.tx
```
Output:
```
bash-5.0# peer channel create -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
2020-03-18 11:15:45.461 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:15:45.465 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:15:45.467 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:15:45.542 UTC [cli.common] readBlock -> INFO 004 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-03-18 11:15:45.548 UTC [channelCmd] InitCmdFactory -> INFO 005 Endorser and orderer connections initialized
2020-03-18 11:15:45.750 UTC [cli.common] readBlock -> INFO 006 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-03-18 11:15:45.752 UTC [channelCmd] InitCmdFactory -> INFO 007 Endorser and orderer connections initialized
2020-03-18 11:15:45.953 UTC [cli.common] readBlock -> INFO 008 Expect block, but got status: &{SERVICE_UNAVAILABLE}
2020-03-18 11:15:45.958 UTC [channelCmd] InitCmdFactory -> INFO 009 Endorser and orderer connections initialized
2020-03-18 11:15:46.161 UTC [cli.common] readBlock -> INFO 00a Received block: 0
```
The `peer channel create` command returns a `genesis block` which will be used to join the channel. You can check that by running `ls` command to review that the file `mychannel.block` has been created.
Output:
```
bash-5.0# ls
crypto           mychannel.block
```
#### Join channel

After the orderer creates the channel, we can have the peers join the channel, again using the peer CLI:

Join  **peer0.coocon.kshrd.com.kh**  to the channel.
```
# Join peer0.coocon.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051

# To join the channel.
peer channel join -b mychannel.block
```
Output:
```
bash-5.0# peer channel join -b mychannel.block
2020-03-18 11:27:11.065 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:27:11.069 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:27:11.071 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:27:11.227 UTC [channelCmd] executeJoin -> INFO 004 Successfully submitted proposal to join channel
```
Join **peer1.coocon.kshrd.com.kh** to the channel.
```
# Join peer1.coocon.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.coocon.kshrd.com.kh:7051

# To join the channel.
peer channel join -b mychannel.block
```
Output:
```
bash-5.0# peer channel join -b mychannel.block
2020-03-18 11:31:45.988 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:31:45.991 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:31:45.993 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:31:46.151 UTC [channelCmd] executeJoin -> INFO 004 Successfully submitted proposal to join channel
```
Join **peer0.webcash.kshrd.com.kh** to the channel.
```
# Join peer0.webcash.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051

# To join the channel.
peer channel join -b mychannel.block
```
Output:
```
bash-5.0# peer channel join -b mychannel.block
2020-03-18 11:34:14.690 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:34:14.694 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:34:14.697 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:34:14.871 UTC [channelCmd] executeJoin -> INFO 004 Successfully submitted proposal to join channel
```
Join **peer1.webcash.kshrd.com.kh** to the channel.
```
# Join peer1.webcash.kshrd.com.kh to the channel.
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer1.webcash.kshrd.com.kh:7051

# To join the channel.
peer channel join -b mychannel.block
```
Output:
```
bash-5.0# peer channel join -b mychannel.block
2020-03-18 11:35:55.904 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:35:55.907 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:35:55.910 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:35:56.085 UTC [channelCmd] executeJoin -> INFO 004 Successfully submitted proposal to join channel
```
#### Update anchor peers
To update Anchor Peer for Coocon Organization.
```
# Set Environment Variable to Peer0 in Coocon Organization
CORE_PEER_LOCALMSPID=CooconMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/coocon.kshrd.com.kh/users/Admin@coocon.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.coocon.kshrd.com.kh:7051

# To update the Anchor Peer
peer channel update -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/CooconMSPanchors.tx
```
Output:
```
bash-5.0# peer channel update -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/CooconMSPanchors.tx
2020-03-18 11:38:12.485 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:38:12.488 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:38:12.488 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:38:12.638 UTC [channelCmd] update -> INFO 004 Successfully submitted channel update
```
To update Anchor Peer for Webcash Organization.
```
# Set Environment Variable to Peer0 in Webcash Organization
CORE_PEER_LOCALMSPID=WebcashMSP
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/webcash.kshrd.com.kh/users/Admin@webcash.kshrd.com.kh/msp
CORE_PEER_ADDRESS=peer0.webcash.kshrd.com.kh:7051

# update the Anchor Peer
peer channel update -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/WebcashMSPanchors.tx
```
Output:
```
bash-5.0# peer channel update -o orderer1.kshrd.com.kh:7050 -o orderer2.kshrd.com.kh:7050 -o orderer3.kshrd.com.kh:7050 -c mychannel -f /etc/hyperledger/configtx/WebcashMSPanchors.tx
2020-03-18 11:40:18.327 UTC [main] InitCmd -> WARN 001 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:40:18.329 UTC [main] SetOrdererEnv -> WARN 002 CORE_LOGGING_LEVEL is no longer supported, please use the FABRIC_LOGGING_SPEC environment variable
2020-03-18 11:40:18.329 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2020-03-18 11:40:18.381 UTC [channelCmd] update -> INFO 004 Successfully submitted channel update
```
Exit from CLI Bash
```
exit
```

### Prepare chaincode
With fabric 2.0, the Chaincode needs to be packaged within a tar archive before deployment. Therefore the code needs to be compiled first.
```
go mod vendor    // For go
./gradlew installDist // For java
```

Now the Code needs to be packaged
```
export CC_SRC_PATH=<path to Chaincode dir>
export VERSION=1
export PKG_FILE=fabcar.tar.gz
peer lifecycle chaincode package ${PKG_FILE} --path ${CC_SRC_PATH} --lang ${CC_RUNTIME_LANGUAGE} --label fabcar_${VERSION}
```
Now a new tar File called has been created within the current directory. This can now be installed onto the peers of the network.

### Install chaincode
With Fabric 2.0, chaincode needs to be installed on every peer using the new Lifecycle Management.
```
// Peer0 - Org Coocon
$ . changeOrg.sh 1
$ peer lifecycle chaincode install ${PKG_FILE}

// Peer1 - Org Coocon
$ . changeOrg.sh 2
$ peer lifecycle chaincode install ${PKG_FILE}

// Peer0 - Org Webcash
$ . changeOrg.sh 3
$ peer lifecycle chaincode install ${PKG_FILE}

// Peer1 - Org Webcash
$ . changeOrg.sh 4
$ peer lifecycle chaincode install ${PKG_FILE}
```
The Output of the above commands should result in a Package ID: ...
The following text needs to be stored within the Variable PACKAGE_ID
```
export PACKAGE_ID=fabcar_1:213192848324892394825239052359
```
The Identifier is now being used to verify the Installed Chaincode:
```
$ peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: fabcar_1:213192848324892394825239052359
```
**Approve the Chaincode**
With the chaincode being installed onto each peer, we need to approve the installment
of the cc with the organizations and the orderers. So this needs to be executed for every Organization!
```
$ . changeOrg.sh 1
$ peer lifecycle chaincode approveformyorg -o localhost:7050\
                                           -o localhost:8050\
                                           -o localhost:9050\
                                           --channelID mychannel\
                                           --name fabcar\
                                           --version 1\
                                           --init-required\
                                           --package-id ${PACKAGE_ID}\
                                           --sequence 1\
                                           --waitForEvent
$ . changeOrg.sh 3
$ peer lifecycle chaincode approveformyorg -o localhost:7050\
                                           -o localhost:8050\
                                           -o localhost:9050\
                                           --channelID mychannel\
                                           --name fabcar\
                                           --version 1\
                                           --init-required\
                                           --package-id ${PACKAGE_ID}\
                                           --sequence 1\
                                           --waitForEvent

```
The output should indicate a success with status (VALID) in both cases!
If both Organizations approve the Chaincode, the following code should return the following:
```
$ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name fabcar --version 1 --sequence 1 --output json --init-required
{
  "approvals":??{
      "CooconMSP": true,
      "WebcashMSP": true
  }
}
```
Both Orgs approved the Chaincode! There should not be a "false"!

### Instantiate chaincode
With Fabric 2.0, there has been a change to the Instantiation Paradigm. The CC now needs to be committed to the Channel first,
then it is being invoked. The following code is expecting the PEER_CON_PARAMS to be set!
```
$ peer lifecycle chaincode commit -o localhost:7050\
                                  -o localhost:8050\
                                  -o localhost:9050\
                                  --channelID mychannel\
                                  --name fabcar\
                                  $PEER_CON_PARAMS\
                                  --version 1\
                                  --init-required\
4x success VALID

```
We can now see whether the CC has been committed to the ledger:
```
$ peer lifecycle chaincode querycommitted --channelID mychannel --name fabcar
Output - the Fabcar contract
```

Finally, the CC can be initialized with the Constructor. Notice that the "--isInit" flag is set here:
```
$ peer chaincode invoke -o localhost:7050\
                        -o localhost:8050\
                        -o localhost:9050\
                        -C mychannel -n fabcar $PEER_CON_PARAMS --isInit\
                        -c '{"function":"initLedger", "Args":[]}'
Output: Invoke successful
```

Now we need to do the same but without the "--isInit" flag!
```
$ peer chaincode invoke -o localhost:7050\
                        -o localhost:8050\
                        -o localhost:9050\
                        -C mychannel -n fabcar $PEER_CON_PARAMS\
                        -c '{"function":"initLedger", "Args":[]}'
Output: Invoke successful
```
Everything is setup now! We can now invoke and query the Chaincode!
```
//Query
$ peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
<All Cars>

$ peer chaincode invoke -o localhost:7050\
                        -o localhost:8050\
                        -o localhost:9050\
                        -C mychannel -n fabcar $PEER_CON_PARAMS\
                        -c '{"function":"changeCarOwner", "Args":["CAR9", "Dave"]}'
Output: invoke successful
```
