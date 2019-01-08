
This will explain how to use the following on Windows 10 x64 (version 1809)  
  
###### Tested with:  
Git 2.20.1.windows.1 (https://github.com/git-for-windows/git/releases/download/v2.20.1.windows.1/Git-2.20.1-64-bit.exe)  
GPG4Win 3.1.5 (https://www.gpg4win.org/get-gpg4win.html)  
Yubikey 4 (https://www.yubico.com/product/yubikey-4-series/)  
Cmder console (http://cmder.net)
  
## WARNINGS, please read and understand these before proceeding:  
  
WARNING 1: This works on my system with the above, but it is not thouroughly vetted.  Your milage may very.  
WARNING 2: You will need a local admin account on the machine, and we will be over-writing files in safe-mode.  
WARNING 3: If you find advice against the below data, you should probably listen to them; this may not be the advisable way to do this setup.  
WARNING 4: If your machine was encrypted with BitLocker, you will need the recovery key.  If it's a company machine, contact IT.  
  
If the above is ok with you, download the above software, make sure to install in this order:  
Git  
GPG4Win  
Cmder  
  
## Setting up a new Yubikey:  
This is todo, but for now, use:  
https://github.com/drduh/YubiKey-Guide  

## Fixing GPG files:   
Open Kleopatra, it should have been installed with GPG4Win  
Click on "Tools", then "Manage Smartcards".  This should show your Yubikey data.  If it does not, try hitting F5.  If nothing still, see troubleshooting below  
Make sure everything is saved, and reboot into Safe Mode:  
* Click on the Start Menu  
* Click on the gear icon to the left  
* Type "safe mode"  
* Click on "Change advanced startup options"  
* Under "Advanced startup", click "Restart now"  
* Once to the "Choose an option" screen, click on "Troubleshoot"  
* Once to the "Troubleshoot" screen, click on "Advanced options"  
* Once to the "Advanced options" screen, click on "Startup Settings", if it doesn't show, look for "See more recovery options"  
* Once to the "Startup Settings" screen, click on "Restart"  
* Once to the ""

## GPG changes:  
  
Open the folder "C:\Program Files (x86)\GnuPG\bin".  
Select and copy all the files (there should be 22).  
Open the folder "C:\Program Files\Git\usr\bin".  
Paste the files here, and over-write existing files when prompted.  
Open the folder "C:\Program Files (x86)\GnuPG\bin".  
Copy "gpg.exe".  
Open the folder "C:\Program Files (x86)\Gpg4win\bin".  
Paste the file here, over-write if prompted.  
Reboot back into your normal system.  
Open the folder "C:\Users\<your username>"  
If you have a ".gnupg" folder, open it, if not, create it and then open it.  
Modify/Create the file "gpg-agent.conf":  
* Ensure it has these lines:  
  * "enable-ssh-support"  
  * "enable-putty-support"  
  
## Git changes:  
  
This is assuming you have already setup your public key on the destination git host.  If not, see Github config.  
Open Cmder  
If you have not previously configured Git, type the following:  
* `git config user.name <Your name as it appears on the certs you made>`  
* `git config user.email <Your e-mail as it appears on the certs you made>`  
Now you should be able to configure the key portion:  
Ensure your Yubikey is currently plugged in.  
Still in Cmder, type:  
* `gpg --card-status`  
  * This should show your card data, if not, see troubleshooting  
* `gpg --list-keys`  
  * This should show a bunch of key data, look for the line "pub", below that, it should show a long key-ID.  Copy this.
* `git config user.signingkey <the key-ID you copied>`  
* `git config --global commit.gpgsign true`  
* `git config gpg.program "C:\Program Files (x86)\Gpg4win\bin\gpg.exe"`
  
If all has gone well, you should now be able to try a commit, and it should prompt for the PIN (not the admin PIN) of your Yubikey.  You should likewise be able to do a `git push`, and provided you entered the unlock PIN recently, may not even get prompted for it.  


## Github config:  
  
Login to your Github account.  
On the upper right, you should see an icon with an arrow, click on that.  
Click on "Settings".  
Click on "SSH and GPG keys".  
Click on "New SSH key", and copy the SSH public key you generated earlier.  
Click on "New GPG key", and send over your GPG public key.  
If they are both showing now, you should be good.  

## Generic SSH endpoint config:  
  
Login to the destination host.  
In the user folder of the desired user (ie: /home/<user>), go into ".ssh" (or create if needed).  
Modify (or create) "authorized_keys".  
Copy in your public SSH key and save the file.  
Ensure the file permissions are 700 (if not, type "chmod 700 authorized_keys".  
This should be good to go, but if it doesn't seem to require the SSH key when you login, reload the service: `sudo systemctl reload sshd.service`.  
  
## Troubleshooting:  
  
#### After removing the card, it seems to stop working:  
Unfortunately, this does happen sometimes, and you have to kill the existing gpg-agent.  From a command prompt try:  
`taskill /F /IM gpg-agent.exe`  
  
Then try `gpg --card-status` again, and see if it has come back.  
  
#### gpg --card-status says it can't contact the agent
First off, make sure there are NO instances of "gpg-agent" running on your machine (under any user).  
Try running `gpg --card-status` again, and it should spawn it's own agent service.  

#### gpg --list-keys doesn't show any keys, whether the yubikey is inserted or not  
GPG isn't aware of your public key.  Run the following:  
`gpg --import <your GPG public key>`  

It should say that it was imported, and should have the keyID, copy this ID.  If not, something has gone wrong in the import.  (If so, are you sure you specified the GPG Public key, NOT the SSH key)  
  
Now you want to make sure that is trusted.  Run the following:  
`gpg --edit-key <that keyID>`  

You should be in a "gpg>" prompt, type "trust" and press enter.  If should prompt for a trust level between 1 and 5.  If you feel like you can be trusted to import your own keyfile ( :-) ) then type "5" and press enter.  When prompted to confirm, press "y" and enter again.  Type "save" and hit enter.  It may say that no changes were saved, but you should be good.  It should drop back to the command prompt.  
  
Try `gpg --list-keys` again.  It should show your key, and right before your name, it should say "[ultimate]".  If it does, the key was imported and trusted successfully.  If not, try the above again.  I did have an instance where I had to trust a key twice to get it to take.  You may also want to stop any existing GPG processes to make sure they get fresh data on the next command (so, look for any of the following tasks: gpg.exe, gpg-agent.exe, kleopatra.exe).  

#### gpg --card-status shows nothing, you have already tried stopping gpg-agent  
Open a command prompt and type:  
`where gpg.exe`  
`where gpg-agent.exe`  
  
The first (and hopefully only) listings for those should be in the same directory (which should be "C:\Program Files\Git\usr\bin").  If they are not, you may need to look for and clean up PATH variables.  That's beyond the scope of this article, but finding instructions on google should be easy.  Likewise, it may be useful to use a tool like Process Explorer from SysInternals to determine the calling processes, and their locations.  Again, it's a bit beyond the scope of this.  
  
#### git commands prompt for your login/say they can't find the key  
Make sure that git is configured to use your specified gpg.exe, you can do this by running:  
`git config gpg.program`  
  
It should be set to "C:\Program Files\Git\usr\bin\gpg.exe".  If not, enter:  
`git config --global gpg.program "C:\Program Files (x86)\Gpg4win\bin\gpg.exe"`  

  
## Closing  
Feel free to contact me, or open an issue, and I can try to help.  I'm no expert, but I can at least try and help.  
