# Making samples for lartpc_mlreco3d
*This page was updated on 11/29/22.*

Want to generate some simulation (or data) samples to feed into a neural network (`lartpc_mlreco3d`) ?
These instructions attempt to explain the process. In a nutshell, after you run all the required LArSoft
modules to generate your sample, you have to run **[Supera](https://github.com/DeepLearnPhysics/Supera/tree/icarus)** which makes MC truth labels and creates a
LArCV file. You can then feed this LArCV file directly into `lartpc_mlreco3d`. 

This tutorial can probably be further simplified. It is written
from the perspective that you might want to do development work - hence creating a development area, cloning
the `sbncode` source code, making a tarball for job submission, etc. Some people might just want to *run* and
don't care about these details - the instructions will be updated (maybe one day) for this simplified workflow.

> Note: **All instructions on this page assume LArSoft >= v09_63_00**. This is using the "new" way of generating MC samples for `lartpc_mlreco3d`.
> Supera is setup to read SimEnergyDepositLite instead of SimEnergyDeposit and MCParticleLite on top of MCParticle.

## 0. Setting up the workspace
### 0.1. `icarusbuild` vs `icarusgpvm` machines
`icarusbuild01` (and `icarusbuild02`) is a physical server with real local space in `/scratch` unlike all other machines used by the collaboration `icarusgpvm0X` where X = 1..N. Most people use `/icarus/data` and `/icarus/app` (accessible from GPVM machines) but the space there is small and you can crush the filesystem. I use `/scratch` on `icarusbuild01` machine to do development work. You do you :)

### 0.2. Filesystems

PNFS (dcache disk storage) is exposed and recommended to use for grid jobs. This is where we should put our code in a tarball once we are done with development, and where the logs/output will be created. It is much bigger than `/icarus/data`. 

Regular UNIX command like `mkdir` work but you should learn to use `ifdh` commands. Prefix most UNIX filesystem commands (`ls`, `rm`, `cp`, etc)  with `ifdh` (e.g. `ifdh ls`, `ifdh cp`, etc) and use `--help` to learn the small differences.

> Avoid using `<tab>` completion and `ls` command on PNFS, it can hammer the filesystem for everyone. Instead use something like `ifdh ls`.

On PNFS there are 2 types of storage spaces:
* `/pnfs/icarus/scratch` are subject to auto-cleanup for files that haven't been touched in a long time. This is where I put the jobs' log and output folders. I copy the output ROOT files to our own servers anyway, so it does not matter if it gets cleaned up every now and then.
* There is also `/pnfs/icarus/persistent` but not recommended most of the time (space is limited and shared by everyone, you easily forget to remove files). I only put there scripts that I want to share/access from multiple machines.

