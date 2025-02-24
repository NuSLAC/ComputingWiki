# Computing resources and management at S3DF

We have to understand two levels of hierarchies, a _facility_ and _repository_, in order to access computing resources effectively and efficiently. 

## Resource allocation and management
When you request computing resources (e.g. submitting a batch job), you would have to specify a facility and repository. There are people who manage/allocate computing resources to a particular combination of a facility and a repository. A manager person is also called a _czar_. If you need to modify the allocation, consult with your czar.

### Facilities

A facility is essentially an "owner" research group of computing servers at S3DF. 
- If you are reading this, the relevant facilities are `neutrino` and `mli`.
- Everyone belongs to at least one facility since s3df account is tied to a facility.
- [Here](https://coact.slac.stanford.edu/facilities), you can check the list of facilities you have an access to.

Here is a summary of the total dedicated resources for `neutrino` and `mli` facilities.

| Facility    | RAM total  | CPU total  | GPU - A100 | | GPU - 2080Ti |
| :---        |   :----:   |   :----:   |   :----:   |     :----:     |
| neutrino    | 6824GB     | 824        | 28         |  10            |
| ml          | 8424GB     | 1224       | 28         |  110           |

You can calculate the table above from the list of computing clusters in S3DF shown towoard the end of [this page](https://s3df.slac.stanford.edu/#/batch-compute). For instance, A100 GPU belongs to an `ampere` cluster. Each server in the `ampere` cluster holds 4 A100 GPUs, 112 CPUs, and 952GB RAM memory. 2080Ti GPUs belong to `turing` cluster. Each `turing` server holds 10 2080Ti GPUs, 40 CPUs, and 160GB RAM memory. The neutrino group owns 7 A100 and 1 2080Ti servers which corresponds to the numbers shown in the table above.

### Repositories

A repository corresponds to an individual research project. Per repository, there is a set maximum limit of computing resources accessible at any time. For instance, if a person A is using computing resources under a project X with close to the maximum limit set for the corresponding repository for X, another person B cannot be allocated additional resources which may exceed the total limit.

- [Here](https://coact.slac.stanford.edu/repos/info), you can check the list of repositories you have an access to.
- The only exception is a `default` repository which everyone belongs to. This is a special repository with no dedicated computing resources. In other words, `default` project can access only opportunistic computing resources (i.e. can access unused computing resources at the time of request, and you may be kicked out of a server anytime when others with dedicated access priority request the resources you are using.)
- The overall list is shown below. Feel free to request an access to a repo relevant to your work. The repos are shown in the format of `facility:repository`.
    - `mli:cider-ml` ... CIDeR-ML collaboration resource under `mli`
    - `neutrino:cider-nu` ... CIDeR-ML collaboration resource under `neutrino`
    - `neutrino:dune` ... for DUNE general compute, meant for CPU-only access.
    - `neutrino:dune-ml` ... for AI/ML related projects in DUNE
    - `neutrino:icarus-ml` ... for AI/ML related projects in SBN
    - `mli:llm-logbook` ... for LLM-PDG projects
    - `neutrino:ml-dev` ... for fundamental AI/ML projects not specific to experiments
    - `neutrino:slacube` ... for SLACube project, meant for CPU-only access.
    - `mli:uq-neutrino` ... for uncertainty quantification related projects in neutrino group.
    - `mli:zoox` ... for ZOOX supported projects within ML group
    - `mli:cmb-ml` ... for CMB projects in the Cosmic Frontier



