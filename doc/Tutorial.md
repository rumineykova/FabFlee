FabFLEE coupled UQ tutorial
=====

In this tutorial we will explain how you can combine a simple stochastic conflict evolution model (Flare) with an agent-based migration model (Flee), perform a set of runs based on different conflict evolutions, and visualize the migrant arrivals with confidence intervals. The Flee agent-based migration model has been used in a *Scientific Reports* paper to make forecasts of forced migration in conflicts (https://www.nature.com/articles/s41598-017-13828-9), while the Flare model is still in prototype stage. In this example, we modelled the 2012 Northern Mali conflict.

![Graphical depiction of population movements in Mali. Background image is courtesy of Orionist (Wikimedia)](https://raw.githubusercontent.com/djgroen/FabFlee/master/doc/mali-arrows-border.png)

<!---
We combine these two models using the FabSim3 automation toolkit (http://fabsim3.readthedocs.io), which features an integrated test infrastructure and a flexible plugin system. Specifically, we will showcase the FabFlee plugin which automates complex simulation workflows involving forced migration models. 
-->
The [FabSim3](https://fabsim3.readthedocs.io/) features an integrated test infrastructure and flexible plugin system. In this tutorial, we showcase the FabFlee Plugin, which automates complex simulation workflows involving forced migration model. We explain how you can combine a simple stochastic conflict evolution model namely the 2012 Northern Mali Conflict

# 1. Setup

This tutorial uses a range of VECMAtk components, as shown in the Tube Map below.

![Graphical depiction of the VECMAtk components used in the FabFlee tutorial](https://raw.githubusercontent.com/djgroen/FabFlee/master/doc/FabFleeMap.png)

The basic tutorial relies primarily on FabSim3, and specifically on the FabFlee plugin of that toolkit. The advanced content in section 3, however, enables you to use several QCG tools as well.

Please refer to https://github.com/djgroen/FabFlee/blob/master/doc/TutorialSetup.md for details on how to download, install and configure the software required for this tutorial.

# 2. Forced migration simulations

In this section we will show you how you can run different types of migration simulations. We first explain how you can do basic analysis using a single model, and then explain how you can perform a coupled application run that features basic uncertainty quantification.

## 2.1 Executing single-model migration simulations

FabFlee comes with a range of sample simulation domains. 

1. To run a single population displacement validation test, simply type:
```
fabsim localhost flee:<conflict_name>,simulation_period=<number>
```

For instance, a basic model for the 2012 Mali conflict can be found in
`(FabSim Home)/plugins/FabFlee/config_files/mali`.
```
fabsim localhost flee:mali,simulation_period=50
```
> _NOTE : regular runs have a `simulation_period` of 300 days, but we use a simulation period of 50 days to reduce the execution time of each simulation in this tutorial_

2. You can copy back any results from completed runs using:
```
fabsim localhost fetch_results
```
The results will then be in a directory inside `(FabSim Home)/results`, which is most likely called `mali_localhost_16`.

This is a little redundant for runs on localhost, but essential if you run on any remote machines, so it is good to get into this habit.

3. You can plot the simulation output using:
```
fabsim localhost plot_output:mali_localhost_16,out
```

## 2.2 Ensembles

Now you may want to run multiple simulations, to see to what extent the definition of the maximum run speed in Flee affects the overall results. To do so, you can create an ensemble definition.

### Step 1: Duplicate configuration directory

Your main configuration directory for this ensemble is in `config_files/mali`. To create a run speed test, it is best to duplicate this directory by:
```
cp -r (FabFlee Location)/config_files/mali (FabFlee Location)/config_files/mali_runspeed_test
```

### Step 2: Create SWEEP directory

Next, you should create a directory named `SWEEP` inside this directory, e.g. through
```
mkdir (FabFlee Location)/config_files/mali_runspeed_test/SWEEP
```

Inside this SWEEP directory, you can then provide modified input files for each particular run instance by creating a subdirectory for it.

<br/> For instance, to create a run instance with a maximum movement speed of 200km/day for people escaping conflict, we can create a subdirectory called `200`, and create a simsetting.csv file in it with the following contents:`"MaxMoveSpeed",200`

To illustrate **simsetting.csv** file:

|MaxMoveSpeed |200 |
|-------------|----|

You can then create similar directories with inputs that have a run speed of 100, or 400. Or if you're too lazy to do that, just copy the contents of `(FabFlee Location)/config_files/mali/example_sweepdir` to `(FabFlee Location)/config_files/mali_runspeed_test/SWEEP`. 

### Step 3: Run an ensemble of run speed tests

To run the ensemble, you can type:
```
fabsim localhost flee_ensemble:mali_runspeed_test,simulation_period=50
```

### Step 4: Analyze the output

You can copy back any results from completed runs using:
```
fabsim localhost fetch_results
```
The results will then be in a directory inside `(FabSim Home)/results` which is most likely called `mali_runspeed_test_localhost_16`.

And you can plot the simulation output using:
```
fabsim localhost plot_uq_output:mali_runspeed_test_localhost_16,out
```
As a reminder: we use `plot_output` to visualize outputs of a single run, and `plot_uq_output` to collate and visualize results from an ensemble.

As output you will get a range of files in the `out` subfolder of your results directory. For example, the image `Niamey-4_V2.png`, which visualizes migrant arrivals in Niamey with 95% confidence intervals based on the move speed, might look like this:

![Arrivals with confidence interval based on movespeed](https://raw.githubusercontent.com/djgroen/FabFlee/master/doc/Niamey-4_V2.png)


## 2.3 Executing coupled migration simulations

we ultimately wish to perform of coupled models, in context of multiscale uncertainty quantification. The relevant workflow comprises the following:

1. We run a set of simple conflict evolution simulations (Flare) in the context of Mali.
2. We gather the conflict evolutions generated by this simulation, and convert them to create input for an ensemble of agent-based migration (Flee) simulation.
3. We run a Flee simulation for each conflict evolution generated.
4. We analyze and visualize a basic result.

### Step 1: Run a Flare ensemble

To run an ensemble of Flare simulations, generating different conflict evolutions, one can simply type:
```
fabsim localhost flare_ensemble:mali,N=10,simulation_period=50,out_dir=flare-out-scratch
```
This generates a range of CSV files, which you can find in `(FabFlee Home)/results-flare/flare-out-scratch`.


### Step 2: Convert Flare output to Flee input

To convert this output to Flee input you can then use.
```
fabsim localhost couple_flare_to_flee:mali,flare_out=flare-out-scratch
```
This generates a SWEEP directory in `(FabFlee Home)/config_files/mali`, which contains all the different conflict evolutions.


### Step 3: Run an ensemble of Flee simulations

To then run a Flee ensemble over all the different configurations, simply type:
```
fabsim localhost flee_ensemble:mali,simulation_period=50
```
Note that for Flee ensembles there is no need to specify the parameter `N`. It simply launches one run for every subdirectory in the `SWEEP` folder.

### Step 4: Analyze the output

You can copy back any results from runs using:
```
fabsim localhost fetch_results
```
The results will then be in a directory inside `(FabSim Home)/results` which is most likely called `mali_localhost_16`.

Assuming this name, you can then run the following command to generate plots:
```
fabsim localhost plot_uq_output:mali_localhost_16,out
```
And you can inspect the plots by examining the `out` subdirectory of your results directory. For instance, if you look at `Bobo-Dioulasso-4_V2.png`, it might look like this:

![Arrivals in Bobo-Dioulasso with confidence interval based on conflict evolution](https://raw.githubusercontent.com/djgroen/FabFlee/master/doc/Bobo.png)

### Step 1-3 in a one-liner

To run a coupled simulation with basic UQ, and basically repeat steps 1-3 in one go, just type:
```
fabsim localhost flee_conflict_forecast:mali,N=10,simulation_period=50
```




# 3. Going the extra mile (content for advanced users)

These advanced tasks are intended for those who have access to the Eagle supercomputer, and who would like to try some of the more advanced features of FabSim3.

## Additional setup tasks for advanced use
Before running any simulation on a remote supercomputer, you'll need to do the following:

- Make sure the target remote machine is properly defined in `(FabSim Home)/deploy/machines.yml` 
- Since that, in Flee, some python libraries such as `numpy` will be used for the job execution, in case of nonexistent of those packages, we recommended to install a _virtualenv_ on the target machine. It can be done by running

	> for QCG machine : `fab qcg install_app:QCG-PilotJob,virtual_env=True`
	>
	> For SLURM machine : `fab eagle install_app:QCG-PilotJob,virtual_env=True`
	> 
	> **NOTE** : the installation path (`virtual_env_path`) is set on `machines.yml` as one of parameters for the target remote machine
	> 
	> By installing this _virtualenv_ on the target remote machine, the [QCG Pilot](https://github.com/vecma-project/QCG-PilotJob) Job service will be also installed alongside with other required dependencies 


### Running the coupled simulation on a supercomputer
```
fabsim eagle flee_conflict_forecast:mali,N=20,simulation_period=50
```
1. Run `fabsim eagle job_stat_update` to check if you jobs are finished or not
2. Run `fabsim eagle fetch_results` to copy back results from `eagle` machine. The results will then be in a directory inside `(FabSim Home)/results`, which is most likely called `mali_eagle_16`
3. Run `fabsim localhost plot_uq_output:mali_eagle_16,out` to generate plots


<!---
### Running an ensemble simulation on a supercomputer using Pilot Jobs
```
fabsim qcg flee_ensemble:mali,N=20,simulation_period=50,PilotJob=true
```
-->

### Running an ensemble simulation on a supercomputer using Pilot Jobs and QCG Broker

```
fabsim qcg flee_ensemble:mali,N=20,simulation_period=50,PilotJob=true
```
1. Run `fabsim qcg job_stat_update` to check if you jobs are finished or not
2. Run `fabsim qcg fetch_results` to copy back results from `qcg` machine. The results will then be in a directory inside `(FabSim Home)/results`, which is most likely called `mali_qcg_16`
3. Run `fabsim localhost plot_uq_output:mali_qcg_16,out` to generate plots

# 4. Acknowledgements

This work was supported by the VECMA and HiDALGO projects, which have received funding from the European Union Horizon 2020 research and innovation programme under grant agreements No 800925 and 824115.