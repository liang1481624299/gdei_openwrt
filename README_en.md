### **Language Switch: [简体中文](README.md)**

> [!NOTE]
> This project is for academic research and technical exchange purposes only. Any direct or indirect consequences arising from the use of this project (including but not limited to data loss, system failure, legal disputes, etc.) shall be borne solely by the user. This project and its authors shall not bear any joint or supplementary liability for any damages or liabilities arising therefrom.

# Campus Network Solution for Dormitories at Guangdong University of Education
This solution addresses common issues with the campus network in dormitories at Guangdong University of Education, such as network lag, insufficient bandwidth, limited number of connectable devices, and inability to access certain websites. The core approach is to bypass campus network monitoring by custom-compiling the OpenWRT firmware and optimizing network settings, thereby enhancing the smoothness and flexibility of network usage. It should be noted that the ideas and methods of this solution are also applicable to campus networks of other universities and are for reference and technical exchange only.


## I. Background and Purpose of the Solution
During my high school years, I was long restricted by the Shunde District Education Network and deeply experienced the impact of network limitations on learning and daily life. After enrolling in Guangdong University of Education, I found that the dormitory campus network had similar problems: video lag during peak hours, frequent disconnections when multiple devices were connected to the network, and inability to access certain external websites required for learning. These issues directly affected online courses, information retrieval, and daily network usage.

Currently, campus network management in domestic universities has formed a mature technical chain. On the front end, terminal detection is implemented through multi-dimensional methods such as User-Agent feature recognition, timestamp dynamic verification, and IPID sequence tracking. On the back end, access blocking is carried out by means of IP address pollution, DNS resolution hijacking, and HTTP response redirection to the campus intranet 404 page. This "detection-response" mechanism loop is the main technical barrier to the free use of the campus network.

Based on the basic knowledge of computer compilation principles and the understanding of the modular architecture of the OpenWRT firmware, I systematically deconstructed the campus network management logic through technical approaches such as custom firmware compilation, protocol stack parameter optimization, and application-layer proxy policy configuration. Finally, this practical and scalable dormitory network optimization solution was formed.


## II. QA and Disclaimer
<details>
<summary>📌 Click to expand: Frequently Asked Questions (QA)</summary>

### 1. Why is this technology open-sourced? Won't the campus network take measures to fix network vulnerabilities?
- Campus network operation and maintenance engineers pay much more attention to technical trends than ordinary students, so there is no such situation as "not knowing". The core reason for not fixing is that **it is technically impossible to fix effectively**.  
- Various detection methods of the campus network are formulated based on patent applications and relevant national standard documents issued by the State Intellectual Property Office of the People's Republic of China. The anti-detection technology of this solution fully complies with the national standard requirements. The principle is to aggregate all network devices in the dormitory into "one IP, one MAC, one UA" for internet access, which is completely consistent with the characteristics of a single device accessing the internet normally. The detection device cannot distinguish between the aggregated device and a single device, and it is even more impossible to block the terminal that accesses the internet normally.  
- From the perspective of operation and maintenance costs, campus network maintenance is restricted by both equipment costs and maintenance costs: adding new detection methods will interfere with the accuracy of the original detection and require more hardware resources to be invested; if detection rules are added blindly, the performance of the campus server will be significantly slowed down, which will affect the internet experience of all teachers and students and the normal teaching progress.  
- From the perspective of processes and practical operations, operation and maintenance personnel need to ask superiors for instructions at all levels and coordinate testing in many ways to adjust the detection strategy, and it is difficult to accurately distinguish between "normal devices" and "aggregated devices". In contrast, this solution only involves technical configuration and has no additional process resistance.

### 2. Will deploying this project pose security risks or legal risks?
**There are no security risks or legal risks**, so you can deploy it with confidence.  

The anti-detection logic of this project is summarized based on the author's long-term technical exploration. It does not involve the modification or damage of campus network equipment throughout the process. All configurations comply with the requirements of national standard documents and do not break through the basic usage framework of the campus network. There is no need to worry about legal or security issues.

### 3. How to solve other problems not mentioned?
If you encounter difficult problems not covered in the solution, you are welcome to leave a message in the **issues** section of the project's GitHub repository. I will reply and assist in solving the problem in a timely manner after seeing it.
</details>

<details>
<summary>⚠️ Click to expand: Disclaimer</summary>

