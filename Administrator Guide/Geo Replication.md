#Managing Geo-replication

Geo-replication provides a continuous, asynchronous, and incremental
replication service from one site to another over Local Area Networks
(LANs), Wide Area Network (WANs), and across the Internet.

Geo-replication uses a master–slave model, whereby replication and
mirroring occurs between the following partners:

-   **Master** – a GlusterFS volume

-   **Slave** – a slave which can be of the following types:

    -   A local directory which can be represented as file URL like
        `file:///path/to/dir`. You can use shortened form, for example,
        ` /path/to/dir`.

    -   A GlusterFS Volume - Slave volume can be either a local volume
        like `gluster://localhost:volname` (shortened form - `:volname`)
        or a volume served by different host like
        `gluster://host:volname` (shortened form - `host:volname`).

    > **Note**
    >
    > Both of the above types can be accessed remotely using SSH tunnel.
    > To use SSH, add an SSH prefix to either a file URL or gluster type
    > URL. For example, ` ssh://root@remote-host:/path/to/dir`
    > (shortened form - `root@remote-host:/path/to/dir`) or
    > `ssh://root@remote-host:gluster://localhost:volname` (shortened
    > from - `root@remote-host::volname`).

This section introduces Geo-replication, illustrates the various
deployment scenarios, and explains how to configure the system to
provide replication and mirroring in your environment.

##Replicated Volumes vs Geo-replication

The following table lists the difference between replicated volumes and
geo-replication:

  Replicated Volumes | Geo-replication
  --- | ---
  Mirrors data across clusters | Mirrors data across geographically distributed clusters
  Provides high-availability | Ensures backing up of data for disaster recovery
  Synchronous replication (each and every file operation is sent across all the bricks) | Asynchronous replication (checks for the changes in files periodically and syncs them on detecting differences)

##Preparing to Deploy Geo-replication

This section provides an overview of the Geo-replication deployment
scenarios, describes how you can check the minimum system requirements,
and explores common deployment scenarios.

##Exploring Geo-replication Deployment Scenarios

Geo-replication provides an incremental replication service over Local
Area Networks (LANs), Wide Area Network (WANs), and across the Internet.
This section illustrates the most common deployment scenarios for
Geo-replication, including the following:

-   Geo-replication over LAN
-   Geo-replication over WAN
-   Geo-replication over the Internet
-   Multi-site cascading Geo-replication

**Geo-replication over LAN**

You can configure Geo-replication to mirror data over a Local Area
Network.

