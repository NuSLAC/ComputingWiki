# Access and Storage
Computing resources are made available at SLAC Data Facility (SDF). While there is the [official documentation of SDF](https://ondemand-dev.slac.stanford.edu/public/doc/#/getting-started). This page serves as a guide specifically for neutrino users. In this page we cover:

* **Gateway**: how can I access computing servers for doing my work?
* **Storage**: where is my space? where to keep my data files?

## Interactive access via `ssh` 

From your laptop/desktop terminal, you can access SDF via [secure shell (SSH)](https://en.wikipedia.org/wiki/Secure_Shell). 
```
ssh $USER@sdf-login.slac.stanford.edu
```

**If you read to the end of this guide**, you will also learn how to access computing resources within a native web-browser on your laptop or desktop :) Keep reading!


### Data transfer to SDF from outside
Data transfer to SDF should be done using an appropriate data transfer node `sdf-dtn.slac.stanford.edu` (dtn stands for data transfer node). For example:
```
scp my_data.root $USER@sdf-dtn.slac.stanford.edu:~/
``` 

**Please do not transfer data through `sdf-login.slac.stanford.edu`**.
You can also use `globus` to transfer data files (that would correctly use the right nodes underneath).

## Storage space 

There are four storage spaces at SDF that serve different purposes. See also the [storage documentation](slac#sdf) at SDF.

### Home space
`$HOME` is where you may want to store **small files (i.e. scripts)**.
This space is accessible from everywhere within SDF (i.e. computing nodes).
While the size is limited ($\approx$25GB), this space is backed-up everyday.
You could recover your work (e.g. software under development) in case of a trouble.

### Group storage
Your primary storage space for large files and is located under `/sdf/group/neutrino`.
This space is accessible from everywhere within SDF like `$HOME`.
However, the space is not backed-up.
Feel free to use the space up to 1TB, and consult with Kazu for the need of more space.
(The hard limit is much larger than 1TB and it's ok to go little above, no panic!)

If this is your first time to use the space, **please create your own space** by executing the command below:
```
mkdir -p /sdf/group/neutrino/$USER
```

### Scratch space
This is a medium-term temporary storage space for large files and is located under `$SCRATCH`.
This space is accessible from everywhere within SDF like `$HOME`.
However, the space is not backed-up and **purged every month** (i.e. your files will be cleaned).

### Local scratch
This is a short-term temporary storage space for large files and is located under `$LSCRATCH`.
The space is not backed-up, and is purged after you log out from the server.
Unlike others, this space is **only accessible within the server you are logged in**.
This is because `$LSCRATCH` is a local disk system and not network mounted.
In return, `$LSCRATCH` offers much faster disk IO (=input/output) and is a recommended location to stage your data files for computing jobs that require fast data IO.  


## Software

We use a software _container_. If you are not familiar, a container is a technology that ships/provides a stack of software and necessary environment to execute a program in a single file. There is a [dedicated section about a container]() in this documentation. However, in here, we focus on how to use `singularity`, a particular container technology used at SLAC and many other academic institutions.

### How to run a container

Our computing workflow usually contains two parts:
1. Development stage requires interactive process for editing and debugging code
2. Production stage does not require interactive process but it may last hours or even days

In the first case, you might work in a terminal (e.g. `bash`), a web-browser (e.g. `Jupyter`) or other interactive sessions (e.g. `IPython`). All of these interactive sessions are initiated by executing a corresponding interpreter program. In the latter, you may simply execute a script like `python my_script.py`. 

So, in both cases, you want to execute a certain program under an environment where necessary softwares for your work are made available. You can do this by _running your command in a container_, and this is how:
```
singularity exec --bind /sdf,$SCRATCH,$LSCRATCH /sdf/group/neutrino/images/latest.sif bash
```

In the example above, we are executing `bash`, which is a command line interpreter. Running the above command is essentially same as **logging into a (virtual) machine where all kinds of pre-installed softwares are available**, and you can develop or test run a program in your session. In order to exit, simply finish your `bash` session in a usual way by typing `exit` or ctrl-D.


As another example, perhaps you want to start a Python interpreter within a container:
```
singularity exec --bind /sdf,$SCRATCH,$LSCRATCH /sdf/group/neutrino/images/latest.sif python3
```

Finally, maybe for a production, you may not want an interactive session but simply a batch processing:
```
singularity exec --bind /sdf,$SCRATCH,$LSCRATCH /sdf/group/neutrino/images/latest.sif python3 -c "print('hello world')"
```
This should exit the container immediately upon finishing the execution of python command `print('hello world')`.


### Understanding `singularity exec`

The format of the command above is `singularity exec $OPTION_FLAGS $IMAGE_FILE $EXECUTABLE [$ARG1 $ARG2 ...]`.
Let's decode this word by word!
* `singularity` is an executable and installed at every server in SDF.
* `singularity exec` is used to execute `$EXECUTABLE` in a container.
* `$IMAGE_FILE` used in this case is `/sdf/group/neutrino/images/latest.sif` which is called a singularity image file. This is like a "virtuam machine itself" containing all softwares, environments, etc. to run a container. It's just a file, and you can copy it to any other Linux platform (e.g. your laptop) and reproduce the exactly same environment. You can also build your own image and upload to anywhere at SDF to use (e.g. `$HOME`). There are other images under the same directory. `latest.sif` is the go-to software environment for the group.
* `$OPTION_FLAGS` used in this case is `--bind /sdf,$SCRATCH,$LSCRATCH`. Running a container is like logging into another machine. This implies even the file systems are different (e.g. `/sdf` space will not exist). Typically you would want to make storage areas visible within a container so you can access data files etc.. This can be done by _binding the sapce(s)_, or simply `--bind` with comma-separated arguments. The only exception to this is `$HOME` which is, by default, bound when running a container.
* `$EXECUTABLE [$ARG1 $ARG2 ...]` is your program. The arguments are optional and up to the program you are running.


### What's in our container?

This is not an exhaustive list but here are the list of softwares that are explicitly installed within `latest.sif`

* `apt-get install`
    * dpkg-dev g++ gcc binutils git wget curl emacs vim openssh-client krb5-user cmake libx11-dev libxpm-dev libxft-dev libxext-dev libssl-dev python3-dev python3-tk python3-pip python3-setuptools libopenblas-dev libxerces-c-dev gdb screen tmux libhdf5-dev nodejs git-lfs libyaml-cpp-dev

* `pip3 install`
    * jedi==0.17.2 numpy wheel zmq six pygments pyyaml cython pytest numba pyzmq pyserial gputil psutil humanize h5py tqdm fire sphinx-rtd-theme scipy tables pandas scikit-image scikit-learn scikit-build Pillow opencv-python matplotlib seaborn cufflinks plotly==4.14.3 plotly_express==0.4.1 "plotly>=5" plotly_express ipython ipykernel jupyter "notebook>=5.3" "ipywidgets>=7.6" jupyterthemes html-parser ipympl dash dash_renderer dash_html_components dash_core_components dash_bootstrap_components "jupyterlab>=3" jupyter-dash psutil requests plotly-geo==1.0.0 rootpy root_numpy uproot

*  `jupyter labextension` 
    * jupyterlab-plotly plotlywidget @jupyter-widgets/jupyterlab-manager jupyter-matplotlib kaleido

* `npm`
    * docsify

* GPU libraries
    * cuda-11.3 cudnn8 cupy 

* ML libraries (through `pip3`)
    * torch==1.10.0+cu113 torchvision==0.11.1+cu113 torchaudio==0.10.0+cu113 tensorflow keras gpytorch botorch scikit-learn numdifftools pyro-ppl MinkowskiEngine torch-scatter torch-sparse torch-cluster torch-spline-conv torch-geometric

* Physics softwares
    * geant4 root larcv2 edep-sim larnd-sim h5flow

## Other softwares
SDF also mounts [CernVM File Systems (CVMFS)](https://cernvm.cern.ch/portal/filesystem). This could be useful for those who are working on legacy softwares (e.g. LArSoft) but you should not mix this with Singularity. In general, we do not recommend performing developmental work using CVMFS and use Singularity instead.




