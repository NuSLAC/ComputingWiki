## Running a standard configuration

?> To run it, make sure you have the latest code version from `DeepLearnPhysics/lartpc_mlreco3d` on the branch `develop`.

!> TODO: provide weights.

### Configuration
This is meant to be an example of a typical configuration: no ghost points, with all the steps of the full chain currently available (UResNet + PPN + CNN/DBSCAN clustering + GNN clustering for showers, tracks, interactions + Kinematics and particle flow + cosmic discrimination). Let us break it down to explain a few things. 

First comes the I/O configuration block. 
?> Technical Note: here we specify `particle_mpv_tree` to be used by `parse_cluster3d_clean_full` and `parse_cluster3d_kinematics_clean`, because the neutrino interaction id wasn't recorded in this sample yet. If you are using a newer sample, ask Laura about skipping this mitigation (not implemented yet).

```
iotool:
  batch_size: 4
  shuffle: False
  num_workers: 4
  collate_fn: CollateSparse
  sampler:
    batch_size: 4
    name: RandomSequenceSampler
  dataset:
    name: LArCVDataset
    data_keys:
      - /gpfs/slac/staas/fs1/g/neutrino/kterao/data/mpvmpr_2020_01_v04/train.root
    limit_num_files: 10
    schema:
      input_data:
        - parse_sparse3d_scn
        - sparse3d_pcluster
      segment_label:
        - parse_sparse3d_scn
        - sparse3d_pcluster_semantics
      cluster_label:
        - parse_cluster3d_clean_full
        - cluster3d_pcluster
        - particle_pcluster
        - particle_mpv
        - sparse3d_pcluster_semantics
      particles_label:
        - parse_particle_points
        - sparse3d_pcluster
        - particle_corrected
      kinematics_label:
        - parse_cluster3d_kinematics_clean
        - cluster3d_pcluster
        - particle_corrected
        - particle_mpv
        - sparse3d_pcluster_semantics
      particle_graph:
        - parse_particle_graph_corrected
        - particle_corrected
        - cluster3d_pcluster
```

Then we start configuring the full chain. The `chain` block defines which parts of the chain you want to use. This is also where you should turn on `enable_ghost` if your sample has ghost points. 
```
model:
  name: full_chain
  modules:
    chain:
      enable_uresnet: True
      enable_ppn: True
      enable_cnn_clust: True
      enable_gnn_shower: True
      enable_gnn_track: True
      enable_gnn_particle: False
      enable_gnn_inter: True
      enable_gnn_kinematics: False
      enable_cosmic: True
      enable_ghost: False
      use_ppn_in_gnn: True
      verbose: False
```
After that, it is time to specify the config for each block of the chain. The following blocks are at the same level as `chain` (i.e. listed under `modules`), hierarchy-wise.

#### 1. UResNet + PPN
This is a UResNet architecture with depth 6 (`num_strides`) and 16 initial filters (`filters`). `features` specifies how many features the input has (in this example, just one, an energy deposit for each voxel). `ghost` should be enabled if your input has ghost points and you require the "5+2" architecture (predicts a binary ghost/non-ghost mask in addition to the semantic segmentation mask). `spatial_size` defines a cube (in voxels) in which all coordinates should fit. `data_dim` is 3 since the sample is three-dimensional. `num_classes` refers to the semantic classes in your labels (not counting the label for ghost point, if applicable).

In the `ppn` block, the eponymous configuration parameters should be identical to the `uresnet_lonely` block (the PPN layers feed off the UResNet network).
`downsample_ghost` should be True if you have ghost points, False otherwise. If you have ghost points, it helps to enable `weight_ppn` (will reward positives more than negatives).

`ppn1_size` and `ppn2_size` (in voxels) specify the spatial size at the intermediate layers of PPN1 and PPN2. They need to be such that `ppn1_size` * 2^i = `spatial_size`, `ppn2_size` * 2^j = `spatial_size`, and depth >= i > j.
```
    # UResNet + PPN
    uresnet_ppn:
      uresnet_lonely:
        num_strides: 6
        filters: 16
        num_classes: 5
        data_dim: 3
        spatial_size: 768
        ghost: False
        features: 1
        model_path: '/gpfs/slac/staas/fs1/g/neutrino/ldomine/chain/new/weights_cnn_clustering1/snapshot-50999.ckpt'
        model_name: 'uresnet_lonely'
      ppn:
        num_strides: 6
        filters: 16
        num_classes: 5
        data_dim: 3
        downsample_ghost: False
        #weight_ppn: 0.9
        ppn1_size: 24
        ppn2_size: 96
        spatial_size: 768
        model_path: '/gpfs/slac/staas/fs1/g/neutrino/ldomine/chain/new/weights_cnn_clustering1/snapshot-50999.ckpt'
        model_name: 'ppn'
```
> Technical Note: `model_path` refers to weights that should be loaded for different parts of the chain. `model_name` is optional and should be specified if there is a naming mismatch between the network's parameters in your current configuration and in the weights that you want to load.
{.is-info}

