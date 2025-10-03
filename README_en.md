### **Language Switch: [简体中文](README.md)**
# Campus Network Dormitory Optimization Solution for Guangdong University of Education
This solution addresses common issues with the campus network in dormitories of Guangdong University of Education, including network lag, insufficient bandwidth, limited number of connectable devices, and inaccessibility to certain websites. The core approach involves custom-compiling OpenWRT firmware and optimizing network settings to bypass campus network monitoring, thereby enhancing the smoothness and flexibility of network usage. It should be noted that the ideas and methods of this solution are also applicable to campus networks of other universities, and are for reference and technical exchange only.


## 1. Background and Purpose of the Solution
During my high school years, I was restricted by the Shunde District Education Network for a long time, and deeply experienced the impact of network restrictions on study and daily life. After enrolling in Guangdong University of Education, I found similar issues with the dormitory campus network: video lag during peak hours, frequent disconnections when multiple devices are connected simultaneously, and inaccessibility to some external websites required for study. These problems directly affect online courses, information retrieval, and daily network use.

Currently, the campus network management system in domestic universities has formed a mature technical chain:
- Frontend detection: Terminal detection is implemented through multi-dimensional methods such as User-Agent feature recognition, timestamp dynamic verification, and IPID sequence tracking.
- Backend blocking: Access blocking is carried out by means of IP address pollution, DNS resolution hijacking, and HTTP response redirection to the campus internal 404 page.

This closed-loop "detection-response" mechanism is the main technical barrier to the free use of the campus network.

Based on basic knowledge of computer compilation principles and an understanding of the modular architecture of OpenWRT firmware, I systematically deconstructed the campus network management logic through technical approaches such as custom firmware compilation, protocol stack parameter optimization, and application-layer proxy policy configuration. Eventually, this dormitory network optimization solution with both practicality and scalability was developed.


## 2. Preparation Phase
### 1. Hardware Equipment
- **Main Router Device**: A device with dual network ports and x86-64 architecture (such as an industrial computer or mini host) is required, or other motherboards that support flashing the OpenWRT system (such as ARM architecture development boards). It should be able to connect both the campus network incoming line and subordinate devices simultaneously. This solution uses the X86-64 architecture for demonstration.
- **Wireless Access Point (AP)**: Prepare a router with wireless functionality as an AP to extend WiFi coverage. It is recommended to choose a model that supports the 802.11ac protocol or above to ensure wireless speed.

### 2. Compilation Environment
- **Operating System**: Ubuntu (version 20.04/22.04 LTS is recommended) or other Linux distributions must be used. This solution uses Ubuntu for operation demonstration.
- **Configuration Requirements**: At least 4GB of memory and 20GB of free hard disk space are recommended. The processor should support multi-threading to speed up the compilation process.

### 3. Tool Software
- **PE System**: Used to boot the device and write the firmware. Pure version PE tools such as Micro PE or Dabaicai PE are recommended.
- **Disk Writing Tool**: [DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download) (download link), used to write the compiled OpenWRT image to the hard disk.
- If download fails or for other reasons, you can download it from the "releases" section of the project.

### 4. Other Preparations
- Campus network account and password: Used to authenticate and access the campus network.
- USB flash drive (8GB or larger): Used to make a PE boot disk and store firmware files.
- Network cables (at least two): One for connecting the campus network interface to the main router, and the other for connecting the main router to the AP device.

### 5. Notes for Users with Little Operational Experience
If you cannot complete the following operations independently, you can directly use the following commands to download the pre-configured files for compilation, or download the already compiled img file from the "releases" section:
```
git clone https://github.com/liang1481624299/gdei_openwrt.git
cd gdei_openwrt
```


## 3. QA and Disclaimer
<details>
<summary>📌 Click to expand: Frequently Asked Questions (QA)</summary>

### 1. Why is this technology open-sourced, and why doesn't the campus network take measures to fix network vulnerabilities?
- Campus network operation and maintenance engineers pay much more attention to technical trends than ordinary students, so there is no such situation as "not knowing". The core reason for not fixing is that **effective technical fixes are not possible**.
- Various detection methods of the campus network are formulated based on patent applications and relevant national standard documents issued by the State Intellectual Property Office of the People's Republic of China. However, the anti-detection technology of this solution fully complies with national standards. Its principle is to aggregate all network devices in the dormitory into "one IP, one MAC, and one UA" for internet access, which is completely consistent with the characteristics of a single device accessing the internet normally. Detection devices cannot distinguish between aggregated devices and single devices, and it is even more impossible to block terminals that access the internet normally.
- From the perspective of operation and maintenance costs, campus network maintenance is restricted by both equipment costs and maintenance costs: adding new detection methods will interfere with the accuracy of existing detection and require more hardware resources; if detection rules are increased blindly, the performance of campus servers will be significantly slowed down, which will affect the internet experience of all teachers and students and the normal teaching progress.
- From the perspective of process and practical operation, operation and maintenance personnel need to report to superiors at all levels, coordinate and test in multiple parties when adjusting detection strategies, and it is difficult to accurately distinguish between "normal devices" and "aggregated devices". In contrast, this solution only involves technical configuration and has no additional process obstacles.

### 2. Will deploying this project pose any security risks or legal risks?
There are **no security risks or legal risks**, and it can be deployed with confidence.

The anti-detection logic of this project is summarized based on long-term personal technical exploration. It does not involve modification or damage to campus network equipment throughout the process. All configurations comply with the requirements of national standard documents and do not exceed the basic usage framework of the campus network. There is no need to worry about legal or security issues.

### 3. How to solve other unmentioned problems?
If you encounter problems not covered in this solution, you are welcome to leave a message in the **issues** section of the project's GitHub repository. I will reply and assist in solving the problem in a timely manner after seeing it.
</details>

<details>
<summary>⚠️ Click to expand: Disclaimer</summary>

1. This project is developed and shared based on open-source protocols (such as the GPL protocol). All codes and configuration files are only for **technical exchange and learning research**, and shall not be used for any commercial purposes or illegal activities.
2. Users must ensure that this solution is deployed in compliance with the campus network usage regulations of Guangdong University of Education and relevant national laws and regulations. The project author shall not bear any joint liability for campus disciplinary actions, legal liabilities, etc. caused by non-compliant use.
3. This project only provides technical ideas and basic configurations. The network environment and device models of different dormitories may vary. The project author shall not bear the responsibility for hardware maintenance or network recovery if problems such as network instability or device failure occur during the deployment process. It is recommended that users have basic computer network knowledge before operation.
4. Any individual or organization is prohibited from tampering with or repackaging the codes and documents of this project for commercial dissemination or profit without the permission of the project author. Violators will be held accountable for relevant legal responsibilities.
5. The project author has the right to update or adjust the content of the solution according to technological development or changes in campus network policies. There is no guarantee that the solution will be applicable to all scenarios for a long time. It is recommended that users pay attention to the latest developments of the project.
</details>


## 4. Start Compilation
### 1. Clone OpenWRT Source Code
Clone the OpenWRT 23.05 version branch (newer versions may have module incompatibility issues). Due to network reasons, some users may need to use a proxy tool for acceleration:
```bash
git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
```

### 2. Copy Source Code Copy
To avoid affecting the original code due to operation errors, first make a copy and perform compilation in the copy:
```bash
cp -r openwrt openwrt_2
```

### 3. Enter the Copy Folder
```bash
cd openwrt_2
```

### 4. Update Package Sources
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

### 7. Update Package Sources Again
Update the package sources again to ensure that the newly added plugins are recognized:
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
Press `Ctrl + O` to save, then press `Ctrl + X` to exit the editor.

### 9. Modify Kernel Compilation Parameters
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

### 10. Modify Kernel Makefile
Similarly, modify the kernel Makefile to ensure MD5 verification compatibility:
```bash
nano ./package/kernel/linux/Makefile  # Edit the kernel compilation configuration file
```
Press `Ctrl + /`, enter `29` and press Enter to locate line 29. Modify it to the following content (note to retain the spaces at the beginning of the line):
```bash
# Note: Keep the leading spaces when pasting the following two lines
#STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
# New line: Use custom vermagic
STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

### 11. Start Compilation Configuration
```bash
make menuconfig
```

### 12. Configure the Target System
After entering `menuconfig`, the operation rules are as follows: double-click ESC to return to the previous page, use the left and right arrow keys on the keyboard to select operations, and use the up and down arrow keys to scroll. Set `Target System` to `x86` and `Subtarget` to `x86_64`.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/3.png)

### 13. Configure Target Image Partitions
Enter `Target Images` and only need to modify `Kernel partition size (in MiB)` and `Root filesystem partition size (in MiB)`. Adjust the specific values according to your own situation:
- `Kernel partition`: The partition for storing the kernel and boot files
- `Root filesystem partition`: The Root partition of OpenWRT

**Tip**: If the compiled firmware is used to flash a hard router, the values here should not be too large to avoid overflowing the flash memory of the hard router. You can refer to the following image for configuration.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/4.png)

### 14. Return to the Main Configuration Interface
After completing the operation, press ESC to return to the previous page until the top title bar displays `OpenWrt Configuration`. If you accidentally exit `make menuconfig` completely, you can repeat step 11 to restart the configuration.
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

### 15. Add Modules and Packages
#### (1) Add the ipid Module
1. Find and enter `Kernel modules -> Other modules`
2. Press Y to select `kmod-rkp-ipid`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/6.png)

#### (2) Return to the Main Configuration Interface
Press ESC twice to return to `OpenWrt Configuration`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

#### (3) Add the UA Modification Module
1. Find and enter `Network -> Routing and Redirection`
2. Press Y to select `ua2f`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/7.png)

