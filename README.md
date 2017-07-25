###########################################################################################
http://apache.claz.org/hadoop/common/
DOWNLOAD .tar.gz (binary distribution)
NOT src.tar.gz (we do not want the source code)
https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
Hadoop requires ssh and rsync!!!!!!!!!!!!!!!
###########################################################################################

###################
Introducing Hadoop
###################
why do we need Hadoop?
    world is generating huge amounts of data and traditional solutions do not scale
    the performance of a single machine does not scale linearly
    we need a system that can
        store massive amounts of data
        process the data in a timely matter
        scale easily as data grows

distributed system
    many cheap computers in a data center work together
        each computer = node
        entire system = cluster
        2x number of computers > 2x storage capacity > 2x performance
    software needs to coordinate all the servers
        partition data onto the different machines
        coordinate computing tasks
        allocate capacity to complete a process
    handle fault tolerance and recovery
        distributed computing = a lot of complexity = Hadoop to the rescue!

HDFS
    distributed file system manages storage of data
    file system that exists on multiple nodes in a cluster
    works on multiple computers

MapReduce
    named after the programming model for parallel processing
    framework to DEFINE data processing tasks
    focus on what logical operations you want to perform on raw data

YARN
    framework RUNS tasks across multiple machines
    manages resources, memory, and storage requirements

series of steps taken when submitting a job to Hadoop:
    user defines map and reduce tasks to perform on the data
        using MapReduce API
        usually written in Java
    map reduce programs are packaged into jobs
    a job is run on the cluster using YARN
        YARN checks whether the cluster or nodes in the cluster have resources to run the job
        then YARN figures out where this job should run
    YARN runs the job and the stores the data in HDFS

#################
Hadoop ecosystem
#################
Hive
SQL query interface
convert SQL like queries to MapReduce behind the scenes

HBase
motivation = need to perform low latency operations on database
add more structure using key-value pairs
essentially a database built on Hadoop

Pig
scripting language
convert unstructured data into structured format
can then store this structured data in HDFS

Spark
distributed computing engine used along with Hadoop
interactive Scala/Python shells
intuitively process datasets

Oozie
tool to schedule workflows on all the Hadoop ecosystem technologies
big companies don’t run processes of 1 variety
chain of processes are constantly munging data

Flume/Sqoop
put data into and get data from Hadoop

#################
Installing Hadoop
#################
Standalone
    runs on a single node with a single process
    default mode of operation
    used to test MapReduce logic
    NO HDFS
    uses local file system for storage
    NO YARN
    no need to negotiate resources
    master/slave setup does not exist

Pseudo-distributed
    simulate an actual cluster for more advanced testing
    runs on a single node
    2 JVM processes simulates as if it is 2 nodes
    master node and slave node
    HDFS used for storage
    using portion of computer’s disk space
    YARN manages resources

Fully distributed
    production environment
    runs on a cluster of machines
    real distributed computing setup
    manual configuration of cluster is complicated
    usually use enterprise editions of Hadoop
    package Hadoop installs
    pre-configured to run on a cluster of machines
    including Cloudera, Hortonworks

###################################
within the hadoop-2.7.3/ directory
###################################
hadoop-2.7.3/etc/
    config files for the Hadoop environment

hadoop-2.7.3/bin/
    various commands

hadoop-2.7.3/bin/hadoop command
    run MapReduce jobs in Hadoop

hadoop-2.7.3/bin/hadoop jar
    run MapReduce jobs on Hadoop cluster

hadoop-2.7.3/share/
    all the Hadoop libraries live here
    jars that you’ll use when you write your MapReduce job


