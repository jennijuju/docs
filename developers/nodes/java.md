---
title: Java
description: The Java implementation of the Aion kernel. You can interact with this kernel directly through the JSON RPC or Protobuff API layers. Further documentation regarding the kernel can be found on the Aion Java Kernel Github repository at https://github.com/aionnetwork/aion
---

This section of the docs goes over how to configure and interact with the Java kernel directly. If you're looking to develop the Aion kernel itself, or how to setup and Aion node, then this section is for you! If you're just looking to create some Dapps, then you can leave this section for now and head on over to the [fundamentals section](/developers/fundamentals).

## Installation

There are two methods for installing the Java kernel on your machine:

1. Installing a build on a [Ubuntu installation](#ubuntu-installation)
2. Using the [Aion Java Kernel Docker Image](#docker-image)

### Native Ubuntu Install

This section walks you through installing the kernel from a package. You can either generate your own package by building the kernel from source, or you can download a [pre-built package from Github](https://github.com/aionnetwork/aion/releases).

We have used the latest build as an example on this page. If you want to use a different version of the Aion kernel you will need to change the commands to fit the version you are using. We do not recommend always using the latest Aion kernel version.

#### Prerequisites

- [Ubuntu 16.04 or later](http://releases.ubuntu.com/16.04/)
- [Open JDK 11](https://download.java.net/java/GA/jdk11/13/GPL/openjdk-11.0.1_linux-x64_bin.tar.gz)
- [Apache Ant 1.10](http://ant.apache.org/bindownload.cgi)
- `8GB` RAM
- CPU with `SSE4.2` support

#### Run the Installation

1. Connect to your machine via SSH or open a terminal if you are working on a local installation.
2. Change to your root `~` directory and update your system:

        cd ~
        sudo apt update -y && sudo apt upgrade -y

3. Download the latest [Aion Java kernel package](https://github.com/aionnetwork/aion/releases) to your computer if you haven't already. You can also use `wget` if you'd prefer:

        wget https://github.com/aionnetwork/aion/releases/download/v0.3.3/aion-v0.3.3.22362091d-2019-03-07.tar.bz2

4. Unzip the package and move the `aion` directory to your home `~` directory:

        tar xvjf aion-v0.3.3.22362091d-2019-03-07.tar.bz2
        mv aion ~/

5. Move to your `aion` directory and create an address & keystore:

        cd ~/
        ~/aion/aion.sh -a create

6. Enter a password and write it down, you'll need it later.
7. The terminal shows an account address. Write it down somewhere, you'll need this later.

        ~/.aion/aion.sh -a create

        > Please enter a password:
        > Please re-enter your password:
        > A new account has been created: 0xa09...

8. The build script creates a keystore file in `aion/keystore`. Create a backup of the file:

    **On a Local Machine**:

        cp ~/aion/mainnet/keystore/* ~/Documents/aion-keystore

    Make a copy of this `keystore` file and store it somewhere other than the computer you are installing the Java kernel on. Move onto the last step.

    **On a Remote Machine over SSH**

        # exit your remote machine
        exit

        # copy the aion keystore to your local machine
        scp user@node_ip_address:~/aion/mainnet/keystore/* ~/Desktop

    Make a copy of this `keystore` file and store it somewhere other than the computer you are installing the Java kernel on. Reconnect to your remote machine:

        ssh user@remote-node-ip

    Move onto the last step.

9. Run the Aion Java kernel:

        ~/aion/aion.sh

The kernel will now begin to _sync_ with the other nodes on the network. This process can take up to 72 hours depending on the speed of your network and the size of the database you are syncing. Everything is complete once that's finished. You have successfully installed the Aion Java kernel on your system.

---

### Docker Image

This section covers how to configure and run the Aion Java kernel Docker container.

#### Quickstart

Follow these steps to get started quickly, or skip this section if you want to learn how to run the container in more detail.

```bash
# Pull the Rust kernel image.
docker pull aionnetwork/aion:0.3.3

# Create some local storage for the container:
docker volume create aion-mainnet

# Run the container:
docker run -it -p 8545:8545 -p 8547:8547 -p 30303:30303 --mount source=aion-mainnet,destination=/aion/mainnet aionnetwork/aion:0.3.3
```

#### Java Prerequisites

To use this Docker image your system must meet the following requirements:

- `8GB` RAM (`16GB` recommended)
- 2 CPU cores
- 1GB HDD space
- Docker `v18.0.0`

The HDD space required only takes the Docker image into account. You will need a significant amount of space for storing the blockchain itself. The database is currently around `22GB` in size, although this can be [pruned](https://docs.aion.network/docs/database#section-state-database-pruning).

#### Install the Java Image

1. Pull down the latest Java Docker image.

        docker pull aionnetwork/aion:0.3.3

        > 0.3.3-latest: Pulling from aionnetwork/aion-staging
        > 6cf436f81810: Downloading  14.83MB/32.37MB
        > ...
        > Digest: sha256: efa8e77...

2. Create local storage for Aion image.

        docker volume create aion-mainnet

        > aion-mainnet

1. Run the image.

        docker run -it -p 8545:8545 -p 8547:8547 -p 30303:30303 --mount source=aion-mainnet,destination=/aion/mainnet aionnetwork/aion:0.3.3

        >                     _____
        >      .'.       |  .~     ~.  |..          |
        >    .'   `.     | |         | |  ``..      |
        >  .''''''''`.   | |         | |      ``..  |
        >.'           `. |  `._____.'  |          ``|
        >
        >                v0.3.3.2886431
        >                    mainnet
        >                   using FVM

    See the [Running the Java Container](#running-the-java-container) section for more information on what arguments to supply with the `docker run` command.

4. Press `CTRL` + `c` to shut down and exit the container.

        19-03-04 10:59:01.821 INFO  GEN  [shutdown]: Starting shutdown process...
        ...
        19-03-04 10:59:05.887 INFO  GEN  [shutdown]: ---------------------------------------------
        19-03-04 10:59:05.888 INFO  GEN  [shutdown]: | Aion kernel graceful shutdown successful! |
        19-03-04 10:59:05.888 INFO  GEN  [shutdown]: ---------------------------------------------

#### Java Arguments and Configuration

There are several arguments that you can supply with the `docker run` command.

##### Configure the Java Kernel

Once the kernel Docker image is pulled you can configure it by running the `docker exec` command in a separate terminal window:

    docker exec -it <CONTAINER_NAME or CONTAINER_HASH> /bin/bash

This starts a standard terminal session within the container. You will need to install a text editor before you can edit any files, as the container doesn't come with one pre-installed:

    sudo apt install nano

    OR

    sudo apt install vim

Then you can edit the `config.xml` file associated with the network you are running. For example, if you are running the Java kernel on Mainnet, then you should edit the `mainnet/config.xml` file. If you are running the Rust kernel on the Testnet (Mastery), then you should edit the `mastery/config.xml` file.

##### Java Networks

By default, running the image will start a node on the mainnet. To specify a network; for instance, the mastery testnet, use:

    docker run aionnetwork/aion:0.3.3 /aion/aion.sh -n mastery

##### Java Ports

The Aion Docker image is configured to run the Java API and RPC servers, as well as allow connections from other Aion nodes. When running the Docker container, it is necessary to publish those ports if you use to wish these functionalities.

| Port | Connection Type |
| ---- | ------- |
| `30303` | P2P |
| `8545` | JSON-RPC |
| `8547` | Java API |

##### Java Storage

In most cases, storage should be attached so that configuration and blockchain sync state can be persisted between each time the kernel is launched. You will need a separate Docker volume for each Aion Network, so it is recommended to include the network name in the volume name. To create a volume:

    docker volume create VOLUME-NAME

To start the Docker image with the volume, where VOLUME-NAME is the volume name and NETWORK is the Aion network name:

    docker run -it --mount source=VOLUME-NAME,destination=/aion/NETWORK aionnetwork/aion:0.3.

For the list of network names, see:

    docker run -it aionnetwork/aion:0.3.3 /aion/aion.sh -h

## Further Documentation

The [Github Wiki](https://github.com/aionnetwork/aion/wiki) section contains detailed information on building and interacting with the kernel. Take a look at the [Installation](#installation) section to find out how to install the Java kernel on your machine. 

## Get Involved

If you want to get involved with the Aion Java kernel development, take a look at the [Github repository](https://github.com/aionnetwork/aion/).