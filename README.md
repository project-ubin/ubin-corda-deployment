# Project Ubin Phase 2 - Corda Deployment

This is a repository for Project Ubin containing collection of scripts to set up a network and perform deployment for Corda workstream. 


## Getting Started

To get started, clone this repository with:
```sh
$ git clone https://github.com/project-ubin/ubin-corda-deployment.git
```
# Set Up New Network

Note: Following steps have been tested in Ubuntu 16.04 LTS

## A. Pre-Requisites
You will need the following components set up/installed:

* 15 Ubuntu (Xenial - LTS 16.04) VMs (11 banks, 1 MAS Central Bank node, 1 MAS as Regulator node, 1 Network Map, 1 Notary) with minimum specifications of 1 core, 3.5 GB RAM
* [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
  installed and available on your path.
* Git

## B. Network setup step:
1. Set up network map
2. Set up notary
3. Set up bank nodes

The script `configure.sh` from `ubin-corda-deployment` repository takes in 5 input parameters:
* Node type (value: networkmap, notary, node)
* Virtual machine (VM) username
* Network Map Name
* Notary type (default value is "nonValidating")
* Network Map IP address

### 1. Set Up Network Map

1\. SSH to Network Map virtual machine.

2\. Clone `ubin-corda-deployment` repository
```sh
$ git clone https://github.com/project-ubin/ubin-corda-deployment.git
```
3\. Determine network map node name (e.q. Network Map)

4\. Get virtual machine IP address with following command:
```sh
$ hostname -i
```
5\. Execute `configure.sh` script with:
```sh
$ sudo ./configure.sh networkmap <<VM Username>> <<Network Map Name>> nonValidating <<Network Map IP>>
```
   Example:

```sh
# Network map name is "Network Map"
# Network map IP address is "10.0.0.47"
# VM username is "azureuser"

chmod +x configure.sh
sudo ./configure.sh networkmap azureuser "Network Map" nonValidating 10.0.0.47
```

Note: Network map IP address is required in the set up of notary node and additional bank nodes

### 2. Set Up Notary

1\. SSH to Notary virtual machine.

2\. Clone `ubin-corda-deployment` repository

```sh
$ git clone https://github.com/project-ubin/ubin-corda-deployment.git
```

3\. Determine Notary node name (e.g. Notary)

4\. Get network map IP Address from the previous step (network map setup).

5\.  Execute `configure.sh` script with:
```sh
$ sudo ./configure.sh networkmap <<VM Username>> <<Network Map Name>> nonValidating <<Network Map IP>>>
```

   Example:
```sh
# Notary name is "Notary"
# Network map IP address is "10.0.0.47"
# VM username is "azureuser"
$ chmod +x configure.sh
$ sudo ./configure.sh notary azureuser "Notary" nonValidating 10.0.0.47
```

### 3. Set Up Bank Nodes

1\. SSH to bank nodes virtual machine.

2\. Clone `ubin-corda-deployment` repository

```sh
$ git clone https://github.com/project-ubin/ubin-corda-deployment.git
```

3\. Determine bank node name (usually the bank SWIFT code).

4\. Get network map IP Address from the previous step (network map setup).

5\. Go to `ubin-corda-deployment` directory.

6\. Verify `config.properties` to ensure `ApproveRedeemURI` is configured with the Central Bank domain name.
```sh
ApproveRedeemURI=http://<<centralbankdomain>>:9001/meps/redeem
```

7\. Execute `configure.sh` script with:

```sh
$ sudo ./configure.sh notary <<VM Username>> <<Notary Name>> nonValidating <<Network Map IP>>
```
   Example:
```sh
# Nodename name is "BankA"
# Network map IP address is "10.0.0.47"
# VM username is "azureuser"

$ chmod +x configure.sh
$ sudo ./configure.sh node azureuser "BankA" nonValidating 10.0.0.47
```
Note: do not name the Corda node with a name containing "node".



# Central Deployment and Server Admin

## A. CorDapp Deployment

1\. Copy CorDapp JARs into VM Node 0 into the following directory with SCP/FTP:
```sh
/home/azureuser/ubin-corda-deployment/plugin
```
2\. Log in to VM Node 0 using SSH
3\. Go to `ubin-corda-deployment` directory

4\. Execute manage.sh to deploy CorDapp to all nodes in the network except the Notary and the Network Map. The script assumes that the nodes are named sequentially (e.g. 0 to 12):
```sh
$ ./manage.sh deploy 0 12
```

This step does the following:

- Delete everything in /app/corda/plugins
- Copy all files from Node 0's /home/azureuser/deploy to the target node's /app/corda/plugins folder
- Restart Corda and webserver in the node
- Repeat for selected Nodes

Note: 0 and 12 in Step 4 represents the range of nodes. If you only require deployment on nodes 2-4, change the parameters to "deploy 2 4".

## B. Restart All Nodes
1\. Log in to VM Node 0 using SSH
2\. Go to `ubin-corda-deployment` directory
3\. Execute `manage.sh` to restart all nodes in the network (e.g. Node 0 - Node 12):
```sh
$ ./manage.sh restart 0 12
```
Note: 0 and 12 in Step 3 represents the range of nodes. If you only require deployment on nodes 2-4, change the parameters to "deploy 2 4".

## C. Stop All Nodes
1\. Log in to VM Node 0 using SSH
2\. Go to `ubin-corda-deployment` directory
3\. Execute `manage.sh` to stop all nodes in the network (e.g. Node 0 - Node 12):
```sh
$ ./manage.sh stop 0 12
```
Note: 0 and 12 in Step 3 represents the range of nodes. If you only require deployment on nodes 2-4, change the parameters to "deploy 2 4".

## D. Clear All Data in Vault
1\. Log in to VM Node 0 using SSH
2\. Check that you are in `/home/azureuser`
3\. Go to `ubin-corda-deployment`
4\. Stop all Corda nodes (node 0 to 12) using:
```sh
$ ./manage.sh stop 0 12
```

5\. Clear all data in network (node 0 to 12) - note that the data is *not recoverable*:

```sh
$ ./manage.sh reset 0 12
```

6\. Restart all nodes 0 to 12:

```sh
$ ./manage.sh restart 0 12
```

## E. Update Node Names
1\. Stop the Corda server and webserver
2\. Go to `/app/corda`
3\. Update `node.conf` with command:
```sh
$ vi node.conf
```
4\. Change `myLegalName` in "O" key :
```sh
"O=BOTKSGSX, L=Singapore, C=Singapore"
```
5\. Delete the `certificates` folder. The certificates will be regenerated automatically upon Corda server start up
6\. To purge the old entries in the Network Map, login to the h2 DB of the Network Map (nm0) and delete the old entries in `NODE_NETWORK_MAP_NODES` table

Note:

As of Corda v1.0, 'CN' / Common Name field in the X500 is no longer supported. It is used only for services identity. In addition, words such as "node" is blacklisted in CordaX500Name and therefore should not be used.

# License

Copyright 2017 The Association of Banks in Singapore

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.