# Training the full chain from scratch on v04

## Which sample?

This page assumes the following datasets, commonly known as `v04` sample:

* `/sdf/group/neutrino/data/mpvmpr_2020_01_v04/train.root` as training set
* `/sdf/group/neutrino/data/mpvmpr_2020_01_v04/test.root` as test set

The first file is used to train the network, the second one is used to validate the training (make sure that the models don't overfit the training set).

Obviously you might have your own sample that you want to train on.

> `v04` samples does not have any "ghost points" which are artifacts that can appear for 2D wire LArTPC detectors such as ICARUS. There are some flags in the configuration that need to change to train on a sample with vs without ghost points. Ask us about it.

## Which configuration?
As for configurations, there's an example of the full chain configuration (which Francois has tested works): `/sdf/group/neutrino/drielsma/pi0/full_chain_pi0_v04.cfg`

This is a so-called inference configuration, which means it is used with a set of weights that has already been optimized. So we need to make some modifications here to start a new training from scratch.

The full chain proceeds in this order:
* Semantic segmentation (classification of each voxel in the image as belonging to one of 5 classes: shower, track, michel, delta or low energy)
* Point proposal (identification of shower fragment start points and track points)
* Fragmentation (make dense shower + track fragments)
* Shower aggregation (put shower fragments together, identify primary shower fragments)
* Track aggregation (put broken track fragments back together)
* Particle aggregation (put particles together into interactions, identify particle ID and primary ID)

So we need to first turn off all the stages downstream of the semantic segmentation and point proposal stages and we need to train that first.

The way this is done is as follows. Copy the configuration above first and modify this code block:

```
    chain:
      enable_uresnet: True
      enable_ppn: True
      enable_cnn_clust: True
      enable_dbscan: True
      process_fragments: True
      use_ppn_in_gnn: True
      use_supp_in_gnn: True
      use_true_fragments: False
      enable_gnn_shower: True
      enable_gnn_track: True
      enable_gnn_inter: True
      enable_gnn_kinematics: False
      enable_cosmic: False
      enable_ghost: False
      verbose: True
```

This block sets what part of the full chain is active. As you can see, most things are right now, so we need to turn everything off but uresnet (the semantic segmentation stage) and PPN (the point proposal stage, which shares a common backbone with the semantic segmentation):

```
    chain:
      enable_uresnet: True
      enable_ppn: True
      enable_cnn_clust: False
      enable_dbscan: False
      process_fragments: False
      use_ppn_in_gnn: True
      use_supp_in_gnn: True
      use_true_fragments: False
      enable_gnn_shower: False
      enable_gnn_track: False
      enable_gnn_inter: False
      enable_gnn_kinematics: False
      enable_cosmic: False
      enable_ghost: False
      verbose: True
```

The second thing is that you want to change the dataset used (this config uses the test set), so that this block:

```
     data_keys:
      #- /sdf/group/neutrino/data/mpvmpr_2020_01_v04/train.root
      - /sdf/group/neutrino/data/mpvmpr_2020_01_v04/test.root
```

becomes this:

```
    data_keys:
      - /sdf/group/neutrino/data/mpvmpr_2020_01_v04/train.root
      #- /sdf/group/neutrino/data/mpvmpr_2020_01_v04/test.root
```

We also want to activate randomization of the batching (essentially we don't want to pick images sequentially, in which case we would always make the same batches. All you need to do is add this block under iotool:

```
  sampler:
    name: RandomSequenceSampler
```

Finally, we turn our attention to the `trainval` section of the configuration. First thing first, we want to train, so we need to turn `train` on, i.e.:

```
  train: True
```

Second, we remove the `model_path`, as we want to start the training over, i.e.:

```
  model_path: ''
```

Third, we modify `iterations` to a sensible number. The data set consists of roughly 128k images and we are passing batches of 64 images at each training step, so each 2000 iterations we complete an epoch (i.e. we look at the equivalent of the dataset size). 50 epochs is a large amount, so let's set `iterations` to 100000:

```
  iterations: 100000
```

Fourth, `checkpoint_step` specifies how frequently we store the weights (used to validate the performance on an independent set). Here 1000 corresponds to 1/2 epoch, which is granular enough:

```
  checkpoint_step: 1000
```

Finally, you can specify the path where the log (which stores loss/accuracy/iteration time/... values at each iteration) will be stored:

```
  log_dir: logs/full_chain/uresnet_ppn
```

and also the path + prefix for the weight files:

```
  weight_prefix: weights/full_chain/uresnet_ppn
```

Now you should have a config file you can use for training! 

**(NB: please do not modify the configuration file place, but rather make a copy!**

## Launch a training

This was the (not so) hard part. The easy part is to launch a training. I typically use files which specify my slurm resource request + the command to run the training. Here's an example:

```
#!/bin/bash 

#SBATCH --account=neutrino 
#SBATCH --partition=neutrino 

#SBATCH --job-name=train_full_chain_uresnet_ppn
#SBATCH --output=batch_outputs/train_full_chain_uresnet_ppn.txt
#SBATCH --error=batch_outputs/train_full_chain_uresnet_ppn.txt

#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem-per-cpu=4g
#SBATCH --time=72:00:00 
#SBATCH --gpus v100:1

singularity exec --bind /sdf/ --nv /sdf/group/neutrino/images/latest.sif bash -c "python3 /path/to/lartpc_mlreco3d/bin/run.py /path/to/full_chain_uresnet_ppn.cfg"
```

The first two `#` lines specify the account, you can use `neutrino`. The next three specify the job name (as it will appear in `squeue`) and the log files where the stdout/stderr of the training job will be dumped. After that, it's related to resources. Here I request 16 cpus, 4gb of RAM per CPU, 3 day run time and one V100 GPU. The final line is the most important and is the command line to run the training. Note that you must specify the path to your local `lartpc_mlreco3d` (please pull the latest version, i.e. 2.6.) and also the path to where your training configuration will live. To launch the job, simply do `sbatch train_fc_uresnet_ppn.sh` with `train_fc_uresnet_ppn.sh` the name of the file where you put the lines above

Once this is launched, you will be able to monitor the job progress by either tail the job log file or the training log file.

## Beyond training UResNet/PPN
There's more once the training of the first step is started, but let's start with this.
