# Active Directory Brute Force Lab
Credit goes to MYDFIR for creating this project and putting together this lab! [MYDFIR Youtube Channel: Cybersecurity Projects Active Directory Project](https://www.youtube.com/watch?v=5OessbOgyEo&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=13))
### Learning Objective
- Hands-on experience with Active Directory, Kali, and Splunk to target a host machine and brute force it's password, all while ingesting telemetry created into Splunk. 
- using Windows Server 2019, Kali Linux (used "Crowbar" for brute force attack), Ubuntu (Splunk Server), Sysmon64, Splunk Universal Forwarder, Windows 10, Powershell, and Cmd Prompt. 

### Tools & Requirements
1. VirtualBox or VMWare
2. Windows VM
3. Windows Server VM
4. Ubuntu VM
5. Kali Linux VM
6. Sysmon
7. Splunk Enterprise
8. Splunk Universal Forwarder


## Step 1: Create a network diagram for an eagle-eyed view of how everything will work.
  ![gpedit](https://i.imgur.com/SpcKAxi.png)


## Step 2: Set up a Windows 10 Machine/Ubuntu Server/Windows Server/Kali Linux VM's
1. Install VirtualBox
2. Download and deploy a [Windows Server 2019 VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
3. Download and deploy a [Windows 10 VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
4. Download and deploy a [Linux Server VM](https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso).
5. Download and deploy a [Kali Linux Server VM](https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso).

## Step 3: Prep all VM's for HomeLab environment 

1. Configure NAT Network
   ![gpedit](https://i.imgur.com/q7GDZji.png)

2. Configure Splunk VM 
   - Configure User Profile
     ![gpedit](https://i.imgur.com/V2PCrYC.png)
   - Ubuntu Install Successful
     ![gpedit](https://i.imgur.com/R4XhHfh.png)
   - Using commmand "ip a" found initial IP address.
     ![gpedit](https://i.imgur.com/OzivsDH.png)
   - Using command "sudo nano /etc/netplan/(press tab to auto complete)" to access netplan and reconfigure IP address.
   - Match netplan to photo below.
     ![gpedit](https://i.imgur.com/7dEXUcb.png)
   - Save and apply netplan.
     ![gpedit](https://i.imgur.com/eVoJeFA.png)
   - Verify connectivity.
     ![gpedit](https://i.imgur.com/Ezv7Q0w.png)
   - Download and Install Splunk Enterprise by accessing download file through Vbox mounting.
     ![gpedit](https://i.imgur.com/qQxlBlQ.png)
     ![gpedit](https://i.imgur.com/m9fKMQ2.png)

3. Configure Windows Target Machine VM
   - Change device name to target-pc
     ![gpedit](https://i.imgur.com/u6e78YR.png)
   - Set static ip address as 192.168.10.100 in alignment with our diagram above. 
     ![gpedit](https://i.imgur.com/RgyfUlV.png)
   - Install Splunk Universal Forwarder after creating an account. [SplunkUniversalForwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)
     ![gpedit](https://i.imgur.com/leZadge.png)
   - Install Sysmon64 Config [Sysmon Olaf Hartong](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml)
     ![gpedit](https://i.imgur.com/BkivQEx.png)
   - SUPER IMPORTANT TO NOT MESS UP! Configure your inputs.conf file inside of (C:\Program Files\SplunkUniversalForwarder\etc\system\local)
     Here is the timestamp just to avoid any messups. [ActiveDirectoryProjectPart3](https://www.youtube.com/watch?v=uXRxoPKX65Q&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=16) @19:26
     [Link to inputs.conf file](https://github.com/MyDFIR/Active-Directory-Project) Pay special attention to the index when we set up splunk receiving to receive telemetry later.
     ![gpedit](https://i.imgur.com/kd97cHj.png)
     After updating inputs.conf file, be sure to restart your splunk universal forwarder service. While you are here select splunk universal forwarder properties and tab over to log on. Make sure that the log on is selected for "local system account" otherwise you may       have troubles later down the road ingesting your data to splunk. Also convenient time to double check that sysmon64 is up and running too.
   - Log into your splunk server through the browser using the address, 192.168.10.10:8000 and use your login credentials when you set up splunk initially. After that head over to "settings" at the top and then "indexes". Hit "new index" and name the index as                "endpoint" and save. Review indexes to verify if splunk saved it.
   - Now head over to "Forwarding and Receiving" in order to configure your receiving port to ingest the data coming from our universal forwarder. Under "Receive data", click "configure receiving" and click "new receiving port", then enter 9997 and hit save.
     Everything should be working smoothly now and you should start seeing telemetry coming through. If not be patient as the data might just need to take a little time. It will show up eventually. To see if it is coming through head to the "Search and Reporting" tab.
     Search for index=endpoint and you can verify that the host is target-pc. If you see this, then things are working correctly.
 
4. Configure Windows Server VM
   - The next step is to change the server vm name to Active Directory and set the static ip address to 192.168.10.7, then install Sysmon64 and Splunk Universal Forwarder on this VM following all the same steps with the inputs.conf file. After these steps are done, log      into your splunk server again and search "index=endpoint", you should not see two hosts and telemetry coming through for both that looks something like this.
     ![gpedit](https://i.imgur.com/FmTGGQX.png)
   - Configure your server with Active Directory Domain Services under the "Roles and Features" in control panel. Click next through all the steps until it begins to install.
     ![gpedit](https://i.imgur.com/ATcmgub.png)
   - Proceed to promote the server to a domain controller within server manager using the caution flag in the top right.
   - Add a new forest. Name the domain whatever you wish but I used masterhand.local
   - Leave everything as default and enter a password for DSRM recovery (we shouldn't need this for the scope of this lab).
   - Click next through everything until the prerequisites check is completed and the server will automatically restart. Log back in to your server.
   - Start creating some users. In the top right under tools in server manager, select "Active Directory Users and Computers". Expand out your domain until you see the folder, "Users". Create a new organizational unit called, "IT" and "HR". We will fill these OU's           later with a few users to try and brute force their passwords.
   - Create a new user inside of the IT Organizational Unit. I used Jenny Smith as my example, created her username and set her password. Create another user inside of HR. I used Terry Smith as my example, created his username and set his password.
     ![gpedit](https://i.imgur.com/6bjM8LW.png)

5. Join Target-PC to Domain Controller.
   - Head to "about" in the "Settings" app. Click on "Advanced system settings" and select the tab "computer name", select "change" and select, "domain" in the member of box. Type in your domain (for mine it was masterhand.local). If you get an error, it is because the      Target-PC machine doesn't know how to resolve the domain. To fix this we need to go and change our DNS settings in our IPv4 Properties. It should be pointed to google's dns, 8.8.8.8. We want to change this to point to our DNS server now which is, 192.168.10.7. Hit      "OK", and open a cmd prompt and run the ipconfig command to verify that the settings took place. If everything worked, head back to the previous window and retry to add the computer to the domain. A new window will pop up and ask for credentials to authenticate         this computer before adding it to the domain. Enter the Administrator account credentials for the server as it will have the proper permissions.
     ![gpedit](https://i.imgur.com/1bgSCk3.png)
     ![gpedit](https://i.imgur.com/KXX8lYl.png)
   - Your computer will restart and then you can log in using credentials from our Active Directory list. Select other user and you wnat to make sure that the "sign in to" is pointing to our domain. Enter the credentials for a user from the AD list and now you will log      in as a domain user!

6. Configure Kali Linux.
   - Boot up your Kali Linux machine and log in using the default credentials.
   - We want to set up a static IP address for this lab, so right click on the ethernet icon on the top right and select "edit connections". Select the first profile, which should be "wired connection 1". Then click on the gear icon at the bottom left of the window.    Select the tab IPv4 Settings and under the method drop down change it from "Automatic (DHCP)" to "Manual". Click on "Add", and type in the static IP we want for this machine, 192.168.10.250. Set the netmask as /24. Set the gateway to 192.168.10.1 and the DNS server to 8.8.8.8. Click save and close out the configurator. Right click the home screen and select, "Open Terminal Here". Type in "ip a" in the terminal and we can observe that the ethernet connection didn't update. In order to refresh the ethernet connection, you'll need to reselect the ethernet icon and click disconnect. Then reselect "wired connection 1" and then verify again using "ip a" in the terminal to see if the ip address updated. Go ahead and ping google.com to verify connectivity as well as your splunk server ip address.
   - Now update and upgrade out repositories using the "apt-get update && sudo apt-get upgrade -y" command. Enter the default kali password.
     ![gpedit](https://i.imgur.com/zPTBlE1.png)

7. Set up your attack.
   - Use the "mkdir" command to set up a new directory called, "ad-project".
   - For our attack we are going to be using a tool called, "Crowbar" to launch our brute force attack against our target machine. Use the command, "sudo apt-get install -y crowbar". Enter the password.
   - Next, we are going to use a popular word list called "rockyou" that comes preinstalled with Kali Linux. To navigate here we will type in the command, "cd /usr/share/wordlists/", then type in the command "ls" to view the contents of the folder. Rockyou should be highlighted in red. We will use gunzip to extract the contents of this file, type "sudo gunzip rockyou.txt.gz". Enter "ls" again and you should see rockyou.txt.
   - Now let's copy this file into our ad-project file on the desktop. Type, "cp rockyou.txt ~/Desktop/ad-project/", then let's change into the ad-project folder. Type, "cd ~/Desktop/ad-project/" enter. Then type "clear" to clear out the screen.
   - Enter, "ls -lh", now we can see some information about the size of this file. It's around 134m which is alot of passwords but we don't want to use all of them for the scope of this lab. So let's shorten it, enter "head -n 20 rockyou.txt > passwords.txt" and this will create a file with only 20 of the potential passwords in the rockyou.txt file. Let's open up the file by entering "cat passwords.txt".
   - In this lab we are going to be targeting a known password and machine just to view some telemetry coming through and get a feel for what a brute force attack may look like on a very simple scale. To do this we will directly put our known password into the rockyou.txt file. Enter, "nano passwords.txt" into the terminal and at the very bottom enter in the password you set for either of the users in your active directory list. For my example, I used "Spellbound*". Save the file by holding ctrl + x and pressing y then enter.
   - Let's verify that our password was saved in the rockyou.txt file. Enter "cat passwords.txt" and we should see our super secret password at the bottom of the list.
   - Before we launch our attack we need to head back over to our windows target machine to enable remote desktop so that we can try to authenticate into the machine using crowbar. Open the about section of the settings application and select "advanced system settings", log in with the administrator account, and select the remote tab at the top. Under remote desktop, we want to select the "allow remote connections to this computer", then hit "select users", click "add" and enter the two users we created and check names. It should look something like this. 
     ![gpedit](https://i.imgur.com/BBllgiZ.png)
   - Close out the other windows and hit apply when prompted.

8. Now for the fun.
   - Head back over to your Kali Linux machine and enter the "clear" command in the terminal so we have some space to work with. Enter, "crowbar -b rdp -u jsmith -C passwords.txt -s 192.168.10.100/32"
     ![gpedit](https://i.imgur.com/CztY4KJ.png)
   - Once you see an "RDP-SUCCESS : 192.168.10.100:3389 - 'user':'password" , then you've successfully brute forced into the target-pc! Pretty cool! Feeling like a hacker yet?

9. Observe the telemetry.
   - Let's head back over to our splunk interface and see what happened during the attack. Here's an example of what your telemetry could look like. I searched directly for the event code 4625 as I know that event id correspondes to a failed log in attempt. 
     ![gpedit](https://i.imgur.com/4lPr8fW.png)
   - Feel free to go and observe what other meta data is inside of your splunk interface. See if you can find where the workstation name and source network address of the attacking machine is located.
  
10. Install Atomic Red Team on your windows machine.
    - 





That's all folks!













