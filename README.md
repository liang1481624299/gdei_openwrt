### **Language Switch: [English](README_en.md)**
# 适用于广东第二师范学院校园网宿舍解决方案
本方案针对广东第二师范学院宿舍校园网的常见问题，解决网络卡顿、带宽不足、连接设备数量受限、部分网站无法访问等问题。核心通过自定义编译OpenWRT固件、优化网络设置避开校园网监测，提升网络使用的流畅度与灵活性。需说明的是，方案思路与方法同样适用于其他高校校园网，仅供参考与技术交流。


## 一、方案背景与初衷
高中时期，我长期受顺德区教育网限制，深刻体会到网络受限对学习与生活的影响。进入广东第二师范学院后，发现宿舍校园网存在类似问题：高峰时段视频卡顿、多设备联网频繁断连、部分学习所需外部网站无法打开，直接影响网课学习、资料查询与日常使用。

目前国内高校校园网管控已形成成熟技术链条：前端通过User-Agent特征识别、时间戳动态校验、IPID序列追踪等多维手段实现终端侦测；后端依托IP地址污染、DNS解析劫持、HTTP响应重定向至校内404页面等方式实施访问阻断。这种“侦测-响应”机制闭环，是校园网络自由使用的主要技术壁垒。

基于计算机编译原理的基础认知，以及对OpenWRT固件模块化架构的理解，我通过固件定制编译、协议栈参数优化、应用层代理策略配置等技术路径，系统性解构校园网管控逻辑，最终形成这套兼具实用性与可扩展性的宿舍网络优化方案。

## 二、QA与免责声明
<details>
<summary>📌 点击展开：常见问题解答（QA）</summary>

### 1. 为什么开源此技术，校园网不会采取网络漏洞修复措施？
- 校园网运维工程师对技术动态的关注度远高于普通学生，不存在“不知道”的情况，不修复核心原因是**技术上无法有效修复**。  
- 校园网的各类检测方法，均基于中华人民共和国国家知识产权局发布的专利申请及相关国标文件制定；而本方案的反侦测技术完全符合国标规定，原理是将宿舍所有网络设备聚合为“一个IP、一个MAC、一个UA”上网，与单台设备正常上网的特征完全一致，检测设备无法区分聚合设备与单台设备，更不可能封禁正常上网的终端。  
- 从运维成本角度，校园网维护存在设备成本、维护成本双重限制：新增检测手段会干扰原有检测准确性，且需投入更多硬件资源；若一味增加检测规则，会显著拖慢校园服务器性能，进而影响全校师生的上网体验与正常教学进度。  
- 从流程与实操角度，运维人员调整检测策略需层层请示上级、多方协调测试，且难以精准分辨“正常设备”与“聚合设备”；相比之下，本方案仅涉及技术配置，无额外流程阻力。

### 2. 部署这个项目会不会有什么安全风险或者法律风险？
**无安全风险与法律风险**，可放心部署。 

本项目的反侦测逻辑基于个人长期技术探索总结，全程不涉及对校园网络设备的修改或损坏，所有配置均遵循国标文件要求，未突破校园网基础使用框架，无需担心法律或安全层面的问题。

### 3. 遇到其他未提及的问题该如何解决？
若遇到方案中未覆盖的疑难杂症，欢迎在项目GitHub仓库的**issues**板块留言，我会在看到后及时回复并协助解决。
</details>

<details>
<summary>⚠️ 点击展开：免责声明</summary>

1. 本项目基于开源协议（如GPL协议）开发与分享，所有代码及配置文件仅用于**技术交流与学习研究**，不得用于任何商业用途或非法活动。  
2. 使用者需确保在符合广东第二师范学院校园网使用规定及国家相关法律法规的前提下部署本方案，因违规使用导致的校园处分、法律责任等，均由使用者自行承担，项目作者不承担任何连带责任。  
3. 本项目仅提供技术思路与基础配置，不同宿舍的网络环境、设备型号可能存在差异，部署过程中若出现网络不稳定、设备故障等问题，项目作者不承担硬件维修或网络恢复责任，建议使用者具备基础的计算机网络知识后再操作。  
4. 禁止任何个人或组织在未经项目作者允许的情况下，将本项目的代码、文档进行篡改、二次封装后用于商业传播或盈利，违者将追究其相关法律责任。  
5. 项目作者有权根据技术发展或校园网政策变化，对方案内容进行更新或调整，不保证方案长期适用于所有场景，建议使用者关注项目最新动态。
</details>


## 三、准备阶段
### 1. 硬件设备
- **主路由设备**：需配备双网口的x86-64架构设备（如工控机、迷你主机），或其他支持刷写OpenWRT系统的主板（如ARM架构开发板），确保可同时连接校园网进线与下级设备。本方案以X86-64架构为例演示。
- **无线接入点**：准备1台支持无线功能的路由器作为AP（接入点），用于扩展WiFi覆盖，推荐支持802.11ac及以上协议的型号，以保证无线速率。

