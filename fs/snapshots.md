---
description: >-
  Snapshots enable the saving of a filesystem state to a directory and can be
  used for backup, archiving and testing purposes.
---

# Snapshots

## About Snapshots

Snapshots allow the saving of a filesystem state to a `.snapshots`directory located under the root filesystem. They can be used for:

* **Physical Backup:** The snapshot directory can be copied into a different storage system, possibly on another site, using either the Weka system Snap-To-Object feature or a third-party software.
* **Logical Backup:** Periodic snapshots enable filesystem restoration to a previous state if logical data corruption occurs.
* **Archive:** Periodic snapshots enable the accessing of a previous filesystem state for compliance or other needs.
* **DevOps Environments:** Writable snapshots enable the execution of software tests on copies of the data.

Snapshots have practically no impact on system performance and can be taken for each filesystem while applications are running. They consume minimal space, according to the actual differences between the filesystem and the snapshots, or between the snapshots, in 4K granularity. By default, snapshots are read-only, and any attempts to change the contents of a read-only snapshot returns an error message. It is possible to create a writable snapshot or update an existing snapshot to be writable. However, a writable snapshot cannot be updated to be read-only.

The Weka system supports the following snapshot operations:

* Viewing snapshots
* Creating a snapshot of an existing filesystem
* Deleting a snapshot
* Accessing a snapshot under a dedicated directory name
* Restoring a filesystem from a snapshot
* Making snapshots writable
* Creating a snapshot of a snapshot (relevant for writable snapshots, or for read-only snapshots before being made writable)
* Listing of snapshots and obtaining their metadata

{% hint style="info" %}
**Note:** The number of snapshots per system is limited to 4,096 (the live filesystem consumes one of the total snapshots count).
{% endhint %}

{% hint style="info" %}
**Note:** The `.snapshots` directory is not listed. Running `ls` on the root of the filesystem will not show the `.snapshots` directory, however, it can be explicitly accessed, e.g. using the `cd .snapshots` command. &#x20;
{% endhint %}

## Managing Snapshots

{% hint style="info" %}
**Note:** To schedule snapshots automatically, refer to the [Snapshot Management](../appendix/snapshot-management.md) section of the Appendix.
{% endhint %}

### Viewing Snapshots

#### Viewing Snapshots Using the GUI

To view the snapshot of a filesystem, click the filesystem Manage Snapshots button.

![View Snapshot of a Filesystem Screen](<../.gitbook/assets/Snap view 3.5.png>)

If the filesystem is tiered, this screen is slightly different and includes an additional button (Upload to Object) and a corresponding Object Status value, as shown below:

![View Snapshot of a Tiered Filesystem Screen](<../.gitbook/assets/Snap view tiered 3.5.png>)

#### Viewing Snapshots Using the CLI

**Command:** `weka fs snapshot`

This command is used to display all snapshots of all filesystems in a single table.

### Creating a Snapshot

#### Creating a Snapshot Using the GUI

From the main snapshot view screen, click Create Snapshot at the top right-hand side of the required filesystem snapshot screen. The Create Snapshot dialog box will be displayed.

![Create Snapshot Dialog Box](<../.gitbook/assets/Snap create diallog 3.5.png>)

Enter the name and access point, determine whether it is writable and the source (the current filesystem or another snapshot). Then click Create to create the snapshot.

#### Creating a Snapshot Using the CLI

**Command:** `weka fs snapshot create`

Use the following command line to add a snapshot:

`weka fs snapshot create <file-system> <name> [--access-point access-point] [--source-snap=<source-snap>] [--is-writable]`

**Parameters in Command Line**

