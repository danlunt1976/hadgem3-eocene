- errors in the atmosphere that cause model crashes are logged in the `job.err` file of the 'coupled' task (search for ERROR)
- ocean crashes are not logged in 'job.err'
- instead, need to open `work/cylc-run/$suiteID/work/$cycle/coupled/ocean.output`
- look for ERROR at the bottom of the file
- so far, I only got the following NEMO error:

```
 ===>>> : E R R O R
		 ===========

  stpctl: the zonal velocity is larger than 20 m/s
  =====
 kt=319514 max abs(U):  5.3387E+07, i j k:   236  331   17

           output of last fields in numwso
```
- i,j,k represent the lat/lon/depth indices where the error occured
- in addition, a dump of the last available ocean fields is written to `work/cylc-run/$suiteID/work/$cycle/coupled/ocean.output` which can be very helpful for debugging
- I started to collect the intsantaneous dumps on JASMIN (`/home/users/ssteinig/pmip4_gws/ARCHER2_archive/debugging`) to hopefully identify some patterns that might help find the causes of the crashes
- first, dump is output per CPU (i.e. 128 files on ARCHER2: output.abort_0000.nc, ..., output.abort_0127.nc)
- this can be rebuilt into a single, global file with the `rebuild_nemo` command (`/work/y07/shared/umshared/bin/rebuild_nemo` on ARCHER2)
- usage (in the directory containing the `output.abort_*.nc` files): 
	``` bash
	rebuild_nemo output.abort 128
	```
- I then transfer the resulting `output.abort.nc` and `ocean.output` files to JASMIN where I create a subfolder for each crash named with the metadata of the run: `$suiteID_lastAvailableDay_ijk_i-index_j-index-k-index`, 
	- e.g.: `u-cn504_21230925_ijk_235_331_17`
	- transfer via: `scp output.abort.nc hpxfer1.jasmin.ac.uk:/home/users/ssteinig/pmip4_gws/ARCHER2_archive/debugging/U_gt_20ms/u-cn504_21230925_ijk_235_331_17`