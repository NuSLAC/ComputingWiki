# Computing
Any computing, no matter how light it is, should be done using a dedicated computing server, sometimes called _workers_, _worker nodes_, or _batch workers_ interchangeably.
The log-in nodes (i.e. where you land after log in through `ssh`) are **NOT** computing servers :) 
We should not run any computing job there.

This section briefly explains how to use [`slurm`](https://slurm.schedmd.com/documentation.html) with which you can access workers to run your job.

There are 3 ways to use slurm:
1. Interactive access through a terminal
2. Batch job submission
3. Interactive access through a web-browser (Jupyter)

This section covers 1 and 2 where we explicitly execute `slurm` commands. In 3, `slurm` is used implicitly by another software. This will be discussed in [another section]().


# Interactive jobs

One basic option is to access a computing resource interactively on a terminal. 
This would look literally like you are _logging (or ssh-ing) into a server dedicated for computing_.
For this, we use a `slurm` command `srun`. 

```
srun --pty /bin/bash
```

Take a look at the demo below.
(BTW this movie is a magic: you can use your mouse to copy and paste typed commands from the movie!)

[![asciicast](https://asciinema.org/a/BsnxzlEUMsQv4lUbHcihbYNsi.svg)](https://asciinema.org/a/BsnxzlEUMsQv4lUbHcihbYNsi)

In the demo, you see I was originally on one of SDF login nodes `sdf-login01` with _AMD EPYC 7542_ CPU with 32 cores. Then using `srun`, I logged into one of workers `rome0233` which has _AMD EPYC 7702_ CPU with 64 cores.  


The option flag `--pty` specifies the command to execute on a worker machine. For an interactive job, it makes sense to pick an interperter language like `bash`. 


# Batch jobs

Once you finish developing your code, you may have a script that runs data processing by a single line of a command. 
You may not want to wait for this process to finish running in an interactive session.
This is when you want to use a __batch job submission__.

Here, let's practice an extremely simple batch job submission.
We will submit a batch job to execute `example.sh` script which contents is shown below.

```
#!/bin/bash
#SBATCH --partition=neutrino
#
#SBATCH --job-name=test
#SBATCH --output=job-%j.out
#SBATCH --error=job-%j.err
#
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=5g
#
#SBATCH --time=10:00
#                                                                                                           
#SBATCH --gpus=geforce_rtx_2080_ti:1                             

uname -a
nvidia-smi
```

A command to submit this script is:
```
sbatch example.sh
```

Here's a demo movie:
[![asciicast](https://asciinema.org/a/miMIDmRZqwkbvZU7qodEEHBAl.svg)](https://asciinema.org/a/miMIDmRZqwkbvZU7qodEEHBAl)


Yep, it's that simple! 

Let's walk through the contents of `example.sh`.
* `#!/bin/bash` tells the terminal excution to use `/bin/bash` (i.e. "bash") interpreter to execute the script.
* `--job-name` specifies the name of this job (i.e. handy if you want to find your job in `squeue`)
* `--output` specifies a text file to contain the standard output from the script
* `--error` specifies a text file to contain the standard error from the script
* `--ntasks` specifies the number of task to run. You can set this to the number main process to run.
* `--cpus-per-task` specifies the number of hyperthreads (i.e. "virtual cores") to be allocated per task. Note our machines have 2 hyperthreads per physical CPU core. This should match with the number of parallel threads/processes run by your main script.
* `--mem-per-cpu` specifies the amount of RAM memory per hyperthread. `5g` means 5 GByte. This job will have 20 GByte memory total with 4 hyperthreads.
* `--time` specifies the maximum amount of time allocated for this job. When this time is reached, if the job is still running, the job will be terminated.

I hope this explains the gists of how to submit a slurm batch job. **You might wonder how much resources (i.e. CPUs, RAM) you can or should request.** This of course depends on your job, but also on availability of resources to us as a group. This is what we cover next.


# Submitting opportunistic jobs

The way we practiced `slurm` commands so far for both `srun` and `sbatch` submit opportunistic jobs. To be more specific, opportunistic jobs are submitted without `--partition` nor `--account` option flags, which will be covered in the next section about jobs using "dedicated" resources.

The term "opportunistic" implies that the jobs run on shared resources when/where they are available and match your needs. **An advantage** of opportunistic jobs is that your target resource pool is the entire computing resources within the facility (SDF or S3DF, not both at the same time!). **A disadvantage** is that your jobs can be kicked out when there is a higher priority jobs, such as jobs submitted by the owners of the computing server(s) your jobs are running on.

Following some guidelines could help aleviate this disadvantage:
* _Checkpoint_ your job as much as possible. 
    * Checkpoint is an intermediate state that is saved while job is running, from which a new job can continue the rest of processing if the current job is killed. Do it if you can although many High Energy Physics software workflow does not include many checkpoints. 
* Submit many short jobs as opposed to a small number of long jobs. 
    * This is particularly helpful if your job is difficult to save a checkpoint frequently. Less checkpoint implies more wasted computing hours when a job is killed. By producing many small jobs, you can reduce the fraction of lost time overall.

In addition, it is useful to know an approximate "unit of computing resource". CPU jobs are typically executed on `rome` or `milan` servers that carries 128 physica CPU cores, corresponding to 256 hyper-threads, with 512GB RAM memory. So a unit of compute is 1 hyper-thread with 2GB memory. One server can host 256 such jobs (dislaimer: in reality the worker host needs some memory for running administrative tasks so it is slightly less than this). So it is good to request a resource in multiplicative factor of this unit (e.g. 4 hyper-threads with 8GB memory).

If you have a concern and want to a consultation on submitting a large number of jobs, please [contact Kazu](mailto:kterao@slac.stanford.edu).


# Requesting dedicated resources

Special jobs such as a ML workflow may request a special resource like GPU. Or, perhaps you need an immediate + guaranteed access to CPUs on our dedicated resources. Such jobs need to be submitted with additional `slurm` flags.  How this works is different for SDF and S3DF.

## SDF

On SDF, `slurm` _partition_ is used to specify the dedicated resources. If you would like to access `neutrino` group's resources, submit a `slurm` job with `--partition=neutrino` flag. Likewise submit with `--partition=ml` to access `ml` group's resources.

Here is an example to access v100 GPU on SDF:
```
srun --partition=neutrino  --gpus v100:1 --pty /bin/bash
```
where `--gpus` flag specify the type and count of GPU request (I requested one GPU of the type NVIDIA Tesla V100).

### Resource units on SDF

The total resource on SDF is shown below.
| Partition   | RAM total  | CPU total  | GPU - A100 | GPU - V100 | GPU - P100 | GPU - 2080Ti | GPU - 1080Ti |
| :---        |   :----:   |   :----:   |   :----:   |   :----:   |   :----:   |    :----:    |    :----:    |
| neutrino    | 800GB | 152 | 0 | 10 | 0 | 10 | 5 |
| ml          | 7254GB | 1168 | 20 | 0  | 0 | 110 | 0 |


Again it is useful to know the unit of resources to submit a dedicated job. SDF hosts three types of GPU servers:
* `psc` workers
    * 24 CPU cores (48 threads), 256GB RAM, x10 NVIDIA GTX 1080Ti (11GB) GPUs
* `tur` workers
    * 24 CPU cores (48 threads), 192GB RAM, x10 NVIDIA RTX 2080Ti (12GB) GPUs
* `volt` workers
    * 16 CPU cores (32 threads), 192GB RAM, x4 NVIDIA Tesla V100 (32GB) GPUs
* `ampt` workers
    * 64 CPU cores (128 threads), 1024GB RAM, x4 NVIDIA Tesla A100 (40GB) GPUs

For GPU jobs, a unit of compute is "per GPU" which yields, roughly,:
* 2 cores (4 threads), 24GB RAM per 1080Ti GPU
* 2 cores (4 threads), 16GB RAM per 2080Ti GPU 
* 4 cores (8 threads), 40GB RAM per V100 GPU
* 16 cores (32 threads), 250GB RAM per A100 GPU
where we spared some RAM for worker's administrative tasks and round down floating points.


## S3DF

At S3DF, `--partition` flag is used to specify the computing cluster ID. A _cluster_ is defined as a collection of homogeneous computing servers (i.e. workers of the same hardware type and configuration). Then an additional `--account` flag is used to specify a group or a project for which the resource usage is tracked and the prioritization is calculated.

To submit a GPU job at S3DF, here's a command you can run:
```
srun --partition ampere --account neutrino --gpus a100:1 --mem-per-cpu=20G --pty /bin/bash
```
where `--partition ampere` and `--gpus a100:1` is somewhat redundant since a cluster's homogeneity implies there's only one kind of GPU (so `a100` specification should not be needed). It is what it is :D



### Resource units on S3DF

The total resource on S3DF is shown below.
| Partition   | RAM total  | CPU total  | GPU - A100 | GPU - V100 | GPU - P100 | GPU - 2080Ti | GPU - 1080Ti |
| :---        |   :----:   |   :----:   |   :----:   |   :----:   |   :----:   |    :----:    |    :----:    |
| neutrino    | 4096GB | 512 | 16 | 0 | 0 | 0 | 0 |
| ml          | 2048GB | 256 |  8 | 0 | 0 | 0 | 0 |


S3DF currently hosts only `ampere` workers which are identical to `ampt` workers on SDF:
* `ampere` workers
    * 64 CPU cores (128 threads), 1024GB RAM, x4 NVIDIA Tesla A100 (40GB) GPUs

A unit of per GPU compute is also the same:
* 16 cores (32 threads), 250GB RAM per A100 GPU
