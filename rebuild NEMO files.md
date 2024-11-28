
# rebuild NEMO files

In NEMO each processor inputs/outputs its own section of the ocean grid to separate files. Therefore, we often need to combine NEMO output files from multiple processors into one file with a global grid (e.g. for analysis). This happens automatically during the [[post-processing]] stage, but can and must sometimes also be done manually (e.g. to generate specific restart files). This can be easily done on [[NEXCS-MONSooN]] with the following shell script: `~hadtq/bin/local_rebuild/rebuild_nemo_login`. 

### ARCHER2 update
same usage on ARCHER2, but different script name, with command:

``` bash
rebuild_nemo <SUITEID>o_<YYYYMMDD>_restart NUMPROCS
# e.g:
rebuild_nemo dd709o_24940101_restart 128
```

### Usage: 

 ```bash
 ~hadtq/bin/local_rebuild/rebuild_nemo_login <SUITEID>o_<YYYYMMDD>_restart NUMPROCS
 
 # Example:
  ~hadtq/bin/local_rebuild/rebuild_nemo_login ca003o_18530801_restart 72
 
 ```


### script: rebuild_nemo_login
```bash
#!/bin/ksh
#
#  Script to run the NEMO rebuild tool to rebuild NEMO
#  diagnostic or restart files.
#
#  The scripts creates the nam_rebuild namelist based upon
#  the command line options you give it (see usage below)
#
#  Ed Blockley, September 2011
#
#

#set -ax
usage ()
{
   echo
   echo "  NEMO Rebuild"
   echo "  ************"
   echo
   echo "  usage: ${0##*/} [-t -c] filebase ndomain [rebuild dimensions]"
   echo
   echo "  flags:  -t num      use num threads"
   echo "          -c num      split 4D vars into time chuncks of size num"
   echo
   exit 1
}

while getopts c:t: opt
do
  case ${opt} in
      t)
         OMP_NUM_THREADS=${OPTARG}
      ;;
      c)
         NCHUNKSIZE=${OPTARG}
      ;;
  esac
done
shift $(expr ${OPTIND} - 1)

if [[ $# < 2 ]] ; then
   usage
fi

script_dir=$(dirname $0)

file=$1
ndomain=$2
DIM1=$3
DIM2=$4
export OMP_NUM_THREADS=${OMP_NUM_THREADS:-16}

if [[ -n ${DIM1} && -n ${DIM2} ]] ; then
   dim_str=" dims '${DIM1}','${DIM2}'"
   dims="dims='${DIM1}','${DIM2}'"
   echo ${dim_str} >> nam_rebuild
fi

echo "file ${file},  num_domains ${ndomain}, num_threads ${OMP_NUM_THREADS}${dim_str}"

cat > nam_rebuild << EOC
&nam_rebuild
filebase='${file}'
ndomain=${ndomain}
EOC
if [[ -n ${dims} ]] ; then
   echo ${dims} >> nam_rebuild
fi
if [[ -n ${NCHUNKSIZE} ]] ; then
   echo " nchunksize=${NCHUNKSIZE}" >> nam_rebuild
fi
echo "/" >> nam_rebuild

# run the rebuild code
${script_dir}/rebuild_nemo.exe
```