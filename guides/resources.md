# Computing resources
There are two types of resources: "shared" and "dedicated". Within the latter, there is a "neutrino" resource meant to be available to all neutrino users. There is also "ml" resource which is available to those who work on machine learning (ML) projects lab-wide.

* **"shared"**
  * includes ALL computing resources at SLAC and provides opportunistic (as opposed to "dedicated") access. Being opportunistic means that your job can run if there is unused computing resource that matches with the resource requested by your job. Since it is not a dedicated access, if someone else submits a dedicated job and if the `slurm` decides to run this new job on the server where your job is running, your job may be killed in the middle.
* **"neutrino"**
  * includes resources specialized for ML work and owned by the Neutrino group. 
* **"ml"**
  * includes resources specialized for ML work and owned by the Machine Learning Initiatives.
  
Here is a summary of the total dedicated resources.  

| Partition   | RAM total  | CPU total  | GPU - A100 | GPU - V100 | GPU - P100 | GPU - 2080Ti | GPU - 1080Ti |
| :---        |   :----:   |   :----:   |   :----:   |   :----:   |   :----:   |    :----:    |    :----:    |
| neutrino    | 4864GB | 664 | 28 | 10 | 0 | 10 | 5 |
| ml          | 9280GB | 1424| 28 | 0  | 0 | 110 | 0 |

If you are looking for a particular type of resource and you cannot find how to access, [please let Kazu know](mailto:kterao@slac.stanford.edu).

