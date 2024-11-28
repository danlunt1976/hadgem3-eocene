[[_HadGEM3 MOC]]

# create spin-up suite
- I spent some time to think about a suite suitable for several hundreds of years of Eocene spin-up
- main things to consider that will impact run time and resource usage are model output (STASH) and domain decomposition (number of processors/nodes)
- will first reduce the STASH to a bare minimum set of variables and then use this to do to a quick performance vs processors vs cost comparison to settle on a config for the Eocene spin-up
- created a new copy of hello world suite **u-ch912** for STASH changes and will work with **u-ch949** in the following
- I reduced the total run length to 1 month and 1 hour wallclock time for short performance benchmarks in the following

## STASH output and dumping frequency
- current suite **u-ch949** uses the full CMIP6 PI output fields (STASH)
- this leads to a significant increase in computing time (to write the files) and used disk space. For example, one model month takes about 42 minutes and amounts to 30 GB of data after post-processing (more details at [[performace on ARCHER2]])
- atmosphere output fields can be changed at `> um > namelist > Model Input and Output > STASH Requests and Profiles > STASH Requests`
- first, I turned off (right-click and select 'Ignore this section') all STASH entries to get a feeling about how important the UM STASH is in terms of computation time + file sizes. This caused the following crash in the model:

	```    
	???!!!???!!!???!!!???!!!???!!! ERROR ???!!!???!!!???!!!???!!!???!!!
	? Error code: 1
	? Error from routine: OASIS_INITA2O
	? Error message:
	? 16 item(s) missing from STASH coupling fields.
	? Please refer to standard output for full details.
	? Error from processor: 186
	? Error number: 26
	```

- so it seems some (i.e. a total of 16) output fields are necessary to write out at the coupling time steps to be used by the OASIS couple
- I turned them back on. Easiest way seems to be to group by 'use_name' and then right-click on 'UPCOUP' and 'Enable this section". Indeed, the UPCOUP inluded 16 fields as mentioned in the error message
- ran the model again, this time got a new error message:

	```
	???!!!???!!!???!!!???!!!???!!! ERROR ???!!!???!!!???!!!???!!!???!!!
	? Error code: 37
	? Error from routine: UKCA_MAIN
	? Error message: Some item addresses not found in D1 array
	? Error from processor: 43
	? Error number: 10
	```

- there is another 'use_name' group called 'UPUKCA' containing around 20-30 fields written at every time step? ('TALLTS'), so I enabled this one also
- **this time the coupled stage ran successfull! This means the suite *u-ch983* seems to contain the bare minimum of UM STASH necessary to run the coupled model!**
- next, I manually added some 2D monthly mean fields that I think should be enough as output fields for a long spin-up (we should get yearly snapshots of all 3D fields via the restart dumps anyways). The fields currently included in suite **u-ch988** are:
	- surface temp, 1.5m temp
	- sea ice fraction
	- surface pressure, sea level pressure
	- TOA SW down and up, OLR
	- 10m u and v winds
	- surface sensible and latent heat fluxes
	- total precip
	- soil moisture
	- snow amount

- did some benchmarks to compare [[performace on ARCHER2]] for dfferent STASH configurations

## NEMO CICE output
- NEMO output can be controlled via `~/roses/<SUITE-ID>/app/nemo_cice/file/iodef.xml`
- CICE output frequency via namelist parameters `histfreq` and `histfreq_n`,
	- default is set to `histfreq='d','h','x','x','x'` and `10,24,1,1,1`;
	- only monthly output via: `histfreq = ’m’,’x’,’x’,’x’,’x’`
	- no output via `histfreq = ’x’,’x’,’x’,’x’,’x’`

## domain decomposition
> The base ARCHER2 charging unit is a node irrespective of how many cores on the node are being used. ARCHER2 has 128 cores per node, and for the UM each MPI task and OpenMP thread is mapped to a separate core. So best tro try to fill each node with maximum numbe rof cores.
- domain decomposition of hello world suite **u-ch912** from [this list of standard jobs](http://cms.ncas.ac.uk/wiki/Archer2):
	- atmosphere: 12 x 16 x 2 OpenMP threads = 384 cores / 128 = 3 nodes
	- ocean: 16 x 8 x 1 OpenMP thread = 128 cores / 128 = 1 node
	- xios = 6 processes / 2 CPUs per node in XIOS = 3 nodes
		- this seems strange, I checked and I also get charged for these three nodes when running the coupled model (total of 7 nodes = 3+1+3). But this should only be one node. On NEXCS, Charlie did set the variable `CPUs per node in XIOS` to 16, so all 6 processes are on the same node. I tried the same and actually only get charged for the one xios node, so a total of 5 nodes for the whole model. I leave it now at 16 in **u-ch949** to use less nodes, but don't understand why it is different in the original standard suite.