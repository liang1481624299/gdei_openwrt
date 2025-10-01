# 广东第二师范学院校园网宿舍优化解决方案

本方案专门针对广东第二师范学院宿舍校园网的常见问题，解决网络卡顿、带宽不够用、连接设备数量受限、部分网站打不开等问题。核心是通过自定义编译 OpenWRT 固件，优化网络设置来避开校园网的监测，让网络用起来更顺畅、更灵活。需要说明的是，这套方案的思路和方法也适用于其他高校的校园网，仅供参考和技术交流。

## 方案背景与初衷

我在高中时，就长期受顺德区教育网限制，很清楚网络受限对学习和生活的影响。到了广东第二师范学院后，发现宿舍校园网也有类似问题：高峰时段视频总卡，多设备同时连网经常断，一些学习需要的外部网站还打不开，这些都直接影响网课学习、查资料和日常使用。目前已知，国内高校校园网的管控体系已形成成熟技术链条：前端通过 User-Agent 特征识别、时间戳动态校验、IPID 序列追踪等多维手段实现终端侦测；后端则依托 IP 地址污染、DNS 解析劫持、HTTP 响应重定向至校内 404 页面等方式实施访问阻断。这种 "侦测 - 响应" 机制的闭环，构成了校园网络自由使用的主要技术壁垒。

基于对计算机编译原理的基础认知，以及 OpenWRT 固件模块化架构的深刻理解，笔者着手通过固件定制编译、协议栈参数优化、应用层代理策略配置等技术路径，系统性解构校园网管控逻辑，最终沉淀为这套兼具实用性与可扩展性的宿舍网络优化方案。

## 准备阶段

1. **硬件设备**：
   - 主路由设备：需配备双网口的 x86-64 架构设备（如工控机、迷你主机），或其他支持刷写 OpenWRT 系统的主板（如 ARM 架构的开发板），确保能同时连接校园网进线和下级设备。
   - 无线接入点：准备一台支持无线功能的路由器作为 AP（接入点），用于扩展 WiFi 覆盖，推荐支持 802.11ac 及以上协议的型号以保证无线速率。

2. **编译环境**：
   - 操作系统：必须使用 Ubuntu（推荐 20.04/22.04 LTS 版本）或其他 Linux 发行版，本方案以 Ubuntu 为例进行操作演示。
   - 配置要求：建议至少 4GB 内存、20GB 空闲硬盘空间，处理器支持多线程以加快编译速度。

3. **工具软件**：
   - PE 系统：用于引导设备并写入固件，推荐使用微 PE 或大白菜等纯净版 PE 工具。
   - 磁盘写入工具：[DiskImage](https://sourceforge.net/projects/diskimage/files/latest/download)（下载链接），用于将编译好的 OpenWRT 镜像写入硬盘。

4. **其他准备**：
   - 校园网账号密码：用于验证接入校园网络。
   - U 盘（8GB 以上）：制作 PE 启动盘及存放固件文件。
   - 网线：至少两根，分别用于连接校园网接口与主路由、主路由与 AP 设备。

## 开始编译

1. 克隆OpenWRT 23.05版本的分支（新版可能存在模块不兼容问题）。由于网络原因，部分用户可能需要使用代理工具加速：
   ```bash
   git clone https://github.com/openwrt/openwrt.git --branch=openwrt-23.05
   ```

2. 为避免操作出错影响原始代码，先复制一份副本，在副本中进行编译：
   ```bash
   cp -r openwrt openwrt_2
   ```

3. 进入OpenWRT副本文件夹：
   ```bash
   cd openwrt_2
   ```

4. 先更新软件包源（feeds）：
   ```bash
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   ```

5. 克隆规避校园网侦测的核心插件源码：
   - UA2F：用于突破User-Agent特征检测
   - rkp-ipid：用于修改IPID序列，避免被追踪
   ```bash
   git clone https://github.com/Zxilly/UA2F.git package/UA2F
   git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
   ```

6. 克隆实用工具与插件源码（含网络加速、音乐解锁等功能）：
   ```bash
   # 修复Tailscale编译冲突
   sed -i '/\/etc\/init\.d\/tailscale/d;/\/etc\/config\/tailscale/d;' feeds/packages/net/tailscale/Makefile
   
   # 克隆必要插件
   git clone https://github.com/asvow/luci-app-tailscale package/luci-app-tailscale  # Tailscale内网穿透
   git clone https://github.com/jerrykuku/luci-theme-argon.git package/luci-theme-argon  # Argon主题
   git clone https://github.com/jerrykuka/luci-app-argon-config.git package/luci-app-argon-config  # 主题配置工具
   git clone https://github.com/lucikap/luci-app-ua2f.git package/luci-app-ua2f  # UA2F图形化配置
   
   # 克隆网易云音乐解锁插件
   cd package
   git clone https://github.com/UnblockNeteaseMusic/luci-app-unblockneteasemusic.git
   cd ..
   
   # 安装Turbo ACC网络加速模块
   curl -sSL https://raw.githubusercontent.com/chenmozhijin/turboacc/luci/add_turboacc.sh -o add_turboacc.sh && bash add_turboacc.sh
   ```

7. 再次更新软件包源，确保新增插件被识别：
   ```bash
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   ```

8. 写入OpenWRT 23.05版本的MD5验证值（用于规避版本检测）：
   ```bash
   nano vermagic  # 创建并编辑vermagic文件
   ```
   在打开的编辑器中输入以下MD5值：
   ```
   47964456485559d992fe6f536131fc64
   ```
   按`Ctrl + O`保存，再按`Ctrl + X`退出编辑器。

9. 修改内核编译参数，让系统读取自定义的MD5值：
   ```bash
   nano ./include/kernel-defaults.mk  # 编辑内核默认配置文件
   ```
   按`Ctrl + /`输入`121`并回车，快速定位到第121行，将原有代码注释并替换为读取自定义MD5的配置：
   ```bash
   # 注释原有行
   #grep '=[ym]' $(LINUX_DIR)/.config.set | LC_ALL=C sort | $(MKHASH) md5 > $(LINUX_DIR)/.vermagic
   # 新增行：读取自定义vermagic文件
   cp $(TOPDIR)/vermagic $(LINUX_DIR)/.vermagic
   ```
   ![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/1.png)

10. 同理修改内核Makefile，确保MD5验证兼容：
    ```bash
    nano ./package/kernel/linux/Makefile  # 编辑内核编译配置文件
    ```
    按`Ctrl + /`输入`29`并回车，定位到第29行，修改为以下内容（注意保留行前空格）：
    ```bash
    # 注意，下面两个要粘贴的注意保留前方空格
    #STAMP_BUILT:=$(STAMP_BUILT)_$(shell $(SCRIPT_DIR)/kconfig.pl $(LINUX_DIR)/.config | $(MKHASH) md5)
    # 新增行：使用自定义vermagic
    STAMP_BUILT:=$(STAMP_BUILT)_$(shell cat $(LINUX_DIR)/.vermagic)
    ```
    ![Project Logo](https://github.com/liang1481624299/gdei_openwrt/blob/main/photo/2.png)

11.配置编译模块
    ```bash
    make menuconfig
    ```
