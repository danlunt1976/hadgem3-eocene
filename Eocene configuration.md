[[_HadGEM3 MOC]]

# HadGEM3 Eocene configuration
We build on experience from PI and Pliocene simulations to set up a HadGEM3 run with DeepMIP-Eocene boundary conditions. I start with a copy of the PI CMIP6 standard suite (u-cm430) that is working on the full 23-cabinet ARCHER2 system and includes the reduced CMIP6 spin-up STASH and file transfer to JASMIN. Therefore, my starting point for the Eocene setup is **u-cm487**.

Adjusting this suite to a DeepMIP Eocene simulation involves four steps. These follow closely the extensive previous work by Charlie Williams and are based on his [Idiot's guide to setting up GC3.1 Eocene suite on NEXCS v3.docx](https://uob-my.sharepoint.com/:w:/g/personal/wb19586_bristol_ac_uk/EcIwjv71mMFDgCKJqzcjB_oBGmRss2O40IxQPvVG6BF4xw?e=vgAmjN). For reference, below are the most important steps:

#### 1. Ocean ancillary files
- **bathymetry:**
	- regrid Herold et al. (2014) bathymetry to ORCA1 grid and NEMO file format (instructions on JASMIN at `/gws/nopw/j04/ncas_climate_vol2/users/cwilliams2011/data1_for_use.d/sweet.d/gc31.d/build.d/ancils.d/bathy.d/readme.txt`)
- **coupling weights:**
	- use NEMO suite **u-ba251@87020** to create mesh_mask.nc from new bathymetry during build stage (details in Charlie's guide; **suite was used on NEXCS, so would need to be updated for use on ARCHER2**)
	- this mesh_mask.nc file can be used to generate new ocean-atmosphere coupling weights updated to the new land-sea mask
	- I will be using weights previously generated by Alistair Sellar from the Met Office, but I also generated very similar (but not identical!) weights on my own (see [[OASIS coupling weights]])
	- new weights are on ARCHER2 at `/home1/home/n02/n02/ssteinig/ancils/DeepMIP-Eocene/gc31/coupling_weights_round2`
- **initial temperature & salinity:**
	- change filled modern, monthly mean climatology following Lunt et al. (2017), but with 15°C reduced to 5°C (10°C also available)
	- constant salinity of 34.7
- **initial sea ice**:
	- zeroed PI sea ice restart file
- **viscosity:**
	- reduce viscosity between 20° N/S to increase equatorial current speed and vertical shear
	- needs to be adapted to Eocene mask

#### -> suite changes
New ocean ancils are read-in by ancillary versions files that can be set in `> install_ancil > file`  and defines directories and file names of all ancils.  I started with default CMIP6 file `$UMDIR/nemo/ocean_ancil_versions/GC3.1_eORCA1v2.2x_nemo_ancils_CMIP6` and included newly generated ancils. Final versions file shown below:

```bash
#########################################################################
# Ancil Version File for eORCA1_v2.2x (GO6 grid) ocean input files
#########################################################################


export HG3DIR=/work/y07/shared/umshared/hadgem3
export CICE_GRID=${CICE_GRID:-${HG3DIR}/grids/seaice/eORCA1v2.2x/CICE_grid_eorca1_v2.2x_razed.nc}
#export CICE_INIT_DIR=${CICE_INIT_DIR:-${HG3DIR}/initial/seaice/eORCA1v2.2x/}
#export CICE_INIT=${CICE_INIT:-$CICE_INIT_DIR/u-aj572i.restart.1977-01-01-00000.nc}
export CICE_INIT=/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/final_ancils_charlie/cice/aq853i.restart.zeroed.nc
export CICE_KMT=${CICE_KMT:-${HG3DIR}/grids/seaice/eORCA1v2.2x/CICE_kmt_eorca1_v2.2x_razed.nc}

#Directories for NEMO inputs
#export NEMO_ANCIL_DIR=${NEMO_ANCIL_DIR:-${HG3DIR}/ancil/ocean/eORCA1v2.2x/}
export NEMO_GRIDS_DIR=${NEMO_GRIDS_DIR:-${HG3DIR}/grids/ocean/eORCA1v2.2x/}
#export NEMO_INIT_DIR=${NEMO_INIT_DIR:-${HG3DIR}/initial/ocean/eORCA1v2.2x/}
#3export NEMO_FORCE_DIR=${NEMO_FORCE_DIR:-${HG3DIR}/forcing/ocean/eORCA1v2.2x/}
#export NEMO_SURFACE_FORCE_DIR=${NEMO_SURFACE_FORCE_DIR:-${HG3DIR}/forcing/ocean/CORE2/}  

#Climatological T & S files for initial conditions (or damping) to use in nemo namelist 
#export NEMO_CLIM_START_TEMP=${NEMO_CLIM_START_TEMP:-$NEMO_INIT_DIR/TS_1950-1954_1-12_EN.4.1.1.g10_eORCA1_v2.2x_L75_degC.nc}
#export NEMO_CLIM_START_SAL=${NEMO_CLIM_START_SAL:-$NEMO_INIT_DIR/TS_1950-1954_1-12_EN.4.1.1.g10_eORCA1_v2.2x_L75_degC.nc}
export NEMO_CLIM_START_TEMP=/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/final_ancils_charlie/ocean/ts_init/EN4_v1.1.1995_2014.monthlymean_eORCA1T_NEMO_L75_gmdmod_5deg.nc
export NEMO_CLIM_START_SAL=/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/final_ancils_charlie/ocean/ts_init/EN4_v1.1.1995_2014.monthlymean_eORCA1T_NEMO_L75_gmdmod_5deg.nc

#Files required in the working directory
#export K1ROWDRG=${K1ROWDRG:-$NEMO_ANCIL_DIR/eORCA_R1_K1_v1.0.nc} 
#export M2ROWDRG=${M2ROWDRG:-$NEMO_ANCIL_DIR/eORCA_R1_M2_v1.0.nc} 
#export AHMCOEF=${AHMCOEF:-$NEMO_ANCIL_DIR/ahmcoef.nc}  
export AHMCOEF=/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/final_ancils_charlie/ocean/ahmcoef.nc
#export BATHY_METER=${BATHY_METER:-$NEMO_ANCIL_DIR/eORCA_R1_bathy_meter_v2.2x.nc} 
export BATHY_METER=/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/final_ancils_charlie/ocean/bathy/eORCA_R1_bathy_meter_v2.0_eocene_chiselled.nc
#export BFR_COEF=${BFR_COEF:-$NEMO_ANCIL_DIR/bfr_coef01.nc}
#export CALVING=${CALVING:-$NEMO_FORCE_DIR/ecalving_v2.2x.nc}
export COORDINATES=${COORDINATES:-$NEMO_GRIDS_DIR/eORCA1_coordinates_nc4.nc_from_MR}
#export MASK_ITF=${MASK_ITF:-$NEMO_ANCIL_DIR/eORCA_R1_maskITF_v1.0.nc}
#export RUNOFF=${RUNOFF:-$NEMO_FORCE_DIR/eORCA1_runoff_v2.2x.nc}
#export SUBBASINS=${SUBBASINS:-$HG3DIR/grids/ocean/eORCA1/subbasins_CMIP6.nc} 
#export GEOTHERM=${GEOTHERM:-$NEMO_FORCE_DIR/goutorbe_1x1_0360_eORCA1_v2.2x.nc} 
```

In addition, the following suite changes were necessary:
- set NEMO ancillary versions file in  `> install_ancil > file` to newly created `/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/ancil_versions/GC3.1_eORCA1v2.2x_nemo_ancils_CMIP6.DeepMIP_Eocene` 
 
#### 2. Atmosphere ancillary files
two-step process:
1. create all required ancils with special ancillary suite (created ancils are based on Eocene mask, but filled with modern data)
	- ancil suite uses new Eocene land-sea mask, orography and river routing files (these can be created following CHarlie's guide)
	- **BUT:** ancil suite (u-ar826@65219) not adapted for ARCHER2 yet, so I will use Charlie's ancils in the following. I copied them from JASMIN (`/gws/nopw/j04/pmip4_vol2/users/cwilliams2011/gc31_code.d/ancils.d/final_ancils`) to ARCHER2 (`/home/n02/n02/ssteinig/work/ancils/DeepMIP-Eocene/gc31/final_ancils_charlie`)
2. go through most of them and 'paleotise' them (i.e. fill Eocene land areas with modern zonal or global mean values
	- detailed description of the necessary steps for each field, references to associated scripts and plots are in [Idiot's guide to setting up GC3.1 Eocene suite on NEXCS v3.docx](https://uob-my.sharepoint.com/:w:/g/personal/wb19586_bristol_ac_uk/EcIwjv71mMFDgCKJqzcjB_oBGmRss2O40IxQPvVG6BF4xw?e=vgAmjN)
	- list of ancils Charlie changed for the DeepMIP Eocene config:
		- **aerosol emissions**:
			- natural SO2, DMS & monoterpene from modern monthly zonal mean clim
			- biomass burning from PI monthly zonal mean clim
			- anthropogenic emissions zeroed
		- **ozone**:
			- from UKESM1 4xCO2 run, zonally palaeotised at every month/level (85 levels)
		- **offline oxidants**:
			- precomputed monthly climatology of oxidant fields (O3, OH, H2O2, HO2, NO3) in HadGEM3 to achieve comparable aerosol loading and consistency in radiative forcing to more complex UKESM (see [MetOffice summary](http://www.jwcrp.org.uk/projects/ukesm/news/aug15/simpae.asp))
			- zonally paleotised at every month/level from PI climatology
		- **vegetation**
			- 

#### 3. Paleotising of atmospheric ancils

#### 4. Linking to new ancils