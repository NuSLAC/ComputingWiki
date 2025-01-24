# Getting started

In this page we go over the basic ingredients to use S3DF:
* Access from a terminal (ssh)
* Storage spaces
* Software stacks 

The official documentation for S3DF can be found [here](https://s3df.slac.stanford.edu/public/doc/#/).

# SSH access from a terminal

You can access S3DF through `s3dflogin.slac.stanford.edu`:
```
$> ssh $USER@s3dflogin.slac.stanford.edu
```
by replacing `$USER` with your Unix user name.

Then there is an additional step to log into a group-specific interactive node. Type:
```
$> ssh neutrino
```
after you log onto s3dflogin.slac.stanford.edu, from the same terminal. 

# Filesystem and storage

There are four storage spaces at SDF that serve different purposes.

* Home space
    * `$HOME` is where you may want to store **small files (i.e. scripts)**. This space is accessible from everywhere within SDF (i.e. computing nodes). While the size is limited ($\approx$25GB), this space is backed-up everyday. You could recover your work (e.g. software under development) in case of a trouble.

* Group software space
    * You are encouraged to maintain your local software repositories under `/sdf/group/neutrino/$USER`. Create a new directory if your `$USER` area is not present. This space is accessible from everywhere within SDF like `$HOME`. However, the space is not backed-up. Feel free to use this space up to 0.5TB, and consult with Kazu for the need of more space.

* Group data storage
    * Your primary storage space for large files and is located under `/sdf/data/neutrino/$USER`. Create a new directory if your `$USER` area is not present. This space is accessible from everywhere within SDF like `$HOME`. This space is backed up unlike the software space. Feel free to use the space up to 1TB, and consult with Kazu for the need of more space. (The hard limit is much larger than 1TB and it's ok to go little above, no panic!)

* Scratch space
    * This is a medium-term temporary storage space for large files and is located under `$SCRATCH`. This space is accessible from everywhere within S3DF like `$HOME`. However, the space is not backed-up and **purged every month** (i.e. your files will be cleaned).

* Local scratch
    * This is a short-term temporary storage space for large files and is located under `$LSCRATCH`. The space is not backed-up, and is purged after you log out from the server. Unlike others, this space is **only accessible within the server you are logged in**. This is because `$LSCRATCH` is a local disk system and not network mounted. In return, `$LSCRATCH` offers much faster disk IO (=input/output) and is a recommended location to stage your data files for computing jobs that require fast data IO.  


## Transfering data
Data transfer to SDF should be done using an appropriate data transfer node `s3dfdtn.slac.stanford.edu` (dtn stands for data transfer node).  For example:
```
scp my_data.root $USER@s3dfdtn.slac.stanford.edu:~/
```

`scp` is the most basic option. Both nodes support better transfer protocoles including `bbc` (SLAC-made multi-streaming data transfer protocole) and `globus` (data transfer protocole shared across many DOE national labs). The `globus` endpoint is `slac#s3df` for S3DF.

**Please do not transfer data through `s3dflogin.slac.stanford.edu`**.