### 2. 编译环境
- **操作系统**：必须使用Ubuntu（推荐20.04/22.04 LTS版本）或其他Linux发行版，本方案以Ubuntu为例操作演示。
- **配置要求**：建议至少4GB内存、20GB空闲硬盘空间，处理器支持多线程以加快编译速度。

### 3. 工具软件
- **PE系统**：用于引导设备并写入固件，推荐使用微PE或大白菜等纯净版PE工具。
- **磁盘写入工具**：[DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download)（下载链接），用于将编译好的OpenWRT镜像写入硬盘。
- 如果无法下载或者其他原因，可以在项目的releases中下载

### 4. 其他准备
- 校园网账号密码：用于验证接入校园网络。
- U盘（8GB以上）：制作PE启动盘及存放固件文件。
- 网线（至少两根）：分别用于连接校园网接口与主路由、主路由与AP设备。

### 5. 面向操作不熟练同学的说明
若无法独立完成以下操作，可直接使用以下命令下载已设置好的配置文件，直接进行编译，或者直接去releases下载已经编译好的img（可以直接跳转到步骤四-16）
```
git clone https://github.com/liang1481624299/gdei_openwrt.git
cd gdei_openwrt
make kernel_menuconfig -j$(nproc) V=cs
```



## 四、开始编译
### 1. 克隆OpenWRT源码
克隆OpenWRT 23.05版本分支（新版可能存在模块不兼容问题）。因网络原因，部分用户需使用代理工具加速：
```bash
git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
```

### 2. 复制源码副本
为避免操作出错影响原始代码，先复制1份副本，在副本中进行编译：
```bash
cp -r openwrt openwrt_2
```

### 3. 进入副本文件夹
```bash
cd openwrt_2
```

### 4. 更新软件包源
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 5. 克隆核心插件源码
克隆用于规避校园网侦测的核心插件源码：
- UA2F：用于突破User-Agent特征检测
- rkp-ipid：用于修改IPID序列，避免被追踪
```bash
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
```

### 6. 克隆实用工具与插件源码
克隆含网络加速、音乐解锁等功能的实用工具与插件源码：
```bash
# 修复Tailscale编译冲突
sed -i '/\/etc\/init\.d\/tailscale/d;/\/etc\/config\/tailscale/d;' feeds/packages/net/tailscale/Makefile

# 克隆必要插件
git clone https://github.com/asvow/luci-app-tailscale package/luci-app-tailscale  # Tailscale内网穿透
git clone https://github.com/jerrykuku/luci-theme-argon.git package/luci-theme-argon  # Argon主题
git clone https://github.com/jerrykuku/luci-app-argon-config.git package/luci-app-argon-config  # 主题配置工具
git clone https://github.com/lucikap/luci-app-ua2f.git package/luci-app-ua2f  # UA2F图形化配置

# 克隆网易云音乐解锁插件
cd package
git clone https://github.com/UnblockNeteaseMusic/luci-app-unblockneteasemusic.git
cd ..

# 安装Turbo ACC网络加速模块
curl -sSL https://raw.githubusercontent.com/chenmozhijin/turboacc/luci/add_turboacc.sh -o add_turboacc.sh && bash add_turboacc.sh
```

### 7. 再次更新软件包源
再次更新软件包源，确保新增插件被识别：
```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 8. 写入MD5验证值
写入OpenWRT 23.05版本的MD5验证值（用于规避版本检测）：
```bash
nano vermagic  # 创建并编辑vermagic文件
```
在打开的编辑器中输入以下MD5值：
```
47964456485559d992fe6f536131fc64
```
按 `Ctrl + O` 保存，再按`Ctrl + X`退出编辑器。

### 9. 修改内核编译参数
修改内核编译参数，让系统读取自定义的MD5值：
```bash
nano ./include/kernel-defaults.mk  # 编辑内核默认配置文件
```
按 `Ctrl + /` 输入 `121` 并回车，快速定位到第121行，将原有代码注释并替换为读取自定义MD5的配置：
```bash
# 注释原有行
#grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | $(MKHASH) md5 > $(LINUX_DIR)/.vermagic
# 新增行：读取自定义vermagic文件
cp $(TOPDIR)/vermagic $(LINUX_DIR)/.vermagic
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/1.png)

### 10. 修改内核Makefile
同理修改内核Makefile，确保MD5验证兼容：
```bash
nano ./package/kernel/linux/Makefile  # 编辑内核编译配置文件
```
按 `Ctrl + /` 输入 `29` 并回车，定位到第29行，修改为以下内容（注意保留行前空格）：
```bash
# 注意，下面两个要粘贴的注意保留前方空格
#STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
# 新增行：使用自定义vermagic
STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
```
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

### 11. 启动编译配置
```bash
make menuconfig
```

### 12. 配置目标系统
进入 `menuconfig` 后，操作规则如下：双击ESC返回上一页，键盘左右键选择操作，上下键实现滚动。将 `Target System` 设置为 `x86` ， `Subtarget` 设置为 `x86_64` 。
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/3.png)

