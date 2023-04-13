---
description: This page describes how to set up, update, monitor, and delete an S3 cluster.
---

# S3 cluster management

## Considerations

The S3 service can be exposed from the cluster hosts, ranging from three hosts to the entire cluster. The service performance scales linearly as the S3 cluster scales.

{% hint style="info" %}
**Note:** Depending on the workload, you may need to use several FE cores to gain maximum performance.
{% endhint %}

## Round-robin DNS or load balancer

To ensure that the various S3 clients balance the load on the different Weka hosts serving S3, configuring a _Round-robin DNS_ entry is recommended. The round-robin DNS resolves the list of hosts' IPs and equally distributes the clients' loads across all hosts.

A DNS server that supports health checks can help with resiliency if any hosts serving S3 become unresponsive.

Even a robust DNS server or load-balancer may become overloaded with an extreme load. You can also use a client-side load balancer, where each client checks the health of each S3 host in the cluster. One such load balancer is the open-source _Sidekick Load Balancer_.



**Related information**

[Round-robin DNS](https://en.wikipedia.org/wiki/Round-robin\_DNS)&#x20;

[Sidekick Load Balancer](https://github.com/minio/sidekick)



**Related topics**

[s3-cluster-management.md](s3-cluster-management.md "mention")

[s3-cluster-management-1.md](s3-cluster-management-1.md "mention")
