### **Language Switch: [简体中文](README.md)**
# Campus Network Dormitory Solution for Guangdong University of Education
This solution addresses common issues with the campus network in dormitories of Guangdong University of Education, including network lag, insufficient bandwidth, limited number of connectable devices, and inaccessibility to certain websites. It primarily enhances network fluency and flexibility by custom-compiling OpenWRT firmware and optimizing network settings to bypass campus network monitoring. It should be noted that the ideas and methods of this solution are also applicable to campus networks of other universities, and are for reference and technical exchange only.


## I. Background and Original Intent of the Solution
During my high school years, I was long restricted by the Shunde District Education Network, which gave me a deep understanding of how network restrictions affect learning and daily life. After enrolling in Guangdong University of Education, I found similar problems with the dormitory campus network: video lag during peak hours, frequent disconnections when multiple devices are connected to the network, and inaccessibility to some external websites required for learning. These issues directly impact online course learning, information retrieval, and daily network use.

Currently, campus network management and control in domestic universities have formed a mature technical chain. On the front end, terminal detection is implemented through multi-dimensional means such as User-Agent feature recognition, timestamp dynamic verification, and IPID sequence tracking. On the back end, access blocking is carried out by means of IP address pollution, DNS resolution hijacking, and HTTP response redirection to the campus 404 page. This closed-loop "detection-response" mechanism is the main technical barrier to the free use of the campus network.

Based on my basic understanding of computer compilation principles and the modular architecture of OpenWRT firmware, I systematically deconstructed the campus network management and control logic through technical approaches such as custom firmware compilation, protocol stack parameter optimization, and application-layer proxy strategy configuration. Finally, this practical and scalable dormitory network optimization solution was formed.

## II. QA and Disclaimer
<details>
<summary>📌 Click to expand: Frequently Asked Questions (QA)</summary>

### 1. Why is this technology open-sourced? Won't the campus network take measures to fix network vulnerabilities?
- Campus network operation and maintenance engineers pay much more attention to technical trends than ordinary students. There is no such thing as "not knowing" about the technology. The core reason for not fixing it is that **it is technically impossible to fix it effectively**.
- Various detection methods of the campus network are formulated based on patent applications and relevant national standard documents issued by the State Intellectual Property Office of the People's Republic of China. The anti-detection technology of this solution fully complies with national standards. Its principle is to aggregate all network devices in the dormitory into "one IP, one MAC, one UA" for internet access, which is completely consistent with the characteristics of a single device accessing the internet normally. Detection devices cannot distinguish between aggregated devices and single devices, and it is even more impossible to block terminals that access the internet normally.
- From the perspective of operation and maintenance costs, campus network maintenance is restricted by both equipment costs and maintenance costs. Adding new detection methods will interfere with the accuracy of original detection and require more hardware resources to be invested. If detection rules are added blindly, the performance of campus servers will be significantly slowed down, which will further affect the internet experience of all teachers and students and the normal teaching progress.
- From the perspective of processes and practical operations, operation and maintenance personnel need to report to superiors at all levels and coordinate testing in various aspects to adjust detection strategies, and it is difficult to accurately distinguish between "normal devices" and "aggregated devices". In contrast, this solution only involves technical configuration and has no additional process obstacles.

### 2. Will there be any security risks or legal risks in deploying this project?
There are **no security risks or legal risks**, and it can be deployed with confidence.

The anti-detection logic of this project is summarized based on the author's long-term technical exploration. It does not involve modifying or damaging campus network equipment throughout the process. All configurations comply with the requirements of national standard documents and do not break through the basic use framework of the campus network. There is no need to worry about legal or security issues.

### 3. How to solve other unmentioned problems?
If you encounter difficult problems not covered in the solution, you are welcome to leave a message in the **issues** section of the project's GitHub repository. I will reply and assist in solving the problem in a timely manner after seeing the message.
</details>

<details>
<summary>⚠️ Click to expand: Disclaimer</summary>

