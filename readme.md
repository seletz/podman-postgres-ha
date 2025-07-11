# PostgreSQL HA using podamn and anasible

## Abstract

This repo is about trying out and testing a PostgreSQL Master/Slave setup running on
RockyLinux and using podman.

## Goals

- To have two VMs running
- One of the VM should provide a PostgreSQL master, the other one a slave
- Test podman / systemd integration
- Use rootless containers
- To test failover scenarios
- Test backup and restore scenarios

## Approach

- I'm doing this on a arm based mac.
- We're going to use **ansible** to set up the two VMs.
- We're using **RockyLinux** as this is the deployment target at work
- We're using PostgreSQL version 16, because that's what is deployed in work

## Tools

- ansible
- UTM for linux VMs on the mac

```bash
$ brew install ansible utm
```

## Step 1: Setup VMs

> [!Note]
> In reality these would be set up and provisioned using `hclouud` on hetzner or similiar
> eliminating the manual installation steps.

VM Parameters:
- 2 Processors
- 4096 MB RAM
- 20 GB disk
- bridge network
- "Apple Virtualisation" checked

Rock Linux installation:
- I configured a `seletz` user with a password and set that user as admin user
- I kept root disabled
- I kept "minimal install"

I set up `pg1` an then clonde that to `pg2`.  After the clone (copy), you need to edit the copy and
randomise the Ethernet MAC.

Network:
- I used `nmcli` to get fixed, manual IP adresses
- I added `pg1` and `pg2` to my `~/.ssh/config` and specified the IP adresses and my identity file for this experiment.

### Details

- Download Rocky Linux 9 for ARM iso:

```bash
$ mkdir downloads
$ cd downloads
$ wget https://dl.rockylinux.org/pub/rocky/9/isos/aarch64/Rocky-9-latest-aarch64-minimal.iso
```

- Start UTM and create a new native (arm based) machine.  Check `Use Apple Virtualisation`.
- Create a `ssh` key to use for this experiment:

```bash
$ ssh-keygen -t ed25519 -C "stefan.eletzhofer@digitalgedacht.de" -f ~/.ssh/podman-postgres-ha-experiment
Generating public/private ed25519 key pair.
Enter passphrase for "/Users/seletz/.ssh/podman-postgres-ha-experiment" (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/seletz/.ssh/podman-postgres-ha-experiment
Your public key has been saved in /Users/seletz/.ssh/podman-postgres-ha-experiment.pub
The key fingerprint is:
SHA256:A4HygZ/jR/zxcjV/ekRGTdkOg3Lu7B8gD+i/+NdvkLQ stefan.eletzhofer@digitalgedacht.de
The key's randomart image is:
+--[ED25519 256]--+
|   . ..      . .=|
|  o o  .  . o o.+|
|   + +.    +   = |
|    = o...  + . +|
|   . o .Soo+.+ = |
|    . ..o.o+o.E o|
|     .  .o ....= |
|         o  o o.o|
|        ..+o ..+.|
+----[SHA256]-----+
```

- Network setup -- set static IPs.  On each VM, do:

```bash
$ nmcli connection modify enp0s1 ipv4.addresses "192.168.200.10/24" ipv4.gateway "192.168.200.1" \
    ipv4.dns "192.168.200.1,8.8.8.8" ipv4.method manual
$ nmcli connection up System enp0s1
```

## Step 2: Ansible Files

```text
ansible
├── group_vars
│ └── all.yml
├── inventory
│ └── hosts.yml
├── playbooks
│ └── site.yml
└── roles
    ├── podman-setup
    │ └── tasks
    │     └── main.yml
    ├── postgres-primary
    │ ├── tasks
    │ │ └── main.yml
    │ └── templates
    │     ├── init-replica.sql.j2
    │     └── postgres-primary.container
    └── postgres-replica
        ├── tasks
        │ └── main.yml
        └── templates
            └── postgres-replica.container

13 directories, 9 files
```

With that, we can now `ping` the servers:

```bash
$ ansible all -i inventory/hosts.yml -m ping
pg2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
pg1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
