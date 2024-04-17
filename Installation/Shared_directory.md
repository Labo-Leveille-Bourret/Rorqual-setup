# Create shared directory for all members of my group on Béluga

See this webpage for details on how to share data between users: https://docs.alliancecan.ca/wiki/Sharing_data

The problem is that, by default, group members do not have access to files produced by other group members. It is possible to change a folder so that it is accessible by everyone in my group with `chmod g+rwx fichier`, but when a user creates a file in this folder, the file is only accessible to them and not to other users. The user would then have to `chmod` each new file it produces to give access to other group members, which is far from ideal.

Instead, it is possible to create a folder that is accessible to everyone in the group, and inside which files are open to everyone by default (r/w/x):
```bash
cd /project/def-bourret/
mkdir shared
chmod -R g+rwx shared
setfacl -R -d -m g::rwx shared
getfacl shared

```

