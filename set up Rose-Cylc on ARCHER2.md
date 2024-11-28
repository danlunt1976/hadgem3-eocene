[[_HadGEM3 MOC]]

# set up Rose-Cylc on ARCHER2
- in contrast to [[NEXCS-MONSooN]], we will not run the UM (Rose/Cylc) on a frontend of the supercomputer, but rather on the NCAS [[PUMA]] service
- this means we configure our suite on [[PUMA]] and submit it from there to the [[ARCHER2]] queue
- so most of the initial configuration needs to be done on PUMA

## setting-up PUMA
- if you don't have an account already, email the [CMS Team](http://cms.ncas.ac.uk/wiki/PumaService)
- you will need both a password and SSH key pair for login
- I did send my public key to Andy Heaps by mail some time ago
- ssh login via `puma.nerc.ac.uk`
- I added PUMA to my `~/.ssh/config` file for convenience as follows:
	```
	Host puma puma.nerc puma.nerc.ac.uk
		Hostname = puma.nerc.ac.uk
		User = wb19586
		IdentityFile = ~/.ssh/id_rsa_puma
		ForwardX11 = yes
		ForwardX11Trusted = yes
	```
- I can then login with `ssh puma` followed by SSH passpharse, PUMA and [[MOSRS]] password

#### SSH setup from PUMA to ARCHER2
- to allow the submission of UM suites from PUMA to ARCHER2 I followed the [NCAS wiki instructions](http://cms.ncas.ac.uk/wiki/Archer2/SshAgentSetup)

1. PUMA environment
	- I already had a `.profile` environment from previous HadCM3 work, so just added the lines from the standard file `~um/um-training/setup/.profile`. The resulting file looks like:
	
		```bash
		#
		met_profile=~ggpjv/.defaults/met.profile
		if [ -r $met_profile ] ; then
			. $met_profile
		fi
		#
		# --- Set ENV = the file holding KSH specific commands
		[[ -f ~/.kshrc ]] && . ~/.kshrc
		#
		if [ -f $HOME/setvars ]
		then
		   . $HOME/setvars
		fi
		#
		export PATH="$PATH:/home/swsvalde/bin:/home/swsvalde/bin/ummodel"
		#
		ulimit -s unlimited

		# ssh-agent setup
		. $HOME/.ssh/ssh-setup

		# required for Rose UM suites
		export UMDIR=/home/um

		export SCRATCH=/export/puma/data-01/training/$USER

		[[ $- != *i* ]] && return # Stop here if not running interactively
		. mosrs-setup-gpg-agent
		```

2. ssh keys
	- generated `id_rsa_archerum` key with: `ssh-keygen -t rsa -b 4096 -C "ARCHER2 UM Workflow" -f ~/.ssh/id_rsa_archerum`
	- this should be automatically installed on ARCHER2 (but could take up to 48 hours) **SPOILER: it actually took 48 hours ...**

3. ssh config
	- I added following entry to my `~/.ssh/config`:

		```
		Host archer2 login.archer2.ac.uk
			Hostname = login.archer2.ac.uk
			User = ssteinig
			IdentityFile = ~/.ssh/id_rsa_archerum
			ForwardX11 = no
			ForwardX11Trusted = no
		```

4. set up ssh-agent
	- allows caching of `archerum` passphrase for some time even if you log out of PUMA
	- `cp ~um/um-training/setup/ssh-setup ~/.ssh` (only needed once)
	- `` eval `ssh-agent -s` ``
	- `ssh-add ~/.ssh/id_rsa_archerum`

5. verify setup
	- try logging in to ARCHER2 without password: `ssh archer2`
	- if `id_rsa_archerum` key is **not yet** installed, output looks like: 
	
		```
		Permission denied (publickey).
		```

	- if `id_rsa_archerum` key is **correctly** installed, output looks like: 
	
		```
		PTY allocation request failed on channel 0
		Comand rejected by policy. Not in authorised list
		Connection to login.archer2.ac.uk closed.
		```
	
	> This error is fine and the expected response! It is not possible to start an interactive login session on ARCHER2 from PUMA. For an interactive session you need to login from your local desktop or via your host institution.

6. specify ARCHER2 host
	- all suites must be submitted to `login.archer2.ac.uk` (`host = login.archer2.ac.uk` in the appropriate `.rc` file)
	- this was already set in my `suite.rc` file for my [[hello world suite on ARCHER2]]

## setting-up ARCHER2
- created `~/.bash_profile` on ARCHER2 an copied the content of `/work/y07/shared/umshared/bin/rose-um-env` that looks like:

	```bash
	#-----------------------------------------------------------------------------
	#
	# Owner: NCAS-CMS
	# Date: 14 March 2017
	# Updated: 21 October 2020 for ARCHER2
	#
	# Description: Set up user's login environment for running the UM under Rose
	#
	#----------------------------------------------------------------------------

	. /etc/bash.bashrc > /dev/null 2>&1

	UMDIR=/work/y07/shared/umshared

	PATH=$PATH:$UMDIR/software/bin:$UMDIR/bin

	PYTHONPATH=$UMDIR/lib/python3.8:$PYTHONPATH

	WORKDIR=`echo $HOME | sed 's/home/work/'`
	DEVTDIR=$WORKDIR
	DATADIR=$WORKDIR

	export UMDIR PATH PYTHONPATH
	export WORKDIR DEVTDIR DATADIR
	```

- changed the permissions of `/home` and `/work` directories to enable the NCAS-CMS team to help with any queries:

	```bash
	chmod -R g+rX /home/n02/n02/ssteinig
	chmod -R g+rX /work/n02/n02/ssteinig
	```
