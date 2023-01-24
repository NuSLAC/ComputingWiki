# Getting started

In this page we go over the basic ingredients to use SDF/S3DF computing facilities:
* Access from a terminal (ssh)
* Storage spaces
* Software stacks 

How-to for these steps are same/similar between SDF and S3DF. Wherever there's a difference, we highlight it clearly. More topics for which SDF differs from S3DF will be covered in separate sections.

Note there are also official documentations for both [SDF](https://sdf.slac.stanford.edu/public/doc/#) and [S3DF](https://s3df.slac.stanford.edu/public/doc/#/).

# SSH access from a terminal

You can access SDF through `sdf-login.slac.stanford.edu`:
```
$> ssh $USER@sdf-login.slac.stanford.edu
```
by replacing `$USER` with your Unix user name.

For S3DF, the gateway DNS is different:
```
$> ssh $USER@s3dflogin.slac.stanford.edu
```
Then there is an additional step to log into a group-specific interactive node. Type:
```
$> ssh neutrino
```
after you log onto s3dflogin.slac.stanford.edu, from the same terminal. 

# Filesystem and storage

There are four storage spaces at SDF that serve different purposes.

* Home space
    * `$HOME` is where you may want to store **small files (i.e. scripts)**. This space is accessible from everywhere within SDF (i.e. computing nodes). While the size is limited ($\approx$25GB), this space is backed-up everyday. You could recover your work (e.g. software under development) in case of a trouble.

* Group storage
    * Your primary storage space for large files and is located under `/sdf/group/neutrino`. This space is accessible from everywhere within SDF like `$HOME`. However, the space is not backed-up. Feel free to use the space up to 1TB, and consult with Kazu for the need of more space. (The hard limit is much larger than 1TB and it's ok to go little above, no panic!)

If this is your first time to use the group storage, **please create your own space** by executing the command below:
```
mkdir -p /sdf/group/neutrino/$USER
```

* Scratch space
    * This is a medium-term temporary storage space for large files and is located under `$SCRATCH`. This space is accessible from everywhere within SDF like `$HOME`. However, the space is not backed-up and **purged every month** (i.e. your files will be cleaned).

* Local scratch
    * This is a short-term temporary storage space for large files and is located under `$LSCRATCH`. The space is not backed-up, and is purged after you log out from the server. Unlike others, this space is **only accessible within the server you are logged in**. This is because `$LSCRATCH` is a local disk system and not network mounted. In return, `$LSCRATCH` offers much faster disk IO (=input/output) and is a recommended location to stage your data files for computing jobs that require fast data IO.  

## Accessing SDF from S3DF

The SDF space:
```
/sdf
```
is visible from S3DF via network mount. The mount point at S3DF is:
```
/fs/ddn/sdf
```
If you have some files to access, such as perhaps old data files you have used on SDF, you could access through this path. **However you are encouraged to physically move files to S3DF because the network bandwidth for this mount is limited to 10Gb/s**. Accessing files through this network mount could result in bottleneck in your process.


## Transfering data
Data transfer to SDF should be done using an appropriate data transfer node `sdf-dtn.slac.stanford.edu` (dtn stands for data transfer node). The corresponding server for S3DF is `s3dfdtn.slac.stanford.edu` For example:
```
scp my_data.root $USER@sdf-dtn.slac.stanford.edu:~/
``` 
for SDF, and for S3DF:
```
scp my_data.root $USER@s3dfdtn.slac.stanford.edu:~/
```

`scp` is the most basic option. Both nodes support better transfer protocoles including `bbc` (SLAC-made multi-streaming data transfer protocole) and `globus` (data transfer protocole shared across many DOE national labs). The `globus` endpoints are `slac#sdf` and `slac#s3df` for SDF and S3DF respectively.

**Please do not transfer data through `sdf-login.slac.stanford.edu`**.

