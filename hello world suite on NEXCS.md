[[_HadGEM3 MOC]]

## hello world suite on NEXCS NEXCS
With the set-up steps ([[getting accounts]], [[setting-up NEXCS]]) completed, you should now be able to login to NEXCS and open the Rose GUI with:
```bash
ssh -Y <USERNAME>@lander.monsoon-metoffice.co.uk 
ssh -Y -A xcslc0
rosie go &
```

To get started, I tried to copy an existing CMIP6 PI suite, adapt it to my own set-up and run it for 1 year.
- search for **u-ar766** in the GUI (revision 119943, owner tillkuhlbrodt), right-click and select `Copy Suite` (this both copies and checks out the suite, making it editable under home directory). 
- I kept the poject-id as `u-cmip6` and only changed the e-mail address of the suite owner, clicked OK and got the new suit **u-by789** in my home directory.
- following Charlie's idiot guide, I edited the configuration of my test PI suite (**u-by789**) by either opening the GUI (right-click on suite and select `Edit Suite`) or editing the individual files directly (at `/home/d02/<USERNAME>/cylc-run/u-by766`)
- **IMPORTANT: if you change settings directly in the files e.g. edit `rose-suite.conf` with a text editor, need to make sure that Rose GUI is closed.  Otherwise, if it is already open, changes made in files will revert back to previous version.  Likewise, if using GUI, need to make sure file is closed.**

### GUI changes for 1 year test run
1. Project code (GUI: `> suite conf > Project accounting`): 
``` 
Use default account = True
Sub-project name = Other
Other subproject name = nexcs-n02
```

2. **Processes/nodes** (GUI: `> suite conf > Domain decomposition > Atmosphere`): can leave this as is for testing purposes
3. **Building** (GUI: `> suite conf > Build and Run`): all need to be turned on.  Can turn off `Build UM/Drivers/Ocean` when it has been run once, but will need to turn back on when suite is copied and/or if suite is cleaned using rose suite-clean.  Only turn on `Post Processing` if archiving has been included (see below).  If running for first time, turn on `Run Reconfiguration` to reconfigure atmosphere start dump
4. **Run length & cycling** (GUI: `> suite conf > Run initialisation and cycling`): for testing purposes e.g. 1 year run, 3 months cycling, 4 hour wallclock time.
5. **Restart dumps**: need to set atmospheric reconfigured start dump (GUI: `> um > namelist > Reconfiguration and ancillary control > General technical options`, or just search for “ainitial” within um app) and sea ice start dump (GUI: `> nemo_cice > Restart files > CICE restart file`) to appropriate locations, because default options may not exist.  These are currently at 
	```
	/home/d05/cwilliams/gc31/restarts/u-aq853/aq853a.da25940101_00
	/home/d05/cwilliams/gc31/restarts/u-aq853/aq853i.restart.2594-01-01-00000.nc
	```

	Can leave non-reconfigured start dump (GUI: `> um > namelist > Model input and output > Dumping and meaning`, or just search for “astart” within um app) as is, because this is not used (overwritten by ainitial).  Also need to remove both NEMO restart files (GUI: `> nemo_cice > Restart files`), so that they begin from climatology
	
### File changes for 1 year test run
1. **Host**: don’t want any of the suggestions from the GUI, instead set `HOST_XC40='xcs-c'` in `/home/d02/<USERNAME>/roses/<SUITEID>/rose-suite.conf`
2. **Queue**: actually more queues available than what is in GUI; set this in `HPC_QUEUE` in `/home/d02/<USERNAME>/roses/<SUITEID>/rose-suite.conf`.  NEXCS has 3 queues: **normal** (4 hours = 14000 seconds), **long12** (12 hours = 43200 seconds) & **long24** (24 hours = 86400 seconds). I used normal for the initial test run.
3. **Resources for fcm building 1**: these are currently set to Met Office settings, so need to change from ‘slurm’ to ‘background’ in `/home/d02/<USERNAME>/roses/<SUITEID>/site/meto_cray.rc` under `[[EXTRACT_RESOURCE]]` section, so:
	```bash
	{{{
		[[EXTRACT_RESOURCE]]
			 [[[job]]]
				 batch system = background
				 execution time limit = PT5M
			 [[[directives]]]
				 --mem=1G
				 --ntasks=1
	 }}}
	```
7. **Resources for fcm building 2**: if suite is being run since demise of exvmsrose, and hasn’t been taken from a previously committed job where this has been done need to insert 2 lines into `/home/d02/<USERNAME>/roses/<SUITEID>/site/meto_cray.rc`: 
	```bash
		[[[remote]]]
    		host = xcs-c
	```
	at the end of the the `[[HPC_SERIAL]]` and `[[OCEANBUILD_RESOURCE]]` blocks.  If suite is committed, shouldn’t need to repeat this again.  But might, especially if error `[FAIL] mirror.target = : incorrect value in declaration` is received