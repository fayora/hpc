# SLURM on CentOS 7.1 HPC ARM Template

Deploys a SLURM cluster with head node and n worker nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ffayora%2Fhpc%2Fmaster%2Fslurm-on-centos7.1-hpc2%2Fazuredeploy.json" target="_blank">
   <img alt="Deploy to Azure" src="http://azuredeploy.net/deploybutton.png"/>
</a>

1. Select an existing resource group or enter the name of a new resource group to create.

2. Select the region for the deployment.

3. Specify a user name and password for the administrator.

4. Specify a user name for the cluster user.

5. Select an OS image.

6. Select the size of the head (master) node and the compute nodes.

7. Specify the number of compute nodes to deploy.

8. Select the size of the data disks that will be connected in a RAID array to the head and compute nodes.

9. Select the storage account type.

10. Select the path for the mounting of the cluster file system.

## Accessing the cluster

Simply SSH to the master node using the IP address of the head node. For example:

```
# ssh azureuser@1.2.3.4
```

You can log into the head node using the admin user and password specified.  Once on the head node you can switch to the HPC user.  For security reasons the HPC user cannot login to the head node directly.

## Running workloads

### HPC User

After SSHing to the head node you can switch to the HPC user specified on creation, the default username is 'hpc'.  

This is a special user that should be used to run work and/or SLURM jobs.  The HPC user has public key authentication configured across the cluster and can login to any node without a password.  The HPC users home directory is a NFS share on the master and shared by all nodes.

To switch to the HPC user.

```
azureuser@master:~> sudo su hpc
azureuser's password:
hpc@master:/home/azureuser>
```

### Shares

The master node doubles as a NFS server for the worker nodes and exports two shares, one for the HPC user home directory and one for a data disk.

The HPC users home directory is located in /share/home and is shared by all nodes across the cluster.

The master also exports a generic data share under /share/data.  This share is mounted under the same location on all worker nodes.  This share is backed by several disks configured as a RAID-0 volume.  You can expect much better IO from this share and it should be used for any shared data across the cluster.

### Running a SLURM job

First you'll need to set the worker nodes to IDLE as they were rebooted during the creation process.  To do this execute:

```
hpc@master:~> sudo scontrol update NodeName=worker[0-3] State=IDLE
```

To verify that SLURM is configured and running as expected you can execute the following.

```
hpc@master:~> srun -N4 hostname
worker0
worker2
worker3
worker1
hpc@master:~>
```

Replace '4' above with the number of worker nodes your cluster was configured with.  The output of the command should print out the hostname of each node.

### MPI

To run MPI applications you'll need to use the Hr-series instances which include InfiniBand and RDMA support.