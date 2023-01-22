# ICARUS production
> If you already have `icaruscode` setup, some of these steps may be unnecessary/redundant. I am writing for the perspective of someone who has never connected to any ICARUS machine to run `icaruscode`, and explaining all the steps to be able to create your own MC simulation samples.
{.is-info}

## 0. Preliminary
Just a few words to explain the bare minimum / context  that you need to understand (or have heard of).

### 0.1. `icarusbuild` vs `icarusgpvm` machines
`icarusbuild01` (and `icarusbuild02`) is a physical server with real local space in `/scratch` unlike all other machines used by the collaboration `icarusgpvm0X` where X = 1..N. Most people use `/icarus/data` and `/icarus/app` (accessible from GPVM machines) but the space there is small and you can crush the filesystem. We will be using `/scratch` on `icarusbuild01` machine to do the development work.

### 0.2. Filesystems

PNFS (dcache disk storage) is exposed and recommended to use for grid jobs. This is where we should put our code in a tarball once we are done with development, and where the logs/output will be created. It is much bigger than `/icarus/data`. 

Regular UNIX command like `mkdir` work but you should learn to use `ifdh` commands. Prefix most UNIX filesystem commands (`ls`, `rm`, `cp`, etc)  with `ifdh` (e.g. `ifdh ls`, `ifdh cp`, etc) and use `--help` to learn the small differences.

> Avoid using `<tab>` completion and `ls` command on PNFS, it can hammer the filesystem for everyone. Instead use something like `ifdh ls`.
{.is-warning}

On PNFS there are 2 types of storage spaces:
* `/pnfs/icarus/scratch` are subject to auto-cleanup for files that haven't been touched in a long time. This is where I put the jobs' log and output folders. I copy the output ROOT files to our own servers anyway, so it does not matter if it gets cleaned up every now and then.
* There is also `/pnfs/icarus/persistent` but not recommended most of the time (space is limited and shared by everyone, you easily forget to remove files). I only put there scripts that I want to share/access from multiple machines.

