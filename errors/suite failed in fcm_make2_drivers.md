# HadGEM3 error: suite failed in fcm_make2_drivers

### problem:

- copied suite [[ca003]] from [[Charlie Williams]] to local experiment [[cf028]]
- see [[cf028]] for applied changes
- started suite; model didn't run, error in **fcm_make2_drivers**:

```bash
# suite failed in fcm_make2_drivers
[FAIL] no configuration specified or found

[FAIL] fcm make -C /home/d02/ssteinig/cylc-run/u-cf028/share/fcm_make_drivers -n 2 -j 1 # return-code=2
2021-05-27T19:58:47Z CRITICAL - failed/EXIT
```
- what is fcm_make2_drivers?

### solution:
- found tickets in NCAS helpdesk:
    - [#3016](http://cms.ncas.ac.uk/ticket/3016): This is a common issue with 2 step builds on xcs see [#2991](http://cms.ncas.ac.uk/ticket/2991) Grenville
    - [#2991](http://cms.ncas.ac.uk/ticket/2991): In site/MONSooN.rc in the HPC section please change: `host = localhost` to `host = xcs-c` FCM 2-step does not like mirroring to the same host that it is extracting to. It is possible that at some point you may find the model will fail with a weird `_mkstemp` error which you will find in the log/suite/log (or err) file. You will then need to switch this back to `host = localhost`. Ideally the fcm_make for postproc should be a one step for Monsoon to stop this problem occurring but that is not a straightforward change. Cheers, Ros
- we are not using `site/MONSooN.rc` but `site/meto_cray.rc`
- following the ticket I changed `host = localhost` to `host = xcs-c` in the *HPC* section (line 33) -> fcm_make2_drivers still failing
- **found another `host = localhost` in the section *HPC_SERIAL* and changed it also to `host = xcs-c` (line 54) -> SUCCESS!!!** fcm_make2_drivers compiled this time