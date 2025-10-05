### **Language Switch: [简体中文](README.md)**
>[!NOTE]
>DISCLAIMER
>This project is intended solely for academic research and technical exchange. Any direct, indirect, or consequential damages arising from the use of this project are the sole responsibility of the user. The project and its contributors shall not be held liable for any legal, financial, or other consequences resulting from its use.

# Campus Network Solution for Dormitories at Guangdong University of Education
This solution addresses common issues with the campus network in dormitories at Guangdong University of Education, including network lag, insufficient bandwidth, limited number of connectable devices, and inaccessibility to certain websites. Its core approach is to enhance network fluency and flexibility by custom-compiling OpenWRT firmware and optimizing network settings to bypass campus network monitoring. It should be noted that the concepts and methods of this solution are also applicable to campus networks of other universities, and are provided for reference and technical exchange only.


## I. Background and Purpose of the Solution
During my high school years, I was restricted by the Shunde District Education Network for a long time, which gave me a deep understanding of how network restrictions affect learning and daily life. After enrolling in Guangdong University of Education, I found similar issues with the dormitory campus network: video lag during peak hours, frequent disconnections when multiple devices are connected, and inaccessibility to some external websites required for learning. These problems directly impact online course participation, information retrieval, and daily network use.

Currently, campus network management and control in domestic universities have formed a mature technical chain. On the front end, terminal detection is implemented through multi-dimensional methods such as User-Agent feature recognition, dynamic timestamp verification, and IPID sequence tracking. On the back end, access blocking is carried out via IP address pollution, DNS resolution hijacking, and HTTP response redirection to the campus 404 page. This closed-loop "detection-response" mechanism is the main technical barrier to the free use of the campus network.

Based on my basic understanding of computer compilation principles and the modular architecture of OpenWRT firmware, I systematically deconstructed the campus network management logic through technical approaches including custom firmware compilation, protocol stack parameter optimization, and application-layer proxy strategy configuration. Ultimately, this practical and scalable dormitory network optimization solution was developed.


## II. QA and Disclaimer
<details>
<summary>📌 Click to expand: Frequently Asked Questions (QA)</summary>

### 1. Why is this technology open-sourced? Won’t the campus network take measures to fix potential vulnerabilities?
- Campus network operation and maintenance engineers pay far more attention to technical developments than ordinary students, so there is no "lack of awareness" about such technologies. The core reason for not fixing it is that **it is technically impossible to fix effectively**.  
- All detection methods used by the campus network are based on patent applications and relevant national standard documents issued by the State Intellectual Property Office of the People's Republic of China. The anti-detection technology in this solution fully complies with national standards. Its principle is to aggregate all network devices in the dormitory into a single unit for internet access, appearing as "one IP, one MAC, one UA". This is identical to the characteristics of a single device accessing the network normally, so detection tools cannot distinguish between aggregated devices and single devices, and it is even impossible to block terminals that access the network normally.  
- From the perspective of operation and maintenance costs, campus network maintenance is constrained by both equipment costs and maintenance costs. Adding new detection methods will interfere with the accuracy of existing detection and require more hardware resources. Blindly increasing detection rules will significantly slow down campus server performance, which in turn affects the network experience of all teachers and students and normal teaching progress.  
- From the perspective of processes and practical operations, operation and maintenance personnel need to report to superiors at all levels and coordinate testing from multiple parties to adjust detection strategies. Additionally, it is difficult to accurately distinguish between "normal devices" and "aggregated devices". In contrast, this solution only involves technical configuration and has no additional process obstacles.

### 2. Are there any security or legal risks in deploying this project?
There are **no security or legal risks**, and it can be deployed with confidence.  

The anti-detection logic of this project is summarized based on long-term personal technical exploration. It does not involve modifying or damaging campus network equipment throughout the process. All configurations comply with the requirements of national standard documents and do not exceed the basic usage framework of the campus network. There is no need to worry about legal or security issues.

### 3. How to resolve other issues not mentioned in the solution?
If you encounter difficult problems not covered in this solution, please leave a message in the **issues** section of the project’s GitHub repository. I will reply and assist in resolving the issue promptly after seeing the message.
</details>

<details>
<summary>⚠️ Click to expand: Disclaimer</summary>

