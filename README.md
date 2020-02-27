
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

# Get an specific node
To reserve a particular node include the folowing:  
	-from qrsh: `-q @<name_of_node>`  
	-from your bash script: `-q *@<name_of_node>` **I havent test this one yet!**    

Also, you can use wild cards to chose a group of nodes. \
For instance, lest supose you want your job to get only the fastest nodes in Hoffman (intel-gold). \
You must add in the head of your bash scrip the folowing:  

`
#!/bin/bash
#$ -l h_rt=8:00:00,h_data=2G,arch=intel-gold* # arch means architecture
`

# Jobs being killed due to memmory issues.
Beacuse Hoffman2 uses a linux system, it will look for the virtual memmory of your job. \
Sometimes the virtual memory higher than the actually memory you have requested with `h_data=`. How much higher? It will depend on the program and \
type of request you are doing.  

For instance, **programs that use java** like GATK use a lot of virtual memory. You can solve this
by increasing the virtual memory with the command `h_vmem=`. 
Because the virtual memory is a space that you shared with other users keep in mind that increasing memory too much will harm other users.  
To avoid memory issues with other users you can get a whole node for your job with: \
`exclusive`. Your header should look something like this `qrsh -l highmem,highp,**exclusive**,h_rt=48:00:00`

Seems to be that another issue with virtual memory is **when you paralilize your jobs** (Thanks kirk's lab for the info !!!). When you include `-pe`, every independent job has its own virtual memory, \
so one posible solution is to include in the header of your script an specific amount of virtual memory with `h_vmem=`. \
This amount should be at least the sum of the virtual memmory of all the independet ones. One easy fix sugested by Jazlyn from kirk's lab is to request a h_vmem equal to the product of h_data times the number of slots requested.  
 
**IMPORTANT* Regardless if you add in your script header `h_vmem=`, every job has its own virtual memory. You can find out what is the amount of virtual memory \
of job by typing:  
`qstat -j <name_of_job> | grep 'resour'` 

# Parallelizing your jobs.
There are two ways to Parallelize your jobs: `-pe shared` and `-pe dc*`.  

`-pe shared`: will look for cores at the same node. In case of `-pe dc*` it will look at cores in different nodes.  

The advantage of `-pe shared` is that once it has started it may run faster. However, because hoffman is looking for a node with \
a particular number of cores, it may take longer to start. In contrast, `-pe dc*` can chose cores from whatever node its availalbe, thus making the waiting time shorter. \
The downside of `-pe dc*` is that it may take longer to run since it has to collect information for multiple nodes. The recomnedation is to use \
`-pe shared` if your program is multi-threaded, which means that uses shared-memory. You should use `-pe dc*` if your program uses a MPI-style that handles multi-node jobs.

For more information go [here](https://github.com/schuang/hoffman2-job-scheduling-tutorial/tree/master/pdf).  
