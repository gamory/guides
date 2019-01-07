
This will explain how to use the following on Windows 10 x64 (version 1809)  
  
Tested with:  
Git 2.20.1.windows.1 (https://github.com/git-for-windows/git/releases/download/v2.20.1.windows.1/Git-2.20.1-64-bit.exe)  
GPG4Win 3.1.5 (https://www.gpg4win.org/get-gpg4win.html)  
Yubikey 4 (https://www.yubico.com/product/yubikey-4-series/)  
Cmder console (http://cmder.net)
  
WARNINGS, please read and understand these before proceeding:  
  
WARNING 1: This works on my system with the above, but it is not thouroughly vetted.  Your milage may very.  
WARNING 2: You will need a local admin account on the machine, and we will be over-writing files in safe-mode.  
WARNING 3: If you find advice against the below data, you should probably listen to them; this may not be the advisable way to do this setup.  
WARNING 4: If your machine was encrypted with BitLocker, you will need the recovery key.  If it's a company machine, contact IT.  
  
If the above is ok with you, download the above software, make sure to install in this order:  
Git  
GPG4Win  
Cmder  
  
Setting up a new Yubikey:  
This is todo, but for now, use:  
https://github.com/drduh/YubiKey-Guide  

Getting it working in Cmder:  
Open Kleopatra, it should have been installed with GPG4Win  
Click on "Tools", then "Manage Smartcards".  This should show your Yubikey data.  If it does not, try hitting F5.  If nothing still, see troubleshooting below  
Make sure everything is saved, and reboot into Safe Mode:  
 - Click on the Start Menu  
 - Click on the gear icon to the left  
 - Type "safe mode"  
 - Click on "Change advanced startup options"  
 - Under "Advanced startup", click "Restart now"  
 - Once to the "Choose an option" screen, click on "Troubleshoot"  
 - Once to the "Troubleshoot" screen, click on "Advanced options"  
 - Once to the "Advanced options" screen, click on "Startup Settings", if it doesn't show, look for "See more recovery options"  
 - Once to the "Startup Settings" screen, click on "Restart"  
 - Once to the ""

GPG changes:  
  
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
 - Ensure it has these lines:  
 - - "enable-ssh-support"  
 - - "enable-putty-support"  
  
Git changes:  
  
Open Cmder  
If you have not previously configured Git, type the following:  
 - "git config user.name <Your name as it appears on the certs you made>"  
 - "git config user.email <Your e-mail as it appears on the certs you made>"  
Now you should be able to configure the key portion:  
Ensure your Yubikey is currently plugged in.  
Still in Cmder, type:  
 - "gpg --card-status"  
 - - This should show your card data, if not, see troubleshooting  
 - "gpg --list-keys"  
 - - This should show a bunch of key data, look for the line "pub", below that, it should show a long key-ID.  Copy this.
 - "git config user.signingkey <the key-ID you copied>"  
 - "git config --global commit.gpgsign true"  
 - "git config gpg.program "C:\Program Files (x86)\Gpg4win\bin\gpg.exe""
  
If all has gone well, you should now be able to try a commit, and it should prompt for the PIN (not the admin PIN) of your Yubikey.  You should likewise be able to do a "git push", and provided you entered the unlock PIN recently, may not even get prompted for it.  