1. This project is developed and shared based on open-source protocols (such as the GPL protocol). All codes and configuration files are for **technical exchange and learning research only**, and shall not be used for any commercial purposes or illegal activities.  
2. Users must ensure that this solution is deployed in compliance with the campus network usage regulations of Guangdong University of Education and relevant national laws and regulations. The project author shall not bear any joint liability for campus disciplinary actions, legal liabilities, etc. caused by non-compliant use; such responsibilities shall be borne solely by the user.  
3. This project only provides technical concepts and basic configurations. There may be differences in network environments and device models across different dormitories. If problems such as network instability or device malfunctions occur during deployment, the project author shall not be responsible for hardware maintenance or network restoration. It is recommended that users have basic computer network knowledge before performing operations.  
4. Any individual or organization is prohibited from tampering with or repackaging the project’s code and documents for commercial dissemination or profit without the permission of the project author. Violators will be held accountable for their relevant legal responsibilities.  
5. The project author has the right to update or adjust the content of the solution based on technological development or changes in campus network policies. The author does not guarantee that the solution will be applicable to all scenarios in the long term. It is recommended that users pay attention to the latest updates of the project.
</details>


## III. Preparation Stage
### 1. Hardware Equipment
- **Main Router Device**: An x86-64 architecture device with dual network ports (such as an industrial computer or mini host) is required, or other motherboards that support flashing the OpenWRT system (such as ARM architecture development boards). Ensure it can connect to both the campus network input line and subordinate devices simultaneously. This solution uses the X86-64 architecture for demonstration.
- **Wireless Access Point (AP)**: Prepare one router with wireless functionality as an AP to expand WiFi coverage. It is recommended to choose a model that supports the 802.11ac protocol or above to ensure wireless speed.

### 2. Compilation Environment
- **Operating System**: Ubuntu (version 20.04/22.04 LTS is recommended) or other Linux distributions must be used. This solution uses Ubuntu for operation demonstration.
- **Configuration Requirements**: At least 4GB of memory and 20GB of free hard disk space are recommended. The processor should support multi-threading to accelerate the compilation speed.

### 3. Tool Software
- **PE System**: Used to boot the device and write firmware. It is recommended to use pure-version PE tools such as WeiPE or Dabaicai.
- **Disk Writing Tool**: [DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download) (download link), used to write the compiled OpenWRT image to the hard disk.
- If you cannot download the above tools, alternative tools can be downloaded from the project’s releases section.

### 4. Other Preparations
- Campus Network Account and Password: Used to authenticate and access the campus network.
- USB Flash Drive (8GB or larger): Used to create a PE boot disk and store firmware files.
- Network Cables (at least two): One for connecting the campus network interface to the main router, and the other for connecting the main router to the AP device.

### 5. Instructions for Users Unfamiliar with Operations
If you cannot complete the subsequent operations independently, you can directly use the following commands to download the pre-configured files for compilation, or go directly to the releases section to download the already compiled img file (you can skip directly to Step 4-16):
```
git clone https://github.com/liang1481624299/gdei_openwrt.git
cd gdei_openwrt
make kernel_menuconfig -j$(nproc) V=cs
```


## IV. Start Compilation
### 1. Clone OpenWRT Source Code
Clone the OpenWRT 23.05 version branch (the new version may have module incompatibility issues). Due to network reasons, some users may need to use a proxy tool to speed up the process:
```bash
git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
```

### 2. Copy the Source Code
To avoid affecting the original code due to operational errors, first make a copy and perform compilation in the copied version:
```bash
cp -r openwrt openwrt_2
```

### 3. Enter the Copied Folder
```bash
cd openwrt_2
```

### 4. Update Software Package Sources
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 5. Clone Core Plugin Source Code
Clone the core plugin source code used to bypass campus network detection:
- UA2F: Used to bypass User-Agent feature detection
- rkp-ipid: Used to modify the IPID sequence to avoid tracking
```bash
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
```

