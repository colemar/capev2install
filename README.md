Part 1 written by rebaker501

_Note: I (JTurn01) did not use physical hardware for my CAPE setup. I used an Azure VM running Ubuntu._

# Capev2 Sandbox Installation Part 1 - Installation 

So, I spent so much time using the official docs for CAPEv2. The Github readme and the install docs conflict a bit. There were also some very key issues called out in the Github issues section. Those are incorporated here. I installed and reinstalled over 40 times. Doomedraven has done a wonderful job of getting this code optimized and continuing on the Cuckoo project. Hopefully this documentation will save some time so that you can start utilizing this terrific project. Also, feel free to copy this documentation. It's for the community.

The official repositories for CAPEv2 and Doomedraven's Tools are below
[CAPEv2 Malware Configuration And Payload Extraction](https://github.com/kevoreilly/CAPEv2)



# Hardware

Let's talk about hardware. I've run the install on a 2U Dell Server from 2021 with 2 CPUs, 16 cores, 128GB RAM, and 30TB of SSD RAID to a small micro PC from HP. This will work on almost anything that can run at least 1 Windows 10  VM. You'll probably want at least a 4 core processor (recent-ish), 16GB RAM, and 128GB SSD minimum. More is better obviously. Don't get too hung up on hardware. Let's get this thing running!

## Install Ubuntu 22.04 LTS Desktop Edition

 1. Download Ubuntu 22.04 LTS Desktop from the official repository
    [Ubuntu 22.04 LTS ISO](https://ubuntu.com/download/desktop)  
   
 2. Create a bootable USB installer for your newly downloaded ISO file. I
    recommend Balena Etcher as it works on Mac and Windows. [Download
    Balena Etcher](https://www.balena.io/etcher/). You'll need at least
    an 8GB USB drive for this task.
    
 3. Boot the PC/server that you've chosen for the CAPEv2 install with the newly created Ubuntu 22.04 LTS USB. 
 4. During the install, choose minimal install, uncheck download updates while installing (we will get to those later from the command line).
 5. Choose your disk formatting. I recommend a to use the entire disk and reinstall completely over any OS that was on that device before. 
 6. When it's time to choose a machine name and user, I'd recommend using *cape* as your username and whatever computer name you happen to choose. Login automatically works for me as the console of this machine is secure. This is purely a risk decision on your part.
 7. Choose your timezone. I set mine to UTC, but this again is a personal choice.
 8. Reboot this PC when instructed (remove the install media).
 9. Once the Ubuntu Desktop is up, you might get a welcome pop up. For Livepatch just click next. We don't need that running. Help Improve Ubuntu? Choose No, don't send system info and click next. Privacy screen, click next. Click Done. (At time of writing this was valid for 22.04.1 LTS).
 10. If/When Software Updater pops up, just close it. 
 11. From the Activities button find Settings. Open Power. Choose Performance, turn off dim screen, and change screen blank time to never (it's at the bottom of that dropdown list). Close settings.
 12. Click Activities and search for Terminal. Might want to drag that to your favorites bar as we will need it. 
 13. From the terminal issue the following commands
		

    sudo apt update && sudo apt upgrade -y && sudo reboot
    
    

 14. Log back into the Ubuntu device and open the Terminal in preparation for the dependencies installation.

 

 

## Getting some dependencies out of the way

This is where the instructions incorporate my experience. Would like to give a shout out to Victor Batiz for the assist on some of this. Always nice to have someone else to help from time to time. 

 1. Let get some packages installed. You can omit openssh-server and vim if you don't want those. I found that if I didn't do this, pyre2 would fail every time during the cape2.sh script that we execute later.
 `sudo apt-get install -y git build-essential cmake ninja-build python3-dev cython3 pybind11-dev python3-pip libre2-dev vim openssh-server acpica-tools net-tools gperf`
 
 2. Once step 1 completes, let's get our KVM installer downloaded.
 3. From a terminal:

    `wget https://raw.githubusercontent.com/kevoreilly/CAPEv2/master/installer/kvm-qemu.sh`

 4. This file will need to be edited with some hardware information. I think this is right, but if it's not, please feel free to put in an issue and give us the assist. Documentation was not good regarding this, so hours upon hours where spent. Here's how I got the data I believe is correct to replace WOOT within the kvm-qemu script. Run the following commands. I recommend doing this from a temp folder you can create in your home directory. Once again, this is my estimation of what to do as the instructions literally say to Google it.
 5. `sudo acpidump > acpidump.out`
 6. `sudo acpixtract -a acpidump.out`
 7. `sudo iasl -d dsdt.dat`  (NOTE: my steps are case sensitive)
 8.  The output of that last command produced a line that contained a 4 digit code. ACPI: DSDT 0x0000000000000000 0213AC (v02 HPQOEM **82BF**     00000000 INTL 20121018). I used 82BF to replace all of the `<WOOT>` instances in the kvm-qemu.sh file.
 9. Set the installer to executable `sudo chmod +x kvm-qemu.sh`
 10. **colemar**: `./kvm-quemu.sh -h` for help, `./kvm-qemu.sh issues` to see known issues
 11. **colemar**: kvm-qemu.sh will execute `systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target` unconditionally.
 12. Execute the installer for KVM from the terminal, replace <username> with current active user `sudo ./kvm-qemu.sh all <username> | tee kvm-qemu.log` # **colemar**: As of commit 76b801b9c (2023-03-14), kvm-qemu.sh will use $SUDO_USER regardless of the optional <username> argument, which means it will always use the current active user, therefore `sudo ./kvm-qemu.sh all | tee kvm-qemu.log` is fine.
 13. `sudo reboot`
 14. Install virtmanager GUI, replace <username> with current active user `sudo ./kvm-qemu.sh virtmanager <username> | tee kvm-qemu-virtmanager.log` # **colemar**: if `ubuntu-desktop`, `ubuntu-desktop-minimal` or any package `ubuntu-desktop*` is installed, then virtmanager was already installed with the `all` option above.
 15. `sudo reboot`

## Install CAPEv2 Sandbox

In this section we install CAPEv2. The installer will need to be edited a little. Additionally, I had some problems with tor, so I disabled that in the config. Obviously, you'd probably want this functionality. Let's get it running and then we can enable extra features one at a time so troubleshooting is easier if there are any problems.

 1. Open the Terminal and let's get going. Go to your home directory `cd ~`
 2. `wget https://raw.githubusercontent.com/kevoreilly/CAPEv2/master/installer/cape2.sh`
 3.  Edit cape2.sh with your favorite editor
 5.  Here are some of the parameters. Verify these.
	> NETWORK_IFACE=virbr0 
	> IFACE_IP="192.168.122.1"
	> PASSWD="<pswd>" Replace <pswd> with your own password
	> USER=cape
 5. `sudo chmod a+x cape2.sh`
 7. `sudo ./cape2.sh all | tee cape.log`
 8. `sudo reboot`
 9. **colemar**: `journalctl -u cape` 'CRITICAL: CuckooCriticalError: Unable to bind ResultServer on 192.168.1.1:2042' ==> change `ip` item to `192.168.122.1` in section `[resultserver]` of file `/opt/CAPEv2/conf/cuckoo.conf`
 10. **colemar**: `cd /opt/CAPEv2/conf;grep -r '192\.168\.1\.'` ==> change ´192.168.1.´ to ´192.168.122.´ by executing `sudo sed -i 's/192\.168\.1\./192.168.122./' $(grep -rl '192\.168\.1\.')`
 11. **colemar**: `cd /opt/CAPEv2/conf;grep -r 'virbr1'` ==> change 'virbr1' to 'virbro' by executing `sudo sed -i 's/virbr1/virbr0/' $(grep -rl 'virbr1')`.
 12. Once rebooted, there is one more hurdle. This one took me a while to figure out. It was buried in the issues section of the Github repository. The database doesn't have proper permissions by default, so this will correct that. **colemar**: database `cape` owner was already `cape`.
 13. From the Terminal `sudo -u postgres psql`
 14. `ALTER DATABASE cape OWNER TO cape;`
 15. `\q`
 16. **colemar**: at this point is **really** better to switch to user cape for the next steps: `sudo -i -u cape`
 17. `cd /opt/CAPEv2`
 18. `pip3 install -r requirements.txt` # **colemar**: this is incorrect as stated in https://capev2.readthedocs.io/en/latest/installation/host/installation.html; in my case it failed at `daphne==3.0.2`; go straight to the next step.
 19. `poetry install` # **colemar**: already done by line 1262 in `cape2.sh`
 20. `sudo -u cape poetry run pip install -r extra/optional_dependencies.txt`
 21.  Run `sudo journalctl -u cape.service`. This will show the log for the CAPE service. My first try showed that some dependencies were missing. Yours may vary, but this is what I had to do next.
 22.  **colemar**: solving dependencies will not suffice to avoid `libvirt: QEMU Driver error : Domain not found: no domain with matching name 'cuckoo1'` and `Error initializing machines`, since there is no virtual machine named 'cuckoo1' yet. The following poetry steps are likely not needed if optional_dependencies.txt command above worked.
 23. `poetry run pip3 install https://github.com/CAPESandbox/peepdf/archive/20eda78d7d77fc5b3b652ffc2d8a5b0af796e3dd.zip#egg=peepdf==0.4.2`
 24. `poetry run pip3 install -U git+https://github.com/DissectMalware/batch_deobfuscator`
 25. `poetry run pip3 install -U git+https://github.com/CAPESandbox/httpreplay`
 26. Work through that until you get no errors from journalctl. You might have to restart the service. The other cape services can be found by typing journalctl -u cape and hitting tab a couple of times. This will list out the other services. You'll need to look at these to figure out if anything is wrong. In your routing.conf you'll need the tor line to reflect what interface KVM is attached to, in my case virbr0.
 27. **colemar**: set `arch = x64` in /opt/CAPEv2/conf/kvm.conf

## Build an Analysis VM

We will need a virtual machine build for analyzing malware samples. A Windows 10 VM will work. You can use Windows 7, Windows 10, Windows 11, Windows Server, Linux, etc. Let's start with one. Disabling the services on Windows 10 is a bit of a pain, but I've found some good scripts to automate the process. You'll need a Windows 10 license and I strongly suggest a copy of MS Office (2019 or 2016 is just fine). The new version (365) might be too chatty on the network. Some people suggest 4GB of RAM for the VMs but I use 8GB. Also, never use less than 2 cores or CPUs. No devices in the past 10 years are single core and malware might look for that. Let's start.

 1. Create a new KVM via Virt-Manager. Do not be attached to the network while installing and after install. Just remove the network card from the configuration for now. We do not want this VM to take updates. # **colemar**: see `https://www.doomedraven.com/2016/05/kvm.html#modifying-kvm-qemu-kvm-settings-for-malware-analysis` for some counter-anti-vm spoofing.
 2. Begin the installation. Choose your language, accept license terms, etc. Custom install, use the entire drive for the install. Installation will proceed on it's own for a bit.
 3. Download some software for the VM onto a USB drive on another machine while we wait. If your VM has internet, use the download links to get the software.
 

> Python3 32bit for Windows (imperative that this is 32bit) When you install Python make sure you install for all users and check the option to include Python in the PATH [Download](http://www.python.org/getit/)
> 
> Older versions of Adobe Acrobat. Try something from 2017 or so. I like to make sure the version I choose is one of the first of that major version. [Adobe Acrobat - Archive](ftp://ftp.adobe.com/pub/adobe/reader/win/) or [Old Versions](http://www.oldversion.com/windows/download/acrobat-reader-xi-11-0-01)
> 
> Download Microsoft Office Installer. You'll need your own licensed copy for this. 2013 or 2016 is fine. I'd stay away from the newer ones.
> 
> Java Runtime [JRE Archive Downloads](https://www.oracle.com/java/technologies/downloads/archive/) or [Old Versions Website](http://www.oldversion.com/windows/java-platform/)
> 
> PowerShell 5.1 (Make sure to get the correct version for your Windows version) [Download PowerShell](https://www.microsoft.com/en-us/download/details.aspx?id=54616)
> 
> Autoruns to discover any update services that could produce network activity [Download Autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns)
> 
> CAPE agent. I like to download it to the guest software install media as it's sometimes a pain to get it from the host to the guest. [CAPE Agent for Windows](https://raw.githubusercontent.com/kevoreilly/CAPEv2/master/agent/agent.py)
> 
> Windows 10 De-Bloat Scripts. Download the entire zip here are you will need multiple scripts from this repo. [Debloat-Windows-10](https://github.com/W4RH4WK/Debloat-Windows-10)
> 
> Windows Defender Remover [Download repo .zip](https://github.com/jbara2002/windows-defender-remover)
>
> Windows 10 Decrapifier. Removes a lot of the noise that Windows 10 generates [Windows 10 Decrapifier](https://gist.github.com/rca/0d265148f7f754e642a79713330bf8f9)

 3. Once the install is back up and asking for user input, select your region  and language. 
 4. Since we are not connected to the internet right now, as we have no NIC, select Continue with limited setup in the lower left corner.
 5. Choose a fun username and a memorable password. Create some security questions with answers. 
 6. Privacy settings...turn them all off.
 7. Not now for Cortana.
 8. Once the install is complete, type virus in the search bar and click Virus & threat protection settings. Click manage settings in the Virus & threat protections settings section. When that opens turn off real time protection, cloud delivered protection, and automatic sample submission. This will allow us to run the removal tool.
 9. Insert the USB drive into the host computer and access it from the new Windows 10 guest. 
 10. Let's run Defender Remover. Right click the Defender Remover .exe file and click Run as Administrator. A command window will appear. Answer Y to remove Defender and confirm with another Y. The software will be removed and your VM will reboot.
 11. PowerShell needs some attention at this point. Open an Administrator PowerShell session.
 12. Type `Set-ExecutionPolicy Unrestricted` and hit enter. Confirm by typing A for all.
 13. Now enter `netsh interface teredo set state disabled` and hit enter.
 14. Start `gpedit.msc` from the PowerShell window.
 15. Go to Computer Configuration > Administrative Templates > Network > DNS Client, and open Turn off Multicast Name Resolution. Enable that.
 16.  One more quick change from the Group Policy editor. Computer Configuration> Administrative Templates> System> Internet Communication Management, and open Restrict Internet Communication. Enable it. This will reduce noise further.
 17. Back to the PowerShell. Open the directory containing the Deploat Scripts. Type the following `ls -Recurse *.ps*1 | Unblock-File`. Now we will execute a few of these. Start with disable-services.ps1, then block-telemetry.ps1, remove-default-apps.ps1, remove-onedrive.ps1. Move to the utils directory and run disable-scheduled-tasks.ps1. This should tighten things up quite a bit now.
 18. Install Edge Browser x64 offline installer.  The browser will ask some questions. Basically answer those where it won't talk to MS as much. 
 19. Open Settings in Edge. Profile Settings - turn all of those off. Privacy - turn off tracking protection. Choose what to clear when exiting browser - turn all of those on. Uncheck typo protection and the other ones in that section and everything in the services section. Home button - Make it about:blank and uncheck preload new tab page.
 20. Search for internet options in Windows search bar. Open that. Make sure about:blank is the home page, delete browser history on exit.
 21. Security tab - Lot's to do here.
     Uncheck enable protected mode. Click Custom Level.
     In the list, change the following:
     - Loose XAML - Enable XAML Browser Applications
     - Run components not signed with Authenticode - Enable
     - Run components signed with Authenticode - Enable
     - Allow ActiveX Filtering - Disable
     - Allow previously unused ActiveX controls to run without prompting - Enable
     - Allow scriptlets - Enable
     - Display video and animation on a webpage - Enable
     - Download signed ActiveX controls - Enable
     - Download unsigned ActiveX controls - Enable
     - Initialize and script ActiveX controls not marked as safe - Enable
     - Only allow approved domains to use ActiveX - Disable
     - Run ActiveX controls and plug-ins - Enable
     - Antimalware software - Disable
     - Script ActiveX controls marked as safe for scripting - Enable
     - Access data sources across domains - Enable
     - Allow dragging ... 2 entries - Enable
     - Allow scripting of Microsoft web browser control and script initiated windows - Enable
     - Allow the TDC control - Enable
     - Allow webpages to use restricted protocols - Enable
     - Allow website to open windows without address - Enable
     - Display mixed content - Enable
     - Don't prompt for client cert - Enable
     - Launching applications and unsafe files - Enable
     - Launching programs and files in IFRAME - Enable
     - Navigate windows and frames across different domains - Enable
     - Render legacy filters - Enable
     - Use pop-up blocker - Disable
     - Use Windows Defender SmartScreen - Disable
     - Userdata persistence - Disable
     - Allow programmatic clipboard access - Disable
     - Allow status bar updates via script- Enable
     - Logon = Anonymous logon
 23. When you try to save these settings you'll get a warning they are not secure. Just accept that and move on.
 24. Privacy tab of Internet Options: Uncheck Turn on pop-up blocker and disable toolbars. Advanced Security Section: Allow active content - check all of those. Uncheck Check For... boxes. Make it permissive. Use your judgement here as things change. 
 25. Install Python 32bit for Windows. Customize the installation. Check the box to add Python to PATH. Make sure everything else is checked including install for all users. Install it all. At the end of the install there is an option to disable PATH length limit. Click that. Y
 26. Type Windows Firewall in the Windows search bar. Open Windows Firewall. Click Turn Windows Defender Firewall on/off. Turn it off.
 27. From the Windows search bar, type services. Open the Services application and look for Windows Update. Double click that and stop it. Change the service startup to disabled.
 28. Open Task Scheduler. In the Microsoft folder look for Windows Update and Windows Defender. Delete all tasks in there. These re-enable those services and we don't want that.
 29. Let's reboot and re-add the network card. That'll be in the Details section of your virt-manager GUI for your VM. Add Hardware button there.
 30. Once you're back up and running set a static IP, DNS, default GW, etc. You'll need a static IP for CAPE configs.
 31. Install Office. Activate it when complete. In Options, go to Security Center. Turn on all macros and anything that looks like it would make office insecure. Do this for Word, PowerPoint, and Excel for sure.
 32. Install Adobe Acrobat. Once again, go into preferences, turn off updates and under Security make sure to make it the least secure. Each version is different, so use your judgement. Make sure to open Acrobat and accept any EULAs. Also make it your default .pdf application otherwise Edge will be the default.
 33. Install Java Runtime. There is a Control Panel control that will allow you to customize the settings. Turn off updates and make everything very insecure. They're all a little different, so use your best judgement.
 34. Open an Administrator Command Prompt. Install Python Image Library (Pillow). `python -m pip install --upgrade pip` and `python -m pip install --upgrade Pillow` are the commands to run here. 
 35. Open autoruns and look to the Logon tab. Delete anything you don't want starting like Java, Adobe, etc. 
 36. Install the CAPE agent. This will be in your agent directory of your CAPE directory on Linux or the repo. It is named agent.py by default. Change it to something different and use a .pyw extension. The W in the pyw extentions hides the Pyhon console when running a Python program in Windows. Add this to your Run registry key. 
 37. Open Regedit. From HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run add a STRING value. Call it whatever you want. Double click it and add the path to youragentname.pyw to the data section. The CAPE agent will start at next reboot. You can verify this by checking Task Manager and looking for a Python32 process. It should be the only one (might be listed twice but don't worry).
 38. Reboot. Once everything comes back up, make sure your agent is running, apps open cleanly, etc. When you have this done, go to snapshots from your virt-manager window and add a snapshot (green plus at bottom left). Name your snapshot something without spaces. I get nervous with spaces in names on linux. That snapshot name is going into a .conf file later on. Once you have your snapshot, you can turn off the VM. 
 39. Clone this VM if you want to but you'll need to change machine name and IP address.
 40. WHEW! All done.

## Optional Additional Configurations
If Windows 10 is the VM


# Capev2 Sandbox Installation Part 2 - Configuring and Running CAPE

At this point you should have a server running Ubuntu 22.04 with CAPE and KVM installed on it. Within KVM you should have a VM (or VMs) with a snapshot taken after configuring the machine; this is the CAPE agent.
**ALL OF THESE STEPS NEED TO BE DONE USING THE CAPE USER! Open a terminal and use command `sudo su - cape -c /bin/bash` to run commands as CAPE user**
## Editing configs 

In this section we are going to change the CAPE configs on the server so that, when CAPE runs, it can find the VM which you created. Make sure you write down the IPs for the server and CAPE agent.

In a terminal (as CAPE user):
1. `cd /opt/CAPEv2/conf/`
2. `nano kvm.conf`
3. Find and change the matching config settings 
   - machines = _your_vm_name_
   - interface = virbr0
   - label = _your_vm_name_
   - platform = _os_on_vm_
   - ip = _ip_of_vm_
   - label = _your_vm_name_
   - ip = _ip_of_vm_
4. `nano cuckoo.conf`
5. Find and change the matching config settings
   - _(Optional)_ machinery_screenshots = on _This allows cape to take pictures during analysis_
   - _Under [resultserver]_ ip = _ip_of_local_server_
   - port = _port_of_local_server_

## Running CAPE

Now you should be ready to run cape from your server. Running cape must be done with the cape user, which was created when you ran the cape2.sh script, so please be sure to use the commands as written.  

If you run into any errors with running CAPE, see the "CAPE Errors" section below.  

1. `cd /opt/CAPEv2/`
2. `poetry run python3 cuckoo.py`
3. If cape is running correctly, you will get an output similar to this:
> Cuckoo Sandbox 2.1-CAPE
> www.cuckoosandbox.org
> Copyright (c) 2010-2015
>
> CAPE: Config and Payload Extraction
> github.com/kevoreilly/CAPEv2
>
> 2020-07-06 10:24:38,490 [lib.cuckoo.core.scheduler] INFO: Using "kvm" machine manager with max_analysis_count=0, max_machines_count=10, and max_vmstartup_count=10
> 2020-07-06 10:24:38,552 [lib.cuckoo.core.scheduler] INFO: Loaded 100 machine/s
> 2020-07-06 10:24:38,571 [lib.cuckoo.core.scheduler] INFO: Waiting for analysis tasks.

## CAPE Errors
  
### _2024-05-17 18:53:22,589 [root] CRITICAL: CuckooCriticalError: No machines available_
This error occurs when you set up your config files wrong. Go back and make sure everything is correct within those files.
  
  
### _AttributeError: 'KVM' object has no attribute 'vms'_ 
This error occurs when you set up your config files wrong. Go back and make sure everything is correct within those files.
  
  
### _libvirt.libvirtError: Domain not found: no domain with matching name 'cuckoo1'_
This error occurs when you set up your config files wrong. Go back and make sure everything is correct within those files.
  
## WORK IN PROGRESS