As mentioned above, there is also so-called "BlueArc" storage which means `/icarus/data` and `/icarus/app`. See the [SBN wiki](https://sbnsoftware.github.io/icaruscode_wiki/Computing_Resources.html#storing-data) for more details. I do not use this type of storage for now.

## 1. Setup working environment

1. `kinit USERNAME@FNAL.GOV`
2. `ssh USERNAME@icarusbuild01.fnal.gov`

If you haven't created your own space under `/scratch` yet:

3. `mkdir /scratch/icarus/USERNAME`

We are going to copy a few setup scripts to this personal space:

4. `cd /scratch/icarus/USERNAME`
5. Copy setup scripts: `scp -r /pnfs/icarus/persistent/users/ldomine/setup/ /scratch/icarus/USERNAME/`

Here you should look at these setup scripts, especially `setup.sh` and make sure the right version for `icaruscode` is specified (`v09_28_01` in this example).

```bash
export MY_LARSOFT_VERSION=v09_28_01
export MY_LARSOFT_FLAVOUR=e20:prof
```

6. Source setup scripts: `source setup/setup.sh`
7. `cd $MRB_TOP && mrbsetenv`

## 2. Cloning the relevant repositories
### 2.1. Begin with `icaruscode`
`mrb` is some wrapper around git that manages larsoft packages for you. `mrb g` is the equivalent of `git clone`. `mrb i` will compile the code. You need to have run `mrbsetenv` in your `$MRB_TOP` directory before you use any `mrb` command.

1. `mrb g icaruscode`
2. `mrb i -j4` to build it (may take some time if this is the first time you are compiling `icaruscode`)

### 2.2. Then add `larcv`
`larcv` is not a larsoft package, so we need to add it separately and copy manually the built libraries to the proper location.

1. `cd $MRB_TOP/srcs`
2. `git clone https://github.com/DeepLearnPhysics/larcv2.git`

Building it from scratch will take a few minutes, you can do something else in the meantime...

3. `cd larcv2 && git checkout develop && source configure.sh && make -j4`
4. Manually copy the libraries: `scp build/lib/* $MRB_TOP/localProducts_larsoft_vXX_XX_XX_e17_prof/icaruscode/vXX_XX_XX/slf6.x86_64.e17.prof/lib/`

### 2.3. Finally add `Supera`
`Supera` is also somewhat independent from the rest of larsoft packages, so we `git clone` it on our own:

1. `cd $MRB_TOP/srcs/icaruscode/icaruscode`
2. `git clone https://github.com/DeepLearnPhysics/Supera.git`
3. Edit CMakeLists in this directory, add (or uncomment)  `add_subdirectory(Supera)` at the end
4. `cd Supera && git checkout feat/mlreco && sh setup.sh icarus`
5. `mrb i -j4` to build everything.

> Every time that you log into `icarusbuild01` again, you will need to do this before you can successfully build `icaruscode`:
> ```
> $ cd /scratch/icarus/USERNAME && source setup.setup.sh
> $ cd $MRB_TOP && source srcs/larcv2/configure.sh
> $ sh srcs/icaruscode/icaruscode/Supera/setup.sh icarus
> $ mrbsetenv
> ```
{.is-info}


## 3. A look at the fhicl files
To make nu simulation samples, I made an example fhicl in `Supera/job/run_nue.fcl`. It relies on the fhicl `Supera/job/supera_nue.fcl` to configure the Supera module.

Going into the details of these fhicl files may be for another wiki page... for the purpose of brevity, let's just say that you can easily change the neutrino type and interaction type as demonstrated in [`run_nue_ccqe.fcl`](https://github.com/DeepLearnPhysics/Supera/blob/feat/mlreco/job/run_nue_ccqe.fcl):

```
#include "run_nue.fcl"

physics.producers.generator.GenFlavors: [ 12 ]
physics.producers.generator.EventGeneratorList: ["CCQE"]
```

There is a list of event type options [here](https://cdcvs.fnal.gov/redmine/projects/nutools/wiki/GENIE_Configuration_Files) (thank you Kazu!).

You can run an interactive test run easily:
```
$ cd $MRB_TOP/srcs/icaruscode/icaruscode/Supera
$ lar -c job/run_nue.xml -n 1
```
The `-n 1` option tells larsoft to generate just 1 event. At the end, you will get a summary of resources usage (memory & time) which will be useful to determine how many events / jobs you need and how much resources to ask for in your job submission.
## 4. Running a job

### 4.1. Writing a XML file for `project.py`
`project.py` is a wrapper written by physicists around the grid scheduler (i.e. SLURM or equivalent). It reads a configuration XML file and submits the job request for you. Also see [here](https://sbnsoftware.github.io/sbndcode_wiki/Using_projectpy_for_grid_jobs.html) for more information about it.

So, we need to write an XML file to configure our job submission for `project.py`. You need a file called something like `run_nue.xml` in your `$MRB_TOP` folder. To get you started you can copy an example from my folder:

1. `ifdh cp -D /pnfs/icarus/persistent/users/ldomine/run_nue.xml $MRB_TOP`

I will copy here the contents so it is easier to go over:

```xml
<?xml version="1.0"?>

<!-- Production Project -->

<!DOCTYPE project [
<!ENTITY release    "v09_28_01">
<!ENTITY releasetag "e20:prof">
<!ENTITY my_version "v00">
<!ENTITY name       "nue">
<!ENTITY file_type  "mc">
<!ENTITY run_type   "physics">
<!ENTITY PNFSpath   "/pnfs/icarus/scratch/users/ldomine/">
]>

<project name="&name;">

  <!-- Group -->
  <group>icarus</group>

  <!-- Project size -->
  <numevents>200</numevents>

  <!-- Operating System -->
  <os>SL7</os>

  <!-- Batch resources -->
  <resource>DEDICATED,OPPORTUNISTIC</resource>

  <!-- Larsoft information -->
  <larsoft>
    <tag>&release;</tag>
    <qual>&releasetag;</qual>
    <local>&PNFSpath;tars/&release;.tar</local>
  </larsoft>

  <!-- Project stages -->
  <fcldir>&PNFSpath;tars/</fcldir>

  <stage name="reco">
		<fcl>run_nue.fcl</fcl>
		<outdir>/pnfs/icarus/scratch/users/ldomine/&name;_&my_version;/&release;</outdir>
		<workdir>/pnfs/icarus/scratch/users/ldomine/&name;_&my_version;_work/&release;</workdir>
		<logdir>/pnfs/icarus/scratch/users/ldomine/&name;_&my_version;_log/&release;</logdir>
    <numjobs>10</numjobs>
		<datatier>larcv</datatier>
		<defname>&name;_&my_version;</defname>
    <memory>8000</memory>
		<!-- <disk>60GB</disk> -->
  </stage>

  <!-- file type -->
  <filetype>&file_type;</filetype>

  <!-- run type -->
  <runtype>&run_type;</runtype>

</project>
```
The `<!ENTITY variable_name "variable_value">` lines define some variables for convenience that you can reuse later in the XML file. This is a single stage job, so fairly simple. The main parameters you want to adjust are
* `<numevents>200</numevents>` this is the TOTAL number of events that you want to generate.
* `<numjobs>10</numjobs>` number of jobs that you want to submit. In this example, it means that each job will have to generate 200 / 10 = 20 events.
* `<fcl>run_nue.fcl</fcl>` name of fhicl file to run
* `<memory>8000</memory>` you want to adapt this based on your own interactive tests and how many events per job you will be generating.
* if needed, you can also add a line at the same level `<jobsub>--expected-lifetime=5h</jobsub>` to specify a job lifetime (default is 8h I believe).
* `<stage name="reco">` the name `reco` is what you will specify to `project.py` when you run it and tell it which stage to run (e.g. `--stage reco`). Here we have only 1 stage.

Obviously, you also need to update the paths for yourself.

> Have to put this advice somewhere: running test jobs is **important**. You do not want to repeatedly submit batches of 500 jobs that fail immediately for stupid reasons.
>
> Do a 1 event interactive run to determine resources usage and don't be too harsh on the limits that you set, to avoid having too many jobs held. 
>
> Do a test job submission to check your XML file. Submit <10 jobs, create a few events. Go through the full pipeline, download the ROOT files and look at them. In a ROOT browser and in an event display. 
>
> Once you are satisfied, there was no unexpected crash and the output looks like you expect, you can start scaling up your job submission.
{.is-warning}


### 4.2. Preparing the tarball and XML on `icarusbuild01.fnal.gov`
You will need the right Kerberos tickets before submitting jobs / using `ifdh` command line tool.
1. `kinit USERNAME@FNAL.GOV && kx509`

Since we modified the tagged `icaruscode` to include things like LArCV and Supera, we need to package it into a tarball that will be distributed to each job. You can copy and use the script `make_tar.sh` that Kazu wrote:

2. `ifdh cp -D /pnfs/icarus/persistent/users/ldomine/make_tar.sh ./`
3. `sh make_tar.sh -d $MRB_INSTALL v09_28_01.tar`

To make it visible by the jobs, I put the tarball / XML / fcl files somewhere under `/pnfs/icarus/scratch/users/USERNAME/your_folder`. Copy the tarball, XML run file and FHICL files to that directory.

4. `ifdh mkdir /pnfs/icarus/scratch/users/USERNAME/your_folder`
5. `ifdh cp -D run_nue.xml v09_28_01.tar srcs/icaruscode/icaruscode/Supera/run_nue.fcl srcs/icaruscode/icaruscode/Supera/supera_nue.fcl /pnfs/icarus/scratch/users/USERNAME/your_folder`

### 4.3. Launching the jobs from `icarusgpvm01.fnal.gov`
> For some reason I am currently not able to start grid jobs from `icarusbuild01.fnal.gov`, so I switch to `icarusgpvm01.fnal.gov` to launch my jobs (this is the purpose of the GPVM machines anyway). 
If you know a better way / a solution, please ping me!
{.is-info}

In a different terminal or after you close your session on `icarusbuild01.fnal.gov`:

6. `ssh USERNAME@icarusgpvm01.fnal.gov`
7. `kinit USERNAME@FNAL.GOV && kx509` (different machine, make sure you have the Kerberos tickets to submit grid jobs)

You need to source / setup a few things to run the job submission smoothly:

8. `source /cvmfs/icarus.opensciencegrid.org/products/icarus/setup_icarus.sh`
9. `setup -B icarusutil v09_26_00 -q:e20:prof` (contains `project.py` script)
10. `setup jobsub_client` (to interact with grid scheduler)

And finally we can start the job submission:

11. `project.py --xml /pnfs/icarus/scratch/users/USERNAME/your_folder/run_nue.xml ---stage reco --submit`



## 5. Monitoring and retrieving the job results

### 5.1. Keeping an eye on the grid jobs
Great, now you can use `jobsub_*` command line tools to interact with your jobs.
* `jobsub_q --user=USERNAME` to watch your jobs.
* `jobsub_rm --user=USERNAME` or `jobsub_rm --jobid xxxxx.@xxxx.fnal.gov` to cancel jobs.
* `jobsub_fetchlog --jobid xxxxx.xx@xxxx.fnal.gov` will retrieve the logs for you. It will bring a `.tar` compressed folder in your current directory. Beware before unzipping it, it contains a lot of log files in the root directory! Create a temporary directory to unzip it (`tar -xvf the_log.tar`) without overwhelming your working directory with log files.

If a job is "held", it means that it went above the requested memory or allocated time. Jobs will be "idle" until they are assigned a slot to start running. 

When the batch of jobs is done, you will receive an email with statistics about the resources usage (use it to refine your resources request, if you only used <50% of memory or time) and how many jobs succeeded (exit code 0). Use `jobsub_fetchlog` to investigate jobs that failed.

### 5.2. Once the jobs are done, retrieving the output
There are probably other ways of doing it. This is how I have been doing it so far, feel free to follow your own path!

To retrieve the job results, I use another script written by Kazu and barely changed by me:
1. `ifdh cp -D /pnfs/icarus/persistent/users/ldomine/link_maker.sh ./ `
It retrieves all the jobs outputs and creates symlinks for you. You may want to edit it to change the `OUTPUT_DIR` to suit your needs:
```python
OUTPUT_DIR  = '/icarus/data/users/%s/nue_symlink/%s' % (USER,UNIQUE_NAME)
```
If you leave it as is, it will create a directory under `/icarus/data/users/%s/nue_symlink` and put all symlinks in that folder.
2. `python3 link_maker.py /pnfs/icarus/scratch/users/USERNAME/your_folder/run_nue.xml`
The output will confirm where the symlinks were put. It uses the information in your XML file to determine a name for the folder. For example, `/icarus/data/users/USERNAME/nue_symlink/larcv_nue_v01`.

The last step is to copy the actual ROOT files to your SDF space (or wherever you plan to store them):
3. `rsync -r -e ssh -L --update --progress -v /icarus/data/users/USERNAME/nue_symlink/larcv_nue_v01 USERNAME@sdf-login.slac.stanford.edu:/sdf/group/neutrino/USERNAME/`

depending on how many files you are retrieving, this may take a while, but fortunately `rsync` can be interrupted and resumed as necessary.
