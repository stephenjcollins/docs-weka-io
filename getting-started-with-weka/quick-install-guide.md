# Quick installation guide

For a complete installation guide, as well as the self-service portal on AWS refer to the following pages:

{% content-ref url="../install/aws/" %}
[aws](../install/aws/)
{% endcontent-ref %}

{% content-ref url="../install/bare-metal/" %}
[bare-metal](../install/bare-metal/)
{% endcontent-ref %}

## Prerequisites

1. [OFED is installed](../install/bare-metal/setting-up-the-hosts/#mellanox-ofed-installation)
2. [NIC devices are configured properly](../install/bare-metal/setting-up-the-hosts/#network-configuration)
3. [NTP is set up](../install/bare-metal/setting-up-the-hosts/#clock-synchronization)

For a complete prerequisite list, refer to the [Prerequisites for Installation](../install/prerequisites-for-installation-of-weka-dedicated-hosts.md) section.&#x20;

We'll consider an example of architecture with 8 identical hosts (named `weka01` to `weka08`). Each host has more than 20 cores, 6 NVME drives, and a single Mellanox NIC.&#x20;

Using Mellanox or Intel E810 NICs simplifies the installation commands. For example, only a single IP for data is required, no need to expose VFs, identification of the interface netmask, and default routing gateway.&#x20;

If the architecture is different, the installation commands should be slightly changed accordingly.

For a full command set, refer to [Weka System Installation Process Using the CLI](../install/bare-metal/using-cli.md) section.

## Installation

Install Weka software on each host:

```bash
# deploy the software on all hosts
pdsh -w weka0[1-8] "curl https://[GET.WEKA.IO-TOKEN]@get.weka.io/dist/v1/install/4.0.0/4.0.0 | sudo sh"

```

## Configuration

From one of the servers, form the cluster, set the cluster name, stripe width, and protection scheme, and enable cloud monitoring (DNS is assumed to be set up, otherwise explicit IPs should be used in the cluster create command):

```bash
# connect to one of the servers and run the rest of the configuration from there
ssh weka01

# form the cluster and set basic stuff
# using bash, you can provide a compact list of hosts; otherwise, a full list of all hosts should be supplied
# weka cluster create weka01 weka02 weka03 weka04 weka05 weka06 weka07 weka08
weka cluster create weka0{1..8}
weka cluster update --cluster-name=WekaProd
weka cloud enable

```

Configure the network, drives, and CPUs per host:

```bash
# configure network, drives, and cores per host
# replace network, drives, and cores configuration with your actual data

for i in {0..7}
do
    weka cluster host dedicate $i on
    
    # add network NICs
    # e.g., weka cluster host net add $i eth1
    weka cluster host net add $i NETDEV
    
    # add the nvme drives; e.g., /dev/nvme0n1, etc.
    weka cluster drive add $i /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1 /dev/nvme4n1 /dev/nvme5n1
    
    # set host cores
    weka cluster host cores $i 19 --frontend-dedicated-cores 1 --drives-dedicated-cores 6
done

```

Check the  configuration:

```bash
# show hosts info (net, cores, etc.)
for i in {0..7}
do
    weka cluster host resources $i
done

# show drives info
weka cluster drive

# show configuration status
weka status

```

If satisfied, apply the configuration:

```bash
# initialize the hosts
weka cluster host apply --all --force

# Verify that the operation is successful
# Check the alerts and verify that the ResourcesNotApplied alert does not show
weka alerts

```

Set a valid license and start the cluster:

```
# set a license (classic or payg), as obtained from get.weka.io
weka cluster license set LICENSE_TEXT_OBTAINED_FROM_GET_WEKA_IO

# start the cluster
weka cluster start-io

```

Check the status:

```bash
# show cluster info and status
weka status

```

You would see a similar output to the following example:

```bash
WekaIO v4.0.0 (CLI build 4.0.0)

       cluster: WekaProd (00569cef-5679-4e1d-afe5-7e82748887de)
        status: OK (8 backends UP, 48 drives UP)
    protection: 6+2
     hot spare: 1 failure domains
 drive storage: 82.94 TiB total, 82.94 TiB unprovisioned
         cloud: connected
       license: Unlicensed

     io status: STARTED 7 seconds ago (96 io-nodes UP, 750 buckets UP)
    link layer: Ethernet
       clients: 0 connected
         reads: 0 B/s (0 IO/s)
        writes: 0 B/s (0 IO/s)
    operations: 0 ops/s
        alerts: 1 active alert, use `weka alerts` to list it

```

Lastly, change the default admin password and ensure no other alerts exist in the system.

Once the Weka system is installed, learn how to view, manage and operate it using either the CLI or the GUI, and perform the first IO to a WekaFS filesystem.

The Weka system supports a RESTful API for automating the interaction with the Weka system,  integrating it into your workflows, and monitoring systems.



**Related topics**

[manage-the-system-using-weka-cli.md](manage-the-system-using-weka-cli.md "mention")

[manage-the-system-using-weka-gui.md](manage-the-system-using-weka-gui.md "mention")

[performing-the-first-io.md](performing-the-first-io.md "mention")

[getting-started-with-weka-rest-api.md](getting-started-with-weka-rest-api.md "mention")
