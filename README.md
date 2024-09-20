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

## Step 2: Prep all VM's for HomeLab environment 

1. Configure NAT Network
   ![gpedit](https://i.imgur.com/q7GDZji.png)
2. Change IP address on Linux VM
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
   

## Step 3: Installing Sysmon (Windows VM)
1. Download and install Sysmon to provide granular telemetry on Windows Endpoints.
2. Download and install [SwiftOnSecurity](https://infosec.exchange/@SwiftOnSecurity) Sysmon config
3. Validate Sysmon is running
![sysmon](https://i.imgur.com/gIDLhzw.png)
## Step 4: Installing LimaCharlie (Windows VM)
1. Create a LimaCharlie account
2. Install LimaCharlie on the Windows VM
  ![limacharlie](https://substackcdn.com/image/fetch/w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff821e410-d4d3-4161-a426-9d8ff348806c_610x392.png)
3. Add a rule to allow LimaCharlie to receive the Sysmon event table

## Step 5: Installing/Creating a payload with Sliver C2 (Linux VM)
1. Download and install Sliver
2. Create a directory for sliver: /opt/sliver
3. Generate a C2 session payload using sliver in /opt/sliver by running ``sliver`` and confirm the new implant configuration
![sliver](https://i.imgur.com/VXa2ZwP.png)

## Step 6: Python Server
1. Start a Python server with the command ``python3 -m http.server 80`` to transfer over the payload generated by Sliver to the Windows VM

## Step 7: Start a Command and Control Session
1. Start an Sliver HTTP listener by running the commands ``sliver-server`` and ``http`` while in Sliver
2. Execute the C2 payload on the Windows VM

## Step 8: Observe the EDR(LimaCharlie) Telemetry
1. Conduct hash analysis using VirusTotal
![hash analysis](https://i.imgur.com/Vx9d4dI.png)
This virus did not show up in VirusTotal because VT has never seen the file! Eric Capuano states "This actually makes a file even more suspicious because nearly everything has been seen by VirusTotal".

## Step 9: Stealing credentials with Sliver
1. In Sliver (still connected to the http listener session) run the command ``procdump -n lsass.exe -s lsass.dmp``
![procdump](https://i.imgur.com/PO1nz69.png)

## Step 10: Detecting the stolen creds with LimaCharlie
1. Look at the timeline of the Windows VM sensor and filter for "SENSITIVE_PROCESS_ACCESS". This will show the event where lsass was accessed.
![Alt text](https://i.imgur.com/2fRg32o.png)
![lsassevent](https://i.imgur.com/gwVpgdS.png)
2. Create a D&R(Detection & Response) rule that will alert anytime this event occurs. This rule specifies that the detection will only look at SENSITIVE_PROCESS_ACCESS where the process ends with lsass.exe. The response section generates a detection report with the name LSASS access.
![d&rbutton](https://i.imgur.com/QBXZeeC.png)
![d&rrule](https://i.imgur.com/HtJu3e0.png)
3. Test the new rul LSASS rule
![Alt text](https://i.imgur.com/wvo3q8d.png)
4. Save the rule as LSASS Accessed
![Alt text](https://i.imgur.com/BebmYh7.png)

## Step 11: Detect LSASS Accessed
1. Run the procdump command again
2. Look in the "Detections" tab of LimaCharlie. As you can see, our new rule worked, and the event is captured!
![Alt text](https://i.imgur.com/0Exlnax.png)

## Step 12: Perform a ransomware attack! (Almost)
1. As Eric Capuano states in his post "Volume Shadow Copies provide a convenient way to restore individual files or even an entire file system to a previous state which makes it a very attractive option for recovering from a ransomware attack". So as an attacker, we are deleting the copies so there is no way to recover from the ransomware attack.
2. Run the ``shell`` command, run the ``vssadmin delete shadows /all`` command and run ``whoami``
![Alt text](https://i.imgur.com/Km7lU2T.png)


## Step 13 Detect and Block the Attack
1. Look in the Detections tab of LimaCharlie
![Alt text](https://i.imgur.com/JZ4pBTg.png)
2. Make a new D&R rule for Shadow Copies Deletion. The action:report tells LimaCharlie to create a Detection report and the action:task is what will be used to block the attack by killing the parent process of the `vssadmin delete shadows /all` command. Run the `vssadmin delete shadows /all` command again and run `whoami`. Whoami didnt return anything because the D&R rule worked successfully. The rule terminated the parent process of `vssadmin delete shadows /all` making the shell hang and `whoami` not returning anything.


That's all folks!