1. This project is developed and shared based on open-source protocols (such as the GPL protocol). All codes and configuration files are only used for **technical exchange and learning research**, and shall not be used for any commercial purposes or illegal activities.  
2. Users shall ensure that this solution is deployed in compliance with the campus network usage regulations of Guangdong University of Education and relevant national laws and regulations. The campus sanctions, legal liabilities, etc. caused by illegal use shall be borne by the user themselves, and the project author shall not bear any joint liability or supplementary liability.  
3. This project only provides technical ideas and basic configurations. The network environment and equipment models of different dormitories may vary. If problems such as network instability and equipment failure occur during the deployment process, the project author shall not bear the responsibility for hardware maintenance or network restoration. It is recommended that users have basic computer network knowledge before operating.  
4. Any individual or organization is prohibited from tampering with or repackaging the code and documents of this project for commercial dissemination or profit without the permission of the project author. Violators will be held accountable for their relevant legal responsibilities.  
5. The project author has the right to update or adjust the content of the solution according to the development of technology or changes in campus network policies. The solution is not guaranteed to be applicable to all scenarios for a long time. It is recommended that users pay attention to the latest developments of the project.
</details>


## III. Preparation Stage
### 1. Hardware Equipment
- **Main Router Device**: An x86-64 architecture device with dual network ports (such as an industrial computer, a mini host) is required, or other motherboards that support flashing the OpenWRT system (such as an ARM architecture development board) to ensure that it can connect to both the campus network incoming line and lower-level devices at the same time. This solution uses the X86-64 architecture as an example for demonstration.
- **Wireless Access Point (AP)**: Prepare a router with wireless function as an AP to expand the WiFi coverage. It is recommended to use a model that supports the 802.11ac protocol or above to ensure the wireless rate.

### 2. Compilation Environment
- **Operating System**: Ubuntu (recommended 20.04/22.04 LTS version) or other Linux distributions must be used. This solution uses Ubuntu as an example for operation demonstration.
- **Configuration Requirements**: It is recommended to have at least 4GB of memory, 20GB of free hard disk space, and a processor that supports multi-threading to speed up the compilation process.

### 3. Tool Software
- **PE System**: Used to boot the device and write the firmware. It is recommended to use pure version PE tools such as WeiPE or Dabaicai.
- **Disk Writing Tool**: [DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download) (download link), used to write the compiled OpenWRT image to the hard disk.
- If the above tools cannot be downloaded, you can download the backup tools in the releases of the project.

### 4. Other Preparations
- Campus network account and password: Used to verify and access the campus network.
- USB flash drive (8GB or more): Used to make a PE boot disk and store firmware files.
- Network cables (at least two): Used to connect the campus network interface to the main router, and the main router to the AP device respectively.

### 5. Instructions for Students Who Are Not Proficient in Operations
If you cannot complete the subsequent operations independently, you can directly use the following command to download the configured files for compilation, or directly go to the releases to download the compiled img file (you can skip directly to Step 4-16):
```
git clone https://github.com/liang1481624299/gdei_openwrt.git
cd gdei_openwrt
make kernel_menuconfig -j$(nproc) V=cs
```


## IV. Start Compilation
### 1. Clone the OpenWRT Source Code
Clone the OpenWRT 23.05 version branch (the new version may have module incompatibility issues). Due to network reasons, some users need to use proxy tools to speed up:
```bash
git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
```

### 2. Copy the Source Code Copy
To avoid affecting the original code due to operation errors, first copy a copy and perform compilation in the copy:
```bash
cp -r openwrt openwrt_2
```

### 3. Enter the Copy Folder
```bash
cd openwrt_2
```

### 4. Update the Package Source
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 5. Clone the Core Plugin Source Code
Clone the core plugin source code used to bypass campus network detection:
- UA2F: Used to break through User-Agent feature detection
- rkp-ipid: Used to modify the IPID sequence to avoid being tracked
```bash
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
```

### 6. Clone the Practical Tool and Plugin Source Code
Clone the practical tool and plugin source code with functions such as network acceleration and music unblocking:
```bash
# Fix Tailscale compilation conflict
sed -i '/\/etc\/init\.d\/tailscale/d;/\/etc\/config\/tailscale/d;' feeds/packages/net/tailscale/Makefile

# Clone necessary plugins
git clone https://github.com/asvow/luci-app-tailscale package/luci-app-tailscale  # Tailscale intranet penetration
git clone https://github.com/jerrykuku/luci-theme-argon.git package/luci-theme-argon  # Argon theme
git clone https://github.com/jerrykuku/luci-app-argon-config.git package/luci-app-argon-config  # Theme configuration tool
git clone https://github.com/lucikap/luci-app-ua2f.git package/luci-app-ua2f  # UA2F graphical configuration

# Clone the NetEase Cloud Music unblocking plugin
cd package
git clone https://github.com/UnblockNeteaseMusic/luci-app-unblockneteasemusic.git
cd ..

# Install the Turbo ACC network acceleration module
curl -sSL https://raw.githubusercontent.com/chenmozhijin/turboacc/luci/add_turboacc.sh -o add_turboacc.sh && bash add_turboacc.sh

# Install vlmcds to activate Windows Office in the local area network
git clone https://github.com/ysc3839/openwrt-vlmcsd.git package/vlmcsd
```

### 7. Update the Package Source Again
Update the package source again to ensure that the newly added plugins are recognized:
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 8. Write the MD5 Verification Value
Write the MD5 verification value of the OpenWRT 23.05 version (used to bypass version detection):
```bash
nano vermagic  # Create and edit the vermagic file
```
Enter the following MD5 value in the opened editor:
```
47964456485559d992fe6f536131fc64
```
Press `Ctrl + O` to save, and then press `Ctrl + X` to exit the editor.

### 9. Modify the Kernel Compilation Parameters
Modify the kernel compilation parameters to make the system read the custom MD5 value:
```bash
nano ./include/kernel-defaults.mk  # Edit the kernel default configuration file
```
Press `Ctrl + /`, enter `121` and press Enter to quickly locate line 121. Comment out the original code and replace it with the configuration for reading the custom MD5:
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
Press `Ctrl + /`, enter `29` and press Enter to locate line 29. Modify it to the following content (note to keep the spaces before the line):
```bash
# Note: The following two lines need to keep the leading spaces
#STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
# New line: Use the custom vermagic
STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

### 11. Start the Compilation Configuration
```bash
make menuconfig
```

### 12. Configure the Target System
After entering `menuconfig`, the operation rules are as follows: double-click ESC to return to the previous page, use the left and right arrow keys on the keyboard to select operations, and use the up and down arrow keys to scroll. Set `Target System` to `x86` and `Subtarget` to `x86_64`.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/3.png)

### 13. Configure the Target Image Partition
Enter `Target Images` and only need to modify `Kernel partition size (in MiB)` and `Root filesystem partition size (in MiB)`. The specific values are adjusted according to your own situation:
- `Kernel partition`: The partition for storing the kernel and boot files
- `Root filesystem partition`: The Root partition of OpenWRT

**Tip**: If the compiled firmware is used to flash the hard router, the value here should not be too large to avoid bursting the hard router's flash memory. You can refer to the following picture for configuration.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/4.png)

### 14. Return to the Main Configuration Interface
After completing the operation, press ESC to return to the previous page until the top title bar displays `OpenWrt Configuration`. If you accidentally exit `make menuconfig` completely, you can repeat Step 11 to restart the configuration.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

### 15. Add Modules and Packages
#### (1) Add the ipid Module
1. Find and enter `Kernel modules -> Other modules`
2. Press Y to select `kmod-rkp-ipid`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/6.png)

#### (2) Return to the Main Configuration Interface and Install VPN (Used for Remote Access to Campus Intranet Services)
1. Press ESC twice to return to `OpenWrt Configuration`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)
2. Find and enter `Network -> VPN`
3. Press Y to select `openvpn-easy-rsa`, `openvpn-openssl`, and `tailscale`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)

#### (3) Add the UA Modification Module
1. Find and enter `Network -> Routing and Redirection`
2. Press Y to select `ua2f`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/7.png)

#### (4) Add the Required Firewall Modules
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
7. You can select the web front-end translation language according to your needs. Here, select all to check
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/18.png)
8. Return to `LuCI`
9. Find and enter `Applications`
10. Press Y to select `luci-app-argon-config`, `luci-app-ttyd`, `luci-app-tailscale`, `luci-app-turboacc`, `luci-app-ua2f`, `luci-app-unblockneteasemusic`, `luci-app-upnp`, and `luci-app-wol`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)
11. Press the right arrow key on the keyboard to move the cursor to `Save` to save (the operation steps are the same as above)
12. Double-click ESC to exit until returning to the terminal command line
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/17.png)

### 16. Configure the Kernel Compilation Parameters (Ensure That the Network Environment Can Access External Resources Normally)
Enter the following command to start the kernel compilation configuration and wait for the selection page to appear again:
```bash
make kernel_menuconfig -j$(nproc) V=cs
```

**Tip**: The compilation waiting time varies depending on the hardware performance. The stronger the performance of the device, the faster the compilation speed.

After executing the command, two situations will occur:
- Some users directly enter the `Linux/x86 5.15.189 Kernel Configuration` interface, and no additional operations are required.
- Some users enter the `General Setup` interface (as shown in the figure below), and need to press ESC twice to return to the `Linux/x86 5.15.189 Kernel Configuration` interface.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/20.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/21.png)

#### (1) Enable the Firewall Kernel of UA2F
1. Find and enter `Networking support -> Networking options`
2. First press Y to select `Network packet filtering framework (Netfilter)`, then press Enter to enter this option
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/22.png)
3. Find and enter `Core Netfilter Configuration`
4. Press Y to select `Netfilter NFNETLINK interface`, `Netfilter LOG over NFNETLINK interface`, `Netfilter connection tracking support`, `Connection tracking netlink interface`, and `NFQUEUE and NFLOG integration with Connection Tracking`
5. Save the configuration and exit.

### 17. Pre-download the Files Required for Compilation (Ensure That the Network Environment Can Access External Resources Normally)
```bash
make download -j$(nproc) V=cs
```

**Tip**: If an error occurs during the process, there is no need to panic. After all downloads are completed, you can execute this command again to re-download the files with errors.

### 18. Compile the OpenWRT System
1. Enter the following command to start compilation:
```bash
make -j$(nproc) V=cs
```

**Tip**: During compilation, ensure that the available running memory of the device is greater than 4GB; select "y" for all when functional options pop up to ensure the compatibility of OpenWRT with the device. If an Error occurs, you can first execute the following command to clean the environment, and then re-execute the compilation command:
```bash
make clean
make dirclean  # More thorough cleaning, which will delete the downloaded packages and configurations
```

### 19. Find the Compilation Result
After the compilation is completed, the image file (in .img format) will be stored in the `./bin/target/x86/64/` directory (the "x86" and "64" in the path are consistent with the architecture selected during compilation).
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/23.png)


## V. Install OpenWRT (Demonstrated Based on the x86_64 Platform)
### 1. Enter the PE System
Insert the made PE boot disk into the device, enter the PE system; ensure that the hard disk where OpenWRT is to be installed has no partitions (the hard disk data needs to be backed up in advance).

### 2. Write the Image File
Open the DiskImage software, select the target hard disk and the compiled OpenWRT image file, and perform the writing operation.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/24.png)

### 3. Restart the Device
After the writing is completed, pull out the PE boot disk and restart the device.


## VI. Configure the OpenWRT Soft Router
### 1. Connect the Device
Connect the devices as shown in the figure below: Campus network interface → OpenWRT main router WAN port, OpenWRT main router LAN port → computer/AP device.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/25.png)

**Tip**: If you are not sure about the WAN port of OpenWRT, you can first connect the computer to any network port; if the computer obtains an IP in the `192.168.1.x` network segment, this network port is the LAN port.

### 2. Enter the Management Panel
Enter `192.168.1.1` in the computer browser to enter the OpenWRT management panel (there is no password by default).
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/26.png)

### 3. Set the Management Password
After entering the panel, first set the administrator password in the "System" module to ensure device security.

### 4. Configure the Anti-detection Auto-start Script
1. Find `System -> Startup` in the left menu and select `Local Startup Script`.
2. Paste the following script content and click the "Save" button in the lower right corner.
```bash
# Start UA2F automatically on boot
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
# The local area network of our school is a Class A network, so this line is commented out; whether to comment it out depends on the type of the campus network intranet where you are located
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

# Modify the TTL value through iptables
iptables -t mangle -A POSTROUTING -j TTL --ttl-set 64

# iptables rejects AC from conducting Flash detection
iptables -I FORWARD -p tcp --sport 80 --tcp-flags ACK ACK -m string --algobm --string " src=\"http://1.1.1." -j DROP
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/27.png)

### 5. Configure Time Synchronization
#### (1) Modify the Time Zone
1. Find `System -> System` in the left menu.
2. Modify the time zone to `Asia/Shanghai` and click "Save & Apply".
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/28.png)

#### (2) Set the NTP Server
In the "Time Synchronization" module, enter the following NTP server addresses (used to provide time service for downstream devices):
```bash
ntp.aliyun.com          # Alibaba Cloud
time1.cloud.tencent.com  # Tencent Cloud
time.ustc.edu.cn        # University of Science and Technology of China
cn.pool.ntp.org        # Global NTP Volunteer Pool
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/29.png)

### 6. Modify the opkg Package Configuration (If an Error Occurs When Updating the Package)
1. Find `System -> Software` in the left menu and select `Configure opkg`.
2. Modify "23.05-SNAPSHOT" in `/etc/opkg/distfeeds.conf` to "23.05". The modified content is as follows:
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
1. Connect the WAN port of the OpenWRT main router to the campus network interface with a network cable.
2. Connect the WAN port of the AP router to the LAN port of the OpenWRT main router.
3. Enter the AP router management interface and set it to "Bridge Mode" or "Wired Repeater Mode".
4. Configure the WiFi name and password of the AP router.
5. Connect all devices to WiFi or a wired network and test whether the network is used normally.

![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/25.png)