1. This project is developed and shared based on open-source protocols (such as the GPL protocol). All codes and configuration files are only used for **technical exchange and learning research**, and shall not be used for any commercial purposes or illegal activities.
2. Users must ensure that this solution is deployed in compliance with the campus network usage regulations of Guangdong University of Education and relevant national laws and regulations. The project author shall not bear any joint liability for campus sanctions, legal liabilities, etc. caused by non-compliant use, which shall be borne by the user themselves.
3. This project only provides technical ideas and basic configurations. There may be differences in network environments and device models in different dormitories. If problems such as network instability and device failures occur during the deployment process, the project author shall not bear the responsibility for hardware maintenance or network restoration. It is recommended that users have basic computer network knowledge before operating.
4. Any individual or organization is prohibited from tampering with or repackaging the code and documents of this project for commercial dissemination or profit without the permission of the project author. Violators will be held accountable for their relevant legal responsibilities.
5. The project author has the right to update or adjust the content of the solution according to technological development or changes in campus network policies. The author does not guarantee that the solution will be applicable to all scenarios for a long time. It is recommended that users pay attention to the latest developments of the project.
</details>


## III. Preparation Stage
### 1. Hardware Equipment
- **Main Router Device**: An x86-64 architecture device with dual network ports (such as an industrial computer or a mini host) is required, or other motherboards that support flashing OpenWRT systems (such as ARM architecture development boards). It should be ensured that the device can connect to both the campus network incoming line and subordinate devices at the same time. This solution uses the X86-64 architecture as an example for demonstration.
- **Wireless Access Point**: Prepare a router that supports wireless functions as an AP (Access Point) for expanding WiFi coverage. It is recommended to choose a model that supports the 802.11ac protocol or above to ensure wireless speed.

### 2. Compilation Environment
- **Operating System**: Ubuntu (version 20.04/22.04 LTS is recommended) or other Linux distributions must be used. This solution uses Ubuntu as an example for operation demonstration.
- **Configuration Requirements**: It is recommended to have at least 4GB of memory and 20GB of free hard disk space. The processor should support multi-threading to speed up the compilation process.

### 3. Tool Software
- **PE System**: Used to boot the device and write firmware. It is recommended to use pure version PE tools such as WeiPE or Dabaicai.
- **Disk Writing Tool**: [DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download) (download link), used to write the compiled OpenWRT image to the hard disk.
- If you cannot download it or for other reasons, you can download it from the releases section of the project.

### 4. Other Preparations
- Campus Network Account and Password: Used to verify access to the campus network.
- USB Flash Drive (8GB or above): Used to make a PE boot disk and store firmware files.
- Network Cables (at least two): One is used to connect the campus network interface to the main router, and the other is used to connect the main router to the AP device.

### 5. Instructions for Students Who Are Not Proficient in Operations
If you cannot complete the following operations independently, you can directly use the following commands to download the configured files for compilation, or directly go to the releases section to download the already compiled img (you can skip directly to Step 4-16):
```
git clone https://github.com/liang1481624299/gdei_openwrt.git
cd gdei_openwrt
make kernel_menuconfig -j$(nproc) V=cs
```



## IV. Start Compilation
### 1. Clone OpenWRT Source Code
Clone the OpenWRT 23.05 version branch (the new version may have module incompatibility issues). Due to network reasons, some users need to use proxy tools to speed up the process:
```bash
git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
```

### 2. Copy a Copy of the Source Code
To avoid affecting the original code due to operation errors, first make a copy and conduct compilation in the copy:
```bash
cp -r openwrt openwrt_2
```

### 3. Enter the Copy Folder
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
- UA2F: Used to break through User-Agent feature detection
- rkp-ipid: Used to modify the IPID sequence to avoid being tracked
```bash
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
```

### 6. Clone Practical Tools and Plugin Source Code
Clone the source code of practical tools and plugins with functions such as network acceleration and music unblocking:
```bash
# Fix Tailscale compilation conflict
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
Update the software package sources again to ensure that the newly added plugins are recognized:
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 8. Write MD5 Verification Value
Write the MD5 verification value of the OpenWRT 23.05 version (used to bypass version detection):
```bash
nano vermagic  # Create and edit the vermagic file
```
Enter the following MD5 value in the opened editor:
```
47964456485559d992fe6f536131fc64
```
Press `Ctrl + O` to save, and then press `Ctrl + X` to exit the editor.

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

### 10. Modify Kernel Makefile
Similarly, modify the kernel Makefile to ensure MD5 verification compatibility:
```bash
nano ./package/kernel/linux/Makefile  # Edit the kernel compilation configuration file
```
Press `Ctrl + /`, enter `29`, and press Enter to locate line 29. Modify it to the following content (note to retain the spaces at the beginning of the line):
```bash
# Note: When pasting the following two lines, be sure to retain the leading spaces
#STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
# New line: Use the custom vermagic
STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