### 13. 配置目标镜像分区
进入 `Target Images` ，仅需修改 `Kernel partition size (in MiB)` 和 `Root filesystem partition size (in MiB)` ，具体数值根据自身情况调整：
- `Kernel partition`：存放内核和引导文件的分区
- `Root filesystem partition`：OpenWRT的Root分区

**提示**：若编译固件用于刷入硬路由，此处数值不宜过大，避免撑爆硬路由闪存，可参考以下图片配置。
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/4.png)

### 14. 返回主配置界面
操作完成后按ESC返回上一页，直至顶部标题栏显示 `OpenWrt Configuration` 。若不小心完全退出 `make menuconfig` ，可重复第11步重新启动配置。
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)

### 15. 添加模块和软件包
#### （1）添加ipid模块
1. 找到并进入 `Kernel modules -> Other modules`
2. 按Y选中 `kmod-rkp-ipid`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/6.png)

#### （2）返回主配置界面，并安装vpn（用于远程访问校内业务）
1. 按两下ESC退回到 `OpenWrt Configuration`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/5.png)
2. 找到并进入 `Network -> VPN`
3. 按Y选中 `openvpn-easy-rsa` `openvpn-openssl` `tailscale`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)

#### （3）添加UA修改模块
1. 找到并进入 `Network -> Routing and Redirection`
2. 按Y选中 `ua2f`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/7.png)

####  (4) 添加需要的防火墙模块
1. 找到并进入 `Network ->Firewall`
2. 按Y选中 `iptables-mod-conntrack-extra` `iptables-mod-filter` `iptables-mod-ipopt` `iptables-nft` `iptables-mod-u32`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/8.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/9.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/10.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/11.png)
4. 回到OpenWRT Configuration
5. 找到并进入 `Network`
6. 按Y选中 `ipset`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/12.png)
7. 先保存一下以防万一
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/13.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/14.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/15.png)
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/16.png)

####  (5) 添加 LuCI 网络管理界面
1. 找到并进入 `LuCI -> Collections ->`
2. 按Y选中 `luci`
3. 回到 `LuCI`
4. 找到并进入 `Modules`
5. 按Y选中 `luci-compat`
6. 找到并进入 `Translations`
7. 可以根据自己想要的语言进行配置网页前端翻译，这边我是全部选上
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/18.png)
8. 回到 `LuCI`
9. 找到并进入 `Applications`
10. 按Y选中 `luci-app-argon-config` `luci-app-ttyd` `luci-app-tailscale` `luci-app-turboacc` `luci-app-ua2f` `luci-app-unblockneteasemusic` `luci-app-upnp` `luci-app-wol`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/19.png)
12. 按键盘方向→键移动光标到 `save` 保存(如同上面保存步骤)
13. 双击ESC退出直到回到终端命令行
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/17.png)

### 16. 开始进行内核编译配置，等待其再次出现选择页面（请确保你的网络环境有魔法）
```bash
make kernel_menuconfig -j$(nproc) V=cs
```

💡Tips：编译时可能会因为不同硬件性能问题，而出现的等待时间不一样，性更强的编译速度快一些，性能弱的则需要耐心等待

这里会有两种情况，有些人直接就进入了 `Linux/x86 5.15.189 Kernel Configuration` 二有些人会进入 `General Setup` （如下图）
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/20.png)


进入了 `Linux/x86 5.15.189 Kernel Configuration` 则无需理会，反之进入了 `General Setup` 请按两次ESC回到 `Linux/x86 5.15.189 Kernel Configuration`
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/21.png)

1. 进行参数设置

#### (1) 开启 UA2F 的防火墙内核

1. 找到并进入 `Networking support -> Networking options`
2. 先按Y选中 `Network packet filtering framework (Netfilter)` 再按回车进入
![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/22.png)

3. 找到并进入 `Core Netfilter Configuration`
4. 按Y选中 `Netfilter NFNETLINK interface` `Netfilter LOG over NFNETLINK interface` `Netfilter connection tracking support` `Connection tracking netlink interface` `NFQUEUE and NFLOG integration with Connection Tracking`
5. 保存并离开

### 17. 开始进行内核编译预下载（请确保你的网络环境有魔法）
```bash
make download -j$(nproc) V=cs
```

💡Tips：若出现报错不要着急，等全部下载完了再执行一遍命令让他重新下载那些报错的

### 18. 大功告成，开始编译OpenWRT系统

1. 输入以下命令开始编译OpenWRT
```bash
make -j$(nproc) V=cs
```

💡Tips：编译时，请确保电脑或者编译服务器的可用运行内存大于4GB，弹出的功能全部选择y，以确保OpenWRT对设备的兼容性，若出现Error报错，运行一下命令在重新执行编译
```bash
make clean
make dirclean  # 更彻底的清理，会删除下载的包和配置
```

### 19. 前往./bin/target/x86(你编译时选择的架构)/x64（你编译时选择的架构分支）/ 这里是存放编译好的所有镜像文件（.img）
