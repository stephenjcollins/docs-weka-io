---
description: >-
  This page describes how to manage quotas to alert or restrict usage of the
  WekaFS filesystem.
---

# Quota management

## Overview

There are several levels on the Weka system where capacity usage can be restricted.&#x20;

1. On an organization level: Set a different organization to manage its own filesystems, where quotas for an organization can be set, as described in the [organization's usage and quota management ](../usage/organizations/#usage-and-quota-management)section.
2. On a filesystem level: Set a different filesystem per department/project.
3. On a directory level: Set a different quota per project directory (useful when users are part of several projects) or per user home directory.

## Directory quotas

The organization admin can set a quota on a directory. Setting a quota will start the process of counting the current directory usage. Until this process is done, the quota will not be taken into account (for empty directories, this process is instantly done).

{% hint style="info" %}
**Note:** Currently, a mount point to the relevant filesystem is required to set a quota on a directory, and the quota set command should not be interrupted until the quota accounting is over.
{% endhint %}

The organization admin sets quotas to inform/restrict users from using too much of the filesystem capacity. For that, only data in the user's control is considered. Hence, the quota doesn't count the overhead of the protection bits and snapshots. It does take into account data and metadata of files in the directory, regardless if tiered or not.&#x20;

### Working with quotas

When working with quotas, consider the following:

* To set a quota, the relevant filesystem must be mounted on the host where the set quota command is to be run.
* When setting a quota, you should go through a new mount-point. This means that if you are using a host that has mounts from Weka versions before 3.10, first unmount all relevant mount point and then mount them again.
* Quotas can be set within nested directories (up to 4 levels of nested quotas are supported) and over-provisioned under the same directory quota tree. E.g., `/home` can have a quota of 1TiB, and each user directory under it can have a quota of 10GiB, while there are 200 users.
* Moving files (or directories) between two directories with quotas, into a directory with a quota, or outside a directory with a quota is not supported. The WekaFS filesystem returns `EXDEV` in such a case, which is usually converted by the operating system to copy and delete but is OS-dependent.
* Quotas and hardlinks:
  * An existing hardlink is not counted as part of the quota.
  * Once a directory has a quota, it is not allowed to create a hardlink to files residing under directories with different (or without) directory quotas.
* Restoring a filesystem from a snapshot turns the quotas back to the configuration at the time of the snapshot.
* Creating a new filesystem from a snap-2-obj does not preserve the original quotas.
* When working with enforcing quotas along with a `writecache` mount-mode, similarly to other POSIX solutions, getting above the quota might not sync all the cache writes to the backend servers. Use `sync`, `syncfs`, or `fsync` to commit the cached changes to the system (or fail due to exceeding the quota).

### Integration with the `df` utility

When a hard quota is set on a directory, running the `df` utility will consider the hard quota as the total capacity of the directory and provide the `use%` relative to the quota. This can help users understand their usage and how close they are to the hard quota.

{% hint style="info" %}
**Note:** The `df` utility behavior with quotas is currently global to the Weka system.&#x20;

To change the global behavior, contact the Weka Support Team.
{% endhint %}

## Manage quotas using the CLI

### Set directory quotas/default quotas

**Command**: `weka fs quota set` / `weka fs quota set-default`

Use the following commands to set a directory quota:

`weka fs quota set <path> [--soft soft] [--hard hard] [--grace grace] [--owner owner]`

It is also possible to set a default quota on a directory. It does not account for this directory (or existing child directories) but will automatically set the quota on **new** directories created directly under it. Use the following command to set a default quota of a directory:

`weka fs quota set-default <path>  [--soft soft] [--hard hard] [--grace grace] [--owner owner]`

#### &#x20;**Parameters**

| **Name** | **Type** | **Value**                                                                                                                                                                          | **Limitations**                                                                      | **Mandatory** | **Default** |
| -------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ------------- | ----------- |
| `path`   | String   | Path to the directory to set the quota on.                                                                                                                                         | The relevant filesystem must be mounted when setting the quota.                      | Yes           | ​           |
| `soft`   | Number   | Soft quota limit; Exceeding this number will be shown as exceeded quota but will not be enforced until the `grace` period is over.                                                 | Capacity in decimal or binary units, e.g.: `1GB`, `1TB`, `1GiB`, `1TiB`, `unlimited` | No            | `unlimited` |
| `hard`   | Number   | Hard quota limit; Exceeding this number will not allow any more writes before clearing some space in the directory.                                                                | Capacity in decimal or binary units, e.g.: `1GB`, `1TB`, `1GiB`, `1TiB`, `unlimited` | No            | `unlimited` |
| `grace`  | Number   | Specify the grace period before the soft limit is treated as a hard limit.                                                                                                         | Format: `1d`, `1w`, `unlimited`                                                      | No            | `unlimited` |
| `owner`  | String   | An opaque string identifying the directory owner (can be a name, email, slack ID, etc.) This owner will be shown in the quota report and can be notified upon exceeding the quota. | Up to 48 characters.                                                                 | No            |             |

{% hint style="info" %}
**Note:** To set advisory-only quotas, use a `soft` quota limit without setting a `grace` period.
{% endhint %}

{% hint style="info" %}
**Note:** When both `hard` and `soft` quotas exist, setting the value of one of them to `0` will clear this quota.
{% endhint %}

### List directory quotas/default quotas

**Command**: `weka fs quota list` / `weka fs quota list-default`

Use the following command to list the directory quotas (by default, only exceeding quotas are listed) :

`weka fs quota list [fs-name] [--snap-name snap-name] [--path path] [--under under] [--over over] [--quick] [--all]`

#### **Parameters**

| **Name**    | **Type** | **Value**                                                                                     | **Limitations**                                                                    | **Mandatory** | **Default**     |
| ----------- | -------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------------- | --------------- |
| `fs-name`   | String   | Shows quota report only on the specified filesystem.                                          | A valid wekafs filesystem name.                                                    | No            | All filesystems |
| `snap-name` | String   | Shows the quota report from the time of the snapshot.                                         | Must be a valid snapshot name and be given along with the corresponding `fs-name.` | No            |                 |
| `path`      | String   | Path to a directory. Shows quota report only on the specified directory.                      | The relevant filesystem must be mounted in the host running the query.             | No            |                 |
| `under`     | String   | A path to a directory under a wekafs mount.                                                   | The relevant filesystem must be mounted in the host running the query.             | No            |                 |
| `over`      | Number   | Shows only quotas over this percentage of usage                                               | 0-100                                                                              | No            |                 |
| `quick`     | Boolean  | Do not resolve inode to a path (provides quicker result if the report contains many entries). |                                                                                    | No            | False           |
| `all`       | Boolean  | Shows all the quotas, not just the exceeding ones.                                            |                                                                                    | No            | False           |

Use the following command to list the directory default quotas:

`weka fs quota list-default [fs-name] [--snap-name snap-name] [--path path]`

#### **Parameters**

| **Name**    | **Type** | **Value**                                                                             | **Limitations**                                                                    | **Mandatory** | **Default**     |
| ----------- | -------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------------- | --------------- |
| `fs-name`   | String   | Shows the default quotas of the specified filesystem only.                            | A valid wekafs filesystem name.                                                    | No            | All filesystems |
| `snap-name` | String   | Shows the default quotas from the time of the snapshot.                               | Must be a valid snapshot name and be given along with the corresponding `fs-name.` | No            |                 |
| `path`      | String   | Path to a directory. Shows the default quotas report only on the specified directory. | The relevant filesystem must be mounted in the host running the query.             | No            |                 |

### Unsetting directory quotas/default quotas

**Command**: `weka fs quota unset` / `weka fs quota unset-default`

Use the following commands to unset a directory quota:

`weka fs quota unset <path>`

Use the following command to unset a default quota of a directory:

`weka fs quota unset-default <path>`

#### **Parameters**

| **Name** | **Type** | **Value**                                 | **Limitations**                                                | **Mandatory** | **Default** |
| -------- | -------- | ----------------------------------------- | -------------------------------------------------------------- | ------------- | ----------- |
| `path`   | String   | Path to the directory to set the quota on | The relevant filesystem must be mounted when setting the quota | Yes           | ​           |
