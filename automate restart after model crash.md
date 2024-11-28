- HadGEM3 generates atmosphere crashes fairly regular
- these are related to instabilities like "North/South halos too small for advection" or negative pressures
- these crashes only happen at higher GMST and happen more often at the very high Eocene temperatures (e.g. every couple of model years at GMST > 30 degC)
- when this happens, I slightly perturb the temperature (theta) in the atmospheric restart file via an automated script at `/home/n02/n02/ssteinig/scripts/perturb_atmosphere.sh`:
```bash
#!/usr/bin/env bash
#
###############################################################################
#
# Convenience wrapper script around the moci 'perturb_theta.py' script to perturb
# theta in the latest UM restart file. More info under 'Restarting if the model blows up'
# at: https://code.metoffice.gov.uk/trac/moci/wiki/tips_CRgeneral
#
###############################################################################
set -e

echo "suiteID: $1";
echo "year: $2";

# load python environment with mule commands (https://ncas-cms.github.io/um-training/post-processing.html#mule)
module load cray-python

# 2. rename latest UM restart file for backup
if [ ! -f ~/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00.orig ]; then
  echo "perturbing: "~/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00
  mv ~/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00 ~/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00.orig
else
  echo "backup of file : "~/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00"already exists."
  echo "exiting now!"
  break
fi

# 3. Perturb the dump and write the result to the original filename
python /home/n02/n02/ssteinig/local_moci_lib/perturb_theta.py /home/n02/n02/ssteinig/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00.orig --output /home/n02/n02/ssteinig/work/cylc-run/u-${1}/share/data/History_Data/${1}a.da${2}0101_00
```
- afterwards, I simply need to retrigger the previously failed `coupled` task in the current cycle
- this can be done via the the `cylc` GUI by right-clicking on `coupled` and selecting `Trigger (run now)`

### let's try to automate this manual process:
- we can use Cylc's built-in mechanisms for triggering tasks based on the state of other tasks, see https://cylc.github.io/cylc-doc/7.8.8/html/running-suites.html?highlight=event%20handlers#task-event-handling
- for example, we can set up a task that is triggered when the `coupled` task fails; so in `suite.rc` under the [[COUPLED]] section we can add:
```
[[[events]]]
	failed handler = perturb_and_retrigger.sh 	
```
- make sure the script is executable `chmod +x perturb_and_retrigger.sh` and move it to `/home/n02/n02/$USER/roses/$SUITE/bin` on PUMA (these scripts will be copied over to the cylc bin directory on ARCHER2 on simulation start or via `rose suite-run --reload`)

``` 
%% NOT WORKING %%
    [[COUPLED]]
        inherit = None, ATMOS, NEMO, CICE, COUPLE_RESOURCE
        script = """
                 . $ROSE_DATA/etc/um_ancils_gl
                 . $ROSE_DATA/etc/nemo_ancils_gl
                 {{TASK_RUN_COMMAND}}
                 """
        [[[events]]]
            handlers = "echo '!!!!!EVENT!!!!!' "
            handler events = submitted, failed
        [[[environment]]]
            ROSE_TASK_APP          = coupled
            CALENDAR = {{CALENDAR}}
            CONTINUE               = $( if [[ $CYLC_TASK_CYCLE_POINT == $CYLC_SUITE_INITIAL_CYCLE_POINT ]]; then echo ""; else echo "true"; fi )
            ROSE_APP_OPT_CONF_KEYS = {{UM_OPT_KEYS}} {{BITCOMP_NRUN_OPT}}
```

```
%% also NOT WORKING %%
    [[COUPLED]]
        inherit = None, ATMOS, NEMO, CICE, COUPLE_RESOURCE
        script = """
                 . $ROSE_DATA/etc/um_ancils_gl
                 . $ROSE_DATA/etc/nemo_ancils_gl
                 {{TASK_RUN_COMMAND}}
                 """
        [[[events]]]
            submitted handler = task_logger.sh
        [[[environment]]]
            ROSE_TASK_APP          = coupled
            CALENDAR = {{CALENDAR}}
            CONTINUE               = $( if [[ $CYLC_TASK_CYCLE_POINT == $CYLC_SUITE_INITIAL_CYCLE_POINT ]]; then echo ""; else echo "true"; fi )
            ROSE_APP_OPT_CONF_KEYS = {{UM_OPT_KEYS}} {{BITCOMP_NRUN_OPT}}


    [[coupled]]
        inherit = None, COUPLED
```