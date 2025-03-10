---
id: system-requirements
title: System Requirements
sidebar_label: System Requirements
description: 'Learn about running an Avail validator using Docker.'
keywords:
  - docs
  - avail
  - node
  - docker
  - validator
  - data availability
image: https://docs.availproject.org/img/avail/AvailDocs.png
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import useBaseUrl from '@docusaurus/useBaseUrl';

## What You Need

To ensure a smooth experience, considering the following:

1. **[<ins>Hardware Configuration</ins>](#hardware):** Verify that your hardware meets the Avail node's requirements.
2. **[<ins>Methods to Run a Node</ins>](#methods-to-run-an-avail-node):** Familiarize yourself with the different Avail node types and the methods to run them.
3. **[<ins>Prerequisites & Installation</ins>](#installation):** Ensure you have the necessary prerequisites and packages installed on your machine.
4. **[<ins>Notable System Configuration</ins>](#notable-configurations):** Pay attention to system configuration details, including port settings and disk setup.

## Hardware

This is the hardware configuration required to set up an Avail node:

| Component                    | Minimum  | Recommended |
| ---------------------------- | -------- | ----------- |
| RAM                          | 4GB      | 8GB         |
| CPU (amd64/x86 architecture) | 2 core   | 4 core      |
| Storage (SSD)                | 20-40 GB | 200-300 GB  |

> While we do not favor any operating system, more secure and stable Linux server distributions (like CentOS) should be preferred over desktop operating systems (like Mac OS and Windows). Also, the minimum storage requirements will change over time as the network grows.
> It is recommended to use more than the minimum requirements to run a robust full node.

## Methods to Run an Avail Node

There are two primary methods to run an Avail node, each with its own set of instructions:

1. **[<ins>Using Binaries</ins>](/docs/operate/node/0020-full-node-binaries.md):** You can follow the instructions provided here for setting up an Avail node using binary releases. This method is suitable for users who prefer manual installation and configuration. For reference, you can also explore the [GitHub Releases](https://github.com/availproject/avail/releases) page for binary releases.

2. **[<ins>Using Docker</ins>](/docs/operate/node/0030-full-node-docker.md):** Alternatively, if you're comfortable with Docker, you can pull Avail node images from the [DockerHub Repository](https://hub.docker.com/r/availj/avail/tags). Docker provides a convenient way to containerize and deploy Avail nodes.
   > The Avail team offers official Docker images specifically designed for running nodes on the Goldberg testnet.

## Setting Up Your Environment

The node deployment guides provide specific instructions tailored for Linux distributions with apt support, such as **Debian**.
Additionally, it's common practice to run nodes on cloud servers.

### Installation

<Tabs groupId="pre-built-release" defaultValue="pre-built-release">
<TabItem value="pre-built-release" label="Pre-Built Release">

Download the client binary and config file for the node you want to run:

| Node Type     | Repository Name                                           | Latest Release                                                                     |
| ------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Full Node** | [<ins>avail</ins>](https://github.com/availproject/avail) | [<ins>v1.8.0.3</ins>](https://github.com/availproject/avail/releases/tag/v1.8.0.3) |

</TabItem>
<TabItem value="source" label="Build from Source">

#### Common Dependencies

Before proceeding, install the required dependencies:

```bash
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustc --version # Verify Rust installation by displaying the version
```

Clone the node repository you would like to run:

| Node Type                                                                              | Build Steps                                         |
| -------------------------------------------------------------------------------------- | --------------------------------------------------- |
| [<ins>**Full Node**</ins>](https://github.com/availproject/avail)                      | `cargo build --release -p data-avail`               |
| [<ins>**Relay Node**</ins>](https://github.com/availproject/avail-light-relay)         | `cd avail-light-relay && cargo build --release`     |
| [<ins>**Bootstrap Node**</ins>](https://github.com/availproject/avail-light-bootstrap) | `cd avail-light-bootstrap && cargo build --release` |

</TabItem>
<TabItem value="docker" label="Docker">

The available Docker images are compatible exclusively with **Linux/amd64** or **x86_64** based CPUs.

<details>
<summary>Install Docker</summary>

If Docker is not already installed on your system, please follow the installation instructions provided [here](https://docs.docker.com/engine/install/). In this guide, we will use the **Ubuntu-specific** installation instructions, but it's advisable to refer to the official guidelines for the most up-to-date information.

To install Docker, execute the following commands:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

At this point, you should have Docker installed. To ensure a more convenient user experience and avoid running Docker as the root user (which can be inconvenient), follow the post-installation steps [here](https://docs.docker.com/engine/install/linux-postinstall/). These steps enable you to interact with Docker without requiring root privileges:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

</details>

| Node Type | Docker Repository Name                                                | Latest Version                                                                                                                                                            |
| --------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Full Node | [<ins>availj/avail</ins>](https://hub.docker.com/r/availj/avail/tags) | [<ins>v1.8.0.3</ins>](https://hub.docker.com/layers/availj/avail/v1.8.0.3/images/sha256-f721dd15c77389a3c94002e22a6abb06bf75dd93812328f09c4871621dc138ad?context=explore) |

</TabItem>
</Tabs>

### Notable Configurations

#### Ports

**Port 30333** is typically required for peer exchange. Ensure that this port, along with any others needed for machine access, is open when configuring your firewall or your cloud provider's security groups.

#### Disk Setup

You'll typically have a root partition for the operating system on one device, and one or more separate devices for storing blockchain data. In these guides, we'll assume the additional storage device is mounted at `/mnt/avail`.

Before mounting the additional disk, it's advisable to format it and create a filesystem. For guidance, you can [<ins>follow these instructions</ins>](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html).

Here's an example of working with a 300 GB device located at `/dev/nvme1n1`. Mount the device with these steps:

```bash
sudo mkdir /mnt/avail
sudo mount -a /dev/nvme1n1 /mnt/avail
```

Use `df -h` to verify the successful configuration.

If everything looks good, create sub-directories for data and configuration files:

```bash
sudo mkdir /mnt/avail/config
sudo mkdir /mnt/avail/state
sudo mkdir /mnt/avail/keystore
```

Depending on your use case and operating system, you may need to add an entry to `/etc/fstab` to ensure the device remains mounted after system reboots.
In our scenario, we'll proceed as follows:

```bash
# Use blkid to get the UUID for the device that we're mounting

blkid

# Edit the fstab file  and add a line to mount your device
# UUID={your uuid}      /mnt/data   {your filesystem}   defaults    0   1
sudo emacs /etc/fstab

# you can use any test editor based on your preference to edit the file here we have used emacs.

# use this to verify the fstab actually works
sudo findmnt --verify --verbose
```

<!--
### Alternate Networks & Releases Information

The instructions provided here are specifically for the **Kate testnet**. If you need to connect to a different network, you may have to download an alternate node version from the [<ins>node releases page</ins>](https://github.com/availproject/avail/releases) and obtain the corresponding chain specification file.

## How Many Tokens Do I Need to Become an Active Validator?

To become an active validator, you'll need a certain number of tokens for staking. Please refer to the [Faucet guide](/docs/about/faucet.md) on obtaining test AVL tokens.

If you have any questions or need further assistance, don't hesitate to get in touch with the Avail team for guidance and support.
-->