![geo-rep_lan](https://cloud.githubusercontent.com/assets/10970993/7412281/a542e724-ef5e-11e4-8207-9e018c1e9304.png)

**Geo-replication over WAN**

You can configure Geo-replication to replicate data over a Wide Area
Network.

![geo-rep_wan](https://cloud.githubusercontent.com/assets/10970993/7412292/c3816f76-ef5e-11e4-8daa-271f6efa1f58.png)

**Geo-replication over Internet**

You can configure Geo-replication to mirror data over the Internet.

![geo-rep03_internet](https://cloud.githubusercontent.com/assets/10970993/7412305/d8660050-ef5e-11e4-9d1b-54369fb1e43f.png)

**Multi-site cascading Geo-replication**

You can configure Geo-replication to mirror data in a cascading fashion
across multiple sites.

![geo-rep04_cascading](https://cloud.githubusercontent.com/assets/10970993/7412320/05e131bc-ef5f-11e4-8580-a4dc592148ff.png)

##Geo-replication Deployment Overview

Deploying Geo-replication involves the following steps:

1.  Verify that your environment matches the minimum system requirement.
2.  Determine the appropriate deployment scenario.
3.  Start Geo-replication on master and slave systems, as required.

##Checking Geo-replication Minimum Requirements

Before deploying GlusterFS Geo-replication, verify that your systems
match the minimum requirements.

The following table outlines the minimum requirements for both master
and slave nodes within your environment:

  Component                | Master                                                                | Slave
  ---                      | ---                                                                   | ---
  Operating System         | GNU/Linux                                                             | GNU/Linux
  Filesystem               | GlusterFS 3.2 or higher                                               | GlusterFS 3.2 or higher (GlusterFS needs to be installed, but does not need to be running), ext3, ext4, or XFS (any other POSIX compliant file system would work, but has not been tested extensively)
  Python                   | Python 2.4 (with ctypes external module), or Python 2.5 (or higher)   | Python 2.4 (with ctypes external module), or Python 2.5 (or higher)
  Secure shell             | OpenSSH version 4.0 (or higher)                                       | SSH2-compliant daemon
  Remote synchronization   | rsync 3.0.7 or higher                                                 | rsync 3.0.7 or higher
  FUSE                     | GlusterFS supported versions                                          | GlusterFS supported versions

##Setting Up the Environment for Geo-replication

**Time Synchronization**

-   On bricks of a geo-replication master volume, all the servers' time
    must be uniform. You are recommended to set up NTP (Network Time
    Protocol) service to keep the bricks sync in time and avoid
    out-of-time sync effect.

    For example: In a Replicated volume where brick1 of the master is at
    12.20 hrs and brick 2 of the master is at 12.10 hrs with 10 minutes
    time lag, all the changes in brick2 between this period may go
    unnoticed during synchronization of files with Slave.

**To setup Geo-replication for SSH**

Password-less login has to be set up between the host machine (where
geo-replication Start command will be issued) and the remote machine
(where slave process should be launched through SSH).

1.  On the node where geo-replication sessions are to be set up, run the
    following command:

        # ssh-keygen -f /var/lib/glusterd/geo-replication/secret.pem

    Press Enter twice to avoid passphrase.

2.  Run the following command on master for all the slave hosts:

        # ssh-copy-id -i /var/lib/glusterd/geo-replication/secret.pem.pub @

##Setting Up the Environment for a Secure Geo-replication Slave

You can configure a secure slave using SSH so that master is granted a
restricted access. With GlusterFS, you need not specify configuration
parameters regarding the slave on the master-side configuration. For
example, the master does not require the location of the rsync program
on slave but the slave must ensure that rsync is in the PATH of the user
which the master connects using SSH. The only information that master
and slave have to negotiate are the slave-side user account, slave's
resources that master uses as slave resources, and the master's public
key. Secure access to the slave can be established using the following
options:

-   Restricting Remote Command Execution

-   Using `Mountbroker` for Slaves

-   Using IP based Access Control

**Backward Compatibility**

Your existing Ge-replication environment will work with GlusterFS,
except for the following:

-   The process of secure reconfiguration affects only the glusterfs
    instance on slave. The changes are transparent to master with the
    exception that you may have to change the SSH target to an
    unprivileged account on slave.

-   The following are the some exceptions where this might not work:

    -   Geo-replication URLs which specify the slave resource when
        configuring master will include the following special
        characters: space, \*, ?, [;

    -   Slave must have a running instance of glusterd, even if there is
        no gluster volume among the mounted slave resources (that is,
        file tree slaves are used exclusively).

### Restricting Remote Command Execution

If you restrict remote command execution, then the Slave audits commands
coming from the master and the commands related to the given
geo-replication session is allowed. The Slave also provides access only
to the files within the slave resource which can be read or manipulated
by the Master.

To restrict remote command execution:

1.  Identify the location of the gsyncd helper utility on Slave. This
    utility is installed in `PREFIX/libexec/glusterfs/gsyncd`, where
    PREFIX is a compile-time parameter of glusterfs. For example,
    `--prefix=PREFIX` to the configure script with the following common
    values` /usr, /usr/local, and /opt/glusterfs/glusterfs_version`.

2.  Ensure that command invoked from master to slave passed through the
    slave's gsyncd utility.

    You can use either of the following two options:

    -   Set gsyncd with an absolute path as the shell for the account
        which the master connects through SSH. If you need to use a
        privileged account, then set it up by creating a new user with
        UID 0.

    -   Setup key authentication with command enforcement to gsyncd. You
        must prefix the copy of master's public key in the Slave
        account's `authorized_keys` file with the following command:

        `command=<path to gsyncd>`.

        For example,
        `command="PREFIX/glusterfs/gsyncd" ssh-rsa AAAAB3Nza....`

### Using Mountbroker for Slaves

`mountbroker` is a new service of glusterd. This service allows an
unprivileged process to own a GlusterFS mount by registering a label
(and DSL (Domain-specific language) options ) with glusterd through a
glusterd volfile. Using CLI, you can send a mount request to glusterd to
receive an alias (symlink) of the mounted volume.

A request from the agent , the unprivileged slave agents use the
mountbroker service of glusterd to set up an auxiliary gluster mount for
the agent in a special environment which ensures that the agent is only
allowed to access with special parameters that provide administrative
level access to the particular volume.

**To setup an auxiliary gluster mount for the agent**:

1.  In all Slave nodes, create a new group. For example, `geogroup`.

2.  In all Slave nodes, create a unprivileged account. For example, ` geoaccount`. Make it a
    member of ` geogroup`.

3.  In all Slave nodes, Create a new directory owned by root and with permissions *0711.*
    For example, create a create mountbroker-root directory
    `/var/mountbroker-root`.

4.  In any one of Slave node, Run the following commands to add options to glusterd vol
file(`/etc/glusterfs/glusterd.vol`)
    in rpm installations and `/usr/local/etc/glusterfs/glusterd.vol` in Source installation.

    ```sh
    gluster system:: execute mountbroker opt mountbroker-root /var/mountbroker-root
    gluster system:: execute mountbroker opt geo-replication-log-group geogroup
    gluster system:: execute mountbroker opt rpc-auth-allow-insecure on
    ```

5.  In any one of the Slave node, Add Mountbroker user to glusterd vol file using,

    ```sh
    gluster system:: execute mountbroker user geoaccount slavevol
    ```

    where slavevol is the Slave Volume name

    If you host multiple slave volumes on Slave, for each of them and add the following options
to the volfile using,

    ```sh
    gluster system:: execute mountbroker user geoaccount2 slavevol2
    gluster system:: execute mountbroker user geoaccount3 slavevol3
    ```

    To add multiple volumes per mountbroker user,

    ```sh
    gluster system:: execute mountbroker user geoaccount1 slavevol11,slavevol12,slavevol13
    gluster system:: execute mountbroker user geoaccount2 slavevol21,slavevol22
    gluster system:: execute mountbroker user geoaccount3 slavevol31
    ```
6.  Restart `glusterd` service on all Slave nodes.

7.  Setup a passwdless SSH from one of the master node to the user on one of the slave node.
For example, to geoaccount.

8.  Create a geo-replication relationship between master and slave to the user by running the
following command on the master node:

    ```sh
    gluster volume geo-replication <master_volume> <mountbroker_user>@<slave_host>::<slave_volume>
    create push-pem [force]
    ```

9.  In the slavenode, which is used to create relationship, run `/usr/libexec/glusterfs/set_geo_rep_pem_keys.sh`
as a root with user name, master volume name, and slave volume names as the arguments.

    ```sh
    /usr/libexec/glusterfs/set_geo_rep_pem_keys.sh <mountbroker_user> <master_volume> <slave_volume>
    ```

### Using IP based Access Control

You can use IP based access control method to provide access control for
the slave resources using IP address. You can use method for both Slave
and file tree slaves, but in the section, we are focusing on file tree
slaves using this method.

To set access control based on IP address for file tree slaves:

1.  Set a general restriction for accessibility of file tree resources:

        # gluster volume geo-replication '/*' config allow-network ::1,127.0.0.1

    This will refuse all requests for spawning slave agents except for
    requests initiated locally.

2.  If you want the to lease file tree at `/data/slave-tree` to Master,
    enter the following command:

        # gluster volume geo-replicationconfig allow-network

    `MasterIP` is the IP address of Master. The slave agent spawn
    request from master will be accepted if it is executed at
    `/data/slave-tree`.

If the Master side network configuration does not enable the Slave to
recognize the exact IP address of Master, you can use CIDR notation to
specify a subnet instead of a single IP address as MasterIP or even
comma-separated lists of CIDR subnets.

If you want to extend IP based access control to gluster slaves, use the
following command:

        # gluster volume geo-replication '*' config allow-network ::1,127.0.0.1

##Starting Geo-replication

This section describes how to configure and start Gluster
Geo-replication in your storage environment, and verify that it is
functioning correctly.

###Starting Geo-replication

To start Gluster Geo-replication

-   Use the following command to start geo-replication between the hosts:

        # gluster volume geo-replication  start

    For example:

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir start
        Starting geo-replication session between Volume1
        example.com:/data/remote_dir has been successful

    > **Note**
    >
    > You may need to configure the service before starting Gluster
    > Geo-replication.

###Verifying Successful Deployment

You can use the gluster command to verify the status of Gluster
Geo-replication in your environment.

**To verify the status Gluster Geo-replication**

-   Verify the status by issuing the following command on host:

        # gluster volume geo-replication  status

    For example:

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir status
        # gluster volume geo-replication Volume1 example.com:/data/remote_dir status
        MASTER    SLAVE                            STATUS
        ______    ______________________________   ____________
        Volume1 root@example.com:/data/remote_dir  Starting....

###Displaying Geo-replication Status Information

You can display status information about a specific geo-replication
master session, or a particular master-slave session, or all
geo-replication sessions, as needed.

**To display geo-replication status information**

-   Use the following command to display information of all geo-replication sessions:

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir status

-   Use the following command to display information of a particular master slave session:

        # gluster volume geo-replication  status

    For example, to display information of Volume1 and
    example.com:/data/remote\_dir

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir status

    The status of the geo-replication between Volume1 and
    example.com:/data/remote\_dir is displayed.

-   Display information of all geo-replication sessions belonging to a
    master

        # gluster volume geo-replication MASTER status

    For example, to display information of Volume1

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir status

    The status of a session could be one of the following:

-   **Initializing**: This is the initial phase of the Geo-replication session;
    it remains in this state for a minute in order to make sure no abnormalities are present.

-   **Not Started**: The geo-replication session is created, but not started.

-   **Active**: The gsync daemon in this node is active and syncing the data.

-   **Passive**: A replica pair of the active node. The data synchronization is handled by active node.
    Hence, this node does not sync any data.

-   **Faulty**: The geo-replication session has experienced a problem, and the issue needs to be
    investigated further.

-   **Stopped**: The geo-replication session has stopped, but has not been deleted.

    The Crawl Status can be one of the following:

-   **Changelog Crawl**: The changelog translator has produced the changelog and that is being consumed
    by gsyncd daemon to sync data.

-   **Hybrid Crawl**: The gsyncd daemon is crawling the glusterFS file system and generating pseudo
    changelog to sync data.

-   **Checkpoint Status**: Displays the status of the checkpoint, if set. Otherwise, it displays as N/A.

##Configuring Geo-replication

To configure Gluster Geo-replication

-   Use the following command at the Gluster command line:

        # gluster volume geo-replication  config [options]

    For example:

    Use the following command to view list of all option/value pair:

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir config

####Configurable Options

The following table provides an overview of the configurable options for a geo-replication setting:

  Option                        | Description
  ---                           | ---
  gluster-log-file LOGFILE 	| The path to the geo-replication glusterfs log file.
  gluster-log-level LOGFILELEVEL| The log level for glusterfs processes.
  log-file LOGFILE 	        | The path to the geo-replication log file.
  log-level LOGFILELEVEL 	| The log level for geo-replication.
  ssh-command COMMAND 	        | The SSH command to connect to the remote machine (the default is SSH).
  rsync-command COMMAND 	| The rsync command to use for synchronizing the files (the default is rsync).
  use-tarssh true 	        | The use-tarssh command allows tar over Secure Shell protocol. Use this option to handle workloads of files that have not undergone edits.
  volume_id=UID 	        | The command to delete the existing master UID for the intermediate/slave node.
  timeout SECONDS 	        | The timeout period in seconds.
  sync-jobs N 	                | The number of simultaneous files/directories that can be synchronized.
  ignore-deletes 	        | If this option is set to 1, a file deleted on the master will not trigger a delete operation on the slave. As a result, the slave will remain as a superset of the master and can be used to recover the master in the event of a crash and/or accidental delete.
  checkpoint [LABEL&#124;now] 	| Sets a checkpoint with the given option LABEL. If the option is set as now, then the current time will be used as the label.

##Stopping Geo-replication

You can use the gluster command to stop Gluster Geo-replication (syncing
of data from Master to Slave) in your environment.

**To stop Gluster Geo-replication**

-   Use the following command to stop geo-replication between the hosts:

        # gluster volume geo-replication  stop

    For example:

        # gluster volume geo-replication Volume1 example.com:/data/remote_dir stop
        Stopping geo-replication session between Volume1 and
        example.com:/data/remote_dir has been successful

##Restoring Data from the Slave

You can restore data from the slave to the master volume, whenever the
master volume becomes faulty for reasons like hardware failure.

The example in this section assumes that you are using the Master Volume
(Volume1) with the following configuration:

    machine1# gluster volume info
    Type: Distribute
    Status: Started
    Number of Bricks: 2
    Transport-type: tcp
    Bricks:
    Brick1: machine1:/export/dir16
    Brick2: machine2:/export/dir16
    Options Reconfigured:
    geo-replication.indexing: on

The data is syncing from master volume (Volume1) to slave directory
(example.com:/data/remote\_dir). To view the status of this
geo-replication session run the following command on Master:

    # gluster volume geo-replication Volume1 root@example.com:/data/remote_dir status

**Before Failure**

Assume that the Master volume had 100 files and was mounted at
/mnt/gluster on one of the client machines (client). Run the following
command on Client machine to view the list of files:

    client# ls /mnt/gluster | wc –l
    100

The slave directory (example.com) will have same data as in the master
volume and same can be viewed by running the following command on slave:

    example.com# ls /data/remote_dir/ | wc –l
    100

**After Failure**

If one of the bricks (machine2) fails, then the status of
Geo-replication session is changed from "OK" to "Faulty". To view the
status of this geo-replication session run the following command on
Master:

        # gluster volume geo-replication Volume1 root@example.com:/data/remote_dir status

Machine2 is failed and now you can see discrepancy in number of files
between master and slave. Few files will be missing from the master
volume but they will be available only on slave as shown below.

Run the following command on Client:

        client # ls /mnt/gluster | wc –l
        52

Run the following command on slave (example.com):

        Example.com# # ls /data/remote_dir/ | wc –l
        100

**To restore data from the slave machine**

1.  Use the following command to stop all Master's geo-replication sessions:

        # gluster volume geo-replication  stop

    For example:

        machine1# gluster volume geo-replication Volume1
        example.com:/data/remote_dir stop

        Stopping geo-replication session between Volume1 &
        example.com:/data/remote_dir has been successful

    > **Note**
    >
    > Repeat `# gluster volume geo-replication  stop `command on all
    > active geo-replication sessions of master volume.

2.  Use the following command to replace the faulty brick in the master:

        # gluster volume replace-brick  start

    For example:

        machine1# gluster volume replace-brick Volume1 machine2:/export/dir16 machine3:/export/dir16 start
        Replace-brick started successfully

3.  Use the following command to commit the migration of data:

        # gluster volume replace-brick  commit force

    For example:

        machine1# gluster volume replace-brick Volume1 machine2:/export/dir16 machine3:/export/dir16 commit force
        Replace-brick commit successful

4.  Use the following command to verify the migration of brick by viewing the volume info:

        # gluster volume info

    For example:

        machine1# gluster volume info
        Volume Name: Volume1
        Type: Distribute
        Status: Started
        Number of Bricks: 2
        Transport-type: tcp
        Bricks:
        Brick1: machine1:/export/dir16
        Brick2: machine3:/export/dir16
        Options Reconfigured:
        geo-replication.indexing: on

5.  Run rsync command manually to sync data from slave to master
    volume's client (mount point).

    For example:

        example.com# rsync -PavhS --xattrs --ignore-existing /data/remote_dir/ client:/mnt/gluster

    Verify that the data is synced by using the following command:

    On master volume, run the following command:

        Client # ls | wc –l
        100

    On the Slave run the following command:

        example.com# ls /data/remote_dir/ | wc –l
        100

    Now Master volume and Slave directory is synced.

6.  Use the following command to restart geo-replication session from master to slave:

        # gluster volume geo-replication  start

    For example:

        machine1# gluster volume geo-replication Volume1
        example.com:/data/remote_dir start
        Starting geo-replication session between Volume1 &
        example.com:/data/remote_dir has been successful

##Best Practices

**Manually Setting Time**

If you have to change the time on your bricks manually, then you must
set uniform time on all bricks. Setting time backward corrupts the
geo-replication index, so the recommended way to set the time manually is:

1.  Stop geo-replication between the master and slave using the
    following command:

        # gluster volume geo-replication stop

2.  Stop the geo-replication indexing using the following command:

        # gluster volume set  geo-replication.indexing of

3.  Set uniform time on all bricks.

4.  Use the following command to restart your geo-replication session:

        # gluster volume geo-replication start

**Running Geo-replication commands in one system**

It is advisable to run the geo-replication commands in one of the bricks
in the trusted storage pool. This is because, the log files for the
geo-replication session would be stored in the \*Server\* where the
Geo-replication start is initiated. Hence it would be easier to locate
the log-files when required.

**Isolation**

Geo-replication slave operation is not sandboxed as of now and is ran as
a privileged service. So for the security reason, it is advised to
create a sandbox environment (dedicated machine / dedicated virtual
machine / chroot/container type solution) by the administrator to run
the geo-replication slave in it. Enhancement in this regard will be
available in follow-up minor release.
