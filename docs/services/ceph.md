
# Ceph Storage

References: 

* http://docs.ceph.com/docs/mimic/start/quick-start-preflight/
* http://docs.ceph.com/docs/mimic/start/quick-ceph-deploy/
* https://nbailey.ca/post/cephfs-kvm-virtual-san/ (OC)

## Manager Node
First, install the ceph deployment tools on the manager node. 

```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
sudo apt-add-repository 'deb https://download.ceph.com/debian-luminous/ bionic main'
sudo apt update
sudo apt install ceph-deploy
```

## OSD Nodes

NTP should be configured on OSD nodes. 

```
sudo apt install ntp
```

The `cephsvc` service account must exist on all the nodes: 

```
sudo useradd -d /home/cephsvc -m cephsvc
sudo passwd cephsvc
> hunter2 or something

echo "cephsvc ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephsvc
sudo chmod 0440 /etc/sudoers.d/cephsvc
```

Install Python on the OSD nodes: 

	ssh {osd1,osd2,osd3} sudo apt-add-repository multiverse
	ssh {osd1,osd2,osd3} sudo apt install python-minimal -y

## SSH Keys

An SSH key should exist on the manager node. 

```
ssh-keygen
...

ssh-copy-id cephsvc@osd1
ssh-copy-id cephsvc@osd2
ssh-copy-id cephsvc@osd3
```

## SSH Config

Specify a user to use for each node: 

```
Host osd1
   Hostname osd1
   User osdsvc
Host osd2
   Hostname osd2
   User osdsvc
Host osd3
   Hostname osd3
   User osdsvc
```

## Firewall Config

Make sure that the correct ports are open: 

```
sudo ufw allow from 10.55.66.0/24 to any port 6800:7300/tcp 
sudo ufw allow from 10.55.66.0/24 to any port 6789
```

# Deploying Ceph Cluster

In this section, separate admin node is used to issue the initial commands to bootstrap the cluster. This requires that the firewall, ssh keys, and packages are installed as specified above. 

## Create Cluster with Ceph-Deploy

Prepare data directory: 

```
mkdir cluster-config
cd cluster-config
```

Specify inital monitor nodes for install: 

	ceph-deploy new osd1 osd2 osd3
		
In `ceph.conf` specify the network of the ceph cluster. Though some documentation indicates this is not mandatory, it appears to fail during monitor deployment if this isn't specified explicitly. 

	public network = 10.55.66.0/24
		
Install the ceph packages on the nodes: 

	ceph-deploy install osd1 osd2 osd3
		
Deploy monitors and gather keys: 

	ceph-deploy mon create-initial
		
Install the ceph keys and cluster configuration to each node: 

	ceph-deploy admin osd1 osd2 osd3
		
Install the manager node: 

	ceph-deploy mgr create osd1
		
## Provision Object Store Daemons 

Create three OSDs. These will claim and __overwrite__ any contents of the specified disk. _Be careful!_

	ceph-deploy osd create --data /dev/sdb osd1
	ceph-deploy osd create --data /dev/sdb osd2
	ceph-deploy osd create --data /dev/sdb osd3
		
Check the health of the cluster

	ssh osd1 sudo ceph health


## Metadata Servers

A metadata node is required to use CephFS. 

Create at least one MDS on the cluster: 

	ceph-deploy mds create osd1 osd2 osd3

## Manager Nodes

At least one manager is required. It is recommended to have several in a cluster for high availability. In this case, add additional managers to the first in the cluster, `osd1`

	ceph-deploy mgr create osd2 osd3

# Ceph Cluster Operations

## Storage Pools

A pool is the lowest level unit of data in Ceph. CephFS, RBD, and Swift are all ways to expose pools to different connectivity types. 

Pool Type      | Fault Tolerance | Storage Space 
-------------- | --------------- | -----------
 Replicated    | High            | Low
 Erasure Coded | Low             | High

 When creating a pool, it's important to pick an appropriate placement group identifier. [Documentation on Placement Groups](http://docs.ceph.com/docs/mimic/rados/operations/placement-groups/). 


#### Example: Create a Replicated Pool

    sudo ceph osd pool create reppool 50 50 replicated

#### Example: Create an Erasure Pool

The basic syntax replaces 'replicated' with 'erasure' to specify the pool type. 

    sudo ceph osd pool create ecpool 50 50 erasure 

Pools can also be tuned balance redundancy and resiliency of the stored data. This is configured with the K and M values: 
* `K` = How many 'chunks' the original data will be divided into for storage. Generally, this is tied to the number of OSDs in the cluster. 
* `M` = Additional replica 'chunks' created to provide redundancy. The data is able to survive the failure up to `M` chunks. 

