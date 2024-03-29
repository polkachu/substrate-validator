# Substrate-based Validator Node Ansible Setup

This repo is to set up a variety of Substrate-based validator nodes. Currently, this script only supports Polkadex mainnet.

## TL/DR

You run one playbook and set up a node. For example:

```bash
ansible-playbook polkadex.yml -e "target=polkadex1"
```

But before you rush with this easy setup, you probably want to read on so you understand the structure of this Ansible program and all the features it offers.

## Some Preparations

First of all, make sure that you have a production inventory file with your confidential server info. You will start by copying the sample inventory file (included in the repo). The sample file gives you a good idea on how to define the inventory.

```bash
cp inventory.sample.ini inventory.ini
```

Needless to say, you need to update the dummy values in the inventory file. For each node:

1. ansible_host: Your server public IP
1. validator_name (optional): We can override the validator_name variable from the group
1. log_name (optional): We can override the log_name variable from the group

For each cluster:

1. validator_name
1. log_name

For all clusters:

1. ansible_user: The sample file assumes `ansible`, but you might have another username. Make sure that the user has `sudo` privilege.
1. ansible_port: The sample file assumes `22`. But if you are like me, you will have a different ssh port other than `22` to avoid port sniffing.
1. ansible_ssh_private_key_file: The sample file assumes `~/.ssh/id_rsa`, but you might have a different key location.
1. log_monitor: Enter your monitor server IP. It is most likely a private IP address if you use a firewall around your private virtual cloud (VPC).
1. telemetry_url: Most likely you will use `wss://telemetry.polkadot.io/submit/`

It is beyond the scope of this guide to help you create a sudo user, alternate ssh port, create a private key, install Ansible on your machine, etc. You can do a quick online search and find the answers. In my experience, Digital Ocean have some quality guides on these topics. Stack Overflow can help you trouble-shoot if you are stuck.

## Basic Cluster Structure

We have a rather opinionated node cluster structure. The basic structure is:

1. Name each polkadex node as `polkadex1`, `polkadex2`, etc. Group all polkadex nodes into `polkadex` group.
1. So or and so forth for all other nodes.

The structure allows you to target `vars` to each node, or each cluster, or the whole cluster.

Make sure that you are familiar with the files in the `group_vars` folder. They follow this clustered structure closely. The files in this folder often need to be changed to stay up to date with the latest releases.

## Playbook Summary

1. The key Ansible playbook is `polkadex.yml`. It will set up a fresh validator from scratch.
1. We also offer separate playbook called `xxx_update_version.yml`. It will update the node version and restart the service.

A generic example for running a playbook is as follows:

```bash
ansible-playbook polkadex.yml -e "target=VALIDATOR_TARGET"
ansible-playbook key_rotation.yml -e "target=VALIDATOR_TARGET"
```

## Playbook Dictionary

| Playbook                      | Description                            |
| ----------------------------- | -------------------------------------- |
| `polkadex_full_setup.yml`     | Set up a fresh polkadex validator node |
| `polkadex_update_version.yml` | Update polkadex validator version      |

## Security and Server Monitoring

We have a rather opinionated security and server monitoring process. The full setup script:

1. Configures firewall and expose/deny ports
2. Installs node_exporter
3. Installs promtail (for log monitoring) that points to an internal log monitoring server
