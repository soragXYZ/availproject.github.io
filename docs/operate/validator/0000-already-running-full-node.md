---
id: already-running-full-node
title: Already Running a Full Node?
sidebar_label: Already Running a Full Node
description: 'Learn about running an Avail validator using binaries.'
keywords:
  - docs
  - avail
  - node
  - validator
image: https://docs.availproject.org/img/avail/AvailDocs.png
---

import useBaseUrl from '@docusaurus/useBaseUrl';

If you're already running a full node and wish to transition to a validator node, please note that simply adding the `--validator` flag after your full node has synced will result in an error expecting an archive database. Follow the steps below to make the switch.

## Step 1. Stop Your Full Node

Before making any changes, safely stop your running full node.

```bash
sudo systemctl stop avail-node.service  # Replace 'avail-node.service' with your service name if different
```

## Step 2. Purge the Database

Since adding the `--validator` flag after syncing expects an archive database, you'll need to purge the existing database.

```bash
# Replace the path with your actual data path
rm -rf /path/to/your/data/directory
```

## Step 3. Update Command Line Flags

Modify the command line used for running your full node to include the `--validator` flag.

For example:

```bash
./data-avail -d ./data --chain goldberg --validator --name MyAvailNode
```

## Step 4. Update Systemd Service File

If you were running your full node as a systemd service, update the service file to reflect the new command
with the `--validator` flag.

```bash
sudo nano /etc/systemd/system/avail-node.service  # Replace 'avail-node.service' with your service name if different
```

- Update the `ExecStart` line with the new command.
- Save and exit the editor.

## Step 5. Restart the Service

Restart the systemd service to apply the changes:

```bash
sudo systemctl start avail-node.service  # Replace 'avail-node.service' with your service name if different
```

## Step 6. Verify Role

Once your node is up and running, verify that the role displays as **"Authority,"** confirming that you are now running a validator node.