As mentioned above, there is also so-called "BlueArc" storage which means `/icarus/data` and `/icarus/app`. See the [SBN wiki](https://sbnsoftware.github.io/icaruscode_wiki/Computing_Resources.html#storing-data) for more details. I do not use this type of storage for now.

> You can setup your working environment on `icarusbuild` or `icarusgpvm` machines. Both will be addressed in the next 2 sections.

### 0.3. Setup a working environment on `icarusbuild` machine

1. `kinit USERNAME@FNAL.GOV`
2. `ssh USERNAME@icarusbuild01.fnal.gov`

If you haven't created your own space under `/scratch` yet:

3. `mkdir /scratch/icarus/USERNAME`

We are going to copy a few setup scripts to this personal space:

4. `cd /scratch/icarus/USERNAME`
5. Copy setup scripts: `scp -r /pnfs/icarus/persistent/users/ldomine/setup/ /scratch/icarus/USERNAME/`

Here you should look at these setup scripts, especially `setup.sh` and make sure the right version for `icaruscode` is specified (`v09_28_01` in this example).

```bash
export MY_LARSOFT_VERSION=v09_63_00
export MY_LARSOFT_FLAVOUR=e20:prof
```

6. Source setup scripts: `source setup/setup.sh`
7. `cd $MRB_TOP && mrbsetenv`

### 0.4. Setup a working environment on `icarusgpvm` machine
The first thing you run should be `source /cvmfs/icarus.opensciencegrid.org/products/icarus/setup_icarus.sh`. 

Then you run `mrb newDev` in the folder of your choice. This will create source, products and build folders for you. Then `mrbsetenv` before starting to work.
```
$ mkdir myFolder && cd myFolder
$ mrb newDev
```

You need to run the following whenever you log in:
```
$ source localProducts_larsoft_v09_63_00_e20_prof/setup
$ mrbsetenv # check that all dependencies are ok
```

You do not touch the build or products folders, only the source one. When you do `mrb i`, changes will be copied over including FHICL file changes.
Check out the [SBN wiki](https://sbnsoftware.github.io/sbndcode_wiki/How_to_setup_your_directory_and_launch_your_first_job.html) for more detailed explanations on this way of working.

## 1. Cloning source code
You will need `sbncode` which contains the code for Supera. 
```
$ mrbsetenv
$ cd $MRB_TOP/srcs
$ mrb g sbncode
```

You will also need experiment-specific code, for example `icaruscode` for ICARUS or `sbndcode` for SBND.
If you don't plan to make any modifications to that code, then it is sufficient to just point to the CVMFS locations:
```
$ setup icaruscode v09_63_00 -q e20:prof
```
Otherwise, also clone them:
```
$ mrb g icaruscode # or sbndcode
```

Then it is time to compile everything as it is (this will take a few minutes):
```
$ mrb i -j4
```

Supera lives in `sbncode/sbncode/Supera`. This is where you want to be for the rest of the tutorial:
```
$ cd srcs/sbncode/sbncode/Supera
```

## 2. What do you actually want to generate?
Here are the typical successive steps to make a simulation sample:

1. Generator stage. This can be GENIE (BNB, NuMI), CORSIKA or other.

2. Optionally: filter. for example, you may want to discard events where the neutrino is
generated outside the active volume.

3. LArG4 stage. G4 propagates the particles generated at step 1 through the liquid argon.

4. DAQ. Detector-dependent, obviously.

5. MCRECO. This module makes `sim::MCTrack` and `sim::MCShower` objects.

6. Detector-specific processing: ROI finding, decoding, hit finding, etc.

7. Cluster3D makes `recob::SpacePoint` objects.

8. Optionally, you can run optical simulation and reconstruction.

9. Optionally, you can run CRT simulation and reconstruction.

10. At analyzer stage, you want to run Supera to make MC truth labels and output a LArCV file.
 
I think this covers most of what you might want to simulate. The official production team usually has different
FHICL files that break down this simulation into stages, but that complicates the workflow with grid jobs. We
don't have a team at hand, so we'll put everything into the same FHICL file to run the simulation from scratch
all the way to Supera at once.

## 3. Include the right FHICL files defining each stage
For our example, this is what we'll include (ICARUS specific):

```
#include "services_icarus_simulation.fcl"
#include "largeantmodules_icarus.fcl"
#include "detsimmodules_ICARUS.fcl"
#include "stage0_icarus_defs.fcl"
#include "stage0_icarus_mc_defs.fcl"
#include "stage1_icarus_defs.fcl"
#include "channelmapping_icarus.fcl"
#include "timing_icarus.fcl"
#include "g4inforeducer.fcl"
#include "mcreco.fcl"
```

For Supera, this  is the key include:
```
#include "supera_modules.fcl"
```


Depending on which generator you want to use, include the right FHICL:
```
#include "corsika_icarus.fcl"
#include "genie_icarus_bnb.fcl"
#include "genie_icarus_numioffaxis.fcl"
```

If you want to run optical simulation/reconstruction, you will need:
```
#include "opdetsim_pmt_icarus.fcl"
```

And if you want to use a filter, also include the proper FHICL:
```
#include "FilterNeutrinoActive.fcl"
```

## 4. Process name and services
```
process_name: NuMISupera

services:
{
	@table::icarus_g4_services
	@table::icarus_detsim_services
	@table::icarus_gen_services
	IFDH: {}
	IICARUSChannelMap: @local::icarus_channelmappinggservice
	IPMTTimingCorrectionService: @local::icarus_pmttimingservice
}
```

Be aware that the process name needs to be unique. It needs to be different
from any process name in your existing LArSoft file, if you do not start from
scratch.

## 5. What is your input?
If you are starting from scratch, then your input will look like this:

```
source:
{
	module_type: EmptyEvent
	timestampPlugin: { plugin_type: "GeneratedEventTimestamp" }
	maxEvents: 10
	firstRun: 1
	firstEvent: 1
}
```

If instead you are starting from an existing LArSoft ROOT file, then you want to use this:

```
source:
{
	module_type: RootInput
}
```

## 6. Define producer / filter / analyzer modules
### 6.1. Define producer modules
This is where you write the list of producer modules you *might* want to use.
```
physics:
{

 producers:
 {
	rns: { module_type: RandomNumberSaver }
	#generator: @local::icarus_genie_BNB # from `genie_icarus_bnb.fcl`
 	generator: @local::icarus_genie_NuMI # from `genie_icarus_numioffaxis.fcl`
	cosmgen: @local::icarus_corsika_cmc
	largeant: @local::icarus_largeant

	# Reco producers
	@table::icarus_stage0_producers
	@table::icarus_stage1_producers
	@table::icarus_stage0_mc_producers

	# Saving MC information
 	mcreco:   @local::standard_mcreco

 	# Saving SimEnergyDeposit in lite version
 	sedlite: @local::sbn_largeant_info_reducer

	# Multi-TPC DAQ
	daq0: @local::icarus_simreadoutboard
	daq1: @local::icarus_simreadoutboard
	daq2: @local::icarus_simreadoutboard
	daq3: @local::icarus_simreadoutboard

	opdaq: @local::icarus_simpmt # opdetsim_pmt_icarus.fcl
 }
(...)
}
```

Check out the sources if you want to understand the full list that you are invoking: stage0 and stage1
producers for ICARUS are defined in [here](https://github.com/SBNSoftware/icaruscode/tree/develop/fcl/reco/Definitions).

### 6.2. Define filter modules
If you want to use a filter, this is where to define it:

```
physics:
{
(...)
 filters:
 {
	  eventActive: @local::icarus_FilterNeutrinoActive # from `FilterNeutrinoActive.fcl`
 }
(...)
}
```

### 6.3. Define analyzer modules
You can run other analyzers, but we'll stick to Supera.

#### Ready-to-use modules to run Supera

Supera "default" configurations are pre-defined for convenience and listed in `job/include/supera_modules`.
The rest of the FHICL files in that directory `job/include` are LArCV configurations meant for Supera.
Unless you know what you are doing, you probably don't need to change anything in these files and you can
use one of them out of the box by invoking the right module from `supera_modules.fcl`.

* `icarus_supera_generator` or `icarus_supera_generator_PMT_CRT` if you have just 1 generator module named
`generator`,
* `icarus_supera_cosmgen` or `icarus_supera_cosmgen_PMT_CRT` if you have just 1 generator module named
`cosmgen`,
* `icarus_supera_MC` or `icarus_supera_MC_PMT_CRT` if you have both `generator` and `cosmgen` defined in
your modules.

If you have generator modules whose names or count are not covered by a combination of `generator` and `cosmgen`,
or if your detector geometry is different from the ICARUS one (in which case you may need to change the box size),
you will have to copy and edit one of the Supera configurations by yourself. Ask us for help!

#### Example
We are only running GENIE, no cosmics, so we want to use `icarus_supera_generator`.
If you also record PMT or CRT information, you want to use `icarus_supera_generator_PMT_CRT`.
 
```
physics:
{
(...)
 analyzers:
 {
   #supera: @local::icarus_supera_MC_PMT_CRT
   supera: @local::icarus_supera_generator_PMT_CRT
 }
(...)
}
```

## 7. Define what to run exactly
You want to put in `simulate` the list of producer modules to run. This always needs to contain
at least the `rns` (random number service) module. The `end_paths` list can contain either `analyze`
(which will make a LArCV out file) and/or `out_stream` (which will make a LArSoft output file). 
Here is a full example:

```
physics:
{
(...)
 simulate: [ rns, generator,
            largeant, sedlite,
            mcreco,
            daq0, daq1, daq2, daq3,
	    MCDecodeTPCROI,
	    @sequence::icarus_stage0_multiTPC_TPC, # decon1droi, roifinder
	    @sequence::icarus_stage0_EastHits_TPC,
	    @sequence::icarus_stage0_WestHits_TPC,
	    cluster3DCryoE,
	    cluster3DCryoW]
 analyze: [supera]
 out_stream: [ out1   ]

 trigger_paths: [simulate]
 #end_paths:     [analyze,out_stream]
 end_paths:     [analyze]
 #end_paths: [out_stream]
}
```

If you also want to run the optical side, you need to add to your `simulate` list these modules:

```
 simulate: [ (...),
            opdaq,
            pmtfixedthr,
            ophit,
            opflashCryoE,
            opflashCryoW,
	    (...) ]
```

To include CRT simulation and reconstruction:
```
 simulate: [ (...),
            daqCRT,
            crthit,
	    (...) ]
```

## 8. Configure each producer module
There are knobs that you want to know about: for example, which flavor of neutrino to produce or what volume
to restrict the generator to. 

### Generator stage
For GENIE:
```
#
# Generator BNB
#
# Fixing error 65 = missing flux files when running neutrino interaction
physics.producers.generator.FluxCopyMethod: "IFDH"
physics.producers.generator.GenFlavors: [ 12 ] # 12 for nue, 14 for numu
#physics.producers.generator.MixerConfig: "map 14:12 -14:-12 12:14 -12:-14" # oscillated
physics.producers.generator.TopVolume: "volCryostat" #"volDetEnclosure" #box including all cryostats, instead of "volCryostat" which is only C:0
```

There is a list of event types [here](https://cdcvs.fnal.gov/redmine/projects/nutools/wiki/GENIE_Configuration_Files) (thanks Kazu!). So you can easily change the generator settings:

```
physics.producers.generator.GenFlavors: [ 12 ]
physics.producers.generator.EventGeneratorList: ["CCQE"]
```

### LArG4
This is where you want to make sure that `StoreDroppedMCParticles` is enabled.
If you don't need optical simulation, you can uncomment the commented lines to
make your simulation a lot faster (no more photon library loading).
```
#
# LArG4
#
physics.producers.largeant.KeepParticlesInVolumes: [ "volCryostat" ]
#services.LArG4Parameters.NoPhotonPropagation: true
services.LArG4Parameters.ParticleKineticEnergyCut: 0.0005
#services.LArPropertiesService.ScintYield: 0
#services.LArPropertiesService.ScintByParticleType: false
physics.producers.largeant.StoreDroppedMCParticles: true
```

### MCRECO
Here is another section where you want to pay attention.
```
#
# MCRECO
#
physics.producers.mcreco.SimChannelLabel: "sedlite"
physics.producers.mcreco.MCParticleLabel: "largeant"
physics.producers.mcreco.MCParticleLiteLabel: "largeant"
physics.producers.mcreco.UseSimEnergyDeposit: false
physics.producers.mcreco.MCRecoPart.SavePathPDGList: [13,-13,211,-211,111,311,310,130,321,-321,2212,2112,2224,2214,2114,1114,3122,1000010020,1000010030,1000020030,1000020040]
physics.producers.mcreco.UseSimEnergyDepositLite: true
physics.producers.mcreco.IncludeDroppedParticles: true
```
`UseSimEnergyDepositLite` needs to be `true`, and so does `IncludeDroppedParticles`.

### Optical simulation
```
# OpHit config
physics.producers.ophit.InputModule: "opdaq"
```

### DAQ
```
#
# DAQ continued - point each of the SimWire instances to a different TPC set
#
physics.producers.daq0.OutputInstanceLabel: "PHYSCRATEDATATPCEE"
physics.producers.daq0.TPCVec:              [ [0, 0], [0, 1] ]
physics.producers.daq1.OutputInstanceLabel: "PHYSCRATEDATATPCEW"
physics.producers.daq1.TPCVec:              [ [0, 2], [0, 3] ]
physics.producers.daq2.OutputInstanceLabel: "PHYSCRATEDATATPCWE"
physics.producers.daq2.TPCVec:              [ [1, 0], [1, 1] ]
physics.producers.daq3.OutputInstanceLabel: "PHYSCRATEDATATPCWW"
physics.producers.daq3.TPCVec:              [ [1, 2], [1, 3] ]
```

### Decoding and all that
Because we want to have all TPCs at once, we need to adjust some of the input/output labels:
```
#
# MCDecodeTPCROI > decon1droi > roifinder
#

physics.producers.decon1droi.RawDigitLabelVec: ["MCDecodeTPCROI:PHYSCRATEDATATPCWW","MCDecodeTPCROI:PHYSCRATEDATATPCWE","MCDecodeTPCROI:PHYSCRATEDATATPCEW","MCDecodeTPCROI:PHYSCRATEDATATPCEE"]
physics.producers.MCDecodeTPCROI.FragmentsLabelVec: ["daq3:PHYSCRATEDATATPCWW","daq2:PHYSCRATEDATATPCWE","daq1:PHYSCRATEDATATPCEW","daq0:PHYSCRATEDATATPCEE"]
physics.producers.MCDecodeTPCROI.OutInstanceLabelVec: ["PHYSCRATEDATATPCWW","PHYSCRATEDATATPCWE","PHYSCRATEDATATPCEW","PHYSCRATEDATATPCEE"]
physics.producers.roifinder.WireModuleLabelVec: ["decon1droi:PHYSCRATEDATATPCWW","decon1droi:PHYSCRATEDATATPCWE","decon1droi:PHYSCRATEDATATPCEW","decon1droi:PHYSCRATEDATATPCEE"]
```

### Cluster3D
Until this becomes default:
```
#
# Cluster3D
#
physics.producers.cluster3DCryoE.Hit3DBuilderAlg.MinPHFor2HitPoints: 0.
physics.producers.cluster3DCryoW.Hit3DBuilderAlg.MinPHFor2HitPoints: 0.
``` 

## 9. Output
This defines what happens if you decide to output a LArSoft file.
```
outputs:
{
 out1:
 {
   module_type: RootOutput
   fileName:    "larsoft.root"
   dataTier:    "reco"
   compressionLevel: 1
   # Only uncomment if selecting events with nu in active volume
   # SelectEvents: [simulate]
 }
}
```
If you are running an event filter, don't forget to uncomment the last line.

## 10. Generate what you want and process with Supera
Time to run! I'll assume you read the above and put together a reasonable FHICL file corresponding to
what you want to run. To actually run it interactively for 1 event, do `lar -c your_fhicl.fcl -n 1`.

### Debugging your FHICL file
It's painful, but here are a few tips:

**Is it tricking you?** Sometimes your FHICL file might not run with the parameters that you think you
put in there. Use the command line tool `fhicl-dump`. For example
 `fhicl-dump you_fhicl.fcl | grep -C 5 yourParameter` to find out
if you are being fooled.

**Corsika not finding files** This can happen if you do not have a proper Kerberos ticket. Do `kinit`
again followed by `kx509`, and try again.

**Segmentation fault** Load the C++ debugger (`setup gdb v9_2` for example) and run your FHICL file in
debug mode to catch what created the segfault:

```
$ gdb --args lar -c your_fhicl.fcl -n 1
> catch throw
> run
...
Segfault
> backtrace 20
(read the traceback and look for hints of what caused the segfault)
```

Sometimes GDB catches a "geometry file not found" exception which comes back over and over. To remove
a breakpoint that you know is not your problem:

```
> info b
(lists all breakpoints, look at their ids #)
> del 1 (for example)
> continue
```

**Geometry mismatch** (Icarus specific) it's possible you are reading data files that were generated with an old geometry.
In that case you need to adapt your FHICL file with a service like this:

```
#include "geometry_icarus.fcl"
services:
{
	(...)
	@table::icarus_geometry_services_overburden_legacy_icarus_v3
}
```

Obviously this might change over time, so ask around if it doesn't work exactly.

### Visualizing the output
Running Supera should have created a LArCV file called `larcv.root`. If you also enabled the LArSoft output, then you'll have a `larsoft.root` file that you can inspect with something like `lar -c eventdump.fcl -n 1 -s larsoft.root`.

## 11. Preparing to run jobs on the grid
At the end of the interactive run, you will get a summary of resources usage (memory & time) which will be useful to determine how many events / jobs you need and how much resources to ask for in your job submission.

>The idea is that we will make a tarball with the source code, put it in a PNFS path that is accessible from jobs, write an XML file to describe job submission details and that will do it!

### Writing a XML file for `project.py`
`project.py` is a wrapper written by physicists around the grid scheduler (i.e. SLURM or equivalent). It reads a configuration XML file and submits the job request for you. Also see [here](https://sbnsoftware.github.io/sbndcode_wiki/Using_projectpy_for_grid_jobs.html) for more information about it.

So, we need to write an XML file to configure our job submission for `project.py`. You need a file called something like `run_nue.xml` in your `$MRB_TOP` folder. To get you started you can copy an example:

```xml
<?xml version="1.0"?>

<!-- Production Project -->

<!DOCTYPE project [
<!ENTITY release    "v09_63_00">
<!ENTITY releasetag "e20:prof">
<!ENTITY my_version "v00">
<!ENTITY name       "nue">
<!ENTITY file_type  "mc">
<!ENTITY run_type   "physics">
<!ENTITY PNFSpath   "/pnfs/icarus/scratch/users/ldomine/">
]>

<project name="&name;">

  <!-- Group -->
  <group>icarus</group>

  <!-- Project size -->
  <numevents>200</numevents>

  <!-- Operating System -->
  <os>SL7</os>

  <!-- Batch resources -->
  <resource>DEDICATED,OPPORTUNISTIC</resource>

  <!-- Larsoft information -->
  <larsoft>
    <tag>&release;</tag>
    <qual>&releasetag;</qual>
    <local>&PNFSpath;tars/&release;.tar</local>
  </larsoft>

  <!-- Project stages -->
  <fcldir>&PNFSpath;tars/</fcldir>

  <stage name="reco">
		<fcl>your_fhicl.fcl</fcl>
		<outdir>/pnfs/icarus/scratch/users/ldomine/&name;_&my_version;/&release;</outdir>
		<workdir>/pnfs/icarus/scratch/users/ldomine/&name;_&my_version;_work/&release;</workdir>
		<logdir>/pnfs/icarus/scratch/users/ldomine/&name;_&my_version;_log/&release;</logdir>
    <numjobs>10</numjobs>
		<datatier>larcv</datatier>
		<defname>&name;_&my_version;</defname>
    <memory>8000</memory>
    <disk>30GB</disk>
    <jobsub>--expected-lifetime=1h -e IFDH_FORCE</jobsub>
  </stage>

  <!-- file type -->
  <filetype>&file_type;</filetype>

  <!-- run type -->
  <runtype>&run_type;</runtype>

</project>
```
The `<!ENTITY variable_name "variable_value">` lines define some variables for convenience that you can reuse later in the XML file. This is a single stage job, so fairly simple. The main parameters you want to adjust are
* `<numevents>200</numevents>` this is the TOTAL number of events that you want to generate.
* `<numjobs>10</numjobs>` number of jobs that you want to submit. In this example, it means that each job will have to generate 200 / 10 = 20 events.
* `<fcl>your_fhicl.fcl</fcl>` name of fhicl file to run
* `<memory>8000</memory>` you want to adapt this based on your own interactive tests and how many events per job you will be generating.
* Edit `--expected-lifetime=5h` to specify a job lifetime (default is 8h I believe). You can estimate this again from your interactive tests.
* `<stage name="reco">` the name `reco` is what you will specify to `project.py` when you run it and tell it which stage to run (e.g. `--stage reco`). Here we have only 1 stage.

Obviously, you also need to update the paths for yourself.

>Have to put this advice somewhere: running test jobs is **important**. You do not want to repeatedly submit batches of 500 jobs that fail immediately for stupid reasons.
>
>Do a 1 event interactive run to determine resources usage and don't be too harsh on the limits that you set, to avoid having too many jobs held. 
>
> Do a test job submission to check your XML file. Submit <10 jobs, create a few events. Go through the full pipeline, download the ROOT files and look at them. In a ROOT browser and in an event display. 
>
> Once you are satisfied, there was no unexpected crash and the output looks like you expect, you can start scaling up your job submission.


### Preparing the tarball and XML file
You will need the right Kerberos tickets before submitting jobs / using `ifdh` command line tool.
1. `kinit USERNAME@FNAL.GOV && kx509`

> Probably don't need to tarball if the new FHICL is the only change, to update in future.

Since we modified the tagged `sbncode` to include our new FHICL and potentially other modifications, we need to package it into a tarball that will be distributed to each job. You can copy and use the script `make_tar.sh` that Kazu wrote:

2. `ifdh cp -D /pnfs/icarus/persistent/users/ldomine/make_tar.sh ./`
3. `sh make_tar.sh -d $MRB_INSTALL v09_63_00.tar`

To make it visible by the jobs, I put the tarball / XML / fcl files somewhere under `/pnfs/icarus/scratch/users/USERNAME/your_folder`. Copy the tarball, XML run file and FHICL files to that directory.

4. `ifdh mkdir /pnfs/icarus/scratch/users/USERNAME/your_folder`
5. `ifdh cp -D run_nue.xml v09_63_00.tar srcs/sbncode/sbncode/Supera/your_fhicl.fcl /pnfs/icarus/scratch/users/USERNAME/your_folder`

### Launching the jobs from a GPVM machine
Make sure you are on a GPVM machine for this section:

6. `ssh USERNAME@icarusgpvm01.fnal.gov`
7. `kinit USERNAME@FNAL.GOV && kx509` (if different machine, make sure you have the Kerberos tickets to submit grid jobs)

You need to source / setup a few things to run the job submission smoothly:

8. `source /cvmfs/icarus.opensciencegrid.org/products/icarus/setup_icarus.sh`
9. `setup -B icarusutil v09_26_00 -q:e20:prof` (contains `project.py` script)
10. `setup jobsub_client` (to interact with grid scheduler)
11. `export IFDH_FORCE=gsiftp`

And finally we can start the job submission:

12. `project.py --xml /pnfs/icarus/scratch/users/USERNAME/your_folder/run_nue.xml ---stage reco --submit`



## 12. Monitoring and retrieving the job results

### 12.1. Keeping an eye on the grid jobs
Great, now you can use `jobsub_*` command line tools to interact with your jobs.
* `jobsub_q --user=USERNAME` to watch your jobs.
* `jobsub_rm --user=USERNAME` or `jobsub_rm --jobid xxxxx.@xxxx.fnal.gov` to cancel jobs.
* `jobsub_fetchlog --jobid xxxxx.xx@xxxx.fnal.gov` will retrieve the logs for you. It will bring a `.tar` compressed folder in your current directory. Beware before unzipping it, it contains a lot of log files in the root directory! Create a temporary directory to unzip it (`tar -xvf the_log.tar`) without overwhelming your working directory with log files.

If a job is "held", it means that it went above the requested memory or allocated time. Jobs will be "idle" until they are assigned a slot to start running. 

When the batch of jobs is done, you will receive an email with statistics about the resources usage (use it to refine your resources request, if you only used <50% of memory or time) and how many jobs succeeded (exit code 0). Use `jobsub_fetchlog` to investigate jobs that failed.

### 12.2. Once the jobs are done, retrieving the output
There are probably other ways of doing it. This is how I have been doing it so far, feel free to follow your own path!

To retrieve the job results, I use another script written by Kazu and barely changed by me:
1. `ifdh cp -D /pnfs/icarus/persistent/users/ldomine/link_maker.sh ./ `
It retrieves all the jobs outputs and creates symlinks for you. You may want to edit it to change the `OUTPUT_DIR` to suit your needs:
```python
OUTPUT_DIR  = '/icarus/data/users/%s/nue_symlink/%s' % (USER,UNIQUE_NAME)
```

If you leave it as is, it will create a directory under `/icarus/data/users/%s/nue_symlink` and put all symlinks in that folder.

2. `python3 link_maker.py /pnfs/icarus/scratch/users/USERNAME/your_folder/run_nue.xml`
The output will confirm where the symlinks were put. It uses the information in your XML file to determine a name for the folder. For example, `/icarus/data/users/USERNAME/nue_symlink/larcv_nue_v01`.

The last step is to copy the actual ROOT files to your SDF space (or wherever you plan to store them):

3. `rsync -r -e ssh -L --update --progress -v /icarus/data/users/USERNAME/nue_symlink/larcv_nue_v01 USERNAME@sdf-login.slac.stanford.edu:/sdf/group/neutrino/USERNAME/`

depending on how many files you are retrieving, this may take a while, but fortunately `rsync` can be interrupted and resumed as necessary.