For this very small cluster, we only need one replica chunk (`M`), and two primary chunks (`K`) to get the job done. This is done by creating a new profile (smallcluster), and then using that profile to provision a new storage pool. 

    sudo ceph osd erasure-code-profile set smallcluster \
        k=2 m=1 crush-failure-domain=host 
    
    sudo ceph osd pool create ecpool2 50 50 erasure smallcluster



## RADOS Block Device

Create a RADOS block device on the cluster: 

Pool Name | Block Device Name
--------- | ----------------
`reppool` | `repdev`

1. Create a pool (Run on a node with admin tools/keys) 

        sudo rbd pool create reppool 50 50 replicated

2. Associate the pool with RADOS block device: 

	    sudo rbd pool init reppool

3. Create a block device on the pool: 

	    sudo rbd create repdev --size 4096 --image-feature layering -p reppool

4. On the Rados client, map the rbd device to a block device: 

	    sudo rbd map repdev --name client.admin -p reppool


5. Format the device as `ext4` (or XFS, or ReiserFS if you like stabbing people)

	    sudo mkfs.ext4 -m0 /dev/rbd/reppool/repdev

6. Mount the device on the ceph client: 

	    sudo mkdir /mnt/repdev

	    sudo mount /dev/rbd/reppool/repdev /mnt/repdev 

7. Establish mount persistence: 

        echo "reppool/repdev        id=admin,keyring=/etc/ceph/ceph.client.admin.keyring" | sudo tee -a /etc/ceph/rbdmap

        sudo systemctl start rbdmap
        sudo systemctl enable rbdmap

        echo "/dev/reppool/repdev    /mnt/repdev   ext4   defaults,noauto 0 0" | sudo tee -a /etc/fstab

## CephFS Shared Filesystem

This will act as a cluster shared volume for the cluster running on the system. 

* [Overview of CephFS](http://docs.ceph.com/docs/mimic/cephfs/)

* [CephFS Best Practices](http://docs.ceph.com/docs/mimic/cephfs/best-practices/)


### Create CephFS

1. Create a pair of pools to store metadata and data for the cephfs cluster: 

        ceph osd pool create cephfs_data 50
        ceph osd pool create cephfs_meta 50

2. Create a CephFS system from the two pools: 

        ceph fs new cephfs cephfs_meta cephfs_data 

* If using erasure coded pools: 

        ceph osd pool set my_ec_pool allow_ec_overwrites true 

[Source](http://docs.ceph.com/docs/mimic/cephfs/createfs/)


### Mount CephFS


1. Install the `cephfs-fuse` package: 

        sudo apt install ceph-fuse

2. Create a mount point with the same name as the cephfs pool (_not_ required but recommended)

        sudo mkdir -p /mnt/cephfs

3. Configure the  `/etc/fstab` file for using FUSE: 

        none    /mnt/cephfs  fuse.ceph ceph.id=admin,_netdev,defaults  0 0


The cephfs [kernel driver](http://docs.ceph.com/docs/mimic/cephfs/fstab/#kernel-driver) can also be used, but it is generally recommended to use FUSE instead. 

## Useful Commands

#### Ceph OSD Status

    sudo ceph osd status

Shows usage, status, and hosts in a convenient chart. 

#### CephFS Status

    sudo ceph fs status

Shows status of ceph file systems, associated pools, and active/passive MDS servers. 

#### Ceph Cluster Status

    sudo ceph -s

Shows basic health information about the cluster, including the active OSD nodes, pools and available space, and the number of placement groups in the cluster. 

#### Ceph Rados Block Device Status

    sudo rbd info reppool/repdev

#### View OSD and Node layout

    sudo ceph osd tree

Displays a basic map of which OSDs are located on which hosts. 

#### Authentication Scheme

    sudo ceph auth list

Lists the keys in use by the system, and briefly their permissions in the cluster. 

#### View Ceph Utilization 

    sudo ceph df

Much like the regular Linux 'df' command for viewing disk usage. 

## Add Placement Groups

Increase the number of Placement Groups: 

    sudo ceph osd pool set reppool pg_num 50
    sudo ceph osd pool set reppool pgp_num 50

These should be equal. 

## Delete a Pool

Edit the config file `/etc/ceph/ceph.conf`

    mon_allow_pool_delete = true

Restart the monitor service

    sudo systemctl restart ceph-mon.target

Delete the pool

    sudo ceph osd pool delete pooltodelete pooltodelete --yes-i-really-really-mean-it

Remove the allow line from the config file, restart the service again. 

