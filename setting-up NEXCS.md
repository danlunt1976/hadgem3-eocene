[[_HadGEM3 MOC]]

# setting-up [[NEXCS-MONSooN]]
### First login to Monsoon
I received an email from Roger Milton with the following instructions to set a personal PIN for the login:
- `ssh <USERNAME>@lander.monsoon-metoffice.co.uk`
- when prompted to enter your PASSCODE, please enter the six-digit number displayed on the front of your token.
- you will now be asked whether you wish to set a new PIN. Please enter y and press return.
- you will now be prompted to enter, and re-enter a PIN of between four and eight digits.
- you have now set your PIN, and will be asked to enter your PASSCODE.
- this final PASSCODE is your new PIN, followed by the six-digit token-code.
- if the token is still displaying the same code that you entered initially, please wait until it changes (after 1 minute).
- if you enter the incorrect details for the final passcode you do not invalidate the PIN you entered. The new PIN is set once the PIN has been entered and confirmed.
- You should now be logged in to exvmslander, and from there can access the HPC login nodes xcslc0 and xcslc1.

### Configuring Subversion access
You need to [configure your subversion access](https://code.metoffice.gov.uk/trac/home/wiki/FAQ) in the file `~/.subversion/servers`. If `~/.subversion/servers` does not exist, run `svn help` once on the command line and add this to the end of file:
```bash
[groups]
metofficesharedrepos = code*.metoffice.gov.uk

[metofficesharedrepos]
# Specify your Science Repository Service user name here
username = myusername
store-plaintext-passwords = no
```

### Password caching on NEXCS
Following the [online documentation](https://code.metoffice.gov.uk/trac/home/wiki/AuthenticationCaching#Monsoon), add the following to the top of your `.bash_profile` file:

```bash
# Provide access to FCM, Rose and Cylc
PATH=$PATH:~fcm/bin

# Ensure .bashrc is sourced in login shells
# (only add this if it is not already done in your .bash_profile)
[[ -f ~/.bashrc ]] && . ~/.bashrc
```

and add the following at the end of your `.bashrc` file:

```bash
[[ $- ! = *i* ]] && return # Stop here if not running interactively
[[ $HOSTNAME = "exvmsrose.monsoon-metoffice.co.uk" || $HOSTNAME = xcslc* ]] && . ~fcm/bin/mosrs-setup-gpg-agent

# Enable bash completion for Rose commands
[[ -f ~fcm/rose/etc/rose-bash-completion ]] && . ~fcm/rose/etc/rose-bash-completion
```

Now, whenever you login to NEXCS you should be prompted for your MOSRS password (twice on first use only). Just press return if you don't want access to the repositories during that login session.

### Setting up password-less access from NEXCS to JASMIN (credits to Charlie)
1. Log in to NEXCS with `ssh <USERNAME>@lander.monsoon-metoffice.co.uk` and `ssh -Y -A xcslc0`
2. Generate ssh key pair `~/.ssh/id_rsa_jasmin` with:
```bash
cd ~/.ssh
<JASMIN-KEY> <JASMIN-KEY>.orig
openssl rsa -in <JASMIN-KEY> -out new.key
mv new.key <JASMIN-KEY>
```
3. Upload the `<JASMIN-KEY>.pub` to your [JASMIN profile](https://accounts.jasmin.ac.uk/account/profile/). Your public key will have been automatically propagated by the JASMIN accounts system to all the machines to which you have access rights, so should already be present in the correct place. Do not attempt to place the public key manually on any host within JASMIN: it will get automatically overwritten.
4. To skip manually loading the SSH key pair for each login, modify (or create if it doesn't exist) `~/.ssh/config` on NEXCS to add:
```bash   
Host hpxfer2 hpxfer2.jasmin.ac.uk
 Hostname hpxfer2.jasmin.ac.uk
 User ssteinig
 IdentityFile ~/.ssh/id\_rsa\_jasmin

Host jasmin login1.jasmin.ac.uk
 Hostname login1.jasmin.ac.uk
 User ssteinig
 IdentityFile ~/.ssh/id\_rsa\_jasmin
```
Note that some older documentation uses the `jasmin-xfer1` and `jasmin-xfer2` that have been or will be soon retired. Therefore adding `hpxfer2.jasmin.ac.uk` will be enough. You should now be able to log in to JASMIN from NEXCS with no password prompt with `ssh hpxfer1.jasmin.ac.uk`.