# Create shared directory for all members of my group on Béluga

See this webpage for details on how to share data between users: https://docs.alliancecan.ca/wiki/Sharing_data

The problem is that, by default, group members do not have access to files 
produced by other group members. It is possible to change a folder so that it 
is accessible by everyone in my group with `chmod g+rwx fichier`, but when a 
user creates a file in this folder, the file is only accessible to them and 
not to other users. The user would then have to `chmod` each new file it 
produces to give access to other group members, which is far from ideal.

Instead, it is possible to create a folder that is accessible to everyone in 
the group, and inside which files are open to everyone by default (r/w/x):
```bash
cd /project/def-bourret/
mkdir shared
chmod -R g+rw shared
setfacl -R -d -m g:def-bourret:rwx shared
getfacl shared

```

However, within this folder, make sure that nobody can write some critical 
folders that include programs installed for all group members (enable only 
write/execute on these folders):
```
cd /project/def-bourret/shared

mkdir -p ./data
chmod -R g-w data
setfacl -R -d -m g:def-bourret:rx data
getfacl data

mkdir -p ./newdata
chmod -R g+w newdata
setfacl -R -d -m g:def-bourret:rwx newdata
getfacl newdata

mkdir -p ./progs
chmod -R g-w progs
setfacl -R -d -m g:def-bourret:rx progs
getfacl progs

mkdir -p ./R
chmod -R g-w R
setfacl -R -d -m g:def-bourret:rx R
getfacl R

mkdir -p hybseqRefs
chmod -R g-w hybseqRefs
setfacl -R -d -m g:def-bourret:rx hybseqRefs
getfacl hybseqRefs

mkdir -p genomeRefs
chmod -R g-w genomeRefs
setfacl -R -d -m g:def-bourret:rx genomeRefs
getfacl genomeRefs

```
