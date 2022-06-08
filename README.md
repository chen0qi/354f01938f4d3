# Welcome
This repo contains ccpfs prototye and its dependent softwares. The softwares are already built and tested in CentOS 7.9.

# Hardware requirement
We use IB verbs to transfer IO requests between clients and servers. Thus, the servers used to perform the tests should be connected with Infiniband network.

# Software depenency installation

Most of the softwares that are needed by ccPFS are installed from the standard repository of CentOS version 7.9. Users can install them using the following commands.

```
yum install -y hwloc hwloc-devel libyaml gcc libatomic mpich-3.2-3.2 mpich-3.2-autoload mpich-3.2-devel  git gcc-c++ libtool boost-devel cmake libuuid-devel openssl-devel libyaml-devel libibverbs 
```

ccPFS uses CaRT from DAOS (https://github.com/daos-stack/daos) to transfer RPCs between lock clients and lock servers.
CaRT depends on MERCURY (https://github.com/mercury-hpc/mercury) and libfabric (https://github.com/ofiwg/libfabric).
These softwares that are not included in the standard repository. We packed these libararies in our artifact. The version of the used libararies are as follows.
* CaRT from DAOS (https://github.com/daos-stack/daos) with the commitID d4a7ae714f45da8828cd523b08039f656510a05d.

* Mercury (https://github.com/mercury-hpc/mercury) with commit ID 932ad534d8e3f8106e85d0931c20ee13f0e42188.

* libfabric (https://github.com/ofiwg/libfabric) with commit ID 5b0a7b2b516d20a3896e26381b8951e64c4824a5.

# benchmarks

* We use IOR to evaluate the performance of ccPFS and implement a ccpfs plugin in it. We include [0001-add-ccpfs-plugin-for-ior-evaluation.patch](./0001-add-ccpfs-plugin-for-ior-evaluation.patch) in this repo, which is generated based on IOR with commit ID 657ff8ad8f136b14e57677dbe0175e8e875346d5.

* We use mpi-tileIO to evaluate the performnace of ccPFS to support overlapped noncontigous IO. We modify the codes to support ccPFS API. We include [0001-modify-mpi-tile-io-to-use-ccPFS-API.patch](./0001-modify-mpi-tile-io-to-use-ccPFS-API.patch) that is generated based on mpi-tile-IO (https://www.mcs.anl.gov/research/projects/pio-benchmark/code/mpi-tile-io-01022003.tgz).

* We use h5bench to generate VPIC-IO workload and hack the code of h5bench to record the time of different phases (including hdf5 create, write, flush). The patch [0001-print-hdf5-create-write-flush-time-during-the-test.patch](./0001-print-hdf5-create-write-flush-time-during-the-test.patch) is generated based on h5bench with commit id 81bd930510453ebe28b324daa329414e44f88e21. 

# How to run the evaluations

The evaluations can be run on nodes with CentOS version 7.9 with the following steps.

1. Extract ccpfs.bin.tar.gz into some directory (in the following, we take /opt as an example).

2. Export /opt/ccpfs as a writeable NFS shared directory by adding "/opt/ccpfs *(rw,no\_root\_squash)" into /etc/exports and restarting the NFS service.

3. Create a directory /home/ccpfs on each node that will run the experiments, and mount the shared directory into /home/ccpfs using the command "mount NFSSERVER\_IP:/opt/ccpfs /home/ccpfs". The mount point /home/ccpfs cannot be changed because we include the customized softwares that we need in ccpfs.bin.tar.gz and use the absolute path with the prefix "/home/ccpfs/install" to avoid software version conflict issues.

4. Install the necessary software dependencies by running the script /home/ccpfs/install-deps.sh on each node that will run the experiments.

5. Assign a directory to store local data for each server. For example, if we use /mnt/nvme1n1/data/data/ to store local data, we execute the following command: ln -s /mnt/nvme1n1/data/data /home/ccpfs/install/ccpfs/data.

6. Enter the directory /home/ccpfs/install/ccpfs/ad/ and record hostnames or IPs of nodes that will run the experiments in the file named hosts, which is needed by mpirun. We need 96 nodes to perform all of the experiments.

7. Enter the directory /home/ccpfs/install/ccpfs/ad/ and run the scripts in the directory to perform the tests. Referring to the README.md file in the directory for details.
