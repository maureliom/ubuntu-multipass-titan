# ubuntu-titan-multipass
This is a cloud-init template that provision a remote workstation configured for Eclipse TITAN development using multipass tool from Canonical
```
#cloud-config

 --------------------------------------------------------------------------------------------------------------------------------
                          UBUNTU WORKSTATION ENVIRONMENT SETUP FOR DEVELOPMENT 
 --------------------------------------------------------------------------------------------------------------------------------
  Author:          Marco Aurelio Micheletto
                   maureliom@hotmail.com
  Created:         October 2025
  Version:         1.0

  License:
      This configuration is distributed under the MIT License.

      Permission is hereby granted, free of charge, to any person obtaining a copy
      of this software and associated documentation files (the "Software"), to deal
      in the Software without restriction, including without limitation the rights
      to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
      copies of the Software, and to permit persons to whom the Software is
      furnished to do so, subject to the following conditions:

      The above copyright notice and this permission notice shall be included in all
      copies or substantial portions of the Software.

      THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
      IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
      FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
      AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
      LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
      OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
      SOFTWARE.

                      Copyright (c) 2025 Marco Aurelio Micheletto.


  Reference Documentation:
    - Multipass documentation:
              https://multipass.run/docs
    - cloud-init reference:
        https://cloudinit.readthedocs.io/en/latest/
    - autoinstall reference:
        https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html 
    - ECLIPSE TITAN
        https://gitlab.eclipse.org/eclipse/titan/titan.core
        https://gitlab.eclipse.org/eclipse/titan/titan.EclipsePlug-ins
        https://projects.eclipse.org/projects/tools.titan

  Description:
    This cloud-init configuration automatically provisions a Ubuntu development 
    environment for use with:
      - Eclipse IDE configured with TITAN Designer plugin
      - Visual Studio Code 
      - Maven
      - Java 11 and 21 (OpendJDK and Temurin )
      - Python
      - TTCN-3
      - XRDP dual connection (RDC and HYPER-V enhanced mode connections)
      - Visual Code
      - TITAN Core
      - TITAN Designer
      - GNOME
      - XFCE4
      - Configured to Ubuntu 24.04

    Can be integrated to any provisioning tool that accepts cloud-ini file:
      - Multipass
      - Packer / Qemu
 -------------------------------------------------------------------------------------------------------------------------------
                          !!!! UBUNTU DROPPED LTS FOR VAGRANT IMAGES !!!!
              https://documentation.ubuntu.com/public-images/public-images-explanation/vagrant/
              
 -------------------------------------------------------------------------------------------------------------------------------
                          BENCHMARK

--------------------------------------------------------------------------------------
| Feature / Laptop                 |          (i5-1135G7, 32 GB DDR4-2667)           | 
| :------------------------------- | :-----------------------------------------------| 
| CPU cores / threads usable by VM | 2 vCPUs max (≈ 50 % CPU load for host)          | 
| CPU load under full VM           | 90–100 % → throttling likely                    | 
| RAM allocated for VM             | 8–12 GB                                         | 
| RAM bandwidth required           | DDR4-2667 (~42 GB/s max) → may saturate         | 
| NVMe throughput demand           | PCIe 3.0 x4 → ~1.5–2 GB/s seq, 0.5–1 GB/s rand  | 
| GPU load (for GUI acceleration)  | Iris Xe 80 EUs → ok for XFCE / light GNOME      | 
| Disk I/O bottleneck              | Likely under heavy builds                       | 
| Thermal throttling risk          | Medium-high (>90 °C under sustained load)       | 
| Multipass backend performance    | `Hyper-V` (best option)                         | 
| Practical max VM spec (zero-lag) | 2 vCPUs, 8 GB RAM, XFCE, pagefile 8–12 GB       | 
--------------------------------------------------------------------------------------
| Expected VM responsiveness       | Fair-poor; freezes possible under build load    | 
--------------------------------------------------------------------------------------


| Feature / Laptop                 |      (i7-1320P, 32 GB LPDDR5-5200, UHD Graphics)            |
| :------------------------------- |:----------------------------------------------------------- |
| CPU cores / threads usable by VM | 4–6 vCPUs (≈ 33–50 % CPU load for host)                     |
| CPU load under full VM           | 55–70 % → smooth operation under load                       |
| RAM allocated for VM             | 16–24 GB                                                    |
| RAM bandwidth required           | LPDDR5-5200 (~83 GB/s total dual channel) → ample           |
| NVMe throughput demand           | PCIe 4.0 x4 → ~5–6 GB/s seq, 2–3 GB/s rand                  |
| GPU load (for GUI acceleration)  | Intel UHD (64 EUs) → usable but limited for GNOME animations|
| Disk I/O bottleneck              | Minimal                                                     |
| Thermal throttling risk          |Low (hybrid design + efficient cooling)                      |
| Multipass backend performance    |`Hyper-V` (recommended)                                      |
| Practical max VM spec (zero-lag) | 4 vCPUs, 16 GB RAM, GNOME/XFCE, pagefile 16 GB              |
--------------------------------------------------------------------------------------------------
| Expected VM responsiveness       | Very good; stable GUI and compile times near native         |
--------------------------------------------------------------------------------------------------


| Feature / Laptop                 |  (i7-12800H, 32 GB DDR5-5200, Iris Xe + GeForce MX550)  |
| :------------------------------- |:--------------------------------------------------------|
| CPU cores / threads usable by VM | 4–6 vCPUs (≈ 40–60 % CPU load for host)                 |
| CPU load under full VM           | 60–80 % → minimal throttling with GPU offload           |
| RAM allocated for VM             | 16–24 GB                                                |
| RAM bandwidth required           | DDR5-5200 (~83 GB/s total dual channel) → ample         |
| NVMe throughput demand           | PCIe 4.0 x4 → ~5–6 GB/s seq, 2–3 GB/s rand              |
| GPU load (for GUI acceleration)  | Iris Xe + MX550 → excellent for GNOME and 3D tasks      |
| Disk I/O bottleneck              | Minimal                                                 |
| Thermal throttling risk          | Medium; may throttle after long compiles (80–85 °C)     |
| Multipass backend performance    | `Hyper-V` (recommended)                                 |
| Practical max VM spec (zero-lag) | 4–6 vCPUs, 16 GB RAM, GNOME, pagefile 16 GB             |
----------------------------------------------------------------------------------------------
| Expected VM responsiveness       | Excellent; smooth response under full load and GPU tasks|
----------------------------------------------------------------------------------------------


----------------------------------------------------------------------------------------------------------------------------------
                          ADJUST VM CAPABILITIES TO WHAT IS DOABLE BASED ON HOST SPECS 
  >  multipass get --keys
       local.dev-workstation.cpus
       local.dev-workstation.memory

  Overview:

  This guide explains how to:
     1. Configure Hyper-V virtual switches with static IPs
     2. Launch and provision a Multipass VM using cloud-init
     3. Enable Enhanced Session mode for GUI
     4. Manage SSH fingerprints (known_hosts)
     5. Use tools to transfer files between host and guest

 --------------------------------------------------------------------------------------------------------------------------------
                          PREPARATION
 
  !!!!    For credentials setup and adjustments follow the tags {USER_INPUT}   !!!!

   Configure credentials in places marked with {USER_INPUT}
       keep standard user multipass password multipass


 -------------------------------------------------------------------------------------------------------------------------------
                          HYPER-V DEPLOY

   Configure host static IPs (Hyper-V)
   Define two vSwitches:
     - "multipass_external" (External)
     - "multipass_internal" (Internal)

   In PowerShell (Run as Administrator):
     > The gateway for the static IP
     > New-NetIPAddress -IPAddress 192.168.138.1 -PrefixLength 24 -InterfaceAlias "vEthernet (multipass_internal)"

   Verify:
     > Get-NetIPAddress -InterfaceAlias "vEthernet (multipass_internal)"
     > Get-NetNat

   Launch Multipass with static networking and cloud-init:
   (run as administrator from PowerShell in clout-init-multipass.yaml folder)

   > multipass launch 24.04 `
         --name dev-workstation `
         --memory 12G `
         --disk 20G `
         --network name-"multipass_external" `
         --network name-"multipass_internal",mode-manual,mac-"52:54:00:4b:ab:cd" `
         --cloud-init cloud-init-mutipass.yaml

   > in Hyper-mode multipass have reserved network adapters (eth0, eth1, eth2) for static ip
     dev-workstation network adapter needs to be higher than reserved ones (here was set eth3)

   Track cloud-init progress:
     From Hyper-V console: wait for login prompt and watch boot messages  tail -f /var/log/cloud-init-output.log
     From SSH multipass@192.168.138.102 (When X11 kicks in):
       $ tail -f /var/log/cloud-init-output.log

   Wait until the message "Cloud-init ... finished" appears.

   Enable enhanced mode (Set-VM -VMName "dev-workstation" -EnhancedSessionTransportType HvSocket)
   Then reboot and test remote desktop connection (Hyper-v Enhanced Mode or RDC).

   RDC: 192.168.138.102:3390

   Verify and manage VM lifecycle:
     > multipass list
     > multipass stop dev-workstation --force
     > multipass start dev-workstation
     > multipass shell dev-workstation

 ----------------------------------------------------------------------------------------------------------------------------
                          VIRTUALBOX DEPLOY

   Launch Multipass only, network setup is after provisioning:
   (run as administrator from PowerShell in clout-init-multipass.yaml folder)

   > multipass launch 24.04 `
         --name dev-workstation `
         --memory 12G `
         --disk 20G `
         --cloud-init cloud-init-mutipass.yaml

   > in VBox-mode multipass have reserved network adapters (enp0s1, enp0s3) and also is VBox who define the name
     of network adapters, if VBox set a different network adapter name from enp0s8,
     adjust it in /etc/netplan/100-vbox-multipass.yaml

   Track cloud-init progress:
     Wait multipass launch times out.
     Then multipass shell dev-workstation 
     watch boot messages  tail -f /var/log/cloud-init-output.log

   Wait until the message "Cloud-init ... finished" appears.
   Leave ssh, go back to host console
   Then multipass stop dev-workstation --force
   Then multipass start dev-workstation
   Wait the start times out
   Confirm it´s up with multipass shell dev-workstation
   Configure static ip on host side

   Configure only internal gateway
     - VBoxManage hostonlyif ipconfig "VirtualBox Host-Only Ethernet Adapter #2" --ip 192.168.138.1 --netmask 255.255.255.0

   Add dev-workstation instance in VirtualBox UI
      - C:\ProgramData\Multipass\data\virtualbox\vault\instances\dev-workstation

   Attach internal gateway
      - VBoxManage modifyvm dev-workstation --nic2 hostonly --hostonlyadapter2 "VirtualBox Host-Only Ethernet Adapter #2"

   Verify:
     > VBoxManage showvminfo dev-workstation --machinereadable
     > multipass info dev-workstation

   Test remote desktop connection (RDC).
   VirtualBox: Just RDC. Because VBox and Hyper-V are in different layers of the OS.
      > Hyper-V owns dev-workstation and manage virtual ethernet, multipass is the proxy.
      > VBox just manage virtual ethernet, multipass manage dev-workstation in headless.

   RDC: 192.168.138.102:3390

   Verify and manage VM lifecycle:
     > multipass list
     > multipass stop dev-workstation --force
     > multipass start dev-workstation
     > multipass shell dev-workstation

 ---------------------------------------------------------------------------------------------------------------------------
                          CONNECTIVITY & FILE TRANSFER

   SSH access:
   If instance was recreated, clear old fingerprint in host:
     > del "%USERPROFILE%\.ssh\known_hosts"

   Update hosts mapping (optional):
     Edit C:\Windows\System32\drivers\etc\hosts.ics
     Example:
       192.168.138.102   dev-workstation

   File Transfer Tools:
     - Multipass mount:
         > multipass mount "C:\Users\YourUser\Projects" dev-workstation:/home/multipass/Shared
     - WinSCP:
         Protocol: SFTP
         Host: 192.168.138.102
         Username: multipass
         Key/File: (same SSH private key as authorized_keys)

 --------------------------------------------------------------------------------------------------------------------------------
                          CLEANUP & MAINTENANCE
 
    Removing instances:
     > multipass stop dev-workstation --force
     > multipass delete dev-workstation
     > multipass purge
     (Then remove stale host entries and fingerprints)

   VBox useful comands and procedures to wipe out stale instances:
      multipass stop dev-workstation --force
      net stop multipass
      manual deletion of "C:\ProgramData\Multipass\data\virtualbox\vault\instances\dev-workstation\
      net start multipass 
      VBoxManage unregistervm uuid --delete
      VBoxManage unregistervm "dev-workstation" --delete 
      VBoxManage list runningvms 
      multipass list  
      VBoxManage controlvm "dev-workstation" poweroff 
      VBoxManage showvminfo dev-workstation --machinereadable 
      VBoxManage list natnets 
      VBoxManage list vms --long | findstr /i "UUID" 
      VBoxManage list hostonlyifs
      VBoxManage startvm dev-workstation --type headless
      VBoxManage.exe showhdinfo "C:\ProgramData\Multipass\data\virtualbox\vault\instances\dev-workstation\vdi-name.vdi"    
      Uninstall multipass to clear cached data

      instance that get stuck in unknown state:
         > stop multipass daemon and all VBox services
         > manual deletion of dev-workstation folder
         > reboot host
         > provision again with multipass launch

 ------------------------------------------------------------------------------------------------------------------------------
                          GNOME BLACK SCREEN (due to XRDP login idle)              

      Black screen due to XRDP Login idle 
        > sudo pkill -u multipass gnome-session-binary
        > sudo pkill -u multipass Xorg
        > sudo rm -rf /tmp/.X10-lock /tmp/.X11-unix/X10
