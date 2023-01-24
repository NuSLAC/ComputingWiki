# Software

We use a software _container_. If you are not familiar, read sections about [what it is](tools/containers-whatis) as well as [how to use it](tools/containers-howto). Here, we list available `Singularity` container images on SDF and S3DF as well as a list of installed softwares. 

# Singularity images

Singularity images are stored under `/sdf/group/neutrino/images` at both SDF and S3DF. Both facilities host the same set of image files. You can use any image under the path and feel free to [ask Kazu](mailto:kterao@slac.stanford.edu) about each image. These images are also hosted at the Docker Hub which might be a better place for you to download from for using on your own machine (laptop/desktop) if you wish.

That being said, especially for new members of the group, there are a few "go-to" images.

#### For general data analysis use
* `latest.sif` ... is the latest recommended image to use if you have no idea how to pick.
* `develop.sif` ... is the latest development head image which could be unstable and have bugs (typically it doesn't).

#### For neutrino event generator study
* coming soon...

## Updating images
Occassionally we build a new image with updated software versions or with additional softwares. When a new candidate image is built and pass basic tests, `develop.sif`, which is a symbolic link, is changed to point to this candidate image. After `develop.sif` survives more advanced tests by more users, `latest.sif`, which is also a symbolic link, is then changed to point to the same image.

So, while `latest.sif` is recommended when you have no clue, it is useful to be aware of which actual image file the link is pointing to. The update of `latest.sif` and `develop.sif` does not happen without a notice, and anyone is welcome to join as a stakeholder to scrutinize a new `develop.sif` before a change is made to update `latest.sif` (so that you won't find that `latest.sif` is changed by a surprise.)

# What's in our container?

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

# Other softwares
SDF also mounts [CernVM File Systems (CVMFS)](https://cernvm.cern.ch/portal/filesystem). This could be useful for those who are working on legacy softwares (e.g. LArSoft) but you should not mix this with Singularity. In general, we do not recommend performing developmental work using CVMFS and use Singularity instead.




