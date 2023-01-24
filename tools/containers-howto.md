# Quick start

Let's say we want to execute a simple `python` command:
```
import torch
```
which requires `pytorch` library to be installed in your environment. 
If we try `python3 -c "import torch"` on the SDF (or S3DF) log-in node, we get an error message.
```
$ python3 -c "import torch"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'torch'
```

We can solve this by running the command inside a container where `pytorch` is installed:
```
singularity exec /sdf/group/neutrino/images/latest.sif python3 -c "import torch"
```
and you should not see an error message this time.

## `singularity exec`

The format of the command we tried above is `singularity exec $OPTION_FLAGS $IMAGE_FILE $EXECUTABLE [$ARG1 $ARG2 ...]`.
Let's decode this word by word!
* `singularity` is an executable and installed at every server in SDF.
* `singularity exec` is used to execute `$EXECUTABLE` in a container.
* `$IMAGE_FILE` used in this case is `/sdf/group/neutrino/images/latest.sif` which is called a singularity image file. This is like a "virtuam machine itself" containing all softwares, environments, etc. to run a container. It's just a file, and you can copy it to any other Linux platform (e.g. your laptop) and reproduce the exactly same environment. You can also build your own image and upload to anywhere at SDF to use (e.g. `$HOME`). There are other images under the same directory. `latest.sif` is the go-to software environment for the group.
* `$OPTION_FLAGS` can alter "how" to run a container. In the above case we had no option flag. We look into more details about the option flags in the later secion.
* `$EXECUTABLE [$ARG1 $ARG2 ...]` is your program. The arguments are optional and the number is arbitrary (i.e. it's up to the program you are running).

## Interactive process
Having an interactive process is critical for code development. You can try:
```
singularity exec /sdf/group/neutrino/images/latest.sif bash
```
which starts a bash interactive process. You should see `Singularity>` in the prompt while you are in the container. Now you can try:
```
Singularity>  python3 -c "import torch"
```
and it should work without an error. Running the above command is essentially same as **logging into a (virtual) machine where all kinds of pre-installed softwares are available**, and you can develop or test run a program in your session. You can exit the container by typing `exit` or pressing ctrl+D. 


You can start an interactive process like above with any other interpreter program. Replace `bash` with `zsh`, `python`, or `ipython` all of which are interpreter programs.


## Option flags

First off, see the full list of option flags.
```
singularity exec --help
```

Here we discuss two most common flags in our workflow.

### Mounting a directory
As said, running a container is like logging into another machine. It puts us in an isolated environment. This is also true about the system paths (e.g. `/sdf` space will not be visible from inside the container). However we would want to make areas such as storage space visible within a container so we can access data files etc.. This can be done by _binding the sapce(s)_, or simply `--bind` with comma-separated arguments. The only exception to this is `$HOME` which is, by default, bound when running a container (i.e. no need to bind the `$HOME` space).

### Using NVIDIA GPUs
Running NVIDIA GPUs require GPU drivers which are _firmwares_ that's attached to a hardware. This is different from software and must be made visible from the host machine for container processes that utilize GPUs. A special flag to enable this is `--nv` and you should use it when you are on a host server machine with NVIDIA GPU(s) **and** you need to use GPUs from within a container.


### Bottomline

Here is probably the most typical singularity command format we use:
```
singularity exec --nv --bind /sdf,/sdf --bind /sdf,/fs,$SCRATCH,$LSCRATCH sdf/group/neutrino/images/latest.sif bash
```
or replace `bash` with any command you would like to execute :)