### 6. Clone Practical Tools and Plugin Source Code
Clone the source code of practical tools and plugins with functions such as network acceleration and music unblocking:
```bash
# Fix Tailscale compilation conflicts
sed -i '/\/etc\/init\.d\/tailscale/d;/\/etc\/config\/tailscale/d;' feeds/packages/net/tailscale/Makefile

# Clone necessary plugins
git clone https://github.com/asvow/luci-app-tailscale package/luci-app-tailscale  # Tailscale intranet penetration
git clone https://github.com/jerrykuku/luci-theme-argon.git package/luci-theme-argon  # Argon theme
git clone https://github.com/jerrykuku/luci-app-argon-config.git package/luci-app-argon-config  # Theme configuration tool
git clone https://github.com/lucikap/luci-app-ua2f.git package/luci-app-ua2f  # UA2F graphical configuration

# Clone NetEase Cloud Music unblocking plugin
cd package
git clone https://github.com/UnblockNeteaseMusic/luci-app-unblockneteasemusic.git
cd ..

# Install Turbo ACC network acceleration module
curl -sSL https://raw.githubusercontent.com/chenmozhijin/turboacc/luci/add_turboacc.sh -o add_turboacc.sh && bash add_turboacc.sh
```

### 7. Update Software Package Sources Again
Update the software package sources again to ensure the newly added plugins are recognized:
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 8. Write MD5 Verification Value
Write the MD5 verification value for the OpenWRT 23.05 version (used to bypass version detection):
```bash
nano vermagic  # Create and edit the vermagic file
```
Enter the following MD5 value in the opened editor:
```
47964456485559d992fe6f536131fc64
```
Press `Ctrl + O` to save, then press `Ctrl + X` to exit the editor.

### 9. Modify Kernel Compilation Parameters
Modify the kernel compilation parameters to allow the system to read the custom MD5 value:
```bash
nano ./include/kernel-defaults.mk  # Edit the kernel default configuration file
```
Press `Ctrl + /`, enter `121`, and press Enter to quickly locate line 121. Comment out the original code and replace it with the configuration for reading the custom MD5:
```bash
# Comment out the original line
#grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | $(MKHASH) md5 > $(LINUX_DIR)/.vermagic
# New line: Read the custom vermagic file
cp $(TOPDIR)/vermagic $(LINUX_DIR)/.vermagic
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/1.png)

### 10. Modify the Kernel Makefile
Similarly, modify the kernel Makefile to ensure MD5 verification compatibility:
```bash
nano ./package/kernel/linux/Makefile  # Edit the kernel compilation configuration file
```
Press `Ctrl + /`, enter `29`, and press Enter to locate line 29. Modify it to the following content (note to retain the spaces at the beginning of the line):
```bash
# Note: The following two lines require retaining leading spaces
#STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
# New line: Use the custom vermagic
STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

### 11. Launch Compilation Configuration
```bash
make menuconfig
```

### 12. Configure the Target System
After entering `menuconfig`, follow these operation rules: Double-click ESC to return to the previous page; use the left and right arrow keys on the keyboard to select operations; use the up and down arrow keys to scroll. Set `Target System` to `x86` and `Subtarget` to `x86_64`.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/3.png)

### 13. Configure Target Image Partitions
Enter `Target Images` and only modify `Kernel partition size (in MiB)` and `Root filesystem partition size (in MiB)`. Adjust the specific values according to your own situation:
- `Kernel partition`: The partition for storing the kernel and boot files
- `Root filesystem partition`: The Root partition of OpenWRT

**Note**: If the compiled firmware is used to flash a hardware router, the values here should not be too large to avoid filling up the router’s flash memory. You can refer to the configuration in the following image.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/4.png)

### 14. Return to the Main Configuration Interface
After completing the operation, press ESC to return to the previous page until the top title bar displays `OpenWrt Configuration`. If you accidentally exit `make menuconfig` completely, you can repeat Step 11 to restart the configuration.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

### 15. Add Modules and Software Packages
#### (1) Add the ipid Module
1. Find and enter `Kernel modules -> Other modules`
2. Press Y to select `kmod-rkp-ipid`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/6.png)

#### (2) Return to the Main Configuration Interface and Install VPN (for Remote Access to Campus Services)
1. Press ESC twice to return to `OpenWrt Configuration`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)
2. Find and enter `Network -> VPN`
3. Press Y to select `openvpn-easy-rsa`, `openvpn-openssl`, and `tailscale`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)

#### (3) Add the UA Modification Module
1. Find and enter `Network -> Routing and Redirection`
2. Press Y to select `ua2f`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/7.png)