#### 2. CNN (aka dense) clustering
`cluster_classes` specifies which semantic class should undergo the CNN clustering. Here we limit it to tracks.
```
    # CNN Clustering config
    spice:
      network_base:
        spatial_size: 768
        data_dim: 3
        features: 4
        leakiness: 0.33
      spatial_embeddings:
        seediness_dim: 1
        sigma_dim: 1
        embedding_dim: 3
        coordConv: True
        model_path: '/gpfs/slac/staas/fs1/g/neutrino/ldomine/chain/new/weights_cnn_clustering1/snapshot-50999.ckpt'
      uresnet:
        filters: 64
        input_kernel_size: 7
        num_strides: 7
        reps: 2
      fragment_clustering:
        s_thresholds: [0., 0., 0., 0.35]
        p_thresholds: [0.95, 0.95, 0.95, 0.95] 
        cluster_all: False
        cluster_classes: [1]
    spice_loss:
      name: se_vectorized_inter
      seediness_weight: 1.0
      embedding_weight: 1.0
      smoothing_weight: 1.0
      min_voxels: 2
```

#### 3. DBSCAN Fragmenter
Here we can specify if we want some semantic classes to be clustered by DBSCAN. The parameter `cluster_classes` is what matters. Here we ask that shower fragments, Michel and Delta be clustered through DBSCAN algorithm. Note that we ignore the last semantic class (lowE energy depositions), which will thus be excluded from the rest of the chain.
```
    # DBScan Fragmenter config
    dbscan_frag:
      dim: 3
      eps: [1.999, 3.999, 1.999, 4.999]
      min_samples: 1
      min_size: [3,3,3,3]
      num_classes: 4 # Ignores LE
      cluster_classes: [0, 2, 3] #[0, 1, 2, 3]
      track_label: 1
      michel_label: 2
      delta_label: 3
      track_clustering_method: 'closest_path'
      ppn_score_threshold: 0.9
      ppn_type_threshold: 0.3
      ppn_distance_threshold: 1.999
      ppn_mask_radius: 5
```
#### 4. GNN clustering
For now there is a separate GNN to cluster shower fragments, track fragments and particles (into interactions). This is the GNN shower clustering configuration:
```
    # Shower GNN config
    grappa_shower:
      base:
        node_type: 0
        node_min_size: 3
        add_start_point: True
        add_start_dir: True
        start_dir_max_dist: 5
      node_encoder:
        name: 'geo'
        use_numpy: False
      edge_encoder:
        name: 'geo'
        use_numpy: False
      gnn_model:
        name: meta 
        edge_feats: 19
        node_feats: 28
        node_classes: 2
        edge_classes: 2
        node_output_feats: 64
        edge_output_feats: 64
        aggr: add
        leakiness: 0.1
        num_mp: 3
    grappa_shower_loss:
      node_loss:
        name: primary
        high_purity: True
        use_group_pred: True
        group_pred_alg: score
      edge_loss:
        name: channel
        high_purity: True
        source_col: 5
        target_col: 6
```
Track GNN configuration:
```
    # Track GNN config
    grappa_track:
      base:
        node_type: 1
        node_min_size: 3
        add_start_point: True
        add_start_dir: True
        start_dir_max_dist: 5
      node_encoder:
        name: 'geo'
        use_numpy: False
      edge_encoder:
        name: 'geo'
        use_numpy: False
      gnn_model:
        name: meta
        edge_feats: 19
        node_feats: 28
        node_classes: 2
        edge_classes: 2
        node_output_feats: 64
        edge_output_feats: 64
        aggr: 'add'
        leakiness: 0.1
        num_mp: 3
    grappa_track_loss:
      edge_loss:
        name: channel
        high_purity: False
        source_col: 5
        target_col: 6
```
Interaction GNN configuration:
```
    # Interaction GNN config
    grappa_inter:
      type_net:
        num_hidden: 32
      vertex_net:
        num_hidden: 32
      base:
        node_type: [0, 1, 2, 3]
        node_min_size: 3
        add_start_point: True
        add_start_dir: True
        start_dir_max_dist: 5
        group_pred: 'score'
        kinematics_mlp: False # Compute PID in grappa_inter, but not momentum
        kinematics_type: False
        kinematics_momentum: False
        vertex_mlp: False
      node_encoder:
        name: 'geo'
        use_numpy: False
      edge_encoder:
        name: 'geo'
        use_numpy: False
      gnn_model:
        name: meta
        edge_feats: 19
        node_feats: 28
        node_classes: 2
        edge_classes: 2
        node_output_feats: 64
        edge_output_feats: 64
        aggr: 'add'
        leakiness: 0.1
        num_mp: 3
    grappa_inter_loss:
      edge_loss:
        name: channel
        loss: CE
        source_col: 6
        target_col: 7
        reduction: sum
        balance_classes: False
        target: group
        high_purity: False
      # Remove the node_loss if you are not predicting PID or vertex
      node_loss:
        name: kinematics
        type_loss: CE
        balance_classes: True
        spatial_size: 768
```

