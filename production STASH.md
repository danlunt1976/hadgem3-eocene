- copied x1 spin-up suite (cz307) to create x1 production suite (dd626) with more STASH for final 100-year climatology
- followed [[copy STASH for HadGEM3]] to copy default CMIP6 spin-up STASH
- this has way too much output files (500 GB per year)
- reduced this in the following way:
	- sorted by package and deactived all entries for: CFday-2d, CFmon-sim, CMIP6-N96HGM3, COSP, COSP_LIDAR, DUST, UKCA diagnostics, UKSHELF, UMTRACK
	- this only leaves the packages CMIP6-core and UKCA Coupling Macro (the latter one is necessary for coupling, I think?!)
	- then sort by 'tim_name' and deactivated: T1HRMN, T24H0Z, T3HMN, T3HR, T3HRMN, T6H, T6HDAYM, T6HMONM, T6HR, T6HRMAX
	- also removed 3D fields from 6-hourly and daily data
	- for 6-hourly data ('UP7'), only keep;
		- ps, u10, v10, tsurf, hur, pr, psl
	- removed some daily and monthly data 
	- removed some 3D fields from TDMPMN
- resulting output streams:
	- **pa:** daily means of selected hydrological params
	- **pd:** daily means of more 2D diagnostics
	- **pf:** daily means of more 2D diagnostics
	- **pi:** 90-day means of some boundary conditions, snow mass, and water contents
	- **pj:** daily means of more 2D diagnostics
	- **pm:** monthly means of dumps 
	- **py:** annual means of dumps
	- **p4**: monthly 3D dynamics on model levels (U, V, W, P) 
	- **p5:** monthly surface and 3D on pressure levels
	- **p6:** 2D daily mean/min/max 
	- **p7:** 2D 6-hourly means (ps, u10, v10, tsurf, hur, pr, psl)
- IMPORTANT: the ozone remapping is not part of the CMIP6 spin-up STASH, so we need to add the `*.po` output stream (which contains the tropopause height) necessary for the remapping to work. I manually added the following requests to the `rose-app.conf` in my new suite:
```
[namelist:umstash_streq(00253_075a957a)]
dom_name='DALLRH'
isec=0
item=253
package='Ozone-redist'
tim_name='TMONMN'
use_name='UPO'

[namelist:umstash_streq(30453_109a16ed)]
dom_name='DIAG'
isec=30
item=453
package='Ozone-redist'
tim_name='TMONMN'
use_name='UPO'

[namelist:umstash_use(upo_b0aae7b0)]
file_id='pp145'
locn=3
!!macrotag=0
use_name='UPO'

[namelist:nlstcall_pp(pp145)]
file_id='pp145'
!!filename=''
filename_base='$DATAM/${RUNID}a.po%C'
l_reinit=.true.
packing=0
reinit_end=-1
reinit_start=0
reinit_step=360
reinit_unit=2
```