#### (4) Add Required Firewall Modules
1. Find and enter `Network -> Firewall`
2. Press Y to select `iptables-mod-conntrack-extra`, `iptables-mod-filter`, `iptables-mod-ipopt`, `iptables-nft`, and `iptables-mod-u32`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/8.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/9.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/10.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/11.png)
3. Return to `OpenWRT Configuration`
4. Find and enter `Network`
5. Press Y to select `ipset`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/12.png)
6. Save the configuration first to prevent accidents
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/13.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/14.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/15.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/16.png)

#### (5) Add the LuCI Network Management Interface
1. Find and enter `LuCI -> Collections`
2. Press Y to select `luci`
3. Return to `LuCI`
4. Find and enter `Modules`
5. Press Y to select `luci-compat`
6. Find and enter `Translations`
7. You can select the web front-end translation language as needed; all languages are selected here
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/18.png)
8. Return to `LuCI`
9. Find and enter `Applications`
10. Press Y to select `luci-app-argon-config`, `luci-app-ttyd`, `luci-app-tailscale`, `luci-app-turboacc`, `luci-app-ua2f`, `luci-app-unblockneteasemusic`, `luci-app-upnp`, and `luci-app-wol`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)
11. Use the right arrow key on the keyboard to move the cursor to `Save` and save (follow the same steps as above)
12. Double-click ESC to exit until returning to the terminal command line
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/17.png)

### 16. Configure Kernel Compilation Parameters (Ensure the Network Environment Can Access External Resources Normally)
Enter the following command to start kernel compilation configuration and wait for the selection page to appear again:
```bash
make kernel_menuconfig -j$(nproc) V=cs
```

**Note**: The compilation waiting time varies depending on hardware performance; devices with stronger performance will have faster compilation speeds.

After executing the command, two scenarios may occur:
- Some users will directly enter the `Linux/x86 5.15.189 Kernel Configuration` interface, requiring no additional operations.
- Some users will enter the `General Setup` interface (as shown in the figure below) and need to press ESC twice to return to the `Linux/x86 5.15.189 Kernel Configuration` interface.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/20.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/21.png)

#### (1) Enable the Firewall Kernel for UA2F
1. Find and enter `Networking support -> Networking options`
2. First press Y to select `Network packet filtering framework (Netfilter)`, then press Enter to access this option
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/22.png)
3. Find and enter `Core Netfilter Configuration`
4. Press Y to select `Netfilter NFNETLINK interface`, `Netfilter LOG over NFNETLINK interface`, `Netfilter connection tracking support`, `Connection tracking netlink interface`, and `NFQUEUE and NFLOG integration with Connection Tracking`
5. Save the configuration and exit.

### 17. Pre-Download Files Required for Compilation (Ensure the Network Environment Can Access External Resources Normally)
```bash
make download -j$(nproc) V=cs
```

**Note**: If an error occurs during the process, there is no need to panic. After all downloads are completed, you can execute this command again to re-download the files that reported errors.

### 18. Compile the OpenWRT System
1. Enter the following command to start compilation:
```bash
make -j$(nproc) V=cs
```

**Note**: During compilation, ensure the device has more than 4GB of available running memory. When function selection pop-ups appear, select "y" for all options to ensure OpenWRT compatibility with the device. If an Error occurs, you can first execute the following commands to clean the environment, then re-execute the compilation command:
```bash
make clean
make dirclean  # More thorough cleaning, which will delete downloaded packages and configurations
```

### 19. Locate the Compilation Result
After compilation is completed, the image files (in .img format) will be stored in the directory `./bin/target/x86/64/` (the "x86" and "64" in the path correspond to the architecture selected during compilation).
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/23.png)


## V. Install OpenWRT (Demonstrated Based on the x86_64 Platform)
### 1. Enter the PE System
Insert the prepared PE boot disk into the device and enter the PE system. Ensure the hard disk where OpenWRT will be installed has no partitions (back up hard disk data in advance).

### 2. Write the Image File
Open the DiskImage software, select the target hard disk and the compiled OpenWRT image file, and perform the writing operation.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/24.png)

### 3. Restart the Device
After the writing is completed, remove the PE boot disk and restart the device.


## VI. Configure the OpenWRT Soft Router
### 1. Connect Devices
Connect the devices as shown in the figure below: Campus network interface → OpenWRT main router WAN port; OpenWRT main router LAN port → Computer/AP device.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/25.png)

