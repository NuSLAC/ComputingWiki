# Accessing samples
Fermilab uses SAM to interact with samples. You need to authenticate to use it:
```
$ kinit
$ kx509
```
To know the statistics of a sample (number of events and number of files):
```
-bash-4.2$ samweb list-definition-files --summary ICARUS_prod_2020A_00_BNB_nue_cosmic_v09_09_00_reco2
File count:	2152
Total size:	55788062350697
Event count:	58339
```
Other useful SAM commands are summarized on SBN wiki:
https://sbnsoftware.github.io/icaruscode_wiki/samples/MCproduction.html

You may need to "pre-stage" a sample before you can use it.
https://cdcvs.fnal.gov/redmine/projects/icarus-production/wiki/How_to_pre-stage_files_and_check_if_you_need_to_do_it

Also see [how to setup and produce ICARUS data](icarus/production.md).

# Experiment-specific numbers
* [ICARUS](icarus/numbers.md)

### General
Cosmics rate: 30kHz


### Light and photons
1 MeV Deposition Photon Count: 24000 photons/MeV (for muon, particle dependent)

Liquid Argon Photon Property (prompt and late light):

* fraction $\alpha=0.23$ of photons goes to $\tau_f = 6 ns$ (**prompt light**).
* fraction $\beta=0.77$ of photons goes to $\tau_s = 1.5 \mu s$. (**late light**) 

Time probability distribution of photons
$\alpha \exp^{-t/\tau_f} + \beta \exp^{-t/\tau_s}$