#### 5. Kinematics & particle flow
```
    # Kinematics GNN config
    grappa_kinematics:
      use_true_particles: False
      momentum_net:
        num_hidden: 32
      base:
        node_type: -1
        node_min_size: -1
        network: complete
        edge_max_dist: -1
        edge_dist_metric: set
        edge_dist_numpy: True
        kinematics_mlp: True # Compute momentum, not PID, in grappa_kinematics
        kinematics_type: False
        kinematics_momentum: True
      node_encoder:
        name: 'mix_debug'
        normalize: True
        geo_encoder:
          more_feats: True
        cnn_encoder:
          name: 'cnn2'
          res_encoder:
            coordConv: True
            pool_mode: 'avg'
            num_features: 64 #256
          network_base:
            spatial_size: 768
            data_dim: 3
            features: 4
            leakiness: 0.33
          uresnet_encoder:
            filters: 32
            num_strides: 9
            num_filters: 16
            reps: 2
            num_classes: 5
            input_kernel: 3
      edge_encoder:
        name: 'mix_debug'
        normalize: True
        geo_encoder:
          more_feats: True
        cnn_encoder:
          name: 'cnn2'
          res_encoder:
            coordConv: True
            pool_mode: 'avg'
            num_features: 32
          network_base:
            spatial_size: 768
            data_dim: 3
            features: 4
            leakiness: 0.33
          uresnet_encoder:
            filters: 32
            num_strides: 9
            num_filters: 32
            reps: 2
            num_classes: 5
            input_kernel: 3
      gnn_model:
        name: nnconv_old #modular_nnconv
        edge_feats: 51
        node_feats: 83 #275
        node_output_feats: 128
        edge_output_feats: 64
        edge_classes: 2
        node_classes: 5
        aggr: 'add'
        leak: 0.33
        num_mp: 3
    grappa_kinematics_loss:
      node_loss:
        name: kinematics
        reg_loss: l2
      edge_loss:
        name: channel
        reduction: sum
        balance_classes: False
        target: particle_forest
        high_purity: False
```

#### 6. Cosmic discrimination
```
    # Cosmic discrimination
    cosmic_discriminator:
      use_input_data: False
      use_true_interactions: False
      model_path: '/gpfs/slac/staas/fs1/g/neutrino/ldomine/chain/new/weights_nu3/snapshot-9999.ckpt'
      res_encoder:
        coordConv: True
        pool_mode: 'avg'
        num_features: 2
      network_base:
        spatial_size: 768
        data_dim: 3
        features: 4
        leakiness: 0.33
      uresnet_encoder:
        num_strides: 9
        filters: 16
        features: 16 # nInputFeatures
    cosmic_loss:
      node_loss:
        name: type
        target_col: 8
        balance_classes: True
```

#### 7. Chain loss weighting
Here you can specify whether you would like to weight some of the losses more than others (or turn them off completely by setting them to zero).
```
    # full chain loss and weighting
    full_chain_loss:
      segmentation_weight: 1.
      clustering_weight: 1.
      ppn_weight: 1.
      particle_gnn_weight: 1.
      shower_gnn_weight: 1.
      track_gnn_weight: 1.
      inter_gnn_weight: 1.
      kinematics_weight: 1.
      kinematics_p_weight: 1.
      kinematics_type_weight: 1.
      flow_weight: 1.
      cosmic_weight: 1.
```

#### End of configuration
> Note that if you enabled previously `use_true_interactions` or `use_true_particles` in the Kinematics or Cosmic discrimination parts, you should add `segment_label` and `cluster_label` (in this order) to the `network_input` list.
{.is-info}