######################################################
test run to ensure proper standalone (default) install
######################################################
mkdir input
cp etc/hadoop/* input/
copy any files into input folder
use as text file input for MapReduce job
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
cat output/*

######################################################
configure pseudo-distributed mode
######################################################
requirements
25% disk space available

SSH installed
run `ssh localhost` to ensure install
Hadoop nodes communicate via secure shell protocol
master node communicates with slave nodes frequently
SSH needs to be passwordless
avoid additional overhead

hadoop-2.7.3/etc/hadoop/ contains all config files
    hadoop-2.7.3/etc/hadoop/hadoop-env.sh
        set JAVA_HOME environment variables
        find JAVA_HOME on mac `/usr/libexec/java_home`
    hadoop-2.7.3/etc/hadoop/*.xml files
        add configuration properties to .xml files
        each file corresponds with 1 component in Hadoop
    hadoop-2.7.3/etc/hadoop/core-site.xml
        not component specific
        config common across Hadoop
        fs.defaultFS specifies we want to use HDFS and it can be accessed via localhost:9000
    hadoop-2.7.3/etc/hadoop/hdfs-site.xml
        set replication factor to be 1 in order to have 1 additional copy of the HDFS in case of failure
    hadoop-2.7.3/etc/hadoop/mapred-site.xml
        create new file to configure MapReduce framework
        specify yarn
        how you want to negotiate for resources on the cluster
    hadoop-2.7.3/etc/hadoop/yarn-site.xml
        specify capacities you want your MapReduce framework to have that will be managed by YARN
        mapreduce_shuffle = allows MapReduce to shuffle and sort keys from the map to the reduce phase

#######################
run psuedo-distributed mode
#######################
MUST RUN `bin/hdfs namenode -format` FIRST
before starting up Hadoop, need to format namenode
namenode = master node
keeps track of all the other nodes on which processes run
think of it as a table of contents for where the data lives within a cluster

`sbin/start-dfs.sh`
start the master and slave nodes of the DFS

run `jps` to check the java processes are running

adding to HDFS
`bin/hadoop fs -mkdir -p /user/cbohara/`
`bin/hadoop fs -put /user/cobohara/input.txt`

https://stackoverflow.com/questions/28379048/data-lost-after-shutting-down-hadoop-hdfs

############
restart HDFS
############
stop HDFS
`sbin/stop-dfs.sh`
rm the folder in /tmp/
`rm -rf hadoop-*`
format name node
`bin/hdfs namenode -format`
start HDFS
`sbin/start-dfs.sh`
make sure data node is running
`jps`
`sbin/start-yarn.sh`
`jps` will show 2 additional processes now running
ResourceManager runs on the master node
NodeManager runs on the slave nodes

#######################
monitoring resources
#######################
HDFS
localhost:50070
YARN
localhost:8088

#######################
Storing Data with HDFS
#######################
how HDFS works
    Hadoop needs to be very fault tolerant
    built using commodity hardware
    not expensive supercomputers
    these cheap computers are prone to failure

suited for batch processing
    Hadoop typically runs very large, long-running jobs that munge data over a long period of time
    take in large dataset of input all at once > process it > write large output
    run without manual intervention
    not used for quick results for users (better to use Spark)

semi-structured data
    not like RDBS with strict rows and columns

data split across multiple machines in the cluster
    HDFS chooses 1 machine at random > designated MASTER / NAME NODE
    name node = table of contents
    use to lookup the data location
    also stores file metadata

HDFS runs Java process on master node
    receives all requests that are made to the cluster
    forwards requests to slave nodes
    no data is actually read from the name node
    master responsible for coordinating the storage across all the other machines = SLAVE / DATA NODES

data nodes contain the actual data / file contents
    large files are split into blocks (partitions)
    each block is stored on a different node on the cluster
    the entire file is not stored on 1 node
    each block = equal size (128 MB) = equal processing time for the same task
    makes it easier for HDFS to deal with data
    it doesn’t matter if the files are of different length
    do not keep multiple copies of the same file
    keep multiple copies of the blocks

optimal block size = 128 MB
    increase block size > reduce parallelism
    decrease block size > increases overhead of coordinating processes

reading a file in HDFS
    use metadata in name node to look up block locations
    read the actual content from the respective locations

##########
using HDFS
##########
point PATH file to bin/ directory to avoid having to type out bin/

`hadoop fs`
used to interact with all file systems
shows all the file system related commands

`bin/hadoop fs -help`

`hdfs dfs`
used only to interact with HDFS

`hadoop fs copyFromLocal [source file] [destination folder]`
destination folder must exist in order for this to work
same as `hadoop fs -put [source file local] [destination folder in HDFS]`

`hadoop fs -cp /test/* /test-dst/`
unpack test/ content using /*

`hadoop copyToLocal /test/* fromhdfs/`
same as `hadoop fs -get [source folder in HDFS] [destination folder local]

HDFS data node fault tolerance
    what are the possible failures on data nodes?
        block on data node can get corrupted
        entire data node can crash
    HDFS uses a replication factor
        replicate every block that is stored on the cluster across multiple machines
        no single machine holds the data for a single file
        replicated block location stored in name node


choosing replica location
maximize redundancy
    increase number of replicas
    store replicas far away from each other = on different racks
    if a disaster occurs to 1 rack the data will still be available on another
minimize write bandwidth
    want to keep blocks close to each other
    computers on the same rack can communicate faster with each other vs computers on diff racks
Hadoop default compromise
    default for distributed mode = 3 replicas
    first replica on 1st rack
    second replica on 2nd rack
    third replica on 2nd rack (not 3rd)

HDFS name node fault tolerance
    entire cluster is completely lost without the name node!
    if the name node fails, file block locations will be completely lost
    backing up name node = critical
meta data files
    2 files are used to reconstruct name node mappings
    fsimage
        snapshot of the complete filesystem at startup
    edits
        contains log info for all changes made across HDFS since cluster was restarted
merging these files is very compute heavy
    bringing a system back online can take a very long time
    stored on the LOCAL file system of the name node, NOT in HDFS
    can store in multiple locations and on remote disks
secondary name node
    much easier way the bring the Hadoop cluster back online
    secondary name node runs on a completely different machine
checkpointing process
    copy over from the fsimage and edits files from the primary name node
    secondary name node will merge the fsimage and edits to update fsimage on secondary
    checkpointed (updated) fsimage from secondary name node will replace the fsimage on the primary name node
    the edits file will then be emptied on both the primary and secondary name node
    checkpointing frequency is configurable in hdfs-site.xml

###############################
Processing Data with MapReduce
###############################
MapReduce paradigm
map
    runs on multiple nodes in a cluster
    map processes work on data on its own machine
    actual code that you write for map phase
    should only process 1 record
    output is a key-value pair
    each node’s map output is then collated together and transferred across the network to a single node
reduce
    takes map phase key-value pair output
    finds all values with the same key
    reduces to final result


word frequency example
    raw data is very large
    data is distributed across many machines in a cluster
    each machine has a partition of input data

each map process on each computer works on one line at a time

running word count example using Python and Hadoop Streaming

first attempt received output below
need to set properties in yarn-site.xml
https://stackoverflow.com/questions/20586920/hadoop-connecting-to-resourcemanager-failed#20607217
`sbin/stop-yarn.sh`
`sbin/start-yarn.sh`

if you need to kill job
https://stackoverflow.com/questions/11458519/how-to-kill-hadoop-jobs
`yarn application -list`
`yarn application -kill [id]`

########################################
Scheduling and Managing Tasks with YARN
########################################
YARN components
    coordinates tasks running on the cluster
    keeps track of all the resources across the cluster
    including CPU, RAM, disk space
    if a node has failed, and all the processes on the node has stopped > YARN assigns new node for the task

Resource Manager (RM)
    runs on single name node
    manages resources across all nodes in the cluster
    schedules tasks across the nodes
Node Manager (NM)
    manages data nodes = runs on each data node
    only responsible for running the tasks on its own node
    communicates with the RM

how YARN works
client submits a job to the RM > RM finds NM with available resources > NM runs job within a container
container not an actual container, but a logical unit within the data node that runs the application
1 NM can have 1+ containers = multiple processes running on 1 data node
container’s application master process
will run the mapper or reducer processes
also will determine if additional processes are required to complete the task
if additional processes are required the complete the task >
NM communicates with RM to ask for more resources >
RM scans cluster for additional nodes >
NM application master process communicates with other NM to share what work needs to be done

################
YARN scheduling
################
YARN determines where processes should be run within the cluster using the location constraint
    an efficient use of the cluster resources = minimize write bandwidth
    don’t want to have to write to different machines spread across the cluster
    assign process to same node where the data to be process lives
    avoid overhead of having to copy data across the cluster
    ex: need to run an additional mapper process on a node, but the node does not have resources available now
    typically the process will wait
    YARN scheduling algorithm can be set in yarn-site.xml
