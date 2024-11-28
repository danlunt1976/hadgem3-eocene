 [[_Dashboard]]

# HadGEM3 MOC
Documentation of my work using the [Met Office HadGEM3](https://www.metoffice.gov.uk/research/approach/modelling-systems/unified-model/climate-models/hadgem3) (Hadley Centre Global Environment Model version 3) climate model. I started to configure and run the model on [[NEXCS-MONSooN]] and now switched to [[ARCHER2]]. The general set-up to get [[HadGEM3]] running on either machine is described in the "getting started" section. Notes and documentation for specific experiments are listed below that, grouped by time period.

## external documentation
I found quite a lot of scattered information with bits and pieces on different sites/repositories. I extracted the information relevant for my work and included it in these notes, but here some links that seem to be up-to-date (in 08/2021) and that I want to remember:
- general ARCHER2 documentation is drafted on [GitHub](https://github.com/ARCHER2-HPC/archer2-docs/) and rendered in HTML [here](https://docs.archer2.ac.uk)
- specifics about HadGEM3 on ARCHER2 can be found in various places around the NCAS CMS [wiki](http://cms.ncas.ac.uk/wiki/Archer2) 
- a good and recent (UM training course 2021) introduction on running HadGEM3 on ARCHER2 is currently develeoped as a [NCAS Unified Model Introduction Course](https://ncas-cms.github.io/um-training/index.html)

## getting started
1. [[getting accounts]]
2. initial set-up (just need one of them)
	
	- **NEXCS**
		1. [[setting-up NEXCS]]
		2. [[hello world suite on NEXCS]]
	
	- **ARCHER2**
		1. [[set up Rose-Cylc on ARCHER2]]
		2. [[hello world suite on ARCHER2]]
		3. [[file transfer ACRHER2 to JASMIN]]
		4. [[create spin-up suite]]
3. Running simulations
	1. [[NEXCS-MONSooN]] 
	2. [[ARCHER2]]
	
## pre-industrial simulations
1. [[Pre-Industrial Control]]

## Eocene simulations
1. [[Eocene configuration]]
2. Ozone remapping

## [[table of experiments]]

## [[performace on ARCHER2]]