**Note**: If you are unsure about the OpenWRT WAN port, you can first connect the computer to any network port. If the computer obtains an IP address in the `192.168.1.x` segment, that port is the LAN port.

### 2. Access the Management Panel
Enter `192.168.1.1` in the computer browser to access the OpenWRT management panel (no password by default).
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/26.png)

### 3. Set a Management Password
After accessing the panel, prioritize setting an administrator password in the "System" module to ensure device security.

### 4. Configure the Anti-Detection Auto-Start Script
1. Find `System -> Startup` in the left menu and select `Local Startup Script`.
2. Paste the following script content and click the "Save" button in the lower right corner.
```bash
# Auto-start UA2F on boot
service ua2f start
service ua2f enable

# Firewall configuration
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53

# Prevent IPID detection
iptables -t mangle -N IPID_MOD
iptables -t mangle -A FORWARD -j IPID_MOD
iptables -t mangle -A OUTPUT -j IPID_MOD
iptables -t mangle -A IPID_MOD -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A IPID_MOD -d 127.0.0.0/8 -j RETURN
# The campus LAN of this university is a Class A network, so this line is commented out; whether to comment it out depends on the intranet type of the campus network you are using
# iptables -t mangle -A IPID_MOD -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A IPID_MOD -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A IPID_MOD -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A IPID_MOD -d 255.0.0.0/8 -j RETURN
iptables -t mangle -A IPID_MOD -j MARK --set-xmark 0x10/0x10

# Prevent clock offset detection
iptables -t nat -N ntp_force_local
iptables -t nat -I PREROUTING -p udp --dport 123 -j ntp_force_local
iptables -t nat -A ntp_force_local -d 0.0.0.0/8 -j RETURN
iptables -t nat -A ntp_force_local -d 127.0.0.0/8 -j RETURN
iptables -t nat -A ntp_force_local -d 192.168.0.0/16 -j RETURN
iptables -t nat -A ntp_force_local -s 192.168.0.0/16 -j DNAT --to-destination 192.168.1.1

# Modify TTL value via iptables
iptables -t mangle -A POSTROUTING -j TTL --ttl-set 64

# iptables blocks AC from Flash detection
iptables -I FORWARD -p tcp --sport 80 --tcp-flags ACK ACK -m string --algobm --string " src=\"http://1.1.1." -j DROP
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/27.png)

### 5. Configure Time Synchronization
#### (1) Modify the Time Zone
1. Find `System -> System` in the left menu.
2. Change the time zone to `Asia/Shanghai` and click "Save & Apply".
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/28.png)

#### (2) Set the NTP Server
In the "Time Synchronization" module, enter the following NTP server addresses (used to provide time synchronization for downstream devices):
```bash
ntp.aliyun.com          # Alibaba Cloud
time1.cloud.tencent.com  # Tencent Cloud
time.ustc.edu.cn        # University of Science and Technology of China
cn.pool.ntp.org        # Global NTP Pool
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/29.png)

### 6. Modify the opkg Software Package Configuration (If update failed)
1. Find `System -> Software` in the left menu and select `Configure opkg`.
2. Change "23.05-SNAPSHOT" in `/etc/opkg/distfeeds.conf` to "23.05". The modified content is as follows:
```bash
src/gz openwrt_core https://downloads.openwrt.org/releases/23.05/targets/x86/64/packages
src/gz openwrt_base https://downloads.openwrt.org/releases/23.05/packages/x86_64/base
src/gz openwrt_luci https://downloads.openwrt.org/releases/23.05/packages/x86_64/luci
src/gz openwrt_packages https://downloads.openwrt.org/releases/23.05/packages/x86_64/packages
src/gz openwrt_routing https://downloads.openwrt.org/releases/23.05/packages/x86_64/routing
src/gz openwrt_telephony https://downloads.openwrt.org/releases/23.05/packages/x86_64/telephony
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/30.png)


## VII. Final Device Connection and Testing
1. Use a network cable to connect the WAN port of the OpenWRT main router to the campus network interface.
2. Connect the WAN port of the AP router to the LAN port of the OpenWRT main router.
3. Access the AP router’s management interface and set it to "Bridge Mode" or "Wired Repeater Mode".
4. Configure the WiFi name and password of the AP router.
5. Connect all devices to WiFi or a wired network and test whether the network works normally.

![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/25.png)