```
  network_input:
    - input_data
  loss_input:
    - segment_label
    - particles_label
    - cluster_label
    - kinematics_label
    - particle_graph
trainval:
  seed: 123
  unwrapper: unwrap_3d_scn
  concat_result: ['seediness', 'margins', 'embeddings', 'fragments', 'fragments_seg', 'shower_fragments', 'shower_edge_index','shower_edge_pred','shower_node_pred','shower_group_pred','track_fragments', 'track_edge_index', 'track_node_pred', 'track_edge_pred', 'track_group_pred', 'particle_fragments', 'particle_edge_index', 'particle_node_pred', 'particle_edge_pred', 'particle_group_pred', 'particles','inter_edge_index', 'inter_node_pred', 'inter_edge_pred', 'node_pred_p', 'node_pred_type', 'flow_edge_pred', 'kinematics_particles', 'kinematics_edge_index', 'clust_fragments', 'clust_frag_seg', 'interactions', 'inter_cosmic_pred', 'node_pred_vtx', 'total_num_points', 'total_nonghost_points']
  gpus: '0'
  weight_prefix: ./weights_trash/snapshot
  iterations: 100000
  report_step: 1
  checkpoint_step: 100
  log_dir: ./chain/log_trash
  train: True
  debug: False
  minibatch_size: -1
  optimizer:
    name: Adam
    args:
      lr: 0.001
```
> If you want to run on CPU, set `gpu` to an empty string `''`.
{.is-info}

### Outputs
Assuming that the previous configuration is stored in a string `cfg`:
```python
from mlreco.main_funcs import process_config, prepare
cfg=yaml.load(cfg,Loader=yaml.Loader)
# pre-process configuration (checks + certain non-specified default settings)
process_config(cfg)
# prepare function configures necessary "handlers"
hs=prepare(cfg)
```
We can run the full chain for one iteration using
```python
data, output = hs.trainer.forward(hs.data_io_iter)
```
Now we have access to both the input data to the network `data` and the outputs of the full chain in `output`.

> Jupyter notebooks visualizing outputs for ghost/noghost: see for now `/gpfs/slac/staas/fs1/g/neutrino/ldomine/chain/Output_Chain_Ghost.ipynb` and `/gpfs/slac/staas/fs1/g/neutrino/ldomine/chain/Output_Chain_NoGhost.ipynb`.

### Analysis / Inference
Available post-processing scripts to record metrics on all stages of the chain at the same time:

```
post_processing:
  # Interaction GNN
  cluster_gnn_metrics+inter:
    ghost: True
    enable_physics_metrics: True
    store_method: per-iteration
    clusts: particles
    edge_pred: inter_edge_pred
    edge_index: inter_edge_index
    node_pred: ''
    target_col: 7
    source_col: 6
    chain: grappa_inter
    filename: cluster-gnn-metrics-inter
  # Shower GNN
  cluster_gnn_metrics+shower:
    ghost: True
    enable_physics_metrics: True
    store_method: per-iteration
    clusts: shower_fragments
    edge_pred: shower_edge_pred
    edge_index: shower_edge_index
    node_pred: shower_node_pred
    target_col: 6
    source_col: 5
    chain: grappa_shower
    filename: cluster-gnn-metrics-shower
  # Track GNN
  cluster_gnn_metrics+track:
    ghost: True
    enable_physics_metrics: True
    store_method: per-iteration
    clusts: track_fragments
    edge_pred: track_edge_pred
    edge_index: track_edge_index
    node_pred: ''
    target_col: 6
    source_col: 5
    chain: grappa_track
    filename: cluster-gnn-metrics-track
  uresnet_metrics:
    store_method: per-iteration
    num_classes: 5
  deghosting_metrics:
    store_method: per-iteration
    method: '5+2'
  ppn_metrics:
    store_method: per-iteration
    num_classes: 5
    mode: select
  cluster_cnn_metrics:
    store_method: per-iteration
    ghost: True
    spatial_size: 768
    enable_physics_metrics: True
    p_thresholds:
      - 0.95
      - 0.9
      - 0.95
      - 0.95
    s_thresholds:
      - 0.0
      - 0.21
      - 0.0
      - 0.35
  cosmic_discriminator_metrics:
    store_method: per-iteration
    ghost: True
    spatial_size: 768
    enable_physics_metrics: True
  kinematics_metrics:
    store_method: per-iteration
    ghost: True
    spatial_size: 768
  vertex_metrics:
    store_method: per-iteration
    ghost: True
    spatial_size: 768
    clusts: particles
```
Comment out or remove the configuration block of any metric script that you do NOT wish to run. If you are not working with ghost points, set `ghost` to `False` in all scripts.

## Training step by step
For better results the chain should be trained step by step. The order is usually the following:

1. UResNet
2. PPN (can be combined with 1.)
3. CNN clustering (if applicable) 

After this point you probably want to freeze the weights for UResNet/PPN/Spice to avoid overtraining (especially for PPN).

4. GNN clustering of tracks and GNN clustering for EM showers can be trained in parallel.
5. Interaction clustering (GNN again)
6. Once the GNN for interaction clustering is trained for a little bit, you can also activate the PID or vertex prediction components that feed off this GNN.
7. The kinematics GNN for predicting particle hierarchy (flow) and momentum.
8. Cosmic discrimination (nu vs cosmic).
