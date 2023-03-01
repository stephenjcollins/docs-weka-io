---
description: >-
  This section aims at a system engineer who is familiar with the GCP concepts
  and experienced with using Terraform to deploy a system on GCP.
---

# Weka installation on GCP

## Weka on GCP overview

Leveraging the GCP advantages, Weka provides a ready-to-deploy GCP-Terraform package that you can customize for installing the Weka cluster on GCP.&#x20;

Within GCP, Weka runs on instances. Each instance can use up to eight partitions of drives on the connected physical server (Weka does not use the drives directly). The drives can be shared with other clients' partitions in the same physical server.

Weka requires a minimum of four VPC networks, and each is associated with each of the instances. The reason for using four VPC networks is that a basic Weka configuration consists of four processes: Compute, Drive, Frontend, and Management. Each of these processes requires a dedicated network interface as follows:

* eth0: Management VPC
* eth1: Compute VPC
* eth2: Frontend VPC
* eth3: Drive VPC

VPC peering is used for communication between the Weka processes. A project in GCP is limited to a maximum of 32 VPC peers.

<figure><img src="../../.gitbook/assets/GCP_overview.png" alt=""><figcaption><p>Server infrastructure in GCP</p></figcaption></figure>

<details>

<summary>Terraform overview</summary>

Terraform is an open-source project from Hashicorp. It creates and manages resources on cloud platforms and on-premises clouds. Unlike AWS CloudFormation, it works with many APIs from multiple platforms and services.

The GCP Console is already installed with Terraform by default. It is the primary tool for deploying Weka on GCP. Terraform can be used outside of GCP or independent of GCP Console.

<img src="../../.gitbook/assets/Terraform_overview.png" alt="" data-size="original">

### How does Terraform work?

A deployment with Terraform involves three phases:

* **Write:** Define the infrastructure in configuration files and customize the project variables provided in the Terraform package.
* **Plan**: Review the changes Terraform will make to your infrastructure.
* **Apply:** Terraform provisions the infrastructure, including the VMs and instances, installs the Weka software, and creates the cluster. Once completed, the Weka cluster runs on GCP.

<img src="../../.gitbook/assets/Terraform_how.png" alt="Terraform phases" data-size="original">

**Related information**

[Terraform Tutorials](https://learn.hashicorp.com/terraform?track=gcp)

[Terraform Installation](https://learn.hashicorp.com/tutorials/terraform/install-cli)

</details>

**Related topics**

[weka-project-description.md](weka-project-description.md "mention")