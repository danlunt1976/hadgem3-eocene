[[_HadGEM3 MOC]]

# hello world suite on ARCHER2

---
## UPDATE 03/2022:
My original notes were written during the initial ARCHER2 launch phase in which only a reduced, 4-cabinet system was available for use. Since then, this has been replaced by the full 23-cabinet system and some instructions (i.e. login node name) have changed. The new updated suite for a working (in 03/2022) PI standard suite including file transfer to JASMIN is now **u-cm430**

---


With the set-up steps ([[getting accounts]], [[set up Rose-Cylc on ARCHER2]]) completed, you should now be able to login to [[PUMA]] and open the Rose GUI with:
```bash
ssh puma
rosie go &
```

To get started, I tried to copy the existing CMIP6 PI suite, adapt it to my own set-up and run it for 1 year.

- see [this list of standard jobs](http://cms.ncas.ac.uk/wiki/Archer2) ported to ARCHER2 for a starting point
- GC3.1 coupled PI Control for CMIP6 is archer2 branch of u-as037
- search for **u-as037** in the GUI (branch: archer2, revision: 191921, owner rosalynhatcher), right-click and select `Copy Suite` (this both copies and checks out the suite, making it editable under home directory). 
- I only changed the title to `PI Control Std Suite for CMIP6 - HadGEM3-GC3.1 N96ORCA1 - hello world 1yr test`, clicked OK and got the new suit **u-ch033** in my home directory.
- following Charlie's idiot guide, I edited the configuration of my test PI suite (**u-ch033**) by either opening the GUI (right-click on suite and select `Edit Suite`) or editing the individual files directly (at `/home/<USERNAME>/roses/u-ch033`)
> **IMPORTANT: if you change settings directly in the files e.g. edit `rose-suite.conf` with a text editor, need to make sure that Rose GUI is closed.  Otherwise, if it is already open, changes made in files will revert back to previous version.  Likewise, if using GUI, need to make sure file is closed.**

## short test run (FAILING!)
- it seems that most of the steps in [[hello world suite on NEXCS]] are not necessary anymore as the suite is already set-up to run on ARCHER2
- to quickly test whether I can run the model at all, I only did 6 changes to the suite:
	
	1. set ARCHER2 account to `n02` in `> suite conf > Project Accounting > Account group for HPC tasks` 
		
		**!this budget code seems to be wrong - need to ask Grenville and update this!**
	
	2.  set ARCHER2 username (if different to PUMA user name) `> suite conf > Machine Options > USERNAME`
	
	3. reduce the run length from 500 to 1 years in the gui at `> suite conf > Run initialisation and cycling > Total Run length`
	
	4. turn off archiving to remote machine, i.e. setting `> suite conf > Build and Run > PP Transfer` to false 
	
	5.  set **archive_root_path** in `> postproc > Post Processing - common settings > Archer Archiving` to `/work/n02/n02/ssteinig/archive`. This will be a temporary area for processing the data on ARCHER2.
	
	6. removed `validate_suite_info` form the suite graph. Apparently, this process only checks the suite metadata against CMIP controlled vocabulary. This is not necessary and will routinely fail in our suites. Therefore, I deleted the whole line 75 from the `suite.rc` file (removed `{{ 'validate_suite_info => install_ancil'  + ( ' & fcm_make2_pp' if POSTPROC else '') + (' & '  + FCMUM_LAST if BUILD_UM else '') + (' & fcm_make2_ocean' if BUILD_OCEAN else '') + (' & recon' if RECON else '') }}`)

- *NOTE: I did another copy of suite **u-as037** and saved it with a title starting with TEMPLATE (suite-id **u-ch089**). This serves as the official reference suite for any changes I do. For example, I can always go to `/home/wb19586/roses` on PUMA and type `diff u-ch089 u-ch033` to list any changes I made. Currently this should look like this:*
 
```terminal
puma:/home/wb19586/roses $ diff u-ch089 u-ch033
Common subdirectories: u-ch089/.svn and u-ch033/.svn
Common subdirectories: u-ch089/app and u-ch033/app
Common subdirectories: u-ch089/bin and u-ch033/bin
Common subdirectories: u-ch089/meta and u-ch033/meta
diff u-ch089/rose-suite.conf u-ch033/rose-suite.conf
31c31
< ARCHER2_GROUP='''n02-cms'''
---
> ARCHER2_GROUP='''n02'''
83c83
< PPTRANSFER=true
---
> PPTRANSFER=false
95c95
< RUNLEN=500,0,0,0,0,0
---
> RUNLEN=1,0,0,0,0,0
103c103
< USERNAME='$LOGNAME'
---
> USERNAME='ssteinig'
diff u-ch089/rose-suite.info u-ch033/rose-suite.info
29c29
< project=Templates
---
> project=u-cmip6
34c34
< title=TEMPLATE - [Std Suite - ARCHER2] HadGEM3-GC3.1 N96ORCA1 PI Control for CMIP6
---
> title=PI Control Std Suite for CMIP6 - HadGEM3-GC3.1 N96ORCA1 - hello world 1yr test
Common subdirectories: u-ch089/site and u-ch033/site
diff u-ch089/suite.rc u-ch033/suite.rc
75d74
< {{ 'validate_suite_info => install_ancil'  + ( ' & fcm_make2_pp' if POSTPROC else '') + (' & '  + FCMUM_LAST if BUILD_UM else '') + (' & fcm_make2_ocean' if BUILD_OCEAN else '') + (' & recon' if RECON else '') }}
```  

## job submission failed
I am currently not able to submit jobs from PUMA to ARCHER2. This fails allready for installing the ancils and compiling the model. It looks like a problem with the ARCHER2 budget code ( `> suite conf > Project Accounting > Account group for HPC tasks`). In my SAFE account it looks the code is `n02`. If I use this I get the following error in the respective `job-activity.log` files:

```terminal
[job-submit cmd] cylc jobs-submit --host=login.archer2.ac.uk --user=ssteinig --remote-mode -- '$HOME/cylc-run/u-ch033/log/job' 18500101T0000Z/clearout/01 18500101T0000Z/install_ancil/01
[job-submit ret_code] 1
[job-submit out] 2021-08-12T09:24:38+01|18500101T0000Z/install_ancil/01|1|None
(ssteinig@login.archer2.ac.uk) 2021-08-12T09:24:38+01 [STDERR] sbatch: error: AssocMaxCpuMinutesPerJobLimit
(ssteinig@login.archer2.ac.uk) 2021-08-12T09:24:38+01 [STDERR] sbatch: error: Batch job submission failed: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)
```

I also tried other options (e.g. `n02-ncas`, `n02-cms`). In these cases I get a slightly different error message:

```terminal
[job-submit cmd] cylc jobs-submit --host=login.archer2.ac.uk --user=ssteinig --remote-mode -- '$HOME/cylc-run/u-ch033/log/job' 18500101T0000Z/clearout/01 18500101T0000Z/install_ancil/01
[job-submit ret_code] 1
[job-submit out] 2021-08-12T09:17:20+01|18500101T0000Z/install_ancil/01|1|None
(ssteinig@login.archer2.ac.uk) 2021-08-12T09:17:20+01 [STDERR] sbatch: error: Batch job submission failed: Invalid account or account/partition combination specified
```

- need to ask Grenville about the correct budget code
> Grenville: Please use n02-SWEET.

## fixing the test run
lets try again with the correct budget code:
- set ARCHER2 account to `n02-SWEET` in `> suite conf > Project Accounting > Account group for HPC tasks` 
- the suite now gets submitted correctly, but the UM build fails.The error message from `fcm_make_um` is:
	```terminal
	[FAIL] config-file=/home/wb19586/cylc-run/u-ch033/work/18500101T0000Z/fcm_make_um/fcm-make.cfg:6 
	[FAIL] config-file= - /home/ros/um/branches/vn10.7_archer2_compile/fcm-make/ncas-ex-cce/um-atmos-safe.cfg 
	[FAIL] /home/ros/um/branches/vn10.7_archer2_compile/fcm-make/ncas-ex-cce/um-atmos-safe.cfg: cannot load config file 
	[FAIL] /home/ros/um/branches/vn10.7_archer2_compile/fcm-make/ncas-ex-cce/um-atmos-safe.cfg: cannot be read 
	[FAIL] No such file or directory
	```
- so the UM 10.7 configuration file has moved
- I [asked the helpdesk](https://cms-helpdesk.ncas.ac.uk/t/um10-7-archer2-configuration-file/190) for the updated path
- need to set `> fcm_make_um > env > Configuration file > config_root_path` to `fcm:um.xm_br/dev/simonwilson/vn10.7_archer2_compile`
- **NOTE: Rosalyn Hatcher said she will update this in the standard suite, so this step might not be necessary anymore in the future**
- the UM should now compile fine and the actual model run `coupled` should be submitted to the ARCHER2 queue
- in my case it was queueing for ~12 hours and then failed after a couple of minutes running with the error:
	```terminal
	????????????????????????????????????????????????????????????????????????????????
	???!!!???!!!???!!!???!!!???!!!       ERROR        ???!!!???!!!???!!!???!!!???!!!
	?  Error code: 99
	?  Error from routine: IOS_INIT:IOS_RUN
	?  Error message: IO Server is not active
	?  Error from processor: 1
	?  Error number: 0
	????????????????????????????????????????????????????????????????????????????????
	```
- found [this ticket](http://cms.ncas.ac.uk/ticket/3406) with the same error for UM10.7, but on NEXCS
> The UM IO Server needs 2 openmp threads to run, so you either need to go back to using 2 openmp threads or turn off the IO server. To turn off the IO server set IO server Processes to 0, in the same rose edit panel you set openmp threads.
- we are already using 2 OpenMP threads for the atmosphere so the IO server should work ...
- but I saw that our NEXCS Eocene suite [[cf290]] did not use any IO Server Processes, so I tried second option from the helpdesk and changed `> suite conf > Domain Decomposition > Atmosphere > IO Server Processes` from 6 to 0
- this worked! Not sure whether this has some real-world performance impact? SHould we try to fix the suite with IO servers?

## TL;DR 
- all changes necessary to get from **u-as037** standard suite to 10 year PI test run with file transfer to JASMIN:
 
 1. set ARCHER2 account to `n02-SWEET` in `> suite conf > Project Accounting > Account group for HPC tasks` 
	
2.  set ARCHER2 username (if different to PUMA user name) `> suite conf > Machine Options > USERNAME`
	
3. reduce the run length from 500 to 10 years in the gui at `> suite conf > Run initialisation and cycling > Total Run length`
	
4. remove `validate_suite_info` form the suite graph. Apparently, this process only checks the suite metadata against CMIP controlled vocabulary. This is not necessary and will routinely fail in our suites. Therefore, I deleted the whole line 75 from the `suite.rc` file (removed `{{ 'validate_suite_info => install_ancil'  + ( ' & fcm_make2_pp' if POSTPROC else '') + (' & '  + FCMUM_LAST if BUILD_UM else '') + (' & fcm_make2_ocean' if BUILD_OCEAN else '') + (' & recon' if RECON else '') }}`)


## next steps
- currently, the model output will be stored at `/home/n02/n02/ssteinig/work/cylc-run/u-ch033` and the post-processing creates directories for each cycle at `/home/n02/n02/ssteinig/work/archive/u-ch033` 
- those archive files should be temporary and instead be transferred to [[JASMIN]] for long-term storage and access
- I made a copy of the running test suite **u-ch033** and will use the new suite **u-ch912** to implement the  [[file transfer ACRHER2 to JASMIN]]