### 11. Start Compilation Configuration
```bash
make menuconfig
```

### 12. Configure the Target System
After entering `menuconfig`, follow these operation rules: Double-click ESC to return to the previous page, use the left and right arrow keys on the keyboard to select operations, and use the up and down arrow keys to scroll. Set `Target System` to `x86` and `Subtarget` to `x86_64`.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/3.png)

### 13. Configure Target Image Partitions
Enter `Target Images` and only modify `Kernel partition size (in MiB)` and `Root filesystem partition size (in MiB)`. Adjust the specific values according to your own situation:
- `Kernel partition`: The partition for storing the kernel and boot files
- `Root filesystem partition`: The Root partition of OpenWRT

**Tip**: If the compiled firmware is used to flash a hard router, the values here should not be too large to avoid filling up the hard router's flash memory. You can refer to the configuration in the following picture.
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
4. Return to OpenWRT Configuration
5. Find and enter `Network`
6. Press Y to select `ipset`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/12.png)
7. Save first to prevent accidents
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/13.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/14.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/15.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/16.png)

#### (5) Add the LuCI Network Management Interface
1. Find and enter `LuCI -> Collections ->`
2. Press Y to select `luci`
3. Return to `LuCI`
4. Find and enter `Modules`
5. Press Y to select `luci-compat`
6. Find and enter `Translations`
7. You can configure the web front-end translation according to the language you want. Here, I selected all languages.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/18.png)
8. Return to `LuCI`
9. Find and enter `Applications`
10. Press Y to select `luci-app-argon-config`, `luci-app-ttyd`, `luci-app-tailscale`, `luci-app-turboacc`, `luci-app-ua2f`, `luci-app-unblockneteasemusic`, `luci-app-upnp`, and `luci-app-wol`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)
12. Use the right arrow key on the keyboard to move the cursor to `save` and save (as in the above saving steps)
13. Double-click ESC to exit until returning to the terminal command line
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/17.png)

### 16. Start Kernel Compilation Configuration and Wait for the Selection Page to Appear Again (Ensure Your Network Environment Has Access to Global Networks)
```bash
make kernel_menuconfig -j$(nproc) V=cs
```

💡 Tips: During compilation, the waiting time may vary due to different hardware performance. Devices with stronger performance will have a faster compilation speed, while those with weaker performance require more patience.

There are two situations here: some people will directly enter `Linux/x86 5.15.189 Kernel Configuration`, while others will enter `General Setup` (as shown in the figure below).
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/20.png)

If you enter `Linux/x86 5.15.189 Kernel Configuration`, no action is needed. If you enter `General Setup` instead, press ESC twice to return to `Linux/x86 5.15.189 Kernel Configuration`.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/21.png)

1. Configure Parameters

#### (1) Enable the Firewall Kernel for UA2F
1. Find and enter `Networking support -> Networking options`
2. First press Y to select `Network packet filtering framework (Netfilter)`, then press Enter to enter
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/22.png)

3. Find and enter `Core Netfilter Configuration`
4. Press Y to select `Netfilter NFNETLINK interface`, `Netfilter LOG over NFNETLINK interface`, `Netfilter connection tracking support`, `Connection tracking netlink interface`, and `NFQUEUE and NFLOG integration with Connection Tracking`
5. Save and exit

### 17. Start Kernel Compilation Pre-Download (Ensure Your Network Environment Has Access to Global Networks)
```bash
make download -j$(nproc) V=cs
```

💡 Tips: If an error occurs, don't worry. Wait for all downloads to complete and then execute the command again to re-download the files that reported errors.

### 18. Completion of Preparation, Start Compiling the OpenWRT System
1. Enter the following command to start compiling OpenWRT:
```bash
make -j$(nproc) V=cs
```

💡 Tips: During compilation, ensure that the available running memory of the computer or compilation server is greater than 4GB. Select "y" for all pop-up functions to ensure the compatibility of OpenWRT with the device. If an "Error" occurs, run the following commands and then re-execute the compilation:
```bash
make clean
make dirclean  # More thorough cleaning, which will delete downloaded packages and configurations
```

### 19. Go to ./bin/target/x86 (the architecture you selected during compilation)/x64 (the architecture branch you selected during compilation)/. All compiled image files (.img) are stored here.