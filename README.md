# Beluga-setup

This serves as a repository of scripts and tutorials useful to the students in 
my laboratory for analyses made on the Beluga cluster of Compute Canada.

## Connecting to the server

The cluster can be reached at this address: `beluga.computecanada.ca`. To 
connect, one needs to configure [two-factor authentification](https://docs.alliancecan.ca/wiki/Multifactor_authentication) and then use ssh 
from the command line (I suggest using the Mac or Unix command line). Recent 
versions of Windows offer a good Unix command line via [WSL](https://learn.microsoft.com/windows/wsl/tutorials/linux). To connect, type 
this (**replacing "$USERNAME" with your Compute Canada username**):  
```bash
ssh $USERNAME@beluga.computecanada.ca

```

The computing server is organized in three partitions, each with different 
uses and sizes:  
  - **`/home`** is a partition of only **50Gb** that can be used to install 
	small programs or to store small files. Each user has his own `/home`.
	- **`/scratch`** is a large partition of **20Tb** that is used to store 
	large files, but that is **temporary**. The files are deleted every month. 
	Use this partition for assembling your data and doing your analyses, but 
	then save the results to your `/project/def-bourret/$USERNAME` space.
	- **`/project/def-bourret/$USERNAME`** is a medium partition of **1Tb** 
	where you can store raw sequence data, as well as analysis results. Make 
	sure you also store a second copy of important files to an external drive 
	(or two). This space is **only accessible to you**
	- **`/project/def-bourret/shared/progs`** is a space accessible to all users 
	of my lab where I will be installing programs that are of broad interest. If 
	you need a program that is not already available through the cluster, please 
	let me know and I will install it there.
	- **`/project/def-bourret/shared/data`** is a space accessible to all users 
	of my lab where I we put data that is useful to several different users, so 
	that we do not have to store several copies of the same data. For instance, 
	sequencing data of HybSeq or 3RAD libraries containing samples of two or 
	more students will be placed there.