| **Name**       | **Type** | **Value**                                                                                                                  | **Limitations**      | **Mandatory** | **Default**                                                                                                                                                                                                                                                                                          |
| -------------- | -------- | -------------------------------------------------------------------------------------------------------------------------- | -------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `file-system`  | String   | A valid filesystem identifier                                                                                              | Must be a valid name | Yes           | ​                                                                                                                                                                                                                                                                                                    |
| `name`         | String   | Unique name for filesystem snapshot                                                                                        | Must be a valid name | Yes           |                                                                                                                                                                                                                                                                                                      |
| `access-point` | String   | Name of the newly-created directory for filesystem-level snapshots, which will serve as the access point for the snapshots | Must be a valid name | No            | Controlled by `weka fs snapshot access-point-naming-convention update <date/name>.` By default it is `<date>` format: `@GMT_%Y.%m.%d-%H.%M.%S` which is compatible with [windows previous versions format for SMB](../additional-protocols/smb-support/#integration-with-windows-previous-versions). |
| `source-snap`  | String   | Must be an existing snapshot                                                                                               | Must be a valid name | No            | Filesystem snapshot the file system                                                                                                                                                                                                                                                                  |
| `is-writable`  | Boolean  | Sets the created snapshot to be writable                                                                                   |                      | No            | False                                                                                                                                                                                                                                                                                                |

### Deleting a Snapshot

#### Deleting a Snapshot Using the GUI

In the main snapshot view screen, select the filesystem to be deleted and click Delete. The Snapshot Deletion window will be displayed.

![Snapshot Deletion Window](<../.gitbook/assets/Snap delete dialog 3.5.png>)

Click Delete to delete the selected snapshot.

#### Deleting a Snapshot Using the CLI

**Command:** `weka fs snapshot delete`

Use the following command line to delete a snapshot:

`weka fs snapshot delete <file-system> <name>`

**Parameters in Command Line**

| **Name**      | **Type** | **Value**                           | **Limitations**      | **Mandatory** | **Default** |
| ------------- | -------- | ----------------------------------- | -------------------- | ------------- | ----------- |
| `file-system` | String   | A valid filesystem identifier       | Must be a valid name | Yes           | ​           |
| `name`        | String   | Unique name for filesystem snapshot | Must be a valid name | Yes           |             |

{% hint style="warning" %}
**Note:** A snapshot deletion cannot happen in parallel to a snapshot upload to the same filesystem. Since uploading a snapshot to a remote object-store might take a while, it is advisable to delete the desired snapshots before uploading to the remote object-store.&#x20;

Also note, this becomes more important when uploading snapshots to both local and remote object stores. While local and remote uploads can progress in parallel, consider the case of a remote upload in progress, then a snapshot is deleted, and later a snapshot is uploaded to the local object-store. In this scenario, the local snapshot upload will wait for the pending deletion of the snapshot (which will happen only once the remote snapshot upload is done).
{% endhint %}

### Restoring a Filesystem from a Snapshot

#### Restoring a Filesystem or Snapshot from Another Snapshot Using the GUI

In the main snapshot view screen, select the filesystem snapshot to be restored and click Restore To. The Select Where to Restore window will be displayed.

![Select Where to Restore Window](<../.gitbook/assets/Snap restore dialog 3.5.png>)

Select the filesystem snapshot restore destination and click Restore.

#### Restoring a Filesystem or Snapshot from Another Snapshot Using the CLI

**Commands:** `weka fs restore` or `weka fs snapshot copy`

Use the following command line to restore a filesystem from a snapshot:

`weka fs restore <file-system> <source-name>`

Use the following command line to restore a snapshot to another snapshot:

`weka fs snapshot copy <file-system> <source-name> <destination-name>`

**Parameters in Command Lines**

| **Name**           | **Type** | **Value**                                                      | **Limitations**               | **Mandatory** | **Default** |
| ------------------ | -------- | -------------------------------------------------------------- | ----------------------------- | ------------- | ----------- |
| `file-system`      | String   | A valid filesystem identifier                                  | Must be a valid name          | Yes           | ​           |
| `source-name`      | String   | Unique name for the source of the snapshot                     | Must be a valid name          | Yes           |             |
| `destination-name` | String   | Name of the destination to which the snapshot should be copied | Must be an existing  snapshot | Yes           |             |

{% hint style="warning" %}
**Note:** When restoring a filesystem from a snapshot (or copying over an existing snapshot), the filesystem data and metadata are changed. Make sure IOs to the filesystem are stopped during this time.
{% endhint %}

### Updating a Snapshot

#### Updating a Snapshot Using the GUI

In the main snapshot view screen, select the filesystem snapshot to be updated and click Edit. The Update Snapshot window will be displayed.

![Update Snapshot Window](<../.gitbook/assets/Snap update dialog 3.5.png>)

Enter the name and access point, and then click Update to update the snapshot.

#### Updating a Snapshot Using the CLI

**Command:** `weka fs snapshot update`

This command changes the snapshot attributes. Use the following command line to update an existing snapshot:

`weka fs snapshot update <file-system> <name> [--new-name=<new-name>] [--access-point=<access-point>]`

**Parameters in Command Line**

| **Name**       | **Type** | **Value**                                                                                   | **Limitations**      | **Mandatory** | **Default** |
| -------------- | -------- | ------------------------------------------------------------------------------------------- | -------------------- | ------------- | ----------- |
| `file-system`  | String   | A valid filesystem identifier                                                               | Must be a valid name | Yes           | ​           |
| `name`         | String   | Unique name for the updated snapshot                                                        | Must be a valid name | Yes           |             |
| `new-name`     | String   | New name for the updated snapshot                                                           | Must be a valid name | No            |             |
| `access-point` | String   | Name of a directory for the snapshot, which will serve as the access point for the snapshot | Must be a valid name | No            |             |

## Working with Snapshots

Note the following concerning working with snapshots:

1. When moving a file in or out of a snapshot directory, or between snapshots, the kernel will implement the move operation as a copy operation, similar to moving a file between two different filesystems. Such operations for directories will fail.
2. If symbolic links are accessed via the `.snapshots` directory, the symlinks with absolute paths can lead to the current filesystem. Consequently, depending on the usage, it may be preferable not to follow symlinks or to use relative paths.
