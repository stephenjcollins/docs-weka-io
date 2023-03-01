---
description: >-
  This page describes the concept of organizations and how different Weka system
  features operate within an organizational context.
---

# Organizations management

Organizations are used for the separation of duties between different groups of users on the same Weka system. So that an organization cannot control or view other organization data. It is possible to create up to 64 organizations.

Within an organization, the Organization Admin manages the logical entities participating in obtaining control of data (the Cluster Admin cannot manage these entities).

The Cluster Admin can perform the following activities:

* Create new organizations and define the Organization Admin.
* Delete existing organizations.
* Monitor per organization the total capacity used by all the organization filesystems.

‌While Cluster Admins are people trusted by the different organizations (for example, have root access to the backend servers), they are obscured from the organization data in the Weka system. The Cluster Admin separation is partial, for example, they can still see the events of all organizations. The Weka system ensures the separation of any sensitive information between the different organizations.

{% hint style="info" %}
**Note:** The data at the hardware level is not separated. While the Weka system is highly scalable and serves IOs fairly among filesystems, there is no QoS guarantee between organizations. The system limits are according to the entire system. Consequently, a single organization's workload or configuration can exhaust the entire cluster limits.
{% endhint %}

## Organization management use cases&#x20;

### Private cloud multi-tenancy

Working with organizations makes it possible to manage different departments. While this requires more configuration, for example, different LDAP configurations are usually unnecessary between different departments in the same organization, the Cluster Admin is fully trusted.

It is possible to separate and obscure specific departments, such as IT, Finance, Life Sciences, Genomics, and even specific projects in departments.

### Logical separation of external user groups

When multiple independent groups use the same infrastructure, the use of multiple organizations provides much better security, obscuration, and separation of data.

## Cluster level entities

The Cluster Admin manages the following entities at the cluster level:

* Hardware
* NFS service (NFS groups and IP/interfaces)
* SMB service
* Filesystem groups - definition of tiering policies for the different groups, while the Organization Admin selects the filesystem group from the predefined list of groups for each filesystem created
* KMS

## Organization level entities

At the organization level, only the relevant Organization Admin manages all system entities, while the users can only view the system entities within the organization.

Cluster Admins **do not** have permissions to view or manage the system entities within the organization, which include the following:

* Filesystems, and the option to mount the filesystems (also a Cluster Adminfile cannot  mount  the filesystems)
* Object store buckets
* LDAP server
* NFS exports (NFS client permissions)

{% hint style="warning" %}
**Note**: Different protocols are not supported other than in the **root** organization.
{% endhint %}

{% hint style="warning" %}
**Note:** Only exports of the 'legacy' NFS stack can be managed within a **non-root** organization.
{% endhint %}

## Manage organizations

Only users defined as Cluster Admins can manage organizations. When no organization is created, the root organization is the default organization and all operations are regular. That is, it is not necessary to authenticate the mounts or supply an organization name when logging in using the GUI/CLI.

Once a new organization is created, the organization name must be provided in every login command, using the `--org` attribute in the `weka user login` command.

## Usage and quota management

Cluster Admins can view an organization's usage (both SSD and total) and can limit usage with quotas per organization. This can be leveraged for charge-backs on either used or allocated capacity of SSD or object store data.

## Organization admin role privileges

When a new organization is created, the Cluster Admin creates an Organization Admin user for the organization, who is the administrator within the organization responsible for managing each [organization level entity.](./#organization-level-entities)

Organization Admins have similar privileges to Cluster Admins, except that these privileges are limited to the organization level. They can perform the following within the organization:

* Create new users
* Delete existing users
* Change user passwords
* Set user roles
* Manage the organization's LDAP configuration

To avoid situations where an Organization Admin loses access to a Weka system cluster, the following restrictions are implemented on Organization Admins:

* Cannot delete themselves
* Cannot change their role