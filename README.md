# Rorqual setup

This serves as a repository of scripts and tutorials useful to the students in 
my laboratory for analyses made on the Rorqual cluster of Compute Canada.

## Connecting to the server

The cluster can be reached at this address: `rorqual.alliancecan.ca`. To 
connect, one needs to configure [two-factor authentification](https://docs.alliancecan.ca/wiki/Multifactor_authentication) and then use ssh 
from the command line (I suggest using the Mac or Unix command line). Recent 
versions of Windows offer a good Unix command line via [WSL](https://learn.microsoft.com/windows/wsl/tutorials/linux). To connect, type 
this (**replacing "$USER" with your Compute Canada username**):  
```bash
ssh $USER@rorqual.alliancecan.ca

```

The computing cluster is organized in three partitions, each with different 
uses and sizes:  
  - **`HOME`** (`/home/$USER`) is a partition of only **50Gb** that can be 
	used to	install small programs or to store small files. Each user has their 
	own `HOME`.
	- **`SCRATCH`** (`/home/$USER/links/scratch`) is a large partition of 
	**20Tb** that is used to store large files, but that is **temporary**. The 
	files are deleted 	every month. Use this partition for assembling your data 
	and doing your analyses, but then save the results to your `PROJECT` space.
	- **`PROJECT`** (`/home/$USER/links/project/def-bourret/$USERNAME`) is a 
	medium partition of **1Tb** where you can store raw sequence data, as well 
	as analysis results. Make sure you also store a second copy of important 
	files to an external drive (or two). This space is **only accessible to **
	**you**.
	- **`PROGS`** (`/project/def-bourret/shared/progs`) is a space accessible to 
	all users of my lab where I will be installing programs that are of broad 
	interest. If you need a program that is not already available through the 
	cluster, please let me know and I will install it there.
	- **`DATA`** (`/project/def-bourret/shared/data`) is a space accessible to 
	all users of my lab where I we put data that is useful to several different 
	users, so that we do not have to store several copies of the same data. For 
	instance, sequencing data of HybSeq or 3RAD libraries containing samples of 
	two or more students will be placed there.

