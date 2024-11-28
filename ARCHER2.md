[[_HadGEM3 MOC]]

# Archer2
[ARCHER2](https://www.archer2.ac.uk) is the new UK national supercomputing service hosted at the University of Edinburgh. It has replaced the first [ARCHER](https://www.archer.ac.uk) version in early 2021. Currently (June 2021), ARCHER2 is running as a small early access system (4 of 23 cabinets are installed, i.e. 1024 of the total 5848 compute nodes) with the full system of 748,544 computing cores opening later in 2021. Theoretically, they [claim](https://www.archer2.ac.uk/about/hardware.html) ARCHER2 could provide 18x the science throughput for HadGEM3 compared to the previous ARCHER machine.

## getting an account:
I am closely following the [quickstart for users](https://docs.archer2.ac.uk/quick-start/overview/) part in the official documentation. 

1. sign up for an [SAFE account](https://safe.epcc.ed.ac.uk/signup.jsp) to manage all of your UK HPC services. Username is sebastian.steinig@bristol.ac.uk; password is stored in 1Password under https://safe.epcc.ed.ac.uk
2. request an ARCHER2 login account:
    - [log into SAFE](https://safe.epcc.ed.ac.uk)
    - go to *Login accounts* and click  *Request New Login Account* button 
    - Need to select project to join. [[SWEET]] is not listed, based on mail from [[Grenville Lister]] (14/01/21) I chose "n02: NCAS". Grenville said SWEET might become a subproject in the future.
    - select the *archer2-4c* machine in the list of available machines (was already selected for me) and click *Next*
    - choose username (**ssteinig**) and SSH public key. I used the key generated on mu UoB iMac:
       ```
	   ssh-rsa ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDbIpkE4kOalJmj2dic4GHrnRMysU7Iqfr4KJuIW7mtBlrLqABI1hkj6rVRnUuA/ayPfBbIst+v7KIqxoKKJ960o5wgsCHIfPwE60stoVWfE6ka9ketLtjLtAthO/ISQ5XxzCRQLoFq7XQ9eShqBtdH7arcH1LW3+o/8EOJkvE4wSdA9kcpE2xMEIC0azV8JUDmWk2gmRCF3IeCGLAXZE04/+VSo5gK83pyv+xYOao5uGnqcubwqM6wJlZKhozYiZRGQDuiYfpKCLq4FzGZX+l3H9nmMbCXJElGNUbFL+DPr6hWreGRJM4iYApiJVZoNjMP9QJFJEpRUlk9ttsAXHg1 wb19586@IT042618
       ```
	- click *Request*
	- I got the following message *"**New Account requested** Your account ssteinig@archer2-4c has been requested, the project's principal investigator or a project manager will be informed of your request and you will be contacted once your machine account has been activated."* 
	- account request has been approved (04/06/21):
	    - Machine:  archer2-4c  
	    - Username: ssteinig  
        - Project:  NCAS (National Centre for Atmospheric Science)
	- I already added iMac SSH key to ARCHER2 section on SAFE website, but you can add more keys for each machine.
	- Collect ARCHER2 password: login to SAFE, go to *Login Accounts* and select ARCHER2 and click button *View Login Account Password*. Use this initial password for first login
	- login with `ssh [userID]@login.archer2.ac.uk` to change password (I saved it in 1Password)
	- Afterwards, if you use default SSH key in `~/.ssh/id_rsa` you can simply login with:
		```
		ssh ssteinig@login.archer2.ac.uk
		ARCHER2 password
		SSH key passphrase
		```
	- SUCCESS!!!
	- note: you can also access ARCHER2 with a SFTP client (at least Transmit5 on a Mac works fine) to quickly upload/download files

