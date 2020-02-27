
# Get the list of nodes

To get the list of nodes that we have purchased but are shared with labs. Type fhe folowing:  
`qhost -l group=mcdb`  
**Note**:acess these nodes with `highp`  

You will notice that we have soemthing like:  
12 slots and ~48G per node  
16 slots and ~48G per node  
16 slots and ~252G per node  
24 slots and ~63G per node  

To list our **rwayne* node. Type fhe folowing:    
`qhost -l group=rwayne`  
**Note**: acess this node with `highmem`

You will notice that we have soemthng like:  
24 slots and 504.8G for a single node  

For the total amount of nodes available to you including the ones outside our group. Type fhe folowing:  
`qhost`  
**Note**: to acess nodes outside our group exlude `highp` and `highmem` form the header of your script. Like this `-l h_rt=8:00:00,h_data=2G`  

# Get a specific node
To reserve a particular node include the folowing:  
	-from qrsh: `-q @<name_of_node>`  
	-from your bash script: `-q *@<name_of_node>` **I havent test this one yet!**    

Also, you can use wild cards to chose a group of nodes. For instance, to get only the fastest nodes in Hoffman (intel-gold). \
You must add in the head of your bash scrip the folowing:  

`
#!/bin/bash
#$ -l h_rt=8:00:00,h_data=2G,arch=intel-gold* # arch means architecture
`

# Jobs being killed due to memory issues.
Beacuse Hoffman2 uses a linux system, it will look for the virtual memmory of your job not h_data. \
Sometimes the virtual memory is higher than the amount you have requested with `h_data=`. How much higher? It depends on the program as well as the 
type of request that you pass to hoffman. Here are two common problems:

1. **Programs that use java** like GATK use a lot of virtual memory. You can solve this
by increasing the virtual memory with the command `h_vmem=`.   
Because the virtual memory is a space that you shared with other users keep in mind that increasing the virtual memory too much, will harm other users.  
To avoid memory issues with other users you can get a whole node for your job with `exclusive`. Your header should look something like this `qrsh -l highmem,highp,exclusive,h_rt=48:00:00`

2. **When you paralilize your jobs** (Thanks kirk's lab for the info !!!). When you include `-pe`, every independent job has its own virtual memory, 
so one posible solution is to include in the header of your script an specific amount of virtual memory with `h_vmem=`. This amount should be at least the sum of the virtual memmory of all the independet ones. One easy fix, sugested by Jazlyn from kirk's lab, is to request a h_vmem equal to the product of h_data times the number of slots requested.  
 
**IMPORTANT** Regardless if you add in your script header `h_vmem=`, every job has its own virtual memory. You can find out what is the amount of virtual memory \
of job by typing:  
`qstat -j <name_of_job> | grep 'resour'` 

# Parallelizing your jobs
There are two ways to parallelize your jobs in Hoffman: `-pe shared` and `-pe dc*`.  

`-pe shared` will look for cores at the same node. In case of `-pe dc*` it will look for cores at different nodes.  

The advantage of `-pe shared` is that it may be faster once it has started to run. However, because hoffman is looking for a node with 
a particular number of cores, it may stay longer waiting at the queue. In contrast, `-pe dc*` can chose cores from whatever node its availalbe, thus the waiting time should be shorter.  

The downside of `-pe dc*` is that it may take longer to run since it has to collect information for multiple nodes. The recomnedation is to use 
**`-pe shared` if your program is multi-threaded**, which means that uses shared-memory. You should use **`-pe dc*` if your program uses a MPI-style** that handles multi-node jobs.

For more information go [here](https://github.com/schuang/hoffman2-job-scheduling-tutorial/tree/master/pdf).  


# Some basics in Hoffman2

`qsub`	#Submit a Job  
`qstat`	#Determine the Status of a Job  
`qhost -j` <node.name>	#Display Node Information  
`qdel`	#Cancel a Job  
`qhold`	#Place a hold on a queued job to prevent it from running  
`qrls`	#Release a job held with qhold  

`qhost -j | grep 'HOS\|n6005'` # this will give you resources being used  
The most important thing to look for is **MEMUSE** which is memory being used  

If you job is pending and notice that you have to modify either the memory or time requested . Remember you can do this with `qalter` with the following steps:  

1. Find all the -l arguments that your job uses. At the shell prompt, enter:  
     `qstat -j jobnumber | grep 'hard resource_list'`
where jobnumber is your job number. 
Note: your job must in the queu to have a jobnumber; find it by typing 'myjob'. Also 
you can only modify things if your job is pending at the queu `qw` if is laredy running `r` 
there is nothign you can do about it.    
The previous command returns something like:  
`hard resource_list: h_data=4000M,h_rt=1209600,highp=true` 
Use the qalter command to fix the job. At the shell prompt, enter:  
`qalter -l h_data=4000M,highp=true,h_rt=288:00:00 jobnumber`  
where 288:00:00 is no larger than the maximum highp h_rt value. In our case no more than 336:00:00  

**What queues can I run my jobs in?**  
`qquota` #If no resources are in use at the moment, qquota will not return any information.  

After you have start runing a job 
use the folowing to know **how much memory you should request**  
`qacct -j job_ID`  

You can find the maximum allowed time with:  
`qconf -sq rwayne_pod.q | grep h_rt`  

`highp-queue-name` is one of the highp queues from  
`qconf -sql`  

get an especific login.node. For instance this is how you get node login1. Important if you were running things as screen and sudenly you lost connection.  
`ssh login1.hoffman2.idre.ucla.edu -l yourUsername`  

Also this is usefule to get node for `lftp` trasfer. You have to get either 'dtn1 or dtn1' to use `lftp` and donwload sequences from UC Berkeley  
`ssh dtn1.hoffman2.idre.ucla.edu -l yourUsername`  
**Note**: you can also use the above to ftp into NCBI

