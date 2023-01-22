# Software containers
We maintain singularity containers for shared software development environment as well as reproducible data processing. This page summarizes a list of containers publicly available and maintained for the group. For ML work, Pytorch is our current base ML library (though we deviate, was tensorflow in 2018).

## Docker containers on the hub
You can find the full disclosure on [this docker hub page](https://hub.docker.com/repository/docker/deeplearnphysics/larcv2/general).

* **ub18.04-cuda10.2** ... (**3.5GB**) ... the base image built on top of NVIDIA cuda10.2 scientific python libraries + pytorch + pytorch_geometric
* **ub18.04-cuda10.2-extra** ... (**3.9GB**) ... + sparse convnet, MinkowskiEngine, and larcv2

Example:
```
$> sudo docker run deeplearnphysics/larcv2:ub18.04-cuda10.2 python3 -c "import numpy;print(numpy.__version__)"
```

## Singularity containers on the hub
Some Docker containers are pulled and converted into Singularity containers and available on the hub:
* **ub18.04-cuda10.2-extra** ... [on the hub](https://cloud.sylabs.io/builder/5efd02f2bf06d949fb170031)
    * e.g. `singularity pull ub18.04-cpu-ana0 library://drinkingkazu/remote-builds/rb-5efd02f2bf06d949fb170031:latest` 

## Singularity containers copy for servers on SLAC network
The singularity container images are downloaded to `/gpfs/slac/staas/fs1/g/neutrino/images/` and are available to processes running on most server machines within the SLAC